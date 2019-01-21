# Fluentd Kinesis Firehose

Fluentd Kinesis Firehose creates a Kubernetes DaemonSet and stream the logs to [Amazon Kinesis Firehose](https://aws.amazon.com/kinesis/data-firehose/)

## Getting started with Fluentd Kinbesis Firehose

### Requirements

The Fluentd kinesis Firehose daemonset requires that an AWS account has already been provisioned with a Kinesis Firehose stream and with its data stores (eg. Amazon S3 bucket, Amazon Elasticsearch Service, etc).

Additionally there should already be a running Kubernetes cluster.

### Configuring Fluentd Kinesis Firehose

The following two parameters is the minimum that has to be configured in [fluentd-kinesis-firehose.yaml](https://github.com/cxcloud/fluentd-kinesis-firehose/blob/master/fluentd-kinesis-firehose.yaml)

| Variable | Description |
| --- | --- |
| FLUENT_KINESIS_STREAMS_REGION | AWS region for Kinesis Firehose |
| FLUENT_KINESIS_DELIVERY_STREAM_NAME | Delivery stream for Kinesis Firehose |

### Install Fluentd Kinesis Firehose

Deploy the Fluentd daemonset to the Kubernetes cluster

```bash
kubectl apply -f .
```

## Fluentd Kinesis Firehose DaemonSet

Fluentd Kinesis Firehose daemonset is using the [fluentd-kubernetes-daemonset](https://github.com/fluent/fluentd-kubernetes-daemonset) Docker image for Kinesis as the base.

### fluent.conf

The Kinesis Docker image contains preset configuration files for Kinesis Data stream that is not compatible with Kinesis Firehose. However, the image is using the [Fluent plugin for Amazon Kinesis](https://github.com/awslabs/aws-fluent-plugin-kinesis) that has support for all Kinesis services. Hence, fluent.conf has to be overwritten by a custom [configuration file](https://github.com/cxcloud/fluentd-kinesis-firehose/blob/master/fluent-conf.yaml) in order to work with Kinesis Firehose.

### parse-json.conf

Kubernetes log stdout from all the containers into log files. These log files contain JSON strings containing the container log line in a "log" attribute. In case the container log is already in JSON formant the [parse-json.conf](https://github.com/cxcloud/fluentd-kinesis-firehose/blob/master/parse-json.yaml) will parse the json object into own key/value pairs. This is useful for example if Elasticsearch is used as a data source for Kinesis Firehose.

### Logging

The Docker image itself contains pre-configured configurations, [systemd.conf and kubernetes.conf](https://github.com/fluent/fluentd-kubernetes-daemonset/tree/master/docker-image/v0.12/debian-kinesis/conf). These configurations can be overwritten if needed the same way as fluent.conf. It's also possible to totally exclude them from the custom [fluent.conf](https://github.com/cxcloud/fluentd-kinesis-firehose/blob/master/fluent-conf.yaml) configuration file.
