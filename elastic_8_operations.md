## choosing your shards

Documents are hashed to a particular shard.
Each shard may be on a different node in a cluster.
Every shard is a self-contained Lucene index of its own.

### how many shards do i need?
- you can’t add more shards later without re-indexing
- but shards aren’t free – you can just make 1,000 of them and stick them on one node at first.
- you want to overallocate, but not too much
- consider scaling out in phases, so you have time to re-index before you hit the next phase

### really? that’s kind of hand-wavy.
- the “right” number of shards depends on your data and your application. there’s no secret formula.
- start with a single server using the same hardware you use in production, with one shard and no replication.
- fill it with real documents and hit it with real queries.
- push it until it breaks – now you know the capacity of a single shard.


### remember replica shards can be added
- read-heavy applications can add more replica shards without re-indexing.
- note this only helps if you put the new replicas on extra hardware!

## adding an index

creating a new index
```bash
PUT /new_index
{
	"settings": {
		"number_of_shards": 10,
		"number_of_replicas": 1
	}
}
```
You can use index templates to automatically apply mappings, analyzers, aliases, etc.

## Index Aliases

An index alias in Elasticsearch is a secondary name for one or more indices. Aliases are often used for several purposes:

- **Simplifying Index Names**: An alias can be used to refer to an index using a simple, user-friendly name.
- **Switching Between Indices**: Aliases can make it easy to switch between indices, such as pointing an alias from an old index to a new one without requiring any changes in the application code that uses the index.
- **Grouping Indices**: An alias can point to multiple indices, allowing you to query multiple indices at once.
- **Filtering**: An alias can include a filter that will be applied automatically whenever the alias is used, enabling you to pre-filter the data.

Here is a practical example that demonstrates some uses of index aliases:

### 1. Creating an Alias for a Single Index

Suppose you have an index named `logs-2023-11`. You can create an alias named `current-logs` for that index:

```json
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "logs-2023-11",
        "alias": "current-logs"
      }
    }
  ]
}
```

### 2. Switching an Alias to a New Index

If you have a new index `logs-2023-12` and want to switch the `current-logs` alias to point to this new index, you can do this atomically:

```json
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "logs-2023-11",
        "alias": "current-logs"
      }
    },
    {
      "add": {
        "index": "logs-2023-12",
        "alias": "current-logs"
      }
    }
  ]
}
```

### 3. Using an Alias with a Filter

If you want to create an alias for logs from a specific user, say with `user_id: 42`, you can do so with a filter:

```json
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "logs-2023-11",
        "alias": "user-42-logs",
        "filter": { "term": { "user_id": "42" }}
      }
    }
  ]
}
```

When you query the `user-42-logs` alias, it will only return documents from the `logs-2023-11` index where `user_id` is `42`.

### 4. Searching with an Alias

You can search using an alias just like you would with an index name. If you've set an alias `current-logs`, you can use it in search:

```json
GET /current-logs/_search
{
  "query": {
    "match_all": {}
  }
}
```

This will return documents from the indices that `current-logs` points to.

Using aliases gives you a lot of flexibility in managing and querying your data. You can transparently perform activities like reindexing, index rollovers for time-based data, and implement multi-tenant patterns where each user's data might live in a separate index but can be accessed through a common alias.

## Index Lifecycle Management

Index Lifecycle Management (ILM) in Elasticsearch is a feature that allows you to automate the lifecycle of your indices. With ILM, you can define policies that dictate how an index should be handled from the moment it's created until it's no longer needed and can be deleted. This includes managing the index's size and performance, and its transition from one state to another, such as from "hot" (actively updated and queried) to "warm" (less frequently accessed) to "cold" (no longer updated and rarely accessed) and finally to "delete."

ILM Policies consist of four primary phases:

1. **Hot**: This is the phase where data is actively written to the index. Indices are optimized for write operations.
2. **Warm**: In this phase, indices are no longer being written to but are still being queried. They may be optimized for less-frequent access.
3. **Cold**: Indices are rarely accessed and no longer updated. They can be moved to slower, cheaper storage if available.
4. **Delete**: Once the data is no longer needed, the index can be deleted.

Each phase has associated actions that can be taken, such as rollover, shrink, force merge, freeze, and delete, to manage the index.

### Practical Example

Let's say you have logs data that you want to manage with an ILM policy.

#### Step 1: Create an ILM Policy

First, create a policy that defines what should happen to indices as they age:

```json
PUT /_ilm/policy/logs_policy
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "30d"
          }
        }
      },
      "warm": {
        "min_age": "30d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "allocate": {
            "require": {
              "data": "warm"
            }
          }
        }
      },
      "cold": {
        "min_age": "90d",
        "actions": {
          "freeze": {}
        }
      },
      "delete": {
        "min_age": "120d",
        "actions": {
          "delete": {
            "delete_searchable_snapshot": true
          }
        }
      }
    }
  }
}
```

In this example policy named `logs_policy`:

