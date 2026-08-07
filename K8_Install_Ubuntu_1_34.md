# K8 V 1.34 Installation Document
Ref:
https://v1-34.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

# DNS Resolution
________________
Make sure nameserver has address 192.168.1.111 for all hosts

oracle@worker126:~$ sudo cat /etc/netplan/50-cloud-init.yaml
network:
  version: 2
  ethernets:
    ens18:
      addresses:
      - "192.168.1.126/24"
      nameservers:
        addresses:
        - 192.168.1.111
        search:
        - home.aarisha.com
      routes:
      - to: "default"
        via: "192.168.1.1"
oracle@worker126:~$


# A. OS Enable Firewall Ports
____________________________

# ---- CONTROL PLANE ----
ufw allow 6443/tcp        # API server
ufw allow 2379:2380/tcp   # etcd
ufw allow 10250/tcp       # Kubelet
ufw allow 10257/tcp       # kube-controller-manager
ufw allow 10259/tcp       # kube-scheduler

# ---- WORKER NODES ----
ufw allow 10250/tcp       # Kubelet
ufw allow 30000:32767/tcp # NodePort services

# ---- CILIUM (all nodes) ----
ufw allow 4240/tcp        # health checks
ufw allow 8472/udp        # VXLAN

# ---- FALCO (all nodes) ----
ufw allow 8765/tcp        # Falco web server
# ufw allow 2801/tcp      # Falcosidekick (if used)

# ---- ADMIN ----
ufw allow 22/tcp          # SSH
ufw enable


# A.1 Control Plane Logs 
```
root@control124:~# ufw allow 6443/tcp        # API server
ufw allow 2379:2380/tcp   # etcd
ufw allow 10250/tcp       # Kubelet
ufw allow 10257/tcp       # kube-controller-manager
ufw allow 10259/tcp       # kube-scheduler
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
root@control124:~#
root@control124:~# ufw allow 4240/tcp        # health checks
ufw allow 8472/udp        # VXLAN
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
root@control124:~# ufw allow 8765/tcp        # Falco web server
Rules updated
Rules updated (v6)
root@control124:~# ufw allow 22/tcp          # SSH
ufw enable
Rules updated
Rules updated (v6)
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
root@control124:~#
```
# A.2 Worker Node 1 logs

```
oracle@worker125:~$ sudo su -
root@worker125:~# ufw allow 10250/tcp       # Kubelet
ufw allow 30000:32767/tcp # NodePort services
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
root@worker125:~# ufw allow 4240/tcp        # health checks
ufw allow 8472/udp        # VXLAN
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
root@worker125:~# ufw allow 8765/tcp        # Falco web server
Rules updated
Rules updated (v6)
root@worker125:~# ufw allow 22/tcp          # SSH
ufw enable
Rules updated
Rules updated (v6)
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
root@worker125:~#

```
# A.3 Worker Node 2 logs

```
root@worker126:~# # ---- WORKER NODES ----
ufw allow 10250/tcp       # Kubelet
ufw allow 30000:32767/tcp # NodePort services

# ---- CILIUM (all nodes) ----
ufw allow 4240/tcp        # health checks
ufw allow 8472/udp        # VXLAN

# ---- FALCO (all nodes) ----
ufw allow 8765/tcp        # Falco web server
# ufw allow 2801/tcp      # Falcosidekick (if used)

# ---- ADMIN ----
ufw allow 22/tcp          # SSH
ufw enable
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
root@worker126:~#


```

# B Check Swap = 0
___________________

free

# B.1 Control Plane

```
root@control124:~# free
               total        used        free      shared  buff/cache   available
Mem:         6067404      425360     5608324        1004      265468     5642044
Swap:              0           0           0
root@control124:~#

```

# B.2 Worker Node 1

```
root@worker125:~# free
               total        used        free      shared  buff/cache   available
Mem:         6067400      430812     5594308        1004      278880     5636588
Swap:              0           0           0
root@worker125:~#

```

# B.3 Worker Node 2

```
root@worker126:~# free
               total        used        free      shared  buff/cache   available
Mem:         6067396      425624     5602156        1004      271972     5641772
Swap:              0           0           0
root@worker126:~#

```

# C containerd Setup : Step 1 — Remove old packages - All Nodes
_______________________________________________________________

sudo apt remove docker.io docker-compose docker-compose-v2 \
                docker-doc podman-docker containerd runc

