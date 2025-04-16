
# Word Count and Web App Deployment on AWS

## Task 1: Word Count on AWS EC2 using PySpark

### Objective
Use PySpark on an AWS EC2 instance to count words in a text file stored in an S3 bucket.

### S3 Bucket Used
- Bucket name: `wordcountcc`
- Input file: `ccwordcount.txt`
- Output folder: `output_folder/`

### Files in this repo:
- `ccwordcount.txt` – Input dataset
- `word_count.py` – PySpark word count logic
- `output_folder_` – Output with part files from the job

### PySpark Word Count Code

```python
from pyspark.sql import SparkSession

# AWS Credentials
AWS_ACCESS_KEY_ID = 'YOUR_ACCESS_KEY' 
AWS_SECRET_ACCESS_KEY = 'YOUR_SECRET_KEY' 

# S3 paths
S3_INPUT = 's3a://wordcountcc/ccwordcount.txt'
S3_OUTPUT = 's3a://wordcountcc/output_folder/'

# Spark Session
spark = SparkSession.builder \
    .appName("WordCount") \
    .config("spark.jars.packages", "org.apache.hadoop:hadoop-aws:3.3.1,com.amazonaws:aws-java-sdk-bundle:1.11.901") \
    .getOrCreate()

# Hadoop S3 Configuration
hadoop_conf = spark.sparkContext._jsc.hadoopConfiguration()
hadoop_conf.set("fs.s3a.access.key", AWS_ACCESS_KEY_ID)
hadoop_conf.set("fs.s3a.secret.key", AWS_SECRET_ACCESS_KEY)

# Word Count Logic
text_file = spark.sparkContext.textFile(S3_INPUT)
counts = text_file.flatMap(lambda line: line.split()) \
                  .map(lambda word: (word, 1)) \
                  .reduceByKey(lambda a, b: a + b)
counts.saveAsTextFile(S3_OUTPUT)
spark.stop()
```

### Commands Used (EC2)

```bash
# Install Java & Python
sudo yum install java-11 -y
sudo yum install python3-pip -y

# Increase /tmp to avoid memory issues
sudo mount -o remount,size=2G /tmp

# Install PySpark
pip3 install pyspark

# Run the Spark job
spark-submit word_count.py
```

---

## Task 2: Deploy Node.js App with Docker on EC2

### Objective
Run a basic web server in a Docker container and expose it via EC2.

### App Files

- `server.js`
- `package.json`
- `Dockerfile`

### Node.js Code

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
    res.send('Hello, World! Running in Docker.');
});

app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});
```

### Dockerfile

```Dockerfile
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Commands (EC2)

```bash
# Install Docker
sudo yum update -y
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user
logout  # re-login after usermod

# Clone or create Node app
mkdir node-webserver && cd node-webserver
npm init -y
npm install express

# Build and run
docker build -t bhargavipotu269/webserver:latest .
docker run -d -p 80:3000 bhargavipotu269/webserver:latest
```

### URLs
- **Web App Access**: http://52.200.243.129/
- **DockerHub Repo**: https://hub.docker.com/repository/docker/bhargavipotu269/webserver/general
- **GitHub Repo**: https://github.com/bhargavi-potu/word-count-spark-Bhargavi/tree/main

---

## Author
Bhargavi Potu
