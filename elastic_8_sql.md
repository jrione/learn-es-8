## SQL in Elastic 8

Elasticsearch SQL is a feature that allows you to execute SQL queries against Elasticsearch indices. As of my last update in January 2022, you can use Elasticsearch SQL via REST API, CLI, or through client libraries that support it. Here's how you can use it in Elasticsearch 8:

### 1. Using the REST API

You can execute SQL by sending requests to the SQL REST endpoint in Elasticsearch. Here's an example using `curl`:

```bash
curl -X POST "http://localhost:9200/_sql?format=txt" -H "Content-Type: application/json" -d'
{
  "query": "SELECT * FROM \"my_index\" LIMIT 10"
}'
```

Replace `my_index` with the name of your index. The `format` parameter specifies the output format (`txt`, `csv`, `json`, etc.).

### 2. Translate SQL to Elasticsearch DSL

If you want to see how an SQL query would look as an Elasticsearch DSL query, you can use the translate API:

```bash
curl -X POST "http://localhost:9200/_sql/translate" -H "Content-Type: application/json" -d'
{
  "query": "SELECT * FROM \"my_index\" WHERE \"my_field\" = 'criteria'"
}'
```

### 3. Using the CLI

Elasticsearch SQL also provides a CLI tool, which can be used if you have downloaded and extracted Elasticsearch or the Elasticsearch SQL CLI standalone tools.

You can start the CLI using the following command:

```bash
bin/elasticsearch-sql-cli
```

Once in the CLI, you can directly type your SQL queries.

### 4. Using Client Libraries

Various Elasticsearch client libraries provide support for executing SQL queries. For instance, using the Elasticsearch Python client, you could run:

```python
from elasticsearch import Elasticsearch

es = Elasticsearch()

response = es.sql.query(body={
    "query": "SELECT * FROM \"my_index\" WHERE \"my_field\" = 'criteria'"
})

print(response)
```

### 5. JDBC and ODBC Drivers

Elasticsearch provides JDBC and ODBC drivers that you can configure to interact with your Elasticsearch cluster using SQL. These can be particularly useful for integrating with tools that require a JDBC or ODBC connection, like Tableau, Excel, or various BI platforms.

### Note:

- Make sure to replace `localhost:9200` with the actual host and port where your Elasticsearch cluster is running.
- The SQL feature in Elasticsearch may not support all the functionalities of SQL in traditional relational databases. It's best to refer to the [Elasticsearch SQL documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql.html) for specifics on supported features, data types, and syntax.
- The above examples use basic text format for readability. Depending on your use case, you may choose JSON or another supported output format.