``` 
oracle@control124:~$ sudo apt remove docker.io docker-compose docker-compose-v2 \
                docker-doc podman-docker containerd runc
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package 'docker.io' is not installed, so not removed
Package 'docker-compose' is not installed, so not removed
Package 'docker-compose-v2' is not installed, so not removed
Package 'docker-doc' is not installed, so not removed
Package 'podman-docker' is not installed, so not removed
Package 'containerd' is not installed, so not removed
Package 'runc' is not installed, so not removed
0 upgraded, 0 newly installed, 0 to remove and 139 not upgraded.
oracle@control124:~$

```
# D containerd Setup : Step 2 — Kernel modules - All Nodes

sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

```
oracle@worker126:~$ sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
overlay
br_netfilter
oracle@worker126:~$

```

# E containerd Setup : Step 3 — Sysctl params - All Nodes

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

```
oracle@control124:~$ sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
overlay
br_netfilter
oracle@control124:~$ sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
* Applying /usr/lib/sysctl.d/10-apparmor.conf ...
* Applying /etc/sysctl.d/10-console-messages.conf ...
* Applying /etc/sysctl.d/10-ipv6-privacy.conf ...
* Applying /etc/sysctl.d/10-kernel-hardening.conf ...
* Applying /etc/sysctl.d/10-magic-sysrq.conf ...
* Applying /etc/sysctl.d/10-map-count.conf ...
* Applying /etc/sysctl.d/10-network-security.conf ...
* Applying /etc/sysctl.d/10-ptrace.conf ...
* Applying /etc/sysctl.d/10-zeropage.conf ...
* Applying /usr/lib/sysctl.d/50-pid-max.conf ...
* Applying /usr/lib/sysctl.d/99-protect-links.conf ...
* Applying /etc/sysctl.d/99-sysctl.conf ...
* Applying /etc/sysctl.d/kubernetes.conf ...
* Applying /etc/sysctl.conf ...
kernel.apparmor_restrict_unprivileged_userns = 1
kernel.printk = 4 4 1 7
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
kernel.kptr_restrict = 1
kernel.sysrq = 176
vm.max_map_count = 1048576
net.ipv4.conf.default.rp_filter = 2
net.ipv4.conf.all.rp_filter = 2
kernel.yama.ptrace_scope = 1
vm.mmap_min_addr = 65536
kernel.pid_max = 4194304
fs.protected_fifos = 1
fs.protected_hardlinks = 1
fs.protected_regular = 2
fs.protected_symlinks = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
oracle@control124:~$

```
# F containerd Setup : Step 4 — Add Docker's apt repo - All Nodes

sudo apt update
sudo apt install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

```
root@control124:~# sudo apt update
sudo apt install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
Get:1 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Hit:2 http://us.archive.ubuntu.com/ubuntu noble InRelease
Get:3 http://us.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Hit:4 http://us.archive.ubuntu.com/ubuntu noble-backports InRelease
Get:5 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [1,041 kB]
Get:6 http://us.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [1,656 kB]
Fetched 2,949 kB in 1s (2,582 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
139 packages can be upgraded. Run 'apt list --upgradable' to see them.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20260601~24.04.1).
ca-certificates set to manually installed.
curl is already the newest version (8.5.0-2ubuntu10.9).
curl set to manually installed.
gnupg is already the newest version (2.4.4-2ubuntu17.4).
gnupg set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 139 not upgraded.
Get:1 https://download.docker.com/linux/ubuntu noble InRelease [48.5 kB]
Hit:2 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu noble InRelease
Hit:4 http://us.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:5 http://us.archive.ubuntu.com/ubuntu noble-backports InRelease
Get:6 https://download.docker.com/linux/ubuntu noble/stable amd64 Packages [58.9 kB]
Fetched 107 kB in 1s (187 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
139 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@control124:~#

```
# G containerd Setup : Step 5 — Install containerd - All Nodes

sudo apt install -y containerd.io

```
root@control124:~# sudo apt install -y containerd.io
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  containerd.io
0 upgraded, 1 newly installed, 0 to remove and 139 not upgraded.
Need to get 23.6 MB of archives.
After this operation, 93.5 MB of additional disk space will be used.
Get:1 https://download.docker.com/linux/ubuntu noble/stable amd64 containerd.io amd64 2.2.5-1~ubuntu.24.04~noble [23.6 MB]
Fetched 23.6 MB in 2s (11.3 MB/s)
Selecting previously unselected package containerd.io.
(Reading database ... 85319 files and directories currently installed.)
Preparing to unpack .../containerd.io_2.2.5-1~ubuntu.24.04~noble_amd64.deb ...
Unpacking containerd.io (2.2.5-1~ubuntu.24.04~noble) ...
Setting up containerd.io (2.2.5-1~ubuntu.24.04~noble) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /usr/lib/systemd/system/containerd.service.
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
root@control124:~#

```

