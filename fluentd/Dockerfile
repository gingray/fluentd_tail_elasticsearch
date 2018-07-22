FROM fluent/fluentd:v1.2.1
RUN mkdir -p /fluentd/buffer
RUN mkdir -p /fluentd/pos
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-rdoc", "--no-ri", "--version", "2.10.3"]
