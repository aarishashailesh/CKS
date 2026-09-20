## Question 1 | SBOM
bom CLI Reference

 

Solve this question on: ```ssh cks9640```

 

Your team received Software Bill Of Materials (SBOM) requests and you have been selected to generate some documents and scans:

1. Using ```bom```:

Generate a SPDX-JSON SBOM of image ```registry.k8s.io/kube-apiserver:v1.31.0```

Store it at ```/course/1/sbom1.json``` on ```cks9640```

2. Using ```trivy```:

Generate a CycloneDX SBOM of image ```registry.k8s.io/kube-controller-manager:v1.31.0```

3. Store it at ```/course/1/sbom2.json``` on ```cks9640```

Using ```trivy```:

Scan the existing SPDX-JSON SBOM at ```/course/1/sbom_check.json``` on cks9640 for known vulnerabilities. Save the result in JSON format at ```/course/1/sbom_check_result.json``` on ```cks9640```

## Question 2 | Runtime Security with Falco
Falco Documentation

 

Solve this question on: ```ssh cks5632```

 

Falco is installed on worker node ```cks5632-node1```. Connect using ```ssh cks5632-node1``` from ```cks5632```. There is a file ```/etc/falco/rules.d/falco_custom.yaml``` with rules that help you to:

1. Find a Pod running image ```httpd``` which modifies ```/etc/passwd```.

Scale the Deployment that controls that Pod down to 0.

2. Find a Pod running image ```nginx``` which triggers rule ```Package management process launched```.

Change the rule log text after ```Package management process launched``` to only include:


```time-with-nanoseconds,container-id,container-name,user-name```
Collect the logs for at least 20 seconds and save them under ```/course/2/falco.log on cks5632```.

Scale the Deployment that controls that Pod down to 0.

 

ℹ️ Use sudo -i to become root which may be required for this question

## Question 3 | Manual Static Security Analysis

Secrets Good Practices Security Checklist

Solve this question on: ```ssh cks9640```

The Release Engineering Team has shared some YAML manifests and Dockerfiles with you to review. The files are located under ```/course/3/files```.

As a container security expert, you are asked to perform a manual static analysis and find out possible security issues with respect to unwanted credential exposure. Running processes as root is of no concern in this task.

Write the filenames which have issues into ```/course/3/security-issues``` on ```cks9640```.

ℹ️ In the Dockerfiles and YAML manifests, assume that the referred files, folders, secrets and volume mounts are present. Disregard syntax or logic errors.

## Question 4 | Pod Security Standard

Pod Security Standards Pod Security Admission

Solve this question on: ```ssh cks6032```

There is a Deployment ```container-host-hacker``` in Namespace ```team-rose``` which mounts ```/run/containerd``` as a hostPath volume on the node where it's running. This means that the Pod can access various data about other containers running on the same node.

To prevent this, configure Namespace ```team-rose``` to ```enforce``` the ```baseline``` Pod Security Standard. Once completed, delete the Pod of the Deployment mentioned above.

Check the ReplicaSet events and write the event/log lines containing the reason why the Pod isn't recreated into ```/course/4/logs``` on ```cks6032```.

## Question 5 | Network Policy

Network Policies Declare Network Policy

Solve this question on: ```ssh cks4933```

Namespace ```team-ivy-private``` contains the Deployment ```api-private``` and a ```NetworkPolicy``` protecting it. Do not make any changes in that Namespace.

In Namespace ```team-ivy-gateway```, implement what the policy in ```team-ivy-private``` requires in order to:

- Ensure Deployment ```gateway-v1``` can access Deployment ```api-private``` only on port ```3000```

- Ensure Deployment ```gateway-v2``` can access Deployment ```api-private``` only on ports ```4000``` and ```5000```

Create a new NetworkPolicy (or multiple) which allows ```gateway-v1``` and ```gateway-v2``` to only have outgoing connections into Namespace ```team-ivy-private```. No incoming traffic control needed.


ℹ️ You can perform connectivity tests like:


```k -n team-ivy-gateway exec POD_NAME -- curl IP_ADDRESS```

## Question 6 | Verify Platform Binaries

Solve this question on: ```ssh cks1428```

 

