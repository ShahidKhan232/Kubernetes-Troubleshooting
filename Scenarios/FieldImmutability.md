# Field Immutability

> This article explains the "field is immutable" error in Kubernetes and how to resolve it when modifying resource configurations.

In this lesson, we explore the common "field is immutable" error encountered when modifying Kubernetes resources, understand why it occurs, and learn how to resolve it effectively. This error typically arises when you attempt to change immutable fields within an existing resource, leading to potential disruption if altered.

## Deploying a Redis Deployment

Initially, we deploy a Redis instance using a Kubernetes Deployment. Below is the configuration provided in the "redis.yaml" file:

```yaml  theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  labels:
    app: redis
spec:
  replicas: 1
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
        image: redis:latest
        ports:
        - containerPort: 6379
```

Apply the configuration using the following command:

```bash  theme={null}
controlplane ~ ➜ k apply -f redis.yaml
deployment.apps/redis-deployment created
```

## Modifying the Deployment

After a successful deployment, imagine you want to add an additional label (e.g., "tier: backend") to the selector's `matchLabels` and the pod template. You update "redis.yaml" as shown below:

```yaml  theme={null}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  labels:
    app: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      tier: backend
  template:
    metadata:
      labels:
        app: redis
        tier: backend
    spec:
      containers:
      - name: redis
        image: redis:latest
        ports:
        - containerPort: 6379
```

Then, if you run:

```bash  theme={null}
controlplane ~ ➜ k apply -f redis.yaml
```

Kubernetes returns an error similar to:

```bash  theme={null}
The Deployment "redis-deployment" is invalid: spec.selector: Invalid value: 
v1.LabelSelector{MatchLabels:map[string]string{"app":"redis", "tier":"backend"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable
```

<Callout icon="lightbulb" color="#1CB2FE">
  This error occurs because the `matchLabels` field in the selector is immutable to prevent unintended disruptions in traffic routing and the overall stability of your application. Kubernetes follows the immutable infrastructure paradigm, which ensures that changes to critical fields require a new resource creation.
</Callout>

## Resolving the Error

To update immutable fields, delete the existing deployment and recreate it with the new configuration. Follow these steps:

1. **Delete the Existing Deployment**

   Execute the command below to delete the current deployment:

   ```bash  theme={null}
   controlplane ~ ➜ k delete deployments.apps redis-deployment
   deployment.apps "redis-deployment" deleted
   ```

2. **Apply the New Configuration**

   Now apply the updated configuration:

   ```bash  theme={null}
   controlplane ~ ➜ k apply -f redis.yaml
   deployment.apps/redis-deployment created
   ```

<Callout icon="triangle-alert" color="#FF6B6B">
  Immutable fields, such as `spec.selector.matchLabels`, cannot be modified after resource creation. Always plan your label configurations ahead of time to avoid unnecessary downtime.
</Callout>

## Key Takeaways

| Action            | Explanation                                                                      |
| ----------------- | -------------------------------------------------------------------------------- |
| Deploy            | Create resources using configuration files (e.g., "redis.yaml").                 |
| Modify            | Changing labels in immutable fields (e.g., `matchLabels`) will raise an error.   |
| Delete & Recreate | To update immutable fields, delete the resource and apply the new configuration. |

* Immutable fields like the selector's `matchLabels` safeguard the application's stability.
* To change immutable configurations, remove the existing resource and recreate it.
* This principle applies to other resources like ConfigMaps and Secrets, where metadata changes might require redeployment.

By understanding the constraints of immutable fields in Kubernetes, you can prevent unexpected behavior and ensure seamless application updates and maintenance.

For further details and related best practices, refer to the official [Kubernetes Documentation](https://kubernetes.io/docs/).