# H containerd Setup : Step 6 — Generate full config and enable SystemdCgroup - All Nodes

sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

## Confirm it took
grep "SystemdCgroup" /etc/containerd/config.toml



```
root@worker126:~# sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

## Confirm it took
grep "SystemdCgroup" /etc/containerd/config.toml
            SystemdCgroup = true
root@worker126:~#

```

# I containerd Setup : Step 7 — Enable and restart containerd - All Nodes

# *** Restart BEFORE proceeding - ensures new config is loaded ***
sudo systemctl daemon-reload
sudo systemctl restart containerd
sudo systemctl enable --now containerd
sudo systemctl status containerd

ls -l /run/containerd/containerd.sock

```
root@control124:~# sudo systemctl daemon-reload
sudo systemctl enable --now containerd
sudo systemctl status containerd

ls -l /run/containerd/containerd.sock
● containerd.service - containerd container runtime
     Loaded: loaded (/usr/lib/systemd/system/containerd.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-06-27 03:39:32 UTC; 3min 53s ago
       Docs: https://containerd.io
   Main PID: 2802 (containerd)
      Tasks: 9
     Memory: 13.8M (peak: 14.9M)
        CPU: 261ms
     CGroup: /system.slice/containerd.service
             └─2802 /usr/bin/containerd

Jun 27 03:39:32 control124.home.aarisha.com containerd[2802]: time="2026-06-27T03:39:32.517522090Z" level=info msg="loading plugin" id=io.containerd.tracing.pr>
Jun 27 03:39:32 control124.home.aarisha.com containerd[2802]: time="2026-06-27T03:39:32.517534523Z" level=info msg="skip loading plugin" error="skip plugin: tr>
Jun 27 03:39:32 control124.home.aarisha.com containerd[2802]: time="2026-06-27T03:39:32.517540785Z" level=info msg="loading plugin" id=io.containerd.internal.v>
Jun 27 03:39:32 control124.home.aarisha.com containerd[2802]: time="2026-06-27T03:39:32.517547568Z" level=info msg="skip loading plugin" error="skip plugin: tr>
Jun 27 03:39:32 control124.home.aarisha.com containerd[2802]: time="2026-06-27T03:39:32.517552888Z" level=info msg="loading plugin" id=io.containerd.ttrpc.v1.o>
Jun 27 03:39:32 control124.home.aarisha.com containerd[2802]: time="2026-06-27T03:39:32.517560823Z" level=info msg="loading plugin" id=io.containerd.grpc.v1.he>
Jun 27 03:39:32 control124.home.aarisha.com containerd[2802]: time="2026-06-27T03:39:32.517755037Z" level=info msg=serving... address=/run/containerd/container>
Jun 27 03:39:32 control124.home.aarisha.com containerd[2802]: time="2026-06-27T03:39:32.517801785Z" level=info msg=serving... address=/run/containerd/container>
Jun 27 03:39:32 control124.home.aarisha.com containerd[2802]: time="2026-06-27T03:39:32.517851158Z" level=info msg="containerd successfully booted in 0.023647s"
Jun 27 03:39:32 control124.home.aarisha.com systemd[1]: Started containerd.service - containerd container runtime.

srw-rw---- 1 root root 0 Jun 27 03:39 /run/containerd/containerd.sock
root@control124:~#


```

# J containerd Setup : Step 8 — Install crictl - All Nodes

VERSION="v1.34.0"
curl -fsSL https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz | \
  sudo tar -zxvf - -C /usr/local/bin

cat <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: false
EOF

```
root@control124:~# VERSION="v1.34.0"
curl -fsSL https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz | \
  sudo tar -zxvf - -C /usr/local/bin

cat <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: false
EOF
crictl
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: false
root@control124:~#

```
# K containerd Setup : Step 9 — Verify - All Nodes

# Verify sysctl
sysctl net.bridge.bridge-nf-call-iptables \
       net.bridge.bridge-nf-call-ip6tables \
       net.ipv4.ip_forward

# Verify modules
lsmod | grep -E 'overlay|br_netfilter'

# Verify containerd CRI — expect RuntimeReady: true, NetworkReady: false
sudo crictl info | grep -A3 '"conditions"'

```
root@control124:~# # Verify sysctl
sysctl net.bridge.bridge-nf-call-iptables \
       net.bridge.bridge-nf-call-ip6tables \
       net.ipv4.ip_forward

# Verify modules
lsmod | grep -E 'overlay|br_netfilter'

# Verify containerd CRI — expect RuntimeReady: true, NetworkReady: false
sudo crictl info | grep -A3 '"conditions"'
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0
root@control124:~# sudo crictl info | grep -E 'RuntimeReady|NetworkReady|status'
  "status": {
        "status": true,
        "type": "RuntimeReady"
        "status": false,
        "type": "NetworkReady"
        "status": true,
root@control124:~#

```

# L. Installing kubeadm, kubelet and kubectl
____________________________________________

# Step 1 — Install prerequisites
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Step 2 — Add Kubernetes GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Step 3 — Add repo as single line (no backslash continuation)
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Step 4 — Verify the file looks correct before continuing
cat /etc/apt/sources.list.d/kubernetes.list

# Step 5 — Update and install
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Step 6 — Verify install
kubeadm version
kubectl version --client
kubelet --version

```
oracle@worker125:~$ # Step 1 — Install prerequisites
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Step 2 — Add Kubernetes GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Step 3 — Add repo as single line (no backslash continuation)
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Step 4 — Verify the file looks correct before continuing
cat /etc/apt/sources.list.d/kubernetes.list

# Step 5 — Update and install
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Step 6 — Verify install
kubeadm version
kubectl version --client
kubelet --version
Hit:1 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:2 https://download.docker.com/linux/ubuntu noble InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu noble InRelease
Get:4 http://us.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Hit:5 http://us.archive.ubuntu.com/ubuntu noble-backports InRelease
Get:6 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [1,041 kB]
Get:7 http://us.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [1,656 kB]
Fetched 2,823 kB in 1s (2,879 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20260601~24.04.1).
curl is already the newest version (8.5.0-2ubuntu10.9).
gpg is already the newest version (2.4.4-2ubuntu17.4).
gpg set to manually installed.
Suggested packages:
  apt-doc aptitude | synaptic | wajig dpkg-dev
The following NEW packages will be installed:
  apt-transport-https
The following packages will be upgraded:
  apt apt-utils libapt-pkg6.0t64
3 upgraded, 1 newly installed, 0 to remove and 136 not upgraded.
Need to get 2,580 kB of archives.
After this operation, 47.1 kB of additional disk space will be used.
Get:1 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 libapt-pkg6.0t64 amd64 2.8.3 [985 kB]
Get:2 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 apt amd64 2.8.3 [1,376 kB]
Get:3 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 apt-utils amd64 2.8.3 [216 kB]
Get:4 http://us.archive.ubuntu.com/ubuntu noble-updates/universe amd64 apt-transport-https all 2.8.3 [3,970 B]
Fetched 2,580 kB in 1s (4,329 kB/s)
(Reading database ... 85333 files and directories currently installed.)
Preparing to unpack .../libapt-pkg6.0t64_2.8.3_amd64.deb ...
Unpacking libapt-pkg6.0t64:amd64 (2.8.3) over (2.7.14build2) ...
Setting up libapt-pkg6.0t64:amd64 (2.8.3) ...
(Reading database ... 85333 files and directories currently installed.)
Preparing to unpack .../archives/apt_2.8.3_amd64.deb ...
Unpacking apt (2.8.3) over (2.7.14build2) ...
Setting up apt (2.8.3) ...
(Reading database ... 85333 files and directories currently installed.)
Preparing to unpack .../apt-utils_2.8.3_amd64.deb ...
Unpacking apt-utils (2.8.3) over (2.7.14build2) ...
Selecting previously unselected package apt-transport-https.
Preparing to unpack .../apt-transport-https_2.8.3_all.deb ...
Unpacking apt-transport-https (2.8.3) ...
Setting up apt-utils (2.8.3) ...
Setting up apt-transport-https (2.8.3) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.7) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...
 systemctl restart packagekit.service

Service restarts being deferred:
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /
Hit:1 http://us.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:4 http://us.archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:6 https://download.docker.com/linux/ubuntu noble InRelease
Get:5 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.34/deb  InRelease [1,230 B]
Get:7 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.34/deb  Packages [14.3 kB]
Fetched 15.5 kB in 1s (21.6 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  kubernetes-cni
The following NEW packages will be installed:
  kubeadm kubectl kubelet kubernetes-cni
0 upgraded, 4 newly installed, 0 to remove and 136 not upgraded.
Need to get 77.5 MB of archives.
After this operation, 293 MB of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.34/deb  kubeadm 1.34.9-1.1 [12.6 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.34/deb  kubectl 1.34.9-1.1 [11.8 MB]
Get:3 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.34/deb  kubernetes-cni 1.7.1-1.1 [39.9 MB]
Get:4 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.34/deb  kubelet 1.34.9-1.1 [13.1 MB]
Fetched 77.5 MB in 3s (27.8 MB/s)
Selecting previously unselected package kubeadm.
(Reading database ... 85337 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.34.9-1.1_amd64.deb ...
Unpacking kubeadm (1.34.9-1.1) ...
Selecting previously unselected package kubectl.
Preparing to unpack .../kubectl_1.34.9-1.1_amd64.deb ...
Unpacking kubectl (1.34.9-1.1) ...
Selecting previously unselected package kubernetes-cni.
Preparing to unpack .../kubernetes-cni_1.7.1-1.1_amd64.deb ...
Unpacking kubernetes-cni (1.7.1-1.1) ...
Selecting previously unselected package kubelet.
Preparing to unpack .../kubelet_1.34.9-1.1_amd64.deb ...
Unpacking kubelet (1.34.9-1.1) ...
Setting up kubeadm (1.34.9-1.1) ...
Setting up kubectl (1.34.9-1.1) ...
Setting up kubernetes-cni (1.7.1-1.1) ...
Setting up kubelet (1.34.9-1.1) ...
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...

Service restarts being deferred:
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
kubeadm version: &version.Info{Major:"1", Minor:"34", EmulationMajor:"", EmulationMinor:"", MinCompatibilityMajor:"", MinCompatibilityMinor:"", GitVersion:"v1.34.9", GitCommit:"ad7c7374b74c04d07ea041d367ecb1a526bdf758", GitTreeState:"clean", BuildDate:"2026-06-11T17:39:19Z", GoVersion:"go1.25.11", Compiler:"gc", Platform:"linux/amd64"}
Client Version: v1.34.9
Kustomize Version: v5.7.1
Kubernetes v1.34.9
oracle@worker125:~$

```

