# Web-Server-Log-Analysis

This repository is designed to test MapReduce jobs using a simple word count dataset.

## Objectives
### Problem Statement
You need to process a web server log file where each entry is recorded as a row in a CSV file. The tasks include:

1. Counting the total number of web requests processed.
2. Analyzing the frequency of HTTP status codes (e.g., 200, 404, 500).
3. Identifying the top 3 most visited URLs.
4. Conducting a traffic source analysis to determine the most common user agents.
5. Detecting IP addresses with more than three failed requests (status 404 or 500).
6. Analyzing traffic trends by computing the number of requests per minute.
7. Implementing partitioning by status code to enhance query performance.

## Setup and Execution

### 1. **Start the Hadoop Cluster**
To initiate the Hadoop cluster, execute:

```bash
docker compose up -d
```

### 2. **Count Total Web Requests**
This query computes the total number of requests in the web server logs:

```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/count web pages'
SELECT COUNT(*) AS total_requests FROM web_server_logs;
```

### 3. **Analyze Status Codes**
This query retrieves the frequency of each HTTP status code (e.g., 200, 404, 500):

```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Analyse status codes'
SELECT status, COUNT(*) AS count FROM web_server_logs GROUP BY status;
```

### 4. **Identify Most Visited Pages**
This query extracts the top three most frequently visited URLs:

```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Most visited pages'
SELECT url, COUNT(*) AS visits
FROM web_server_logs
GROUP BY url
ORDER BY visits DESC
LIMIT 3;
```

### 5. **Traffic Source Analysis**
This query identifies the most commonly used user agents (browsers):

```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/traffic source'
SELECT user_agent, COUNT(*) AS count
FROM web_server_logs
GROUP BY user_agent
ORDER BY count DESC;
```

### 6. **Detect Suspicious Activity**
This query finds IP addresses with more than three failed requests (status codes 404 or 500):

```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Detect Suspicious'
SELECT ip, COUNT(*) AS failed_requests
FROM web_server_logs  
WHERE status IN (404, 500)
GROUP BY ip
HAVING COUNT(*) > 3;
```

### 7. **Analyze Traffic Trends**
This query calculates the number of requests per minute to observe traffic patterns:

```bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Analyse Traffic'
SELECT substr(`timestamp`, 1, 16) AS time_minute, COUNT(*) AS requests
FROM web_server_logs
GROUP BY substr(`timestamp`, 1, 16)
ORDER BY time_minute;
```

### 8. **Implement Partitioning**
To optimize query performance, this step creates a partitioned Hive table based on HTTP status codes:

```bash
CREATE TABLE web_logs_partitioned (
    ip STRING,
    `timestamp` STRING,
    url STRING,
    user_agent STRING
)
PARTITIONED BY (status INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

SET hive.exec.dynamic.partition.mode=nonstrict;
INSERT OVERWRITE TABLE web_logs_partitioned PARTITION (status)
SELECT ip, `timestamp`, url, user_agent, status FROM web_server_logs;
```

### 9. **View Partitioning Output**
To check the output of the partitioned table, use:

```bash
'/user/hive/output/web_serverlogs_partitioned'
SELECT * FROM default.web_logs_partitioned LIMIT 100;
```

### 10. **Access Hive Server Container**
To open an interactive Bash session inside the Hive server container, run:

```bash
docker exec -it hive-server /bin/bash
```

### 11. **Copy Output from HDFS to Local Filesystem (Inside the Container)**
Retrieve query results stored in HDFS and save them locally inside the container:

```bash
hdfs dfs -get /user/hive/output /tmp/output
```

### 12. **Exit the Container**
To exit the Hive server container:

```bash
exit
```

### 13. **Check Current Working Directory on the Host**
Verify the present working directory where you will copy the output files:

```bash
pwd
```

### 14. **Copy Output Files from Docker Container to Host Machine**
Copy the `/tmp/output` directory from the Hive server container to your project directory on the host machine:

```bash
docker cp hive-server:/tmp/output /........(Use the directory path obtained from `pwd`)
```

### 15. **Commit Changes to GitHub**

```bash
git add .
git commit -m "Updated analysis results"
git push origin main
```

