### force shards for certain index to be stored in specific nodes in cluster

In Elasticsearch 8, you can force shards of a certain index to be stored on specific nodes within the cluster using Shard Allocation Filtering. These filters can be set using index settings which allow you to include, exclude, or require certain nodes based on custom node attributes that you define or built-in attributes like `_name`, `_ip`, `_id`, or `_host`.

Here's how you can do it:

1. **Define custom node attributes** on nodes where you want the shards to be allocated. You add these settings in the `elasticsearch.yml` configuration file on each node. For example:

```yaml
node.attr.my_attribute: "value1"
```

2. **Restart** the nodes after changing the `elasticsearch.yml` to apply the new settings.

3. **Configure index settings** to use these attributes for shard allocation. This can be done when creating a new index or updating an existing one using the Index Settings API. For example:

```json
PUT /my_index/_settings
{
  "index.routing.allocation.include.my_attribute": "value1",
  "index.routing.allocation.exclude.my_attribute": "value2",
  "index.routing.allocation.require.my_attribute": "value3"
}
```

In this example:

- `include` will allow shard allocation on nodes with `my_attribute` set to `value1`.
- `exclude` will prevent shard allocation on nodes with `my_attribute` set to `value2`.
- `require` will force shard allocation only on nodes with `my_attribute` set to `value3`.

If you want to use built-in attributes for a quick setup, you could do something like:

```json
PUT /my_index/_settings
{
  "index.routing.allocation.require._name": "node-1"
}
```

This would force all shards for `my_index` to be allocated on the node with the name `node-1`.

Make sure to replace `my_index` with the name of your index, and modify the attribute names and values to match your specific use case.

**Important Notes:**

- Use these settings with caution because if you set them incorrectly or too restrictively, you could cause shards not to be allocated properly, leading to a yellow or red cluster status.
- You should always carefully plan your shard allocation strategy in accordance with your cluster's capacity, resilience, and performance requirements.
- Using shard allocation filtering can also help with optimizing search performance by collocating related shards on the same node or within the same rack, reducing search latency.

Remember to check the official Elasticsearch documentation for any specific changes or additional options in Elasticsearch 8 regarding shard allocation.