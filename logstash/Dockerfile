FROM opensearchproject/logstash-oss-with-opensearch-output-plugin:7.13.4
RUN rm -f /usr/share/logstash/pipeline/logstash.conf
ADD /pipeline/ /usr/share/logstash/pipeline/