# M. Control Plane Specific - kubeadm init
__________________________________________

sudo kubeadm init \
  --apiserver-advertise-address=192.168.1.124 \
  --pod-network-cidr=10.244.0.0/16 \
  --skip-phases=addon/kube-proxy \
  --cri-socket unix:///run/containerd/containerd.sock

```
oracle@control124:~$ sudo kubeadm init \
  --apiserver-advertise-address=192.168.1.124 \
  --pod-network-cidr=10.244.0.0/16 \
  --skip-phases=addon/kube-proxy \
  --cri-socket unix:///run/containerd/containerd.sock
I0627 14:31:37.643506    6206 version.go:260] remote version is much newer: v1.36.2; falling back to: stable-1.34
[init] Using Kubernetes version: v1.34.9
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [control124.home.aarisha.com kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.1.124]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [control124.home.aarisha.com localhost] and IPs [192.168.1.124 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [control124.home.aarisha.com localhost] and IPs [192.168.1.124 127.0.0.1 ::1]
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
[kubelet-check] The kubelet is healthy after 770.445µs
[control-plane-check] Waiting for healthy control plane components. This can take up to 4m0s
[control-plane-check] Checking kube-apiserver at https://192.168.1.124:6443/livez
[control-plane-check] Checking kube-controller-manager at https://127.0.0.1:10257/healthz
[control-plane-check] Checking kube-scheduler at https://127.0.0.1:10259/livez
[control-plane-check] kube-scheduler is healthy after 4.489162ms
[control-plane-check] kube-controller-manager is healthy after 4.678096ms
[control-plane-check] kube-apiserver is healthy after 2.00150947s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node control124.home.aarisha.com as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node control124.home.aarisha.com as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: 9epghq.50goni44wrkqh5fv
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

sudo kubeadm join 192.168.1.124:6443 --token vjhill.6n2alpb7yxzy4a8n \
        --discovery-token-ca-cert-hash sha256:17379bb0fffe04aca377a24567ab3022e02dc04c559d4e52990309be1e3f95b9  \
        --cri-socket unix:///run/containerd/containerd.sock

oracle@control124:~$

```
# N Copy .kube/config file

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

```
oracle@control124:~$   mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
oracle@control124:~$

```


# P Join othe Nodes on two worker nodes
_______________________________________

sudo kubeadm join 192.168.1.124:6443 --token vjhill.6n2alpb7yxzy4a8n \
        --discovery-token-ca-cert-hash sha256:17379bb0fffe04aca377a24567ab3022e02dc04c559d4e52990309be1e3f95b9  \
        --cri-socket unix:///run/containerd/containerd.sock

