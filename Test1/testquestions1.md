## Question 1 | Contexts


Solve this question on: ```ssh cks3477```

 
On ```cks3477``` you have access to multiple clusters through kubectl contexts. Write all context names into ```/course/1/contexts``` on ```cks3477```, one per line.

From the kubeconfig extract the client certificate of user ```restricted@infra-prod``` and write it decoded to ```/course/1/cert```.

## Question 2 | Image Vulnerability Scanning

Solve this question on: ```ssh cks8930```


The Vulnerability Scanner trivy is installed on your main terminal. Use it to scan the following images for known CVEs:

- nginx:1.30.2-alpine

- registry.k8s.io/kube-apiserver:v1.35.1

- registry.k8s.io/kube-controller-manager:v1.35.2

- istio/pilot:1.28.0

Write all image names (including the tag, exactly as listed above) that are not affected by ```CVE-2025-68121``` or ```CVE-2026-45447``` into ```/course/2/good-images``` on cks8930.

## Question 3 | Apiserver Security

Controlling Access to the Kubernetes API

Solve this question on: ```ssh cks8930```

You received a list from the DevSecOps team which performed a security investigation of the cluster. The list states the following about the apiserver setup:

- Accessible through a NodePort Service

Change the apiserver setup so that:

- Accessible through a ClusterIP Service

ℹ️ Use sudo -i to become root which may be required for this question

## Question 4 | ServiceAccount Token Expiration

Configure Service Accounts for Pods Managing Service Accounts

Solve this question on: ```ssh cks5608```
 

Update file ```/course/4/stream-multiplex.yaml``` with the following changes:

Pods should have annotation ```token-lifetime``` with value ```1200``` (annotation is informational only)

ServiceAccount ```stream-multiplex``` should be used

Disable automounting of ServiceAccount tokens

The ServiceAccount token should be mounted at ```/var/run/secrets/custom/``` with an expiration of ```1200s```

Apply the Deployment and ensure it's running without errors.

## Question 5 | CIS Benchmark

Securing a Cluster

Solve this question on: ```ssh cks7262```

You're asked to evaluate specific settings of the cluster against the CIS Benchmark recommendations. Use the kube-bench tool which is already installed on the nodes.

Connect to the worker node using ```ssh cks7262-node1``` from ```cks7262```.

On the controlplane node ensure (correct if necessary) that the CIS recommendations are set for:

1. The ```--profiling``` argument of the kube-controller-manager

2. The ownership of directory ```/var/lib/etcd```

On the worker node ensure (correct if necessary) that the CIS recommendations are set for:

3. The permissions of the kubelet configuration ```/var/lib/kubelet/config.yaml```

4. The ```--client-ca-file``` argument of the kubelet

ℹ️ Use sudo -i to become root which may be required for this question

## Question 6 | Immutable Root FileSystem

Configure a Security Context

Solve this question on: ```ssh cks2546```

The Deployment immutable-deployment in Namespace ```team-purple``` should run immutable. It's created from file ```/course/6/immutable-deployment.yaml``` on cks2546. Even after a successful break-in, it shouldn't be possible for an attacker to modify the filesystem of the running container.

1. Modify the Deployment in a way that no processes inside the container can modify the local filesystem, only the /tmp directory should be writable. Don't modify the Docker image.

2. Save the updated YAML under ```/course/6/immutable-deployment-new.yaml``` on cks2546 and update the running Deployment.

 ## Question 7 | Pod Security Standard and Admission

Pod Security Standards Pod Security Admission

Solve this question on: ```ssh cks5608```

Implement specific security policies in Namespace ```team-sepia```.

1. Configure Pod Security Admission in mode ```audit``` for level ```baseline```

2. Configure Pod Security Admission in mode ```warn``` for level ```restricted```

3. Afterwards create the Pod from ```/course/7/bad-pod.yaml``` and write any warnings or errors into ```/course/7/bad-pod.log```

## Question 8 | Docker Configuration and Usage
 

Solve this question on: ```ssh cks4024```

 

Docker containers on ```cks4024``` should run more isolated from each other by disabling inter-container communication.

