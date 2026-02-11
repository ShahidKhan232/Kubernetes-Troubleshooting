# Config Out of Date

> This lesson explores configuration changes in Kubernetes and how to troubleshoot issues with environment variable updates in running pods.

In this lesson, we explore a common scenario encountered in Kubernetes where configuration changes (such as updates to ConfigMaps or Secrets) are not immediately reflected in running pods. This guide walks you through inspecting deployments, testing environment configurations, and troubleshooting issues related to environment variable updates.

## Viewing Deployments in the Production Namespace

Begin by listing all deployments in the "production" namespace:

```bash  theme={null}
k get deployments.apps -n production
NAME          READY  UP-TO-DATE  AVAILABLE  AGE
mysql         1/1    1           1          3m3s
two-tier-app  0/1    1           0          3m3s
web-app       1/1    1           1          3m3s
```

## Inspecting the Web Application Deployment

Start with the web application deployment by examining its YAML definition:

```yaml  theme={null}
selector:
  matchLabels:
    app: node-env-app
strategy:
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 25%
  type: RollingUpdate
template:
  metadata:
    creationTimestamp: null
    labels:
      app: node-env-app
  spec:
    containers:
      - envFrom:
          - configMapRef:
              name: web-message
        image: rakshithraka/node-env-app
        imagePullPolicy: Always
        name: web-app
        ports:
          - containerPort: 3000
            protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    schedulerName: default-scheduler
    securityContext: {}
    terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
    lastTransitionTime: "2024-06-08T21:25:36Z"
```

Focus on the environment configuration where the web application receives its environment variables from a ConfigMap named **web-message**. To inspect this ConfigMap, retrieve its details by running:

```bash  theme={null}
k get cm -n production web-message -o yaml
```

The YAML output reveals a single key-value pair:

```yaml  theme={null}
apiVersion: v1
data:
  MESSAGE: Hello, World
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"MESSAGE":"Hello, World"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"web-message","namespace":"production"}}
  creationTimestamp: "2024-06-08T21:24:54Z"
  name: web-message
  namespace: production
  resourceVersion: "7974"
  uid: 5aee4439-24b7-462c-b35c-ad7725c109c2
```

## Testing the Web Application

To test the web application, set up port forwarding for the web service in a separate terminal:

```bash  theme={null}
k port-forward -n production svc/web-svc 3000:3000
```

With port forwarding active, verify the app's response by sending a request:

```bash  theme={null}
curl localhost:3000
```

The HTML response displays "Hello, World" – the message sourced from the **web-message** ConfigMap.

## Updating the ConfigMap

Suppose you want to update the message from "Hello, World" to "Hello, KodeKloud." Edit the ConfigMap with:

```bash  theme={null}
k edit cm -n production web-message
```

After saving the changes, you might expect that curling the web service returns the updated message. However, if you run:

```bash  theme={null}
curl localhost:3000
```

the response still shows "Hello, World." To further verify, execute the following command to inspect the pod’s environment variables:

```bash  theme={null}
k exec -n production -it web-app-5c4d9cb496-sjxr5 -- /bin/sh
# printenv | grep MESSAGE
MESSAGE=Hello, World
```

<Callout icon="lightbulb" color="#1CB2FE">
  Pods do not automatically update if a mounted ConfigMap changes. To see the updated configuration, you must restart the pod.
</Callout>

To update the running configuration, restart the deployment:

```bash  theme={null}
k rollout restart deployment -n production web-app
```

Then test the application again:

```bash  theme={null}
curl localhost:3000
```

You should now see "Hello, KodeKloud" rendered as the new message.

## Troubleshooting the Two-Tier Application

Next, let’s investigate the issue with the two-tier application, which is not becoming ready due to a failing readiness probe. The probe uses the following command to verify connectivity to the MySQL server:

```bash  theme={null}
exec [sh -c mysql -u root -p${MYSQL_PASSWORD} -h ${MYSQL_HOST} -e 'SELECT 1'] delay=0s timeout=1s period=10s #success=1 #failure=3
```

This command prevents the pod from receiving traffic if a connection to the MySQL database fails.

### Diagnosing the Readiness Probe Failure

To diagnose the problem, execute the following command to inspect the environment variables within the two-tier app pod:

```bash  theme={null}
k exec -n production -it two-tier-app-7b7798b66b-wzb4n -- /bin/sh
# printenv | grep MYSQL
MYSQL_PORT_3306_TCP_ADDR=10.105.100.67
MYSQL_PASSWORD=admin
MYSQL_PORT_3306_TCP_PORT=3306
MYSQL_HOST=mysql
MYSQL_SERVICE_HOST=10.105.100.67
MYSQL_PORT_3306_TCP_PROTO=tcp
MYSQL_USER=root
MYSQL_PORT_3306_TCP=10.105.100.67:3306
MYSQL_SERVICE_PORT=3306
MYSQL_DB=mydb
```

Here, notice that the **MYSQL\_HOST** variable is set to "mysql". This value is provided by a secret named **app-secrets**. To inspect the secret, run:

```bash  theme={null}
k get secret -n production app-secrets -o yaml
```

Within the secret, the problematic field is encoded in base64. Verify the correct hostname by running:

```bash  theme={null}
echo "bXlzcHcw" | base64 -d
```

This command outputs:

```MySQL  theme={null}
mysql
```

Even after correcting the secret, the pod might still use the old value because it was not restarted. To refresh the environment variables, restart the two-tier application deployment:

```bash  theme={null}
k rollout restart deployment -n production two-tier-app
```

Once restarted, monitor the pod’s status and events to ensure it becomes ready and successfully connects to the MySQL server.

## Final Thoughts

These examples illustrate that updating a ConfigMap or Secret does not automatically refresh the environment variables in running pods. Kubernetes requires a pod or deployment restart for changes to take effect. For large-scale environments with many pods, manually tracking which resources need a restart can be challenging.

<Callout icon="lightbulb" color="#1CB2FE">
  Whenever you modify an external ConfigMap or Secret that is mounted onto pods, ensure that you restart the affected pods or deployments. This step is essential for the changes to be applied to the running configuration.
</Callout>

In an upcoming lesson, we will discuss strategies to manage these challenges more efficiently in your Kubernetes clusters.

For more detailed Kubernetes concepts and best practices, check these resources:

* [Kubernetes Basics](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)
* [Kubernetes Documentation](https://kubernetes.io/docs/)
* [Docker Hub](https://hub.docker.com/)
* [Terraform Registry](https://registry.terraform.io/)