```
oracle@worker125:~$ sudo kubeadm join 192.168.1.124:6443 --token 9epghq.50goni44wrkqh5fv \
        --discovery-token-ca-cert-hash sha256:ac1c9e57002e9488c16b08381e255fefdf57e5a95ff8d97533a90357f6810c17 \
        --cri-socket unix:///run/containerd/containerd.sock
[preflight] Running pre-flight checks
[preflight] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[preflight] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.
W0627 15:13:47.381516    6239 configset.go:77] Warning: No kubeproxy.config.k8s.io/v1alpha1 config is loaded. Continuing without it: configmaps "kube-proxy" is forbidden: User "system:bootstrap:9epghq" cannot get resource "configmaps" in API group "" in the namespace "kube-system"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/instance-config.yaml"
[patches] Applied patch of type "application/strategic-merge-patch+json" to target "kubeletconfiguration"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 501.434969ms
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

oracle@worker125:~$
```

# Q Verify Nodes
________________

kubectl get nodes

```
oracle@control124:~$ kubectl get nodes
NAME                          STATUS     ROLES           AGE    VERSION
control124.home.aarisha.com   NotReady   control-plane   2m6s   v1.34.9
worker125.home.aarisha.com    NotReady   <none>          18s    v1.34.9
worker126.home.aarisha.com    NotReady   <none>          11s    v1.34.9
oracle@control124:~$

```


# O Install Cilium CNI - On Control Plane
_________________________________________

--- Step1 Install Helm on control plane

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version

```
oracle@control124:~$ curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 11929  100 11929    0     0  45986      0 --:--:-- --:--:-- --:--:-- 45880
Downloading https://get.helm.sh/helm-v3.21.2-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
version.BuildInfo{Version:"v3.21.2", GitCommit:"125963406833fe0525be91f46c8b5b0f22fb9e32", GitTreeState:"clean", GoVersion:"go1.26.4"}
oracle@control124:~$

```
--- Step 2 Install Cilium CNI (Control Plane Only)

# Add Cilium helm repo
helm repo add cilium https://helm.cilium.io/
helm repo update

```
oracle@control124:~$ helm repo add cilium https://helm.cilium.io/
"cilium" has been added to your repositories
oracle@control124:~$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "cilium" chart repository
Update Complete. ⎈Happy Helming!⎈
oracle@control124:~$

```

# Set your control plane IP
API_SERVER_IP=192.168.1.124   # e.g. 192.168.1.124
API_SERVER_PORT=6443

helm install cilium cilium/cilium \
  --namespace kube-system \
  --set kubeProxyReplacement=true \
  --set k8sServiceHost=${API_SERVER_IP} \
  --set k8sServicePort=${API_SERVER_PORT} \
  --set hubble.relay.enabled=false \
  --set hubble.ui.enabled=false

# Verify nodes are ready

kubectl get nodes

```
oracle@control124:~$ kubectl get nodes
NAME                          STATUS   ROLES           AGE     VERSION
control124.home.aarisha.com   Ready    control-plane   6m46s   v1.34.9
worker125.home.aarisha.com    Ready    <none>          4m58s   v1.34.9
worker126.home.aarisha.com    Ready    <none>          4m51s   v1.34.9
oracle@control124:~$ 

```
# Watch Cilium come up
kubectl -n kube-system get pods --watch

# R Verify Pods
__________________

kubectl get pods -A

oracle@control124:~$ kubectl get pods -A
NAMESPACE     NAME                                                  READY   STATUS      RESTARTS   AGE
kube-system   cilium-envoy-f8t5g                                    1/1     Running     0          10m
kube-system   cilium-envoy-jwmpz                                    1/1     Running     0          10m
kube-system   cilium-envoy-x5d8f                                    1/1     Running     0          10m
kube-system   cilium-nk8bp                                          1/1     Running     0          6m37s
kube-system   cilium-operator-b8d96fcdf-8lpg7                       1/1     Running     0          10m
kube-system   cilium-operator-b8d96fcdf-zsxgd                       1/1     Running     0          10m
kube-system   cilium-prcq2                                          1/1     Running     0          6m24s
kube-system   cilium-prw2d                                          1/1     Running     0          6m37s
kube-system   coredns-66bc5c9577-58jpl                              1/1     Running     0          12h
kube-system   coredns-66bc5c9577-pchqr                              1/1     Running     0          12h
kube-system   etcd-control124.home.aarisha.com                      1/1     Running     0          12h
kube-system   hubble-generate-certs-8def919bf5-tsjc7                0/1     Completed   0          52s
kube-system   kube-apiserver-control124.home.aarisha.com            1/1     Running     0          12h
kube-system   kube-controller-manager-control124.home.aarisha.com   1/1     Running     0          12h
kube-system   kube-scheduler-control124.home.aarisha.com            1/1     Running     0          12h


# S Install Falco - All 3 nodes
________________________________

# Step 1 — Add Falco repo

