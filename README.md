# Ansible k8s Spark
[![Work with Ansible](https://img.shields.io/badge/Work%20with-Ansible-brightgreen.svg)](https://img.shields.io/badge/Work%20with-Ansible-brightgreen.svg) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) 

An Ansible playbook deploys a scalable Spark cluster in Kubernetes. It is a try of this [Running Spark on Kubernetes](https://spark.apache.org/docs/latest/running-on-kubernetes.html) feature. 

 - The recommended way for running Spark applications on Kubernetes cluster is using [spark-on-k8s-operator](https://github.com/GoogleCloudPlatform/spark-on-k8s-operator). 
 - To setup Spark standalone cluster on Kubernetes cluster, please refer to my another project, [Ansible k8s Helm Spark](https://github.com/yungshun317/ansible-k8s-helm-spark). 
 - With Docker Swarm mode, refer to [Ansible Docker Swarm Spark](https://github.com/yungshun317/ansible-docker-swarm-spark).

## Requirements

|Package|Version|  
|:-----:|:-----:|  
|python2|2.7.12|  
|ansible|2.7.2|

## Usage
Run `ansible-playbook` softly. 
```sh
$ ansible-playbook k8s-spark.yml
```

Based on description from [deployment instruction](https://spark.apache.org/docs/latest/running-on-kubernetes.html), submission process container several steps:

1. Spark creates a Spark driver running within a Kubernetes pod.
2. The driver creates executors which are also running within Kubernetes pods and connects to them, and executes application code.
3. When the application completes, the executor pods terminate and are cleaned up, but the driver pod persists logs and remains in “completed” state in the Kubernetes API until it’s eventually garbage collected or manually cleaned up.

So, in fact, you have no place to submit a job until you starting a submission process, which will launch a first Spark's pod (driver) for you. And after application completes, everything terminated.

```sh
$ kubectl exec -it spark-master-768d6764b6-499dn bash
bash-4.4# /opt/spark/bin/spark-shell --master k8s://https://10.236.1.1:6443
Error: Client mode is currently not supported for Kubernetes.
Run with --help for usage help or --verbose for debug output

$ kubectl scale deployment spark-worker --replicas 3
deployment.extensions/spark-worker scaled

$ kubectl get pods
NAME                                      READY   STATUS    RESTARTS   AGE
nginx-deployment-5ff4d574f9-drxhs         1/1     Running   0          16h
spark-master-768d6764b6-499dn             1/1     Running   0          16h
spark-worker-6998f6d979-5qlmf             1/1     Running   0          112s
spark-worker-6998f6d979-jppjq             1/1     Running   0          112s
spark-worker-6998f6d979-p7c2h             1/1     Running   0          16h

$ kubectl create serviceaccount spark
serviceaccount/spark created

$ kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default
clusterrolebinding.rbac.authorization.k8s.io/spark-role created

$ ./bin/spark-submit --master k8s://https://10.236.1.1:6443 --deploy-mode cluster --name spark-pi --class org.apache.spark.examples.SparkPi --conf spark.executor.instances=2 --conf spark.kubernetes.container.image=yungshun317/spark --conf spark.kubernetes.driver.pod.name=spark-pi-driver --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark local:///opt/spark/examples/jars/spark-examples_2.11-2.3.1.jar
2018-12-06 11:19:50 WARN  Utils:66 - Your hostname, mesos-master resolves to a loopback address: 127.0.1.1; using 10.236.1.1 instead (on interface enp3s0)
2018-12-06 11:19:50 WARN  Utils:66 - Set SPARK_LOCAL_IP if you need to bind to another address
2018-12-06 11:19:51 INFO  LoggingPodStatusWatcherImpl:54 - State changed, new state: 
	 pod name: spark-pi-driver
	 namespace: default
	 labels: spark-app-selector -> spark-4e636dfeb942454fa6676575c670d99c, spark-role -> driver
	 pod uid: cb0ff32a-f905-11e8-a294-00e081ce995a
	 creation time: 2018-12-06T03:19:51Z
	 service account name: spark
	 volumes: spark-token-8pgc2
	 node name: N/A
	 start time: N/A
	 container images: N/A
	 phase: Pending
	 status: []
2018-12-06 11:19:51 INFO  LoggingPodStatusWatcherImpl:54 - State changed, new state: 
	 pod name: spark-pi-driver
	 namespace: default
	 labels: spark-app-selector -> spark-4e636dfeb942454fa6676575c670d99c, spark-role -> driver
	 pod uid: cb0ff32a-f905-11e8-a294-00e081ce995a
	 creation time: 2018-12-06T03:19:51Z
	 service account name: spark
	 volumes: spark-token-8pgc2
	 node name: mesos-driver
	 start time: N/A
	 container images: N/A
	 phase: Pending
	 status: []
2018-12-06 11:19:51 INFO  LoggingPodStatusWatcherImpl:54 - State changed, new state: 
	 pod name: spark-pi-driver
	 namespace: default
	 labels: spark-app-selector -> spark-4e636dfeb942454fa6676575c670d99c, spark-role -> driver
	 pod uid: cb0ff32a-f905-11e8-a294-00e081ce995a
	 creation time: 2018-12-06T03:19:51Z
	 service account name: spark
	 volumes: spark-token-8pgc2
	 node name: mesos-driver
	 start time: 2018-12-06T03:19:51Z
	 container images: yungshun317/spark
	 phase: Pending
	 status: [ContainerStatus(containerID=null, image=yungshun317/spark, imageID=, lastState=ContainerState(running=null, terminated=null, waiting=null, additionalProperties={}), name=spark-kubernetes-driver, ready=false, restartCount=0, state=ContainerState(running=null, terminated=null, waiting=ContainerStateWaiting(message=null, reason=ContainerCreating, additionalProperties={}), additionalProperties={}), additionalProperties={})]
2018-12-06 11:19:51 INFO  Client:54 - Waiting for application spark-pi to finish...
2018-12-06 11:20:02 INFO  LoggingPodStatusWatcherImpl:54 - State changed, new state: 
	 pod name: spark-pi-driver
	 namespace: default
	 labels: spark-app-selector -> spark-4e636dfeb942454fa6676575c670d99c, spark-role -> driver
	 pod uid: cb0ff32a-f905-11e8-a294-00e081ce995a
	 creation time: 2018-12-06T03:19:51Z
	 service account name: spark
	 volumes: spark-token-8pgc2
	 node name: mesos-driver
	 start time: 2018-12-06T03:19:51Z
	 container images: yungshun317/spark:latest
	 phase: Running
	 status: [ContainerStatus(containerID=docker://185cd7cf127000311d03fbd885bbd05e4c40285558164e4c8d6507478da0722a, image=yungshun317/spark:latest, imageID=docker-pullable://yungshun317/spark@sha256:5e7c40f503b0383b0324dbab759f05966294356decaeecf55909e3e50256a6e9, lastState=ContainerState(running=null, terminated=null, waiting=null, additionalProperties={}), name=spark-kubernetes-driver, ready=true, restartCount=0, state=ContainerState(running=ContainerStateRunning(startedAt=Time(time=2018-12-06T03:20:02Z, additionalProperties={}), additionalProperties={}), terminated=null, waiting=null, additionalProperties={}), additionalProperties={})]
2018-12-06 11:20:21 INFO  LoggingPodStatusWatcherImpl:54 - State changed, new state: 
	 pod name: spark-pi-driver
	 namespace: default
	 labels: spark-app-selector -> spark-4e636dfeb942454fa6676575c670d99c, spark-role -> driver
	 pod uid: cb0ff32a-f905-11e8-a294-00e081ce995a
	 creation time: 2018-12-06T03:19:51Z
	 service account name: spark
	 volumes: spark-token-8pgc2
	 node name: mesos-driver
	 start time: 2018-12-06T03:19:51Z
	 container images: yungshun317/spark:latest
	 phase: Succeeded
	 status: [ContainerStatus(containerID=docker://185cd7cf127000311d03fbd885bbd05e4c40285558164e4c8d6507478da0722a, image=yungshun317/spark:latest, imageID=docker-pullable://yungshun317/spark@sha256:5e7c40f503b0383b0324dbab759f05966294356decaeecf55909e3e50256a6e9, lastState=ContainerState(running=null, terminated=null, waiting=null, additionalProperties={}), name=spark-kubernetes-driver, ready=false, restartCount=0, state=ContainerState(running=null, terminated=ContainerStateTerminated(containerID=docker://185cd7cf127000311d03fbd885bbd05e4c40285558164e4c8d6507478da0722a, exitCode=0, finishedAt=Time(time=2018-12-06T03:20:21Z, additionalProperties={}), message=null, reason=Completed, signal=null, startedAt=Time(time=2018-12-06T03:20:02Z, additionalProperties={}), additionalProperties={}), waiting=null, additionalProperties={}), additionalProperties={})]
2018-12-06 11:20:22 INFO  LoggingPodStatusWatcherImpl:54 - Container final statuses:


	 Container name: spark-kubernetes-driver
	 Container image: yungshun317/spark:latest
	 Container state: Terminated
	 Exit code: 0
2018-12-06 11:20:22 INFO  Client:54 - Application spark-pi finished.
2018-12-06 11:20:22 INFO  ShutdownHookManager:54 - Shutdown hook called
2018-12-06 11:20:22 INFO  ShutdownHookManager:54 - Deleting directory /tmp/spark-1031a01b-cbe9-4b3e-988f-500a7acd7d6a

$ kubectl get pod spark-pi-driver
NAME                READY   STATUS      RESTARTS   AGE
spark-pi-driver     0/1     Completed   0          3m10s
```

## Outline
The playbook does:
1. Install Spark.
2. Build and push Spark Docker images using `docker-image-tool.sh`.
3. Create an nginx Deployment.
4. Create Spark master and worker Deployments.
5. Create Spark master and worker Services.

Note that I did not create `serviceaccount` and `clusterrolebinding` in the playbook, so I have to create them manually like above.

## Tech
This project uses:
* [Ansible](https://www.ansible.com/) - Ansible is open source software that automates software provisioning, configuration management, and application deployment.
* [Docker](https://github.com/docker/docker-ce) - a computer program that performs operating-system-level virtualization, also known as containerization.
* [Kubernetes](https://kubernetes.io/) - an open-source system for automating deployment, scaling, and management of containerized applications.
* [Spark](https://spark.apache.org/) - Apache Spark™ is a unified analytics engine for large-scale data processing.

## Todos
 - Write an Ansible playbook for [spark-on-k8s-operator](https://github.com/GoogleCloudPlatform/spark-on-k8s-operator).
 - Try [Helm Chart for Spark History Server](https://github.com/helm/charts/tree/master/stable/spark-history-server).
 - Be familiar with using Spark on Kubernetes cluster.

## License
[Ansible k8s Spark](https://github.com/yungshun317/ansible-k8s-spark) is released under the [MIT License](https://opensource.org/licenses/MIT) by [yungshun317](https://github.com/yungshun317).