1. Add ```"icc": false``` to the Docker config and ensure the Docker daemon is using the updated settings

2. Create two Docker containers named ```container1``` and ```container2``` which should

- have image ```nginx:1-alpine```

- restart ```always```

- keep running in the background

As a result, the containers should not be able to ping each other on their IP addresses.


ℹ️ Run all Docker commands as root. Use sudo -i to become root

 ## Question 9 | AppArmor Profile

AppArmor

Solve this question on: ```ssh cks7262```

Some containers need to run more securely. There is an existing AppArmor profile located at ```/course/9/profile``` on ```cks7262``` for this.

1. Install the AppArmor profile on node ```cks7262-node1```.

Connect using ```ssh cks7262-node1``` from ```cks7262```

2. Add label ```security=apparmor``` to the node

3. Create a Deployment named ```apparmor``` in Namespace ```default``` with:

- One replica of image ```nginx:1-alpine```

- NodeSelector for ```security=apparmor```

- Single container named ```c1``` with the AppArmor profile enabled only for this container

The Pod might not run properly with the profile enabled. Write the logs of the Pod into ```/course/9/logs``` on ```cks7262``` so another team can work on getting the application running.
 

ℹ️ Use sudo -i to become root which may be required for this question

## Question 10 | Container Runtime Sandbox gVisor

Runtime Class

Solve this question on: ```ssh cks7262```

Team purple wants to run some of their workloads more securely. Worker node ```cks7262-node1``` is already configured so that containerd supports the runsc/gvisor runtime.

Connect to the worker node using ```ssh cks7262-node1``` from ```cks7262```.

1. Create a RuntimeClass named ```gvisor``` with handler ```runsc```

2. Create a Pod that uses the RuntimeClass. The Pod should be in Namespace ```team-purple```, named ```gvisor-test``` and of image ```nginx:1-alpine```

3. Ensure the Pod only ever runs on a node named ```cks7262-node1```

4. Write the output of the ```dmesg``` command of the successfully started Pod into ```/course/10/gvisor-test-dmesg``` on ```cks7262```

## Question 11 | Secret Management

Secrets Managing Secrets using kubectl

Solve this question on: ```ssh cks2546```

There is Secret ```db-con``` in Namespace ```team-khaki-us-east-ad1```. Update the password to ```4c!29f_Ee2e``` and ensure all Pods currently using the Secret will work with the updated value.

Move Secret ```user-data``` from Namespace ```team-khaki-us-east-ad1``` to ```team-khaki-us-east-ad2```.

Convert ConfigMap ```app-data``` in Namespace ```team-khaki-us-east-ad1``` to a Secret and delete the ConfigMap afterwards. Ensure all Pods that used the ConfigMap will continue to work and are now using the values from the Secret.


 
## Question 12 | ImagePolicyWebhook

Admission Control in Kubernetes

Solve this question on: ```ssh cks4024```

Team White created an ImagePolicyWebhook solution at ```/course/12/webhook``` on ```cks4024``` which needs to be enabled for the cluster. There is an existing and working ```webhook-backend``` Service in Namespace ```team-white``` which will be the ImagePolicyWebhook backend.


1. Create an AdmissionConfiguration at ```/course/12/webhook/admission-config.yaml``` which contains the following ImagePolicyWebhook configuration in the same file:

```
imagePolicy:
  kubeConfigFile: /etc/kubernetes/webhook/webhook.yaml
  allowTTL: 10
  denyTTL: 10
  retryBackoff: 20
  defaultAllow: true
```
2. Configure the apiserver to:

- Mount ```/course/12/webhook``` at ```/etc/kubernetes/webhook```

- Use the AdmissionConfiguration at path ```/etc/kubernetes/webhook/admission-config.yaml```

- Enable the ImagePolicyWebhook admission plugin

As a result, the ImagePolicyWebhook backend should prevent container images containing ```danger-danger``` from being used. Any other image should still work.

ℹ️ Create a backup of ```/etc/kubernetes/manifests/kube-apiserver.yaml``` outside of ```/etc/kubernetes/manifests``` so you can revert in case of issues
 
