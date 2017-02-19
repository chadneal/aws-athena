# AWS Athena

Scripts for setting up AWS Athena with common AWS Services.

### NGINX

**Sample Request**
54.255.254.242 - - [19/Feb/2017:19:00:26 +0000] "GET / HTTP/1.1" 200 3770 "-" "Amazon Route 53 Health Check Service; ref:6ba00c53-2538-40ee-b47f-b01eb3dfd95e; report http://amzn.to/1vsZADi" "-"
```bash
remote_addr string                54.255.254.242
remote_user string,               -
time_local string,                [19/Feb/2017:19:00:26 +0000]
http_verb string,                 \"GET
url string,                       /
http_ver string                   HTTP/1.1\"
status int                        200
body_bytes_sent int               3770
http_referer string               "-"
http_user_agent string            "Amazon Route 53 Health Check Service; ref:6ba00c53-2538-40ee-b47f-b01eb3dfd95e; report http://amzn.to/1vsZADi" "-" 
```

**Parsing Regex**
```bash
remote_addr string,               ([0-9\\.]+) -  \\[([^\\]]*)\\]
remote_user string,               ([^ ]*)
time_local string,                \\[([^\\]]*)\\]
http_verb string,                 \"([^ ]*)
url string,                       ([^ ]*)
http_ver string,                  ([^ ]*)\"
status int,                       ([0-9]*)
body_bytes_sent int,              ([0-9]*)
http_referer string,              ([-0-9]*)
http_user_agent string            \"(.*)\" \"(.*)\"
```

**DDL**
```bash
CREATE EXTERNAL TABLE IF NOT EXISTS five3.nginx (
  remote_addr string,
  remote_user string,
  time_local string,
  http_verb string,
  url string,
  http_ver string,
  status int,
  body_bytes_sent int,
  http_referer string,
  http_user_agent string
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
  "input.regex" = "([0-9\\.]+) - ([^ ]*) \\[([^\\]]*)\\] \"([^ ]*) ([^ ]*) ([^ ]*)\" ([0-9]*) ([0-9]*) \"(.*)\" \"(.*)\""
  ) 
-- Be explicit where we are going to store this table
LOCATION 's3://logs-bucket/logs/kfh/'

```

**Query**
```sql

/* Query */
SELECT * FROM five3.nginx LIMIT 100;

/* 500s */
SELECT
  time_local,
  remote_addr,
  status,
  url
FROM five3.nginx
WHERE status>=500 AND status<600
ORDER BY time_local DESC
LIMIT 100;

/* Unique IPs */
SELECT remote_addr, COUNT(*) as count
FROM five3.nginx
WHERE status>=200 AND status<300
GROUP BY remote_addr
ORDER BY count DESC;
```
