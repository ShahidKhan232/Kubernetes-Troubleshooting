# kubectl diff

> This article explores the usage of the kubectl diff command to compare YAML configurations with deployed resources in a Kubernetes cluster.

In this article, we explore the usage of the `kubectl diff` command. This powerful tool allows you to compare the configuration specified in your YAML files against the corresponding resources deployed in your Kubernetes cluster. It is particularly useful for identifying discrepancies and ensuring consistency between your intended state and the actual state of your environment.

<Callout icon="lightbulb" color="#1CB2FE">
  Before using `kubectl diff`, ensure you have the correct context set for your Kubernetes cluster.
</Callout>

## Basic Command Usage

The command to initiate a diff is simple:

```bash  theme={null}
kubectl diff
```

As the name suggests, `kubectl diff` displays the differences between the resource defined in your configuration file and the live resource in your cluster.

## Example: Comparing a Redis Deployment

Consider the following Redis Deployment YAML file as an example:

```yaml  theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: redis
  name: redis-deployment
  labels:
    app: redis
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:6.2.0
          ports:
            - containerPort: 6379
```

Suppose you query the deployments in the `redis` namespace on your cluster:

```bash  theme={null}
controlplane ~ ➜ k get deployments.apps -n redis
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment    2/2     2            2           37m
```

In this case, you can observe that the deployed resource contains 2 replicas, whereas the YAML file specifies 3 replicas. To compare your configuration file with the live state, run:

```bash  theme={null}
kubectl diff -f redis.yaml
```

The output will highlight the differences between the file and the deployed resource. Below is an example of what the diff output might look like:

```diff  theme={null}
diff -u -N /tmp/LIVE-1719420227/apps.v1.Deployment.redis-deployment /tmp/MERGED-905484664/apps.v1.Deployment.redis.redis-deployment
--- /tmp/LIVE-1719420227/apps.v1.Deployment.redis-deployment	2024-04-28 00:41:59.388069648 +0000
+++ /tmp/MERGED-905484664/apps.v1.Deployment.redis.redis-deployment	2024-04-28 00:41:59.388069648 +0000
@@ -6,7 +6,7 @@
 kubectl.kubernetes.io/last-applied-configuration: |
   {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"redis"},"name":"redis-deployment","namespace":"redis"},"spec":{"replicas":2,"selector":{"matchLabels":{"app":"redis"}},"template":{"metadata":{"labels":{"app":"redis"}},"spec":{"containers":[{"image":"redis:6.2.0","name":"redis", ... }]} }}
 creationTimestamp: "2024-04-28T00:04:43Z"
 generation: 1
 labels:
   app: redis
 name: redis-deployment
@@ -15,7 +15,7 @@
   uid: 2a5ed860-51b0-485d-b8cd-056b49324328
 spec:
   progressDeadlineSeconds: 600
-  replicas: 2
+  replicas: 2
   revisionHistoryLimit: 10
   selector:
     matchLabels:
```

The diff output clearly indicates that while the file defines 3 replicas, the active deployment in the cluster currently has 2 replicas. This example demonstrates how `kubectl diff` can guide you in cross-referencing your configuration files with live deployments.

<Callout icon="triangle-alert" color="#FF6B6B">
  Ensure that any changes you make based on diff outputs are tested before applying them to production environments. This prevents unintended disruptions.
</Callout>

## Conclusion

The `kubectl diff` command is an indispensable tool for Kubernetes administrators, allowing you to verify the differences between your declared configuration and the deployed state. By leveraging this command, you can maintain consistency across your environments and manage changes more confidently.

We hope this guide has provided a clear understanding of how to use `kubectl diff`. Happy deploying, and see you in the next lesson!

## Further Reading

* [Kubernetes Documentation](https://kubernetes.io/docs/)
* [Kubernetes Diff Plugin FAQ](https://kubernetes.io/docs/reference/command-line-tools-reference/kubectl-diff/)
