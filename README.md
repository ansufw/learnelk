# ELK

## Getting Started

- clean previous test files [optional]
```sh
rm -rf logs data
```

### Filebeat with input from TCP and output to NDJSON

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


## Troubleshoot

- problem: `Exiting: error loading config file: config file ("filebeat.yml") must be owned by the user identifier (uid=0) or root` 
  - solution: `sudo chown root:root filebeat.yml`
- problem: `Exiting: failed to create Beat meta file: open /usr/share/filebeat/data/meta.json.new: permission denied`
  - solution: add `user: root` to the compose yaml file under the `filebeat` service
- problem: cannot open the log file, permission denied
  - solution: `sudo chown root:root logs/tcp-logs.log*`

## Searching and Putting Data

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


