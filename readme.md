# Fluentd config with Elasticsearch
Basically this config solve one problem when you have a lot of log files and you want to push them into ElasticSearch.
If you google a while you see that you need an `tail` plugn with `elasticsearch` plugin. There Docker Compose config to handle this

Assumes:

1. ElasticSearch available on host machine on port 9200 (you can wrap in to a same network but you need to remove network mode and put your own network tag)
2. All path assume that you have user `app` like `/home/app` change it to you own
3. No auto restat for container because assume that its one time task

Information about paths:
` /home/app/logs:/fluentd/log/` path where logs are stored on your machine
`/home/app/flunetd/conf:/fluentd/etc/` path where config for fluentd is stored
`/home/app/flunetd/buffer:/fluentd/buffer` path where file buffer need to be store
`/home/app/fluentd/pos:/fluentd/pos` path where position of reading files need to be stored for plugin in_tail


Bit more information about config

```
<source>
  @type tail #plugin type for read from files
  path /fluentd/log/*.log #assume that you need push all files in directory with ext *.log
  tag fluentd #tag for fluentd 
  format json #assume that you logs in valid json format each line json object
  time_key time #assume that your time key is field time in json object 
  time_format %Y-%m-%dT%H:%M:%S%z #time format in time key
  read_from_head true #mean that all files will be readed form begining
</source>

<filter **> #maybe you don't need this filter because he do one thing its keep time key in record basically you can remove it 
  @type record_transformer
  enable_ruby
  <record>
    time ${time.strftime('%Y-%m-%dT%H:%M:%S%z')}
    from_file true
  </record>
</filter>

<match **> #elasticsearch plugin config
    @type elasticsearch
    host 127.0.0.1
    port 9200
    logstash_format true #keeplogstash format
    include_tag_key true 
    logstash_prefix fluentd-files
    logstash_dateformat %Y
    tag_key @log_name

    time_key time #time key for record
    time_key_format %Y-%m-%dT%H:%M:%S%z #format that was set above
    reconnect_on_error true #reconnect each time on error
    reload_on_failure true 
    reload_connections false
    request_timeout 120s #it set 120s becuase if your elasticsearch server is slow 
    include_timestamp true
    <buffer> #thats important part because by default using memorry buffer and it will overflow very fast if you have enough logs
      @type file
      flush_interval 20s
      retry_type periodic
      retry_forever true
      retry_wait 10s
      chunk_limit_size 4Mb
      queue_limit_length 4096
      total_limit_size 60Gb
      path /fluentd/buffer/elastic.buff
    </buffer>
</match>
```
PS

I've spent a lot of time to find right config buts maybe I was just so lucky. I'm not sure thats all of these working properly or most efficiently if you find something that maybe improve please point me about that. Thank you.

* [Fluentd in_tail](https://docs.fluentd.org/v1.0/articles/in_tail)
* [Fluentd ElastciSearch](https://docs.fluentd.org/v1.0/articles/out_elasticsearch)
* [Fluentd ElastciSearch repo](https://github.com/uken/fluent-plugin-elasticsearch)
* [Fluentd Buffers](https://docs.fluentd.org/v1.0/articles/buffer-section)
* [Fluentd Buffer Overview](https://docs.fluentd.org/v1.0/articles/buffer-plugin-overview)
* [Fluentd Buffer File](https://docs.fluentd.org/v1.0/articles/buf_file)
