# kubectl logs

> This guide explores the `kubectl logs` command for retrieving logs from Kubernetes pods to aid in troubleshooting applications.

In this guide, we explore one of the fundamental Kubernetes troubleshooting commands: `kubectl logs`. Retrieving logs is often the first step when debugging issues with applications running in Kubernetes pods.

## Basic Log Retrieval

To view the logs for a specific pod, use the following command:

```bash  theme={null}
kubectl logs
```

Typically, you will specify the namespace (if needed) and the pod's name. For instance, consider a pod named `notes-app-deployment-d4fcc5ccd-5fl7z` running in the `uat` namespace. Execute the following command to retrieve its logs:

```bash  theme={null}
controlplane ~ ➜ k logs -n uat notes-app-deployment-d4fcc5ccd-5fl7z
> notes-app@1.0.0 start /app
> node app.js
App is running on port 3000
controlplane ~ ➜
```

The output above shows the log messages produced during the startup of the application.

<Callout icon="lightbulb" color="#1CB2FE">
  For a complete list of `kubectl logs` options and other Kubernetes commands, check out the [Kubernetes Documentation](https://kubernetes.io/docs/).
</Callout>

## Viewing Logs in Multicontainer Pods

In more advanced scenarios, a pod may run multiple containers (e.g., an init container, a sidecar container, and the main application container). When you need to retrieve logs from all containers simultaneously, use the `--all-containers` flag:

```bash  theme={null}
controlplane ~ → k logs -n uat notes-app-deployment-d4fcc5ccd-5fl7z
> notes-app@1.0.0 start /app
> node app.js

App is running on port 3000

controlplane ~ → k logs -n uat notes-app-deployment-d4fcc5ccd-5fl7z --all-containers
> notes-app@1.0.0 start /app
> node app.js

App is running on port 3000

controlplane ~ → k get pods
NAME                READY   STATUS    RESTARTS   AGE
logs-generator      1/1     Running   1 (6m25s ago)   16m
multi-container-pod 2/2     Running   0          7m54s

controlplane ~ →
```

While this approach consolidates logs from every container in the pod, it may become challenging to determine the source container for each log entry.

<Callout icon="triangle-alert" color="#FF6B6B">
  Avoid using `--all-containers` when you need to diagnose issues specific to a single container. Explicitly targeting a container clarifies the troubleshooting process.
</Callout>

## Retrieving Container-Specific Logs

To view logs for a particular container in a multicontainer pod, use the `-c` flag along with the container’s name. First, determine the container names from the pod specification using a JSONPath query:

```bash  theme={null}
k get pod multi-container-pod -o jsonpath='{.spec.containers}'
```

For a more readable output, pipe the result to `jq`:

```bash  theme={null}
k get pod multi-container-pod -o jsonpath='{.spec.containers}' | jq
```

This command will list containers such as `nginx-container` and `cron-logger`.

### Example: Viewing Logs for nginx-container

To see the logs from the `nginx-container`, run:

```bash  theme={null}
controlplane ~ ➜ k logs multi-container-pod -c nginx-container
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2024/04/06 22:42:21 [notice] 1#1: using the "epoll" event method
2024/04/06 22:42:21 [notice] 1#1: nginx/1.25.4
2024/04/06 22:42:21 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2024/04/06 22:42:21 [notice] 1#1: OS: Linux 5.4.0-1106-gcp
2024/04/06 22:42:21 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2024/04/06 22:42:21 [notice] 1#1: start worker processes
2024/04/06 22:42:21 [notice] 1#1: start worker process 79
2024/04/06 22:42:21 [notice] 1#1: start worker process 80
2024/04/06 22:42:21 [notice] 1#1: start worker process 81
2024/04/06 22:42:21 [notice] 1#1: start worker process 83
2024/04/06 22:42:21 [notice] 1#1: start worker process 84
2024/04/06 22:42:21 [notice] 1#1: start worker process 86
2024/04/06 22:42:21 [notice] 1#1: start worker process 87
2024/04/06 22:42:21 [notice] 1#1: start worker process 89
2024/04/06 22:42:21 [notice] 1#1: start worker process 91
2024/04/06 22:42:21 [notice] 1#1: start worker process 92
2024/04/06 22:42:21 [notice] 1#1: start worker process 93
2024/04/06 22:42:21 [notice] 1#1: start worker process 94
controlplane ~ ➜
```

### Example: Viewing Logs for cron-logger

If you need logs for the `cron-logger` container, execute:

```bash  theme={null}
controlplane ~ ✗ k logs multi-container-pod -c cron-logger
Cron logger started. Logging messages every 10 seconds.
Sat Apr  6 22:42:21 UTC 2024 - Regular log message.
Sat Apr  6 22:42:22 UTC 2024 - Regular log message.
Sat Apr  6 22:42:23 UTC 2024 - Regular log message.
Sat Apr  6 22:42:24 UTC 2024 - Regular log message.
Sat Apr  6 22:42:25 UTC 2024 - Regular log message.
Sat Apr  6 22:42:26 UTC 2024 - Regular log message.
Sat Apr  6 22:42:27 UTC 2024 - Regular log message.
Sat Apr  6 22:42:28 UTC 2024 - Regular log message.
```

By using the `-c` flag, you can accurately isolate logs for individual containers within a multicontainer pod. This separation is vital for efficient troubleshooting and gaining targeted insights into each container's behavior.

## Conclusion

Using `kubectl logs` is an integral part of managing and troubleshooting Kubernetes applications. Whether you are working with single-container pods or more complex multicontainer setups, understanding how to access and interpret logs will significantly enhance your ability to diagnose and resolve application issues.

For further details and advanced use cases, visit the official [Kubernetes Documentation](https://kubernetes.io/docs/).

Happy troubleshooting!