- Indices will roll over when they reach 50GB or are 30 days old.
- After 30 days, they will be moved to the warm phase, where they will be shrunk to a single shard and allocated to warm nodes if available.
- After 90 days, the indices will move to the cold phase and will be frozen to reduce their footprint in the cluster.
- After 120 days, the indices will be deleted.

#### Step 2: Apply the ILM Policy to an Index Template

Next, apply this ILM policy to an index template so that it automatically applies to new indices that match the pattern:

```json
PUT /_index_template/logs_template
{
  "index_patterns": ["logs-*"], 
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "index.lifecycle.name": "logs_policy", 
      "index.lifecycle.rollover_alias": "logs"
    }
  }
}
```

Here, `logs_policy` is applied to any new index that matches the pattern `logs-*`. The `rollover_alias` is specified, which is required for rollover to work correctly.

#### Step 3: Create the First Index with the Rollover Alias

When creating the first index, you need to set up the rollover alias properly:

```json
PUT /logs-000001 
{
  "aliases": {
    "logs": {
      "is_write_index": true
    }
  }
}
```

Here, `logs-000001` is the first index in the series. The alias `logs` is used for writing, and `is_write_index` is set to true, indicating that this is the index that new documents should be written to.

With these steps completed, your Elasticsearch cluster will now automatically manage the lifecycle of your logs indices according to the `logs_policy`. As time passes and the conditions specified in the policy are met, Elasticsearch will automatically perform actions like rolling over to a new index, shrinking indices, freezing them, and eventually deleting old indices, all without manual intervention.

## Index Templates

Index templates in Elasticsearch are used to define settings and mappings that should be applied to new indices when they are created. An index template ensures that new indices follow a set of predefined settings and mappings, which can help with consistency and management of your indices, especially in scenarios where you are creating indices regularly, for example with time-series data like logs or metrics.

In Elasticsearch 8.x, there are composable index templates which allow you to compose multiple templates together.

### Composable Index Templates:

A composable index template can have multiple components:

- **Template**: This includes settings, mappings, and alias configurations.
- **Index Patterns**: These define which index names the template should be applied to.
- **Priority**: This determines the order of template application when multiple templates match an index name.
- **Data Streams**: Whether the template should apply to data streams.
- **Components**: You can have component templates that contain reusable parts for your index templates.

### Practical Example:

Let's create an example scenario where we want to apply a template to all indices that start with `logs-`.

#### Step 1: Define Component Template (Optional)

You can define a component template if you have settings or mappings that you want to reuse:

```json
PUT /_component_template/template_logs_settings
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    }
  }
}

PUT /_component_template/template_logs_mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "message": {
          "type": "text"
        },
        "severity": {
          "type": "keyword"
        }
      }
    }
  }
}
```

Here, we have defined two component templates, one for settings and another for mappings.

#### Step 2: Create Index Template

Now we create the index template that uses the component templates:

```json
PUT /_index_template/logs_template
{
  "index_patterns": ["logs-*"], 
  "template": {
    "settings": {
      "index.lifecycle.name": "logs_policy"
    },
    "aliases": {
      "logs": {}
    }
  },
  "composed_of": ["template_logs_settings", "template_logs_mappings"],
  "priority": 200
}
```

In this index template named `logs_template`:

- It applies to any index that matches `logs-*`.
- It uses the lifecycle policy `logs_policy` from our ILM example for automatic management.
- It creates an alias `logs` for each matching index.
- It composes the template from the two component templates we defined earlier.
- It has a priority of 200, meaning it will take precedence over templates with a lower priority.

#### Step 3: Index Creation

Now, when you create an index that matches the pattern `logs-*`, Elasticsearch applies the `logs_template`. For instance, creating an index named `logs-2023-11`:

```json
PUT /logs-2023-11
```

This new index will automatically inherit the settings and mappings defined by `logs_template`, which include the settings and mappings from `template_logs_settings` and `template_logs_mappings`.

This ensures that the index has a single shard and replica, uses the mappings defined for `@timestamp`, `message`, and `severity`, and is set up with the `logs` alias and ILM policy for lifecycle management. 

The power of index templates comes from their ability to standardize index configurations and to help automate index management, which is crucial for maintaining large and potentially complex Elasticsearch clusters.

## Index Snapshot

Index snapshots in Elasticsearch are backups of indices at a point in time. They are stored in a repository, which is a storage location that can be either on a local filesystem, a remote storage service like S3, HDFS, Azure, GCS, or a S3-compatible service such as MinIO.

Snapshots can be used for various purposes, such as:

- **Backup and Restore**: To backup your indices and restore them in case of data loss.
- **Migration**: To migrate data from one cluster to another, even if the clusters are running different versions of Elasticsearch.
- **Data Snapshot**: To take a snapshot of your data at a specific point in time for analysis.

### Practical Example with MinIO as the Snapshot Repository

#### Step 1: Set Up MinIO

MinIO is a high performance, distributed object storage system. It is software-defined and is 100% open source under the Apache V2 license. Before taking Elasticsearch snapshots and storing them in MinIO, you need to have MinIO set up and running. You would have an access key and a secret key for authentication, and a bucket where snapshots will be stored.