There are four Kubernetes server binaries located at ```/course/6/binaries``` on ```cks1428```. You're provided with the following verified sha512 values for these:

kube-apiserver


```de0868c542a91e3be91ca02d9941a5a0530f6234b1324fe69782ec30bed950f59ef1931455a951f2fad02f26e6e205e4029c7f75aa76b6211f2d7983de07251d```

kube-controller-manager


```d3639ee2c51356d2f136da465407173e0113c52d0d0dc98a3602055f56360286dc68c645444ba6898a59d82d551e446edd107f22829242242d96e7e50ed11512```

kube-proxy


```d137a1b06a8e222eeb60007d93c28e0997f06c74548ba3e8ea7b0107c9d297f48eec8c4cbbd2ea41e53fc8bf885323f21a33238083d37754b63ef539ba71e7dd```

kubelet


```4cab451654202cee3dd4c2f937a045e6a832765762744beb4b69f2aadc87ab5762a4b970b6a2b8530de570cc532d370791d99276fafaddcc81b4a9e933665511```

Delete those binaries that don't match the sha512 values above.

 ## Question 7 | KubeletConfiguration

Configuring kubelets using kubeadm

 

Solve this question on: ```ssh cks9640```

 

You're asked to update the cluster's ```KubeletConfiguration```. Implement the following changes the Kubeadm way, so new nodes added to the cluster will receive the changes too:

1. Set these 
- Set ```containerLogMaxSize``` to ```5Mi```

- Set ```containerLogMaxFiles``` to ```3```

2. Apply the changes for the Kubelet on ```cks9640```

3. Apply the changes for the Kubelet on ```cks9640-node1```. Connect with ```ssh cks9640-node1``` from ```cks9640```

 

ℹ️ Use sudo -i to become root which may be required for this question

## Question 8 | CiliumNetworkPolicy

Cilium Documentation

Solve this question on: ```ssh cks6032```

In Namespace ```team-iris``` a Default-Allow strategy for all Namespace-internal traffic was chosen. There is an existing CiliumNetworkPolicy ```default-allow``` which ensures this and which should not be altered. That policy also allows cluster-internal DNS resolution.

Now it's time to deny and authenticate certain traffic. Create 3 CiliumNetworkPolicies in Namespace ```team-iris``` to implement the following requirements:

1. Create a ```Layer 3``` policy named ```p1``` to:

- Deny outgoing traffic from Pods with label ```type=messenger``` to Pods with label ```type=database```

2. Create a ```Layer 4``` policy named ```p2``` to:

- Deny outgoing ICMP ```EchoRequest``` traffic from Deployment ```transmitter``` to Pods with label ```type=database```

3. Create a ```Layer 3``` policy named ```p3``` to:

- Enable Mutual Authentication for outgoing traffic from Pods with label ```type=database``` to Pods with label ```type=messenger```

ℹ️ All Pods in the Namespace run plain Nginx images with open port 80. This allows simple connectivity tests like:


```k -n team-iris exec POD_NAME -- curl database```

 ## Question 9 | Certificates and Signing Requests

Certificates and Certificate Signing Requests Manage TLS Certificates in a Cluster

Solve this question on: ```ssh cks7984```

Create and approve the CertificateSigningRequest from ```/course/9/csr-app-6c63ce3f.yaml```, then download the decoded certificate to ```/course/9/app-6c63ce3f.crt```.

Create and deny the CertificateSigningRequest from ```/course/9/csr-app-dc6fdc2d.yaml```, then store the ```kubectl describe``` output from that resource at ```/course/9/csr-app-dc6fdc2d.log```.

Using the template below, create a CertificateSigningRequest YAML for ```/course/9/new.csr``` and store it at ```/course/9/new.csr.yaml```. The NAME should be the same as the ```CN``` subject of the ```new.csr``` file.


