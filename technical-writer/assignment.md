# Debug a Kubernetes Cluster

Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. 

In a Kubernetes cluster: 

* Nodes are the worker machines (physical servers or virtual machines) that provide the computing power to launch your containerized applications.   
* Control plane nodes act as the “brain” of the cluster.  
* Worker nodes host pods.   
* Pods host containers. A container is a software package containing everything needed to execute an application.  
* Namespaces are used to organize pods. For example, you can create namespaces for different users or environments. 

You can deploy a Kubernetes cluster on cloud providers such as AWS, Azure, or GCE, or on bare metal (a physical server). You can also launch a cluster locally using Minikube, Docker Desktop, or Kubernetes in Docker. 

## Debug a Cluster with `kubectl`

You can use the `kubectl` command-line interface (CLI) to interact with, manage, and debug a Kubernetes cluster. The `kubectl` CLI acts as a bridge to the cluster's control plane by translating your terminal commands into requests that the Kubernetes cluster can understand. You can install `kubectl` on macOS, Windows, or Linux. 

### List Pods

The `get pods` command lists all Kubernetes pods within a specified namespace. It provides a high-level overview of each pod’s lifecycle status, and helps you identify which pods are failing. 

This command uses the following format: 

`kubectl get pods -n <namespace_name>`

For example, to list all pods in the `secondary` namespace: 

```shell
kubectl get pods -n secondary
```  

```shell                   
NAME      READY   STATUS    RESTARTS   AGE  
nginx-1   1/1     Running   0          20s  
nginx-2   1/1     Running   0          12s
```

If you don’t specify a namespace, `get pods` returns the pods in the `default` namespace. To list all pods in all namespaces, execute: 

```shell
kubectl get pods --all-namespaces
``` 

### Retrieve Logs

The `kubectl logs` command retrieves logs from a specified pod or container. Logs provide application-level debug information. 

Use the following command formats to retrieve logs: 


* From a Pod: `kubectl logs <pod_name>`
* From a Specific Container: `kubectl logs <pod_name> -c <container_name>`
* For All Containers in a Pod: `kubectl logs <pod_name> --all-containers`
* In a Specific Namespace: `kubectl logs <pod_name> -n <namespace_name>`


For example, to retrieve the logs for the `nginx-0` pod: 

```shell
kubectl logs nginx-0
```

```shell
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/05/03 17:57:48 [notice] 1#1: using the "epoll" event method
2026/05/03 17:57:48 [notice] 1#1: nginx/1.29.8
2026/05/03 17:57:48 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19) 
2026/05/03 17:57:48 [notice] 1#1: OS: Linux 6.12.76-linuxkit
2026/05/03 17:57:48 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/05/03 17:57:48 [notice] 1#1: start worker processes
2026/05/03 17:57:48 [notice] 1#1: start worker process 33
2026/05/03 17:57:48 [notice] 1#1: start worker process 34
2026/05/03 17:57:48 [notice] 1#1: start worker process 35

```

You can also write log output to a local text file. For example:

```shell
kubectl logs nginx-0 > nginx-0-logs.txt
```

### Execute Commands in a Container

The `kubectl exec` command lets you execute commands directly inside an active container in a Kubernetes pod. You can use it to gather diagnostic data and debug your application's environment in real-time.

This command uses the following format: 

`kubectl exec <pod_name> -- <command>`

For example, to open an interactive bash shell in the `nginx-0` pod:

```shell
kubectl exec -it nginx-0 -- /bin/bash
```

```shell
root@nginx-0:/# ls
bin   dev		   docker-entrypoint.sh  home  media  opt   product_uuid  run	srv  tmp  var
boot  docker-entrypoint.d  etc			 lib   mnt    proc  root	  sbin	sys  usr
root@nginx-0:/# 

```

### Debug a Container

You may encounter issues that can’t be evaluated using standard diagnostic methods like `kubectl exec` – when a pod has crashed, for example. The `kubectl debug` command lets you create a temporary container in a pod alongside your application. You can create a clone of a crashed pod for diagnostic purposes, or use a temporary container to launch tools that aren’t available in the original container image.

Use the following command format to add an interactive temporary container to a running pod: 

`kubectl debug <pod_name> -n <namespace_name> -it --image=<debug_image>`

To create a copy of a pod with a new name for troubleshooting: 

`kubectl debug <pod_name> -n <namespace_name> -it --image=<debug_image> --copy-to=<new_pod_name>`

The following example creates an interactive debugging session in the `nginx-1` pod: 

```shell
kubectl debug nginx-1 -n secondary -it --image=busybox
```

```shell
Defaulting debug container name to debugger-d7cd2.
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
/ #
```
And here we create a copy of `nginx-2` named `nginx-2-debugger`:

```shell
kubectl debug nginx-2 -n secondary -it --image=busybox --copy-to=nginx-2-debugger
```

```shell
Defaulting debug container name to debugger-qm9tg.
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
/ # 
```

## Debug Workflow

We recommend the following debug workflow: 

1. Start with `kubectl get pods` to list each pod’s status and identify failing pods.  
2. Use `kubectl logs` to retrieve application-level information.  
3. Use `kubectl exec` to execute debug commands inside active containers.  
4. Use `kubectl debug` to create a clone of a crashed pod, or to create a temporary container and launch diagnostic tools. 

## Resources

* [Troubleshooting Kubernetes Clusters](https://kubernetes.io/docs/tasks/debug/debug-cluster/)  
* [Command Line Tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)  
* [Kubernetes Overview](https://kubernetes.io/docs/concepts/overview/)

