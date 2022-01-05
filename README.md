## Clone the repository 
Clone the repository and run `docker-compose up` this may require you change some systemctl parameters to get elasticsearch to run without issues e.g
`sudo sysctl -w vm.max_map_count=262144` this sets the kernel param temporarily.

## Run the following queries in the opensearch dashboard
Once the docker containers are successfully running, visit http://127.0.0.1:5601 and use admin:admin as the credentials and then visit the following url http://127.0.0.1:5601/app/dev_tools#/console and perform the following:

```
PUT _index_template/logstash-template
{
  "index_patterns": [
    "my-data-stream",
    "logs-*"
  ],
  "data_stream": {},
  "priority": 100
}


PUT _index_template/logs-template-nginx
{
  "index_patterns": "logs-nginx",
  "data_stream": {
    "timestamp_field": {
      "name": "request_time"
    }
  },
  "priority": 200,
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}

PUT _data_stream/logs-nginx
```

## Modify the docker-compose file
Now that elasticsearch is running and the stream has been created uncomment the logstash portion of the docker-compose file and run
`docker-compose up --build` this will create the logstash container with the opensearch plugin. We need to do this eitherwise logstash will try and make the index instead of a stream.

## Send data to logstash
Use the following command to send log data to logstash, this will then trigger the error as logstash is unable to create the stream.

```
curl -H "content-type: application/json" -XPUT 'http://127.0.0.1:5044' -d '{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}'
```

You should receive something like the following message
```
opensearch-logstash      | [2022-01-05T02:47:57,583][WARN ][logstash.outputs.opensearch][main][4f325355d5a2b6a9d459af1c83aa2ca4b8cd489ec67c5a6de26d78da966041a3] Could not index event to OpenSearch. {:status=>400, :action=>["index", {:_id=>nil, :_index=>"logs-nginx", :routing=>nil}, {"@version"=>"1", "@timestamp"=>2022-01-05T02:47:57.469Z, "post_date"=>"2009-11-15T14:12:12", "user"=>"kimchy", "headers"=>{"content_length"=>"110", "http_host"=>"127.0.0.1:5044", "request_path"=>"/", "http_accept"=>"*/*", "request_method"=>"PUT", "http_version"=>"HTTP/1.1", "http_user_agent"=>"curl/7.74.0", "content_type"=>"application/json"}, "host"=>"172.21.0.1", "message"=>"trying out Elasticsearch"}], :response=>{"index"=>{"_index"=>"logs-nginx", "_type"=>"_doc", "_id"=>nil, "status"=>400, "error"=>{"type"=>"illegal_argument_exception", "reason"=>"only write ops with an op_type of create are allowed in data streams"}}}}
```
