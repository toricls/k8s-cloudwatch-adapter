[![Build Status](https://travis-ci.org/chankh/k8s-cloudwatch-adapter.svg?branch=master)](https://travis-ci.org/chankh/k8s-cloudwatch-adapter)

# Kubernetes AWS CloudWatch Metrics Adapter

An implementation of the Kubernetes [Custom Metrics API and External Metrics
API](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-metrics-apis)
for AWS CloudWatch metrics.

This adapter allows you to scale your Kubernetes deployment using the [Horizontal Pod
Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) (HPA) with
metrics from AWS CloudWatch.

**This project is currently in Alpha status, use at your own risk.**

## Deploy
Requires a Kubernetes cluster with Metric Server deployed, Amazon EKS cluster is fine too.

Create a config for the adapter. This config is very much similar to the query structure used by
[CloudWatch GetMetricData API](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_GetMetricData.html).
Below is a sample for a custom metric named `sqslength` that is getting the average for SQS metric
`ApproximateNumberOfMessagesVisible`.

```yaml
series:
  - name: sqslength
    resource:
      resource: "deployment"
    queries:
      - id: sqs_helloworld
        metricStat:
          metric:
            namespace: "AWS/SQS"
            metricName: "ApproximateNumberOfMessagesVisible"
            dimensions:
              - name: QueueName
                value: "helloworld"
          period: 300
          stat: Average
          unit: Count
        returnData: true
```

Save this file as cloudwatch.yaml and create a config map from this file.

```
kubectl -n custom-metrics create configmap k8s-cloudwatch-adapter --from-file=cloudwatch.yaml

```

Next deploy the adapter to your Kubernetes cluster.

```
kubectl apply -f https://raw.githubusercontent.com/chankh/k8s-cloudwatch-adapter/master/deploy/adapter.yaml
```

### Verifying the deployment
You can also can also query the api to available metrics:

```bash
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq .`
```

