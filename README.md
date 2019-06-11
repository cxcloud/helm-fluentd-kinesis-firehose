# Fluentd Kinesis Firehose Helm Chart

Fluentd Kinesis Firehose Helm Chart creates a Kubernetes DaemonSet and stream the logs to [Amazon Kinesis Firehose](https://aws.amazon.com/kinesis/data-firehose/)

## Getting started

### Requirements

The Fluentd kinesis Firehose daemonset requires that an AWS account has already been provisioned with a Kinesis Firehose stream and with its data stores (eg. Amazon S3 bucket, Amazon Elasticsearch Service, etc). Available is a CX Cloud provided Terraform module, [terraform-kinesis-firehose-elasticsearch](https://github.com/cxcloud/terraform-kinesis-firehose-elasticsearch) for helping with the installation.

Additionally there should already be a running Kubernetes cluster.

### Installing the chart

Clone the repository:

```bash
git clone git@github.com:cxcloud/helm-fluentd-kinesis-firehose.git
```

To install the chart with the release name `my-release` into the namespace `kube-system`:

```bash
helm install helm-fluentd-kinesis-firehose --name my-release --namespace kube-system
```

### Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```bash
helm delete --purge my-release
```

### Configuration

The following table list the configurable parameters of the chart and their default values.

| Parameter | Description | Default |
| --- | --- | --- |
| image.repository | Repository for docker image  | fluent/fluentd-kubernetes-daemonset |
| image.tag | Docker image tag | v0.12-debian-kinesis |
| image.pullPolicy | Image pull policy | IfNotPresent |
| nameOverride | Override the name for resources | Empty string |
| fullnameOverride | Override the full name for resources | Empty string |
| fluentEnvs.assumeRole | Should Fluentd assume IAM role for accessing Kinesis | false |
| fluentEnvs.roleARN | AWS IAM role | Empty string |
| fluentEnvs.roleSession | Role session | Empty string |
| fluentEnvs.kinesisRegion | AWS region for Kinesis | eu-west-1 |
| fluentEnvs.kinesisStreamName | Kinesis stream name | cxcloud |
| fluentEnvs.kinesisIncludeTimeKey | Include time key | true |
| fluentEnvs.bufferChunkLimitSize | Buffer chunk limit size | 2M |
| fluentEnvs.bufferQueueLimitLength | Buffer queue limit length | 32 |
| fluentConf | fluent.conf content | Not set |
| systemdConf | systemd.conf content | Not set |
| kubernetesConf | kubernetes.conf content | Not set |
| resources | Resources allocation (Requests and Limits) | {} |
| rbac.create | Whether RBAC resources are created | true |
| instanceProfileCredentials | AWS instance profile credentials | false |
| instanceProfileIPAddress | IP Address to get instance profile credentials | 169.254.169.254 |
| instanceProfilePort | Port to get instance profile credentials | 80 |

### Overriding configurations

Parameters can be overridden when installing the chart. For example assuming an IAM role for another AWS account were Kinesis Firehose is running:

```bash
helm install helm-fluentd-kinesis-firehose --name my-release --namespace kube-system \
  --set fluentEnvs.useRole=true \
  --set fluentEnvs.roleARN=arn:aws:iam::012345678901:role/KinesisFirehose \
  --set fluentEnvs.roleSession=kops-nodes
```

Alternatively, a YAML file ([values.yaml](values.yaml)) that specifies the values for the parameters can be provided while installing the chart:

```bash
helm install helm-fluentd-kinesis-firehose --name my-release --namespace kube-system -f values.yaml
```

## Fluentd DaemonSet

The daemonset is using the [fluentd-kubernetes-daemonset](https://github.com/fluent/fluentd-kubernetes-daemonset) Docker image for Kinesis as the base.

### fluent.conf

The Kinesis Docker image contains preset configuration files for Kinesis Data stream that is not compatible with Kinesis Firehose. However, the image is using the [Fluent plugin for Amazon Kinesis](https://github.com/awslabs/aws-fluent-plugin-kinesis) with support for all Kinesis services. Hence, fluent.conf has to be overwritten by a custom [configuration file](https://github.com/cxcloud/helm-fluentd-kinesis-firehose/blob/master/templates/fluent-conf.yaml) in order to work with Kinesis Firehose.

### kubernetes.conf

The Docker image itself contains a pre-configured configuration, [kubernetes.conf](https://github.com/fluent/fluentd-kubernetes-daemonset/blob/master/docker-image/v0.12/debian-kinesis/conf/kubernetes.conf). The content can be overwritten from the `kubernetesConf` variable in configurations.

### systemd.conf

The Docker image itself contains a pre-configured configuration, [systemd.conf](https://github.com/fluent/fluentd-kubernetes-daemonset/blob/master/docker-image/v0.12/debian-kinesis/conf/systemd.conf). The content can be overwritten from the `systemdConf` variable in configurations.

### parse-json.conf

Kubernetes log stdout from all the containers into log files. These log files contain JSON strings containing the container log line in a "log" attribute. In case the container log is already in JSON formant the [parse-json.conf](https://github.com/cxcloud/helm-fluentd-kinesis-firehose/blob/master/templates/parse-json.yaml) will parse the json object into own key/value pairs. This is useful for example if Elasticsearch is used as a data source for Kinesis Firehose.

## License

MIT © [CXCloud](https://docs.cxcloud.com)
