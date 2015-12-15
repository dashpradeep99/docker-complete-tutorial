# Lab A : Setting up an ELK stack

> **Difficulty**: ???

> **Time**: ???

> **Tasks**:
>
* [Introduction](#introduction)
* [Task 1: Allocate storage space for logs](#task-1-allocate-space)
* [Task 2: Compose containers](#task-2-compose-containers)
* [Task 3: Connect and browse logs](#task-3-browse-logs)

## Introduction

One of the more popular logging stacks is composed of Elasticsearch, Logstash and Kibana or ELK. The following example lab demonstrates how to set up an example deployment which can be used for logging.  

## Task 1: Allocate storage space for logs

We are all about containers, and our logging solution will be container-ized!    First, lets create some space to put some data

 docker volume create --name orca-elasticsearch-data

## Task 2: Compose containers

We haven't composed our containers so that you can see and start each container separately

       $ docker run -d \
       --name elasticsearch \ 
       -v orca-elasticsearch-data:/usr/share/elasticsearch/data \
       elasticsearch elasticsearch -Des.network.host=0.0.0.0

       $ docker run -d \
       -p 514:514 \
       --name logstash \
       --link elasticsearch:es \
       logstash \
       sh -c "logstash -e 'input { syslog { } } output { stdout { } elasticsearch { hosts => [ \"es\" ] } } filter { json { source => \"message\" } }'"

       $ docker run -d \ 
       --name kibana \
       --link elasticsearch:elasticsearch \
       -p 5601:5601 \
       kibana

## Task 3: Connect and browse logs

- You can then browse to your host system running kibana, port 5601 and browse log/event entries. You should specify the "time" field for indexing.  Note: When deployed in production, you should secure kibana (not described in this doc)

- Some Example Searches.   Here are a few examples demonstrating some ways to view the aggregated log data:

-- Show all the modifications on the system

       type:"api" AND (tags:"post" OR tags:"put" OR tags:"delete")

-- Show all access from a given user

       username:"admin"

-- Show all authentication failures on the system

       type:"auth fail" 


Now that we have a logging solution setup, we can later instruct DUCP to use our logging solution to aggregate logging.

## Conclusion

At this point you have successfully set up a simple ELK stack and retrieved common log queries.

### Share on Twitter!

<p>
<a href="http://ctt.ec/R3e56" target=“_blank”>
<img src="http://www.wyntercon.com/wp-content/uploads/2015/04/twitter-bird-blue-on-white-small.png" width="100" height="100">
</p>

## Clean up

If you plan to do another lab, you need to cleanup your EC2 instances. Cleanup removes any environment variables, configuration changes, Docker images, and running containers. To do a clean up, log into each EC2 instance and run the following:

```bash
$ source /home/ubuntu/cleanup.sh
```

## Related information
