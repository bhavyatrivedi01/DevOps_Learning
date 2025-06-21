
finOps :- Cost Optimization
What is  ELK Stack :- 
Elastic search- Elasticsearch is a distributed search and analytics engine, scalable data store, and vector database built on Apache Lucene. It's optimized for speed and relevance on production-scale workloads. Use Elasticsearch to search, index, store, and analyze data of all shapes and sizes in near real time.
Logstash:- Logstash is a lightweight, open-source, server-side data processing pipeline that allows you to collect data from various sources, transform it on the fly, and send it to your desired destination. It is most often used as a data pipeline for Elasticsearch, an open-source analytics and search engine.
Kibana :-  Kibana is a data visualization and exploration tool used for log and time-series analytics, application monitoring, and operational intelligence use cases.

scenario:- Xyz company uses ELK stack (ElasticSearch , Logstash , Kibana). They do all their log analysis through logstash.
logstash stores almost all their company specific infrastructure related logs (Application log, Kubernetes , Infrastructure logs , Jenkins (VAT , staging , pre-prod , prod). 
Jenkins related notification related to failed jobs sent using email and slack.
Company is looking for cost reduction for the services they are using on AWS.
Key Point :- Most log files stored into logstash are storing from build logs of Jenkins , which they only kept for backup and don’t really do any analysis on those logs.
Suggested Solution:- All the logs related to Jenkins file are created when developers run different jobs during the day and that is a biggest chunk of data , this data is only being kept as backup storage , so this data should be stored into s3 bucket or s3 deep archive , which turns out to be the cheapest option among all aws services.

Shell Script (Credit: Abhishek Veeramalla):-
#!/bin/bash
# Author: Abhishek Veeramalla

# Variables
JENKINS_HOME="/var/lib/jenkins"  # Replace with your Jenkins home directory
S3_BUCKET="s3://your-s3-bucket-name"  # Replace with your S3 bucket name
DATE=$(date +%Y-%m-%d)  # Today's date

# Check if AWS CLI is installed
if ! command -v aws &> /dev/null; then
    echo "AWS CLI is not installed. Please install it to proceed."
    exit 1
fi

# Iterate through all job directories
for job_dir in "$JENKINS_HOME/jobs/"*/; do
    job_name=$(basename "$job_dir")
    
    # Iterate through build directories for the job
    for build_dir in "$job_dir/builds/"*/; do
        # Get build number and log file path
        build_number=$(basename "$build_dir")
        log_file="$build_dir/log"

        # Check if log file exists and was created today
        if [ -f "$log_file" ] && [ "$(date -r "$log_file" +%Y-%m-%d)" == "$DATE" ]; then
            # Upload log file to S3 with the build number as the filename
            aws s3 cp "$log_file" "$S3_BUCKET/$job_name-$build_number.log" --only-show-errors
            
            if [ $? -eq 0 ]; then
                echo "Uploaded: $job_name/$build_number to $S3_BUCKET/$job_name-$build_number.log"
            else
                echo "Failed to upload: $job_name/$build_number"
            fi
        fi
    done
done

Implematation:-
Ssh into ec2 instance
Install Jenkins service on Ec2 instance
 
Create job
 
Install aws cli
-	Apt install aws-cli
Configure aws
Input security access key
Locate Jenkins build logs folder
/var/lib/Jenkins/jobs/<name-of-your-job>/builds/1
Create s3 bucket
 
Execute shellscript
 
 