curl -fsSL https://falco.org/repo/falcosecurity-packages.asc | \
  sudo gpg --dearmor -o /usr/share/keyrings/falco-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/falco-archive-keyring.gpg] \
  https://download.falco.org/packages/deb stable main" | \
  sudo tee /etc/apt/sources.list.d/falcosecurity.list

sudo apt-get update

```

```

# Step 2 — Install Falco non-interactively with modern eBPF

sudo FALCO_FRONTEND=noninteractive \
     FALCO_DRIVER_CHOICE=modern_ebpf \
     apt-get install -y falco

```
[POST-INSTALL] Trigger deamon-reload:
[POST-INSTALL] Enable 'falco-modern-bpf.service':
Created symlink /etc/systemd/system/falco.service → /usr/lib/systemd/system/falco-modern-bpf.service.
Created symlink /etc/systemd/system/multi-user.target.wants/falco-modern-bpf.service → /usr/lib/systemd/system/falco-modern-bpf.service.
[POST-INSTALL] Start 'falco-modern-bpf.service':
Scanning processes...
Scanning candidates...
Scanning linux images...

Running kernel seems to be up-to-date.

Restarting services...

Service restarts being deferred:
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.

```
# Step 3 Verify

# Check which service was created
sudo systemctl status falco-modern-bpf

# Check Falco version
falco --version

# Verify it's detecting events
sudo journalctl -u falco-modern-bpf --no-pager | tail -20

```

oracle@control124:~$ # Check which service was created
sudo systemctl status falco-modern-bpf

# Check Falco version
falco --version

# Verify it's detecting events
sudo journalctl -u falco-modern-bpf --no-pager | tail -20
● falco-modern-bpf.service - Falco: Container Native Runtime Security with modern ebpf
     Loaded: loaded (/usr/lib/systemd/system/falco-modern-bpf.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-06-27 17:26:02 UTC; 1min 45s ago
       Docs: https://falco.org/docs/
   Main PID: 11167 (falco)
      Tasks: 17 (limit: 7012)
     Memory: 65.7M (peak: 66.2M)
        CPU: 2.426s
     CGroup: /system.slice/falco-modern-bpf.service
             └─11167 /usr/bin/falco -o engine.kind=modern_ebpf

Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: Loading rules from:
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]:    /etc/falco/falco_rules.yaml | schema validation: ok
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]:    /etc/falco/falco_rules.local.yaml | schema validation: none
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: The chosen syscall buffer dimension is: 8388608 bytes (8 MBs)
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: Starting health webserver with threadiness 4, listening on 0.0.0.0:8765
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: Loaded event sources: syscall
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: Enabled event sources: syscall
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: Opening 'syscall' source with modern BPF probe.
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: One ring buffer every '2' CPUs.
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: Trying to open the right engine!
Sat Jun 27 17:27:47 2026: Falco version: 0.44.1 (x86_64)
Sat Jun 27 17:27:47 2026: Falco initialized with configuration files:
Sat Jun 27 17:27:47 2026:    /etc/falco/config.d/engine-kind-falcoctl.yaml | schema validation: ok
Sat Jun 27 17:27:47 2026:    /etc/falco/config.d/falco.container_plugin.yaml | schema validation: ok
Sat Jun 27 17:27:47 2026:    /etc/falco/falco.yaml | schema validation: ok
Sat Jun 27 17:27:47 2026: System info: Linux version 6.8.0-124-generic (buildd@lcy02-amd64-019) (x86_64-linux-gnu-gcc-13 (Ubuntu 13.3.0-6ubuntu2~24.04.1) 13.3.0, GNU ld (GNU Binutils for Ubuntu) 2.42) #124-Ubuntu SMP PREEMPT_DYNAMIC Tue May 26 13:00:45 UTC 2026
Falco version: 0.44.1
Libs version:  0.25.4
Plugin API:    3.12.0
Engine:        0.62.0
Driver:
  API version:    10.0.0
  Schema version: 4.3.0
  Default driver: 10.2.0+driver
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: container: Enabled 'cri' container engine.
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: container: * enabled container runtime socket at '/run/containerd/containerd.sock'
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: container: * enabled container runtime socket at '/run/crio/crio.sock'
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: container: * enabled container runtime socket at '/run/k3s/containerd/containerd.sock'
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: container: * enabled container runtime socket at '/run/host-containerd/containerd.sock'
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: container: Enabled 'containerd' container engine.
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: container: * enabled container runtime socket at '/run/host-containerd/containerd.sock'
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: container: Enabled 'lxc' container engine.
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: container: Enabled 'libvirt_lxc' container engine.
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: container: Enabled 'bpm' container engine.
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: Loading rules from:
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]:    /etc/falco/falco_rules.yaml | schema validation: ok
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]:    /etc/falco/falco_rules.local.yaml | schema validation: none
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: The chosen syscall buffer dimension is: 8388608 bytes (8 MBs)
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: Starting health webserver with threadiness 4, listening on 0.0.0.0:8765
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: Loaded event sources: syscall
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: Enabled event sources: syscall
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: Opening 'syscall' source with modern BPF probe.
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: One ring buffer every '2' CPUs.
Jun 27 17:26:02 control124.home.aarisha.com falco[11167]: [libs]: Trying to open the right engine!
oracle@control124:~$

```
# T Verify App Apmor
_____________________

