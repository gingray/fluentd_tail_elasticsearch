# Fluentd config with Elasticsearch
Basically this config solve one problem when you have a lot of log files and you want to push them into ElasticSearch.
If you google a while you see that you need an `tail` plugn with `elasticsearch` plugin. There Docker Compose config to handle this

Assumes:
1. ElasticSearch available on host machine on port 9200 (you can wrap in to a same network but you need to remove network mode and put your own network tag)
2. All path assume that you have user `app` like `/home/app` change it to you own
3. No auto restat for container because assume that its one time task