ℹ️ Use sudo -i to become root which may be required for this question


## Question 13 | CiliumNetworkPolicy Metadata Server

Cilium Documentation

Solve this question on: ```ssh cks8930```

There is a metadata service available at ```http://192.168.100.21:9055```through which nodes can access sensitive data. Access to this needs to be restricted from Pods.

In Namespace ```metadata-access``` create a CiliumNetworkPolicy named default to:

1. Allow egress to ```0.0.0.0/0```

2. Allow egress to Endpoints in the same Namespace

3. Allow egress to Endpoints in the ```kube-system``` Namespace (this covers DNS resolution)

4. Deny egress to ```192.168.100.21``` on port ```9055```

ℹ️ There are existing plain Nginx Pods with open port 80 in the Namespace which can be used for testing but need to remain unchanged. Perform simple connectivity tests like:

```
k -n metadata-access exec POD_NAME -- curl URL
``` 

 ## Question 14 | ETCD Secret Encryption

Encrypting Confidential Data at Rest

Solve this question on: ```ssh cks7262```

An internal security audit requires secrets in the cluster to be encrypted. The team already created the needed EncryptionConfiguration at ```/etc/kubernetes/etcd/ec.yaml```.

1. The Apiserver should mount ```/etc/kubernetes/etcd``` on the host to ```/etc/kubernetes/etcd``` inside the container

2. The Apiserver should use the EncryptionConfiguration from ```/etc/kubernetes/etcd/ec.yaml``` inside the container

3. All Secrets in Namespace ```team-magenta``` should be stored encrypted in ETCD

## Question 15 | Configure TLS on Ingress

Ingress

Solve this question on: ```ssh cks2546```

In Namespace ```team-pink``` there is an existing Nginx Ingress resource named secure which accepts two paths, ```/app``` and ```/api```, pointing to different ClusterIP Services.

From your main terminal you can connect to it, for example:

- HTTP: ```curl -v http://secure-ingress.test:31080/app```

- HTTPS: ```curl -kv https://secure-ingress.test:31443/app```

Right now it uses a default TLS certificate generated by the Nginx Ingress Controller.

You're asked to instead use the key and certificate provided at ```/course/15/tls.key``` and ```/course/15/tls.crt```. As it's a self-signed certificate you need to use ```curl -k``` when connecting to it.

## Question 16 | Runtime Security with Falco

Falco Documentation

Solve this question on: ```ssh cks5608```

Add two new Falco rules to ```/etc/falco/falco_rules.local.yaml```:

1. Named ```Custom Rule 1``` with priority ```WARNING```. It should find all containers that access files on the host whose full path starts with  ```/etc/kubernetes ```. It should output logs as:

     ```custom_rule_1 file={{FILEPATH}} container={{CONTAINER_ID}} ```

2. Named  ```Custom Rule 2 ``` with priority  ```INFO ```. It should find all processes that perform  ```kill ``` syscalls. It should output logs as:

     ```custom_rule_2 event_signal=%evt.arg.sig event_pid=%evt.arg.pid container={{CONTAINER_ID}} ```

Only create the new rules **without additional macros or lists**.

Run Falco with your implemented rules for at least 30 seconds and write the produced logs into  ```/course/16/logs ```.

 ## Question 17 | Audit Log Policy

Auditing

Solve this question on:  ```ssh cks3477 ```

Audit Logging has been enabled in the cluster with an Audit Policy located at  ```/etc/kubernetes/audit/policy.yaml ``` on  ```cks3477 ```.

1. Change the apiserver setting so that only one backup of the logs is stored.

2. Alter the Policy so that it only stores logs:

- From Secret resources, level Metadata

- From "system:nodes" userGroups, level RequestResponse

After you update the Policy, make sure to empty the log file so it only contains entries according to your changes, for example using  ```echo > /etc/kubernetes/audit/logs/audit.log ```.

ℹ️ You can use yq to render JSON in a more readable form, for example  ```cat data.json | yq -p json -o json ```

ℹ️ Use  ```sudo -i ``` to become root which may be required for this question

 
 