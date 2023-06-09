## What does it mean to self heal?
When a pod is scheduled to a node, the kubelet on that node runs its containers and keeps them running as long as the pod exists. The kubelet will restart a container if its main process crashes. But if the application inside of the container throws an error which causes it to continuously restart, Kubernetes has the ability to heal it by using the correct diagnostic and then following the pod’s restart policy.

Within containers, the kubelet can react to two kinds of probes:

## Liveness  
the kubelet uses these probes as an indicator to restart a container. A liveness probe is used to detect when an application is running and is unable to make progress. When a container gets in this state, the pod’s kubelet can restart the container via its restart policy.
Readiness - This type of probe is used to detect if a container is ready to accept traffic. You can use this probe to manage which pods are used as backends for load balancing services. If a pod is not ready, it can then be removed from the list of load balancers.

## Types of health checks for liveness and readiness probes
By configuring liveness and readiness probes to return diagnostics for your containerized applications, Kubernetes can react appropriately, increasing your application’s overall uptime. For a probe to perform a diagnostic on the container’s health, the kubelet calls the handler implemented in the container for the probe:

**ExecAction** - executes a command inside the container.
TCPSocketAction - performs a TCP check against the container’s IP address on a specified port.
HTTPGetAction - performs an HTTP GET request on the container’s IP.
A handler can then return the following:

**Success** - the diagnostic passed on the container.
Fail - the container failed the diagnostic and will restart according to its restart policy.
Unknown - the diagnostic failed and no action will be taken.
When to use liveness and readiness probes?
Liveness and readiness probes are both configured in the pod’s YAML file. Each type has different use cases.

**Liveness use case**
As mentioned, liveness probes are used to diagnose unhealthy containers. They can detect a problem in your service when it cannot progress, and will restart the problematic container according to its restart policy that hopefully sorts out your services’ problem.


For example, you may want to include an Exec liveness probe inside of your container to detect when an application has transitioned to a broken state and cannot recover unless it’s restarted:

**LivenessPobe example**

![test][code-liveness]

This YAML snippet shows a Pod with a single container. It indicates that the kubelet perform a liveness probe every 5 seconds and that it should wait 5 seconds again before performing the first probe. The kubelet executes the command cat /tmp/healthy in the container and if it succeeds, it returns 0 indicating it is healthy. If it returns a non-zero value, the kubelet kills the container and restarts it.
# Readiness use case
A readiness probe on the other hand is useful in the case of an application being temporarily unable to serve traffic. For example, maybe the application needs to load a large dataset or some configuration files during the startup phase. Even though the service is running, it’s not fully available, and in this scenario, Kubernetes may have trouble scaling it up and could fail. With a readiness probe, Kubernetes waits until the service is fully started before it sends traffic to the new copy, as an example.
      
      readynessProbe:
        exec:
          command:
          - cat
          - /temp/healthy
        initialDealySeconds: 5
        periodSeconds: 5

Readiness probes are configured similarly to liveness probes. The only difference is that you use the readinessProbe field instead of the livenessProbe field.

# When not to use liveness and readiness probes
If you have a process inside of your container that is able to crash on its own when it becomes unhealthy or encounters an error, it is not necessary to use a liveness probe. In this case, the kubelet follows whatever action is specified in the Pod’s restartPolicy.

In the case of readiness probes, use them if you’d like to send traffic to a pod only when a probe succeeds. In this case, it may be that the liveness probe and the readiness probe are the same, however having the readiness probe available means the pod will startup first before traffic starts flowing to it. This might be the case if you have a container that needs to load configuration files or other data before it starts up successfully.

If you want to drain the requests after the Pod is deleted, you do not need a readiness probe on deletion. The pod will automatically puts itself into an unready state regardless of whether a readiness probe exists. The pod will remain in the unready state while it waits for all of the containers in the Pod to stop


[code-liveness]: https://images.contentstack.io/v3/assets/blt300387d93dabf50e/blt560764a88564eaf5/5cb73468b790c0112239256a/YAML-snippet.png