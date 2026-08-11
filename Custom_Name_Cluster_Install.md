# Setup Custom cluster name and User Name for k8

File: kubeadm-config.yaml
```
apiVersion: kubeadm.k8s.io/v1beta4
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: "192.168.1.146"
  bindPort: 6443
nodeRegistration:
  criSocket: "unix:///run/containerd/containerd.sock"
skipPhases:
  - "addon/kube-proxy"
---
apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
clusterName: "workload-prod-cluster"
networking:
  podSubnet: "10.244.0.0/16"
```
## Setup Control Plane

sudo kubeadm init --config=kubeadm-config.yaml

```
oracle@control146:~$ sudo kubeadm init --config=kubeadm-config.yaml
I0811 02:19:48.048528    4159 version.go:260] remote version is much newer: v1.36.3; falling back to: stable-1.35
[init] Using Kubernetes version: v1.35.7
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [control146.home.aarisha.com kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.1.146]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [control146.home.aarisha.com localhost] and IPs [192.168.1.146 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [control146.home.aarisha.com localhost] and IPs [192.168.1.146 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/instance-config.yaml"
[patches] Applied patch of type "application/strategic-merge-patch+json" to target "kubeletconfiguration"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 553.98µs
[control-plane-check] Waiting for healthy control plane components. This can take up to 4m0s
[control-plane-check] Checking kube-apiserver at https://192.168.1.146:6443/livez
[control-plane-check] Checking kube-controller-manager at https://127.0.0.1:10257/healthz
[control-plane-check] Checking kube-scheduler at https://127.0.0.1:10259/livez
[control-plane-check] kube-scheduler is healthy after 3.132722ms
[control-plane-check] kube-controller-manager is healthy after 3.695059ms
[control-plane-check] kube-apiserver is healthy after 1.501645283s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node control146.home.aarisha.com as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node control146.home.aarisha.com as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: b1eoah.svrtbhp8vxjte3in
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Configured RBAC rules to allow the API server kubelet client certificate to access the kubelet API
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.146:6443 --token b1eoah.svrtbhp8vxjte3in \
        --discovery-token-ca-cert-hash sha256:7abee9aaed71ed5c84a873ad11bbcb67322d1b4125dad4286f5184e8e4249823
oracle@control146:~$


```

## Set up kube-context

```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
oracle@control146:~$   mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

oracle@control146:~$ 
```

## Extract the client certificate data and client key data into a new user name
```
kubectl config set-credentials workload-admin \
  --client-certificate=$(kubectl config view --raw -o jsonpath='{.users[?(@.name=="kubernetes-admin")].user.client-certificate-data}') \
  --client-key=$(kubectl config view --raw -o jsonpath='{.users[?(@.name=="kubernetes-admin")].user.client-key-data}')


kubectl config set-context workload-context \
  --cluster=workload-prod-cluster \
  --user=workload-admin
```
Logs

```
oracle@control146:~$ kubectl config set-credentials workload-admin \
  --client-certificate=$(kubectl config view --raw -o jsonpath='{.users[?(@.name=="kubernetes-admin")].user.client-certificate-data}') \
  --client-key=$(kubectl config view --raw -o jsonpath='{.users[?(@.name=="kubernetes-admin")].user.client-key-data}')
User "workload-admin" set.
oracle@control146:~$ kubectl config set-context workload-context \
  --cluster=workload-prod-cluster \
  --user=workload-admin
Context "workload-context" created.
oracle@control146:~$
```

## Join Worker Nodes

```
sudo kubeadm join 192.168.1.146:6443 --token b1eoah.svrtbhp8vxjte3in \
        --discovery-token-ca-cert-hash sha256:7abee9aaed71ed5c84a873ad11bbcb67322d1b4125dad4286f5184e8e4249823 \
         --cri-socket unix:///run/containerd/containerd.sock
```

Logs

```
oracle@worker148:~$ sudo kubeadm join 192.168.1.146:6443 --token b1eoah.svrtbhp8vxjte3in \
        --discovery-token-ca-cert-hash sha256:7abee9aaed71ed5c84a873ad11bbcb67322d1b4125dad4286f5184e8e4249823 \
         --cri-socket unix:///run/containerd/containerd.sock
[preflight] Running pre-flight checks
[preflight] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[preflight] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.
W0811 02:29:31.452046    4621 configset.go:77] Warning: No kubeproxy.config.k8s.io/v1alpha1 config is loaded. Continuing without it: configmaps "kube-proxy" is forbidden: User "system:bootstrap:b1eoah" cannot get resource "configmaps" in API group "" in the namespace "kube-system"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/instance-config.yaml"
[patches] Applied patch of type "application/strategic-merge-patch+json" to target "kubeletconfiguration"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 500.704651ms
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

oracle@worker148:~$

oracle@worker149:~$ sudo kubeadm join 192.168.1.146:6443 --token b1eoah.svrtbhp8vxjte3in \
        --discovery-token-ca-cert-hash sha256:7abee9aaed71ed5c84a873ad11bbcb67322d1b4125dad4286f5184e8e4249823 \
         --cri-socket unix:///run/containerd/containerd.sock
[preflight] Running pre-flight checks
[preflight] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[preflight] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.
W0811 02:29:39.303233    4663 configset.go:77] Warning: No kubeproxy.config.k8s.io/v1alpha1 config is loaded. Continuing without it: configmaps "kube-proxy" is forbidden: User "system:bootstrap:b1eoah" cannot get resource "configmaps" in API group "" in the namespace "kube-system"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/instance-config.yaml"
[patches] Applied patch of type "application/strategic-merge-patch+json" to target "kubeletconfiguration"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 500.640087ms
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

oracle@worker149:~$

```