sudo aa-status

```
oracle@control124:~$ sudo aa-status
apparmor module is loaded.
104 profiles are loaded.
16 profiles are in enforce mode.
   /usr/bin/man
   /usr/lib/snapd/snap-confine
   /usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   cri-containerd.apparmor.d
   lsb_release
   man_filter
   man_groff
   nvidia_modprobe
   nvidia_modprobe//kmod
   plasmashell
   plasmashell//QtWebEngineProcess
   rsyslogd
   tcpdump
   ubuntu_pro_apt_news
   unix-chkpwd
   unprivileged_userns
0 profiles are in complain mode.
0 profiles are in prompt mode.
0 profiles are in kill mode.
88 profiles are in unconfined mode.
   1password
   Discord
   MongoDB Compass
   QtWebEngineProcess
   brave
   buildah
   busybox
   cam
   ch-checkns
   ch-run
   chrome
   crun
   devhelp
   element-desktop
   epiphany
   evolution
   firefox
   flatpak
   geary
   github-desktop
   goldendict
   ipa_verify
   kchmviewer
   keybase
   lc-compliance
   libcamerify
   linux-sandbox
   loupe
   lxc-attach
   lxc-create
   lxc-destroy
   lxc-execute
   lxc-stop
   lxc-unshare
   lxc-usernsexec
   mmdebstrap
   msedge
   nautilus
   notepadqq
   obsidian
   opam
   opera
   pageedit
   podman
   polypane
   privacybrowser
   qcam
   qmapshack
   qutebrowser
   rootlesskit
   rpm
   rssguard
   runc
   sbuild
   sbuild-abort
   sbuild-adduser
   sbuild-apt
   sbuild-checkpackages
   sbuild-clean
   sbuild-createchroot
   sbuild-destroychroot
   sbuild-distupgrade
   sbuild-hold
   sbuild-shell
   sbuild-unhold
   sbuild-update
   sbuild-upgrade
   scide
   signal-desktop
   slack
   slirp4netns
   steam
   stress-ng
   surfshark
   systemd-coredump
   thunderbird
   toybox
   trinity
   tup
   tuxedo-control-center
   userbindmount
   uwsgi-core
   vdens
   virtiofsd
   vivaldi-bin
   vpnns
   vscode
   wpcom
5 processes have profiles defined.
5 processes are in enforce mode.
   /usr/local/bin/etcd (4810) cri-containerd.apparmor.d
   /usr/local/bin/kube-controller-manager (4831) cri-containerd.apparmor.d
   /usr/local/bin/kube-scheduler (4849) cri-containerd.apparmor.d
   /usr/local/bin/kube-apiserver (4864) cri-containerd.apparmor.d
   /usr/sbin/rsyslogd (729) rsyslogd
0 processes are in complain mode.
0 processes are in prompt mode.
0 processes are in kill mode.
0 processes are unconfined but have a profile defined.
0 processes are in mixed mode.
oracle@control124:~$ 
```

# U Label Worker Nodes
______________________

kubectl label node worker125.home.aarisha.com node-role.kubernetes.io/worker=
kubectl label node worker126.home.aarisha.com node-role.kubernetes.io/worker=

```
oracle@control124:~$ k label node worker125.home.aarisha.com node-role.kubernetes.io/worker=
k label node worker126.home.aarisha.com node-role.kubernetes.io/worker=


node/worker125.home.aarisha.com labeled
node/worker126.home.aarisha.com labeled
oracle@control124:~$
```

# V Setup Alias K for kubectl
_____________________________

# Add to ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'complete -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc

# Test
k get nodes

``` 
oracle@control124:~$ # Add to ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'complete -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc

# Test
k get nodes
NAME                          STATUS   ROLES           AGE   VERSION
control124.home.aarisha.com   Ready    control-plane   13h   v1.34.9
worker125.home.aarisha.com    Ready    <none>          13h   v1.34.9
worker126.home.aarisha.com    Ready    <none>          13h   v1.34.9
oracle@control124:~$
```