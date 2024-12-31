# Logstash + Vespa = ❤️

Index CSV data into Vespa using Logstash. You only need Docker/Podman/...

Here's how:
```
# clone the repo
git clone https://github.com/radu-gheorghe/logstash-vespa-output-demo.git
cd logstash-vespa-output-demo

# run the docker compose file with Vespa and Logstash
docker compose up # or podman compose up

# once everything settles, you can check Vespa for the documents
curl -XPOST -H "Content-Type: application/json" -d\
  '{  "yql": "select * from sources * where true"}'\
   'http://localhost:8080/search/' | jq .
```

## Details

### Vespa

An awesome search engine. More details [here](https://docs.vespa.ai/en/overview.html). The first container in the compose file is a Vespa container.

### Logstash

A very flexible ETL tool. More details [here](https://www.elastic.co/logstash). The second container in the compose file is a Logstash container, which is configured to index the CSV data into Vespa. Here's how:
1. First, we need a Vespa [application package](https://docs.vespa.ai/en/application-packages.html). Here, [blog_posts_app](blog_posts_app) contains a simple application package, configuring everything from number of nodes to schema.
2. During Logstash startup, we deploy the application package to Vespa, to make sure that e.g. the schema is there.
3. Also during Logstash startup, we install the [Vespa output plugin for Logstash](https://github.com/vespa-engine/vespa/tree/master/integration/logstash-plugins/logstash-output-vespa).
4. Finally, we run Logstash with [logstash.conf](logstash.conf), which reads the CSV file, parses it, and writes the documents to Vespa.

### CSV data

A simple CSV file with blog posts. You can replace it with your own data, just make sure to update:
- [logstash.conf](logstash.conf) to parse the right fields.
- The [Vespa schema](blog_posts_app/schemas/post.sd) to match the fields.
- If you change the document type from `post` to something else, make sure to update:
  - the schema file name: it needs to match the document type name within it
  - `document_type` in [logstash.conf](logstash.conf#L39)
  - the document type in [services.xml](blog_posts_app/services.xml#L77)
- If you change the CSV file name, make sure to update:
  - [logstash.conf](logstash.conf#L5) to point to the right filename
  - the volume path in [docker-compose.yml](docker-compose.yml#L24).