## Install Cilium CNI - On Control Plane

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add cilium https://helm.cilium.io/
helm repo update

API_SERVER_IP=192.168.1.146 
API_SERVER_PORT=6443

 helm install cilium cilium/cilium \
  --namespace kube-system \
  --set kubeProxyReplacement=true \
  --set k8sServiceHost=${API_SERVER_IP} \
  --set k8sServicePort=${API_SERVER_PORT} \
  --set hubble.relay.enabled=false \
  --set hubble.ui.enabled=false

```
Logs

```
oracle@control146:~$ curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 12252  100 12252    0     0  56472      0 --:--:-- --:--:-- --:--:-- 56722
Downloading https://get.helm.sh/helm-v3.21.3-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
oracle@control146:~$ helm repo add cilium https://helm.cilium.io/
"cilium" has been added to your repositories
oracle@control146:~$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "cilium" chart repository
Update Complete. ⎈Happy Helming!⎈
oracle@control146:~$ API_SERVER_IP=192.168.1.146
oracle@control146:~$ API_SERVER_PORT=6443
oracle@control146:~$ helm install cilium cilium/cilium \
  --namespace kube-system \
  --set kubeProxyReplacement=true \
  --set k8sServiceHost=${API_SERVER_IP} \
  --set k8sServicePort=${API_SERVER_PORT} \
  --set hubble.relay.enabled=false \
  --set hubble.ui.enabled=false
NAME: cilium
LAST DEPLOYED: Tue Aug 11 02:35:30 2026
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
You have successfully installed Cilium with Hubble.

Your release version is 1.20.0.

For any further help, visit https://docs.cilium.io/en/v1.20/gettinghelp
oracle@control146:~$
```
## Watch the pods startup and verify the nodes

kubectl -n kube-system get pods --watch
kubectl get nodes -o wide

```
oracle@control146:~$ kubectl -n kube-system get pods --watch
NAME                                                  READY   STATUS    RESTARTS   AGE
cilium-25tdq                                          0/1     Running   0          35s
cilium-envoy-6pt9r                                    0/1     Running   0          35s
cilium-envoy-jtghl                                    0/1     Running   0          35s
cilium-envoy-tr985                                    0/1     Running   0          35s
cilium-hp8jq                                          0/1     Running   0          35s
cilium-operator-784d787dbb-hvszq                      1/1     Running   0          35s
cilium-operator-784d787dbb-vm4lj                      1/1     Running   0          35s
cilium-tdqf6                                          0/1     Running   0          35s
coredns-7d764666f9-chcts                              0/1     Pending   0          15m
coredns-7d764666f9-v5xsl                              0/1     Pending   0          15m
etcd-control146.home.aarisha.com                      1/1     Running   0          16m
kube-apiserver-control146.home.aarisha.com            1/1     Running   0          16m
kube-controller-manager-control146.home.aarisha.com   1/1     Running   0          16m
kube-scheduler-control146.home.aarisha.com            1/1     Running   0          16m
cilium-tdqf6                                          0/1     Running   0          38s
cilium-tdqf6                                          1/1     Running   0          38s
cilium-25tdq                                          0/1     Running   0          38s
cilium-hp8jq                                          0/1     Running   0          38s
cilium-25tdq                                          1/1     Running   0          38s
cilium-hp8jq                                          1/1     Running   0          38s
cilium-envoy-jtghl                                    0/1     Running   0          40s
cilium-envoy-6pt9r                                    0/1     Running   0          40s
cilium-envoy-6pt9r                                    1/1     Running   0          40s
cilium-envoy-jtghl                                    1/1     Running   0          40s
coredns-7d764666f9-v5xsl                              0/1     Pending   0          16m
coredns-7d764666f9-chcts                              0/1     Pending   0          16m
coredns-7d764666f9-v5xsl                              0/1     ContainerCreating   0          16m
coredns-7d764666f9-chcts                              0/1     ContainerCreating   0          16m
coredns-7d764666f9-chcts                              0/1     Running             0          16m
coredns-7d764666f9-v5xsl                              1/1     Running             0          16m
coredns-7d764666f9-chcts                              1/1     Running             0          16m
^C
oracle@control146:~$ kubectl get nodes -o wide
NAME                          STATUS   ROLES           AGE    VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION      CONTAINER-RUNTIME
control146.home.aarisha.com   Ready    control-plane   16m    v1.35.6   192.168.1.146   <none>        Ubuntu 24.04 LTS   6.8.0-134-generic   containerd://2.2.5
worker148.home.aarisha.com    Ready    <none>          7m8s   v1.35.6   192.168.1.148   <none>        Ubuntu 24.04 LTS   6.8.0-134-generic   containerd://2.2.5
worker149.home.aarisha.com    Ready    <none>          7m1s   v1.35.6   192.168.1.149   <none>        Ubuntu 24.04 LTS   6.8.0-134-generic   containerd://2.2.5
oracle@control146:~$
```