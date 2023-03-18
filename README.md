# Automation-of-Kubernetes-Cluster-Hardening
This is a small project to make a Kubernetes Cluster more secure by applying rules to the files of cluster. 

This project is using Centos 7 as base platform to run and configure the K8s cluster.

PRE-REQUISTIES:
Operating System: Centos 7 or any Red-Hat Distribution platform.
Processor: i5 and above with 8 CPUs
RAM: 16 GB
Disk Size : 100 GB or above

Now we have to create a master and worker node.
You can create more than one worker, its up to you. For now we are going with one master and one worker.

For Master Node :
Centos 7
4 CPUs
8 GB RAM
60 GB Hard Disk Space

For Worker Node :
Centos 7
2 CPUs
2 GB RAM
20 GB Hard Disk Space

Now that we have configure our machine for master and worker, we are going to install kubernetes in both master and worker.
first we install in master node then after that we will follow the same procedure in worker node.