#### Step 2: Register a Snapshot Repository in Elasticsearch

You'll need to register a snapshot repository in your Elasticsearch cluster pointing to your MinIO instance:

```json
PUT /_snapshot/my_minio_repository
{
  "type": "s3",
  "settings": {
    "bucket": "elasticsearch-snapshots",
    "endpoint": "http://minio:9000",
    "protocol": "http",
    "access_key": "minioaccesskey",
    "secret_key": "miniosecretkey",
    "s3_path_style_access": true
  }
}
```

Here, `my_minio_repository` is the name of the repository in Elasticsearch, and `elasticsearch-snapshots` is the name of the bucket in MinIO.

- `endpoint`: The URL where your MinIO is accessible.
- `access_key` and `secret_key`: Your MinIO credentials.
- `s3_path_style_access`: Set this to true since MinIO uses path-style access.

#### Step 3: Take a Snapshot

To take a snapshot of an index called `my_index`, use the following command:

```json
PUT /_snapshot/my_minio_repository/my_snapshot
{
  "indices": "my_index",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

This command takes a snapshot named `my_snapshot` of `my_index` and stores it in `my_minio_repository`.

- `ignore_unavailable`: If set to true, missing indices (those that do not exist) will not cause the snapshot to fail.
- `include_global_state`: If set to false, the snapshot will not include the global cluster state.

#### Step 4: Restore from a Snapshot

To restore `my_index` from `my_snapshot`, use the following command:

```json
POST /_snapshot/my_minio_repository/my_snapshot/_restore
{
  "indices": "my_index",
  "include_global_state": false
}
```

This will restore the `my_index` index from the `my_snapshot` snapshot.

Note that if `my_index` already exists, you'll need to close or delete it before you can restore from the snapshot. Restored indices are automatically opened once the restore operation is complete.

#### Important Considerations

- The repository should be set up with the correct permissions so that the Elasticsearch cluster can access it.
- The cluster may need to be configured to allow for S3 repository types, which might involve installing the repository-s3 plugin.
- Make sure you regularly take snapshots and test the restore process to ensure your backup strategy is effective.
- Always monitor the health and space usage of your MinIO instance to ensure it can accommodate the snapshots.

This is a high-level overview and the specific commands and settings might vary based on your Elasticsearch version, your MinIO setup, network configuration, and security requirements. Always refer to the official documentation for the most accurate and up-to-date instructions.


## Snapshot Lifecycle Management

In Elasticsearch, you can schedule automated snapshot creation by using the Snapshot Lifecycle Management (SLM) feature. This allows you to define policies that dictate when to take snapshots, what to include, and how to retain them.

Here’s how you can create an SLM policy to schedule snapshots:

1. **Define the Snapshot Lifecycle Policy:**

   You create an SLM policy by sending a `PUT` request to the `_slm/policy` endpoint with a JSON body specifying the policy details. Below is an example of an SLM policy that creates daily snapshots of an index named `my_index`:

   ```json
   PUT /_slm/policy/daily-snapshots
   {
     "schedule": "0 30 1 * * ?", // This cron schedule means every day at 1:30 am
     "name": "<my_index-snapshot-{now/d}>", // Snapshot names will include the date
     "repository": "my_minio_repository", // The repository you set up for snapshots
     "config": {
       "indices": ["my_index"], // The index or indices you want to include
       "ignore_unavailable": false, // Set to true if you want the snapshot to succeed even if some indices are unavailable
       "include_global_state": false // Set to true if you want to include the cluster state
     },
     "retention": {
       "expire_after": "30d", // Snapshots older than 30 days will be deleted
       "min_count": 5, // Keep at least 5 snapshots
       "max_count": 50 // But no more than 50 snapshots
     }
   }
   ```

2. **Start and Manage the Lifecycle Policy:**

   After you've created a policy, it will automatically run according to the schedule. You can start, stop, or view the status of an SLM policy using the SLM API:

   - To see all defined policies:
     
     ```json
     GET /_slm/policy
     ```

   - To manually trigger a policy to take a snapshot:

     ```json
     POST /_slm/policy/daily-snapshots/_execute
     ```

   - To see the execution history and status of the policy:

     ```json
     GET /_slm/policy/daily-snapshots/_history
     ```

   - To disable a policy (without deleting it):

     ```json
     POST /_slm/policy/daily-snapshots/_stop
     ```

   - To re-enable a policy:

     ```json
     POST /_slm/policy/daily-snapshots/_start
     ```

3. **Monitoring and Alerts:**

   You might also want to set up monitoring and alerts for the snapshot lifecycle. This can be done using Watcher to create an alert if a snapshot fails, or integrating with other external monitoring tools you might have.

Keep in mind that SLM is a feature available from Elasticsearch 7.4 onwards, so it should be available in your Elasticsearch 8.x setup. Also, ensure that your cluster has the required permissions to interact with the snapshot repository, especially when using a repository like MinIO.
