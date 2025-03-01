# ELK

## Getting Started

- clean previous test files [optional]
```sh
sudo rm -rf logs data
```

### 1. Filebeat with input from TCP and output to file

- change ownership
```sh
sudo chown root:root filebeat-1.yml
```

- start services
```sh
docker compose -f compose-1-f.yml up -d
```

#### Testing

- send request to Filebeat
```sh
echo '{"apasaja": "This is JSON from netcat"}' | nc localhost 5000
```

### 2. Run Elasticsearch Only

#### 2.1. Starting Elasticsearch

- start services
```sh
docker compose -f compose-2-e.yml up 
```

- wait for the container to be ready
```sh
curl -X GET "http://localhost:9200/_cluster/health?pretty" | grep status
```

- if the status is green, the container is ready. if red, re-run the container
```sh
docker compose -f compose-2-e.yml down -v
docker compose -f compose-2-e.yml up
```

**a note about status**
- A status of green indicates that everything is working correctly.
- A status of yellow indicates that some shards are not allocated, but the cluster is still functional.
- A status of red indicates that some shards are missing, and the cluster is not fully functional.

#### 2.2. Searching and Putting Data

- searching data
```sh
curl -X GET "localhost:9200/my_index/_search?q=content:sample"
```

- getting document by id
```sh
curl -X GET "localhost:9200/my_index/_doc/1"
```

- putting data
```sh
curl -X PUT "localhost:9200/my_index/_doc/1" -H 'Content-Type: application/json' -d'
{
  "title": "My first document",
  "content": "This is some sample content.",
  "timestamp": "2023-10-27T10:00:00"
}'
```

### 3. Run Filebeat -> Elasticsearch

#### Start Services

```sh
docker compose -f compose-3-fe.yml up 
```

if you get `Exiting: failed to create Beat meta file: open /usr/share/filebeat/data/meta.json.new: permission denied`, add `user: root` to the compose yaml file under the `filebeat` service

```sh
sudo chown root:root filebeat-3.yml
```

#### Testing

- putting data via Filebeat

```sh
# Send logs from finance service
echo '{"service":"finance","message":"{\"transaction_id\":\"tx123\",\"amount\":500,\"currency\":\"USD\",\"status\":\"completed\"}"}' | nc localhost 5140

# Send logs from authentication service 
echo '{"service":"authentication","message":"{\"user_id\":\"user456\",\"action\":\"login\",\"status\":\"success\",\"ip\":\"192.168.1.1\"}"}' | nc localhost 5140

# Send logs from subscription service
echo '{"service":"subscription","message":"{\"plan\":\"premium\",\"user_id\":\"user789\",\"action\":\"renew\",\"period\":\"monthly\"}"}' | nc localhost 5140
```

- checking the status of the cluster 

```sh
# Query finance logs
curl -X GET "localhost:9200/finance-*/_search?pretty"

# Query authentication logs
curl -X GET "localhost:9200/authentication-*/_search?pretty"

# Query subscription logs
curl -X GET "localhost:9200/subscription-*/_search?pretty"
```

### 4. Run Filebeat -> Elasticsearch -> Kibana

#### Seting

```sh
docker compose -f compose-4-fek.yml up
```

#### Testing

- putting data
```sh
# Send logs from finance service
echo '{"service":"finance","message":"{\"transaction_id\":\"tx123\",\"amount\":500,\"currency\":\"USD\",\"status\":\"completed\"}"}' | nc localhost 5140

# Send logs from authentication service 
echo '{"service":"authentication","message":"{\"user_id\":\"user456\",\"action\":\"login\",\"status\":\"success\",\"ip\":\"192.168.1.1\"}"}' | nc localhost 5140

# Send logs from subscription service
echo '{"service":"subscription","message":"{\"plan\":\"premium\",\"user_id\":\"user789\",\"action\":\"renew\",\"period\":\"monthly\"}"}' | nc localhost 5140
```

- checking via Kibana

1. Once Kibana loads, navigate to "Management" → "Stack Management"
2. Go to "Index Patterns" ("Data View" for version 8) and click "Create index pattern"
3. Create index patterns for each of your services:
  - `finance-*` for finance logs
  - `authentication-*` for authentication logs
  - `subscription-*` for subscription logs
  - `logs-*` for any uncategorized logs
4. Set up dashboards:
  - Go to "Dashboard" → "Create dashboard"
  - Add visualizations based on your data
  - You can create separate dashboards for each service or a combined overview





## Troubleshoot

- problem: `Exiting: error loading config file: config file ("filebeat.yml") must be owned by the user identifier (uid=0) or root` 
  - solution: `sudo chown root:root filebeat.yml`
- problem: `Exiting: failed to create Beat meta file: open /usr/share/filebeat/data/meta.json.new: permission denied`
  - solution: add `user: root` to the compose yaml file under the `filebeat` service
- problem: cannot open the log file, permission denied
  - solution: `sudo chown root:root logs/tcp-logs.log*`