```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: {{NAME}}
spec:
  groups:
  - system:authenticated
  request: {{REQUEST}}
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

## Question 10 | Istio Security and mTLS
Istio Documentation

Solve this question on: ```ssh cks1428```

Deployment ```one``` runs in Namespace ```team-sedum``` and communicates with Deployment ```two``` via a Service of the same name.

Istio has been installed in the cluster. Enable Istio sidecar injection for the whole Namespace and ensure all current and future Pods are running with the Istio proxy sidecar.

## Question 11 | Secrets in ETCD

etcd Documentation Operating etcd clusters for Kubernetes

 

Solve this question on: ```ssh cks4933```

 

There is an existing Secret called ```database-access``` in Namespace ```team-daisy```.

1. Read the complete Secret content directly from ETCD (using ```etcdctl```) and store it into ```/course/11/etcd-secret-content``` on ```cks4933```

2. Write the plain decoded value of the Secret's key ```pass``` into ```/course/11/database-password``` on ```cks4933``` 

 

ℹ️ Use sudo -i to become root which may be required for this question


 
## Question 12 | Hack Secrets
Using RBAC Authorization

 

Solve this question on: ```ssh cks1428```

 

You're asked to investigate a possible privilege escalation using the pre-defined context. The context authenticates as user ```restricted``` which has only limited permissions and shouldn't be able to read Secret values.

1. Switch to the restricted context with:


- ```k config use-context restricted@workload-prod```

2. Try to find the ```password``` values of the Secrets ```secret1```, ```secret2``` and ```secret3``` in Namespace ```restricted``` using context ```restricted@workload-prod```

3. Write the decoded plaintext values into files ```/course/12/secret1```, ```/course/12/secret2``` and ```/course/12/secret3``` on ```cks1428```

4. Switch back to the default context with:


- ```k config use-context kubernetes-admin@kubernetes```


## Question 13 | RBAC Operator
Using RBAC Authorization

 

Solve this question on: ```ssh cks4933```

 

Operator ```cert-signer``` is installed in Namespace ```team-lilac``` and it communicates with the K8s API.

It's crashing. Check the error logs and create any missing permissions via ```RBAC``` until it runs without permission errors and without container restarts.

In addition, allow the operator's ServiceAccount to approve CertificateSigningRequests.

Create only the minimally needed permissions when possible.


 ## Question 14 | Syscall Activity
Falco Documentation

 

Solve this question on: ```ssh cks5632```

 

There are Pods in Namespace ```team-tulip```. A security investigation noticed that some processes running in these Pods are using the Syscall ```kill```, which is forbidden by an internal policy of Team Yellow.

Find the offending Pod(s) and remove these by reducing the replicas of the parent Deployment to 0.

You can connect to the worker node using ```ssh cks5632-node1``` from ```cks5632```

## Question 15 | Apiserver TLS Settings
kube-apiserver

 

Solve this question on: ```ssh cks7984```

 

Set the TLS min version of the Apiserver to version ```1.3.```

Afterwards use ```curl --tls-max 1.2 --tlsv1.2``` to call the Apiserver and write the full output including any errors to ```/course/15/curl.log```.

## Question 16 | Docker Image Attack Surface
 

Solve this question on: ```ssh cks5632```

 

There is a Deployment ```image-verify``` in Namespace ```team-maple``` which runs image ```registry.killer.sh:5000/image-verify:v1```. DevSecOps has asked you to improve this image by:

1. Changing the base image to ```alpine:3.22```

2. Not installing ```curl```

3. Updating ```nginx``` to use the version constraint ```>=1.18.0```

4. Running the main process as user ```myuser```

Do not add any new lines to the Dockerfile, just edit existing ones. The file is located at ```/course/16/image/Dockerfile```.

Tag your version as ```v2```. You can build, tag and push using:


```
cd /course/16/image
podman build -t registry.killer.sh:5000/image-verify:v2 
podman run registry.killer.sh:5000/image-verify:v2  # to test your changes
podman push registry.killer.sh:5000/image-verify:v2
```

Make the Deployment use your updated image tag ```v2```.

 

ℹ️ Make sure to run podman as user candidate and not root

 ## Question 17 | Update Kubernetes
Upgrading kubeadm clusters

 

Solve this question on: ```ssh cks9640```

 

The cluster is running Kubernetes ```1.34.8```, update it to ```1.35.6```.

Use ```apt``` package manager and ```kubeadm``` for this.

Use ```ssh cks9640-node1``` from ```cks9640``` to connect to the worker node.

 

ℹ️ Use sudo -i to become root which may be required for this question

 
 
