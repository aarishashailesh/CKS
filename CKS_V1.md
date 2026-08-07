# Q 1

`Label Namespace`
kubectl label ns payments  project=myproject

`Default Deny Policy`
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress-egress
  namespace: payments
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

`Allow Ingress and Egress from namespace and pod with matching labels only`
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payments-network-policy
  namespace: payments
spec:
  podSelector:
    matchLabels:
      app: api-server
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend 
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend

# Q 2 remove the cluster-admin binding, then create a Role and RoleBinding granting ci-bot only the ability to list and get Pods and read Pod logs in the ci-cd namespace. Finally, verify that ci-bot cannot list Secrets.

kubectl delete clusterrolebinding ci-bot-admin -n  ci-cd

kubectl create role ci-bot-role -n ci-cd --verb=get --verb=list --verb=watch --resource=pods

kubectl create rolebinding ci-bot-admin -n  ci-cd --role=ci-bot-role --serviceaccount=ci-cd:ci-bot

kubectl auth can-i list secrets -n ci-cd --as=system:serviceaccount:ci-cd:ci-bot

# Q 3 
    (1) Patch the default ServiceAccount in the prod namespace to disable auto-mounting. 
    (2) Create a new pod manifest for web-app that explicitly sets automountServiceAccountToken: false.`
    (3) Verify no token is mounted inside the running pod

1) 
kubectl get sa -na prod
kubectl edit sa sa-name -na prod

Add this

apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-name
automountServiceAccountToken: false
...


2) 
kubectl get pod web-app -n prod -o yaml > web-app.yaml

vi web-app.yaml

Add this

 apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  serviceAccountName: sa-name
  automountServiceAccountToken: false

kubectl replace -f web-app.yaml --force

3) 
kubectl get pod web-app -o yaml | grep mount -c 5


# Q 4 
1) Create an AppArmor profile named deny-write 
2) Load it on a node - specific 192.168.1.125
3) Deploy a pod that enforces it on the node
4) Verify running pod and write inside the container is blocked.

Ref:
https://kubernetes.io/docs/tutorials/security/apparmor/#example


1) 
2) 

sudo apparmor_parser -q <<EOF
#include <tunables/global>

profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>

  file,

  # Deny all file writes.
  deny /** w,
}
EOF'

3) 

kubectl get nodes --show-labels

kubernetes.io/hostname=worker125.home.aarisha.com


https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/security/hello-apparmor.yaml

apiVersion: v1
kind: Pod
metadata:
  name: hello-apparmor
spec:
  securityContext:
    appArmorProfile:
      type: Localhost
      localhostProfile: k8s-apparmor-example-deny-write
  containers:
  - name: hello
    image: busybox:1.28
    command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
  nodeSelector:
    kubernetes.io/hostname: worker125.home.aarisha.com

kubectl create -f 

4) 

You can verify that the container is actually running with that profile by checking /proc/1/attr/current:

kubectl exec hello-apparmor -- cat /proc/1/attr/current
The output should be:

k8s-apparmor-example-deny-write (enforce)

Finally, you can see what happens if you violate the profile by writing to a file:

kubectl exec hello-apparmor -- touch /tmp/test
touch: /tmp/test: Permission denied
error: error executing remote command: command terminated with non-zero exit code: Err

# Q 5
1) Apply the RuntimeDefault seccomp profile to restrict the container to only the syscalls needed by the container runtime. 
2) Verify the profile is active using /proc inside the pod.


Link:
https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/security/seccomp/ga/default-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: default-pod
  labels:
    app: default-pod
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: test-container
    image: hashicorp/http-echo:1.0
    args:
    - "-text=just made some more syscalls!"
    securityContext:
      allowPrivilegeEscalation: false


kubectl exec default-pod -- cat /proc/1/attr/current

# Q 6 Recreate pod with the following security constraints
 1) Container must run as UID 1000. 
 2) Root filesystem must be read-only. 
 3) Privilege escalation must be disabled. 
 4) All Linux capabilities must be dropped. 
 5) Mount an emptyDir at /tmp so the app can still write temporary files.


Most of these are available while searching for  **runAsUser**

https://kubernetes.io/docs/tasks/configure-pod-container/security-context/


apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /tmp
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL

Verify
kubectl exec security-context-demo -- capsh --print

# Q 7 Your cluster has OPA Gatekeeper v3.23.0 installed
  1) Write a ConstraintTemplate named K8sBlockPrivileged 
  2) A Constraint that enforces it cluster-wide. Any pod that sets securityContext **privileged: true** must be rejected at admission time. 
  3) Verify by attempting to create a privileged pod (expect rejection) and a non-privileged pod (expect success).

`It is expected in the exam that code for the ConstraintTemplate will be provided`

---
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8sblockprivileged
spec:
  crd:
    spec:
      names:
        kind: K8sBlockPrivileged
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sblockprivileged

        # Helper rule to gather all containers in the Pod spec
        get_container[container] {
          container := input.review.object.spec.containers[_]
        }
        get_container[container] {
          container := input.review.object.spec.initContainers[_]
        }
        get_container[container] {
          container := input.review.object.spec.ephemeralContainers[_]
        }

        # Main violation rule
        deny[{"msg": msg}] {
          # Only evaluate if the resource being created/updated is a Pod
          input.review.object.kind == "Pod"
          
          # Find a container using our helper rule
          get_container[container]
          
          # Check if the boolean field is explicitly set to true
          container.securityContext.privileged == true
          
          msg := sprintf("Privileged container '%v' is not allowed. 'securityContext.privileged' must be omitted or set to false.", [container.name])
        }

---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sBlockPrivileged
metadata:
  name: constraint-k8sblockprivileged
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  
# Q 8 gVisor (runsc) is installed on worker126 (192.168.1.126). 

1) Create a RuntimeClass named sandboxed that uses the runsc handler. 
2) Deploy a pod named untrusted-app in namespace sandbox that uses this RuntimeClass. 
3) Verify the pod runs on worker126 and that it is using the gVisor runtime.

Search for k8.io gvisor

https://kubernetes.io/docs/concepts/containers/runtime-class/



1) Edit /etc/containerd/config.toml

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.${HANDLER_NAME}]

**Here is content for runc**

[plugins."io.containerd.cri.v1.runtime".containerd.runtimes.runc]
  runtime_type = "io.containerd.runc.v2"

[plugins."io.containerd.cri.v1.runtime".containerd.runtimes.runc.options]
  SystemdCgroup = true

**Using above, create entry for runsc**

[plugins."io.containerd.cri.v1.runtime".containerd.runtimes.runcs]
  runtime_type = "io.containerd.runsc.v2"

[plugins."io.containerd.cri.v1.runtime".containerd.runtimes.runsc.options]
  SystemdCgroup = true

systemctl stop containerd
systemctl start containerd

2) File - rc-sandboxed.yaml

# RuntimeClass is defined in the node.k8s.io API group
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  # The name the RuntimeClass will be referenced by.
  # RuntimeClass is a non-namespaced resource.
  name: sandboxed 
# The name of the corresponding CRI configuration
spec:
  handler: runsc

**The spec: was not in the URL above**

k create ns sanbdox
k create -f  rc-sandboxed.yaml -n sandbox 

3) Get label of the node worker worker126

k get node --show-labels


4) Create a pod using runsc and a node selector of worker126

k run mypod --image=nginx $do > my_pod.yaml

vi my_pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  runtimeClassName: sandboxed

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mypod
  name: mypod
spec:
  runtimeClassName: sandboxed
  containers:
  - image: nginx
    name: mypod
    resources: {}
  nodeSelector:
    k8.io.label: worker126
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

5) Verify pod is running on worker126

k get pod mypod -o wide

k get pod mypod -o yaml | grep runsc

k get pod mypod -o yaml | grep sandboxed


# 9 A team wants to deploy an old nginx image (nginx:1.19). Before allowing it, you must scan it for vulnerabilities using Trivy. 

1) Scan nginx:1.19 and output results to /tmp/nginx-scan.txt. 
2) Identify any CRITICAL severity CVEs. 
3) The team should use nginx:stable-alpine instead — scan it and compare. 
4) Document which image is safe to deploy.

1) 
trivy image  nginx:1.19 > /tmp/nginx-scan.txt

2) 
trivy image  nginx:1.19 | grep -i critical -C 5

3) 

trivy image  nginx:stable-alpine > /tmp/nginx-stable-scan.txt

4) 

trivy image  nginx:1.19 |  grep -i critical -C 5
trivy image  nginx:stable-alpine |  grep -i critical -C 5

`Less critical findings for nginx:stable-alpine`

# 10 The API server on control124 is not currently configured with audit logging. Configure an audit policy that: 
1) Logs all access to Secrets at the Metadata level. 
2) Logs Pod create/delete events at the RequestResponse level. 
3) Ignores all other events (None). 
4) Write the policy to /etc/kubernetes/audit-policy.yaml 
5) Update the API server manifest to enable the policy. 
6) Verify logs are written to /var/log/kubernetes/audit.log.


1) Search for audit ak kubernetes.io
through
4) 

https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/audit/audit-policy.yaml

`Filename: /etc/kubernetes/audit-policy.yaml`

apiVersion: audit.k8s.io/v1 # This is required.
kind: Policy
# Don't generate audit events for all requests in RequestReceived stage.
omitStages:
  - "RequestReceived"
rules:
  # Log secret changes in all other namespaces at the Metadata level.
  - level: Metadata
    resources:
    - group: "" # core API group
      resources: ["secrets"]

 # Log pod changes at RequestResponse level
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["pods/create", "pods/delete"]

  # Don't log any event
  - level: None
    resources:
    - group: ""
      resources: ["Event"]

**Save above yaml to /etc/kubernetes/audit-policy.yaml**

5) Update /etc/kubernetes/manifests/kube-api-server.yaml. First take a backup.
  1)   
  - --audit-policy-file=/etc/kubernetes/audit-policy.yaml
  - --audit-log-path=/var/log/kubernetes/audit.log

  2) Update Mount Vol and Volume Path too

  volumeMounts:
  - mountPath: /etc/kubernetes/audit-policy.yaml
    name: audit
    readOnly: true
  - mountPath: /var/log/kubernetes/
    name: audit-log
    readOnly: false

  volumes:
- name: audit
  hostPath:
    path: /etc/kubernetes/audit-policy.yaml
    type: File

- name: audit-log
  hostPath:
    path: /var/log/kubernetes/
    type: DirectoryOrCreate

**Save file etc/kubernetes/manifests/kube-api-server.yaml to restart kube-api-sever**

6) tail -f /var/log/kubernetes/audit.log

  Create test scenario
  k run pod1 --image=nginx
  k delete pod1
  k get secret -A

# Q 11

Falco 0.44.1 is running on your cluster using the modern_ebpf driver. 
1) Verify Falco is running and healthy. 
2) Identify the built-in rule that detects shell spawning inside a container. 
3) Trigger the rule by exec-ing into a running container and spawning a shell. 
4) Confirm the alert appears in Falco's output. 
5) Write a custom Falco rule that alerts when the file /etc/passwd is read inside any container, with priority WARNING

1) 
systemctl status falco

2) 
cat /etc/falco/falco_rules.yaml | grep  "shell" -A 10 
cat /etc/falco/falco_rules.yaml | grep  "spawn" -A 10 

3) 
k exec -it test-pod -- sh

4) 
journalctl -u falco _TRANSPORT=stdout | grep -i "Terminal shell"



Here is exactly what each part of that command does to find your alert:

`journalctl`: The core Linux utility used to query and view logs generated by systemd.

`-u falco`: Filters the logs to show only messages from the Falco service.

`_TRANSPORT=stdout`: Filters the logs further to only show messages Falco sent to standard output (stdout), hiding internal systemd process management messages (like "Started Falco Service").| 

`grep -i "Terminal shell"`: Pipes that clean stream into grep to pull out only the specific lines containing the phrase "Terminal shell" (ignoring case).

5) 
Copy matching rule snippet from /etc/falco/falco_rules.yaml
Edit /etc/falco/falco_rules.local.yaml and add that rule
Change priority to WARNING


# Q 12 A pod named legacy-app mounts a database password via an environment variable sourced directly from a Secret. This is insecure because env vars are visible in process listings, crash dumps, and log output. 
1) Create a Secret named db-creds in namespace apps with key password=S3cr3tPwd. 
2) Deploy a pod that mounts the Secret as a file (volume mount) rather than an env var. 
3) Verify the file is accessible at /etc/secrets/password inside the pod. 
4) Verify the password does NOT appear in the pod environment (env | grep password).


1) 
k create secret db-creds --from-literal=password=S3cr3tPwd -n app

2) 
k run sec-mount-pod --image=nginx -n app $do > sec-mount-pod.yaml

vi sec-mount-pod.yaml `Add secret as mount /etc/secrets/password`

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: sec-mount-pod
  name: sec-mount-pod
  namespace: app
spec:
  containers:
  - image: nginx
    name: sec-mount-pod
    volumeMounts:
      # name must match the volume name below
      - name: secret-volume
        mountPath: /etc/secrets
        readOnly: true
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
  # The secret data is exposed to Containers in the Pod through a Volume.
volumes:
  - name: secret-volume
    secret:
      secretName: db-creds

k apply -f sec-mount-pod.yaml
nsloo
3)  

k exec sec-mount-pod -n app -it -- bash
ls  -l /etc/secrets/password
cat /etc/secrets/password

4) 
k exec sec-mount-pod -n app -it -- bash

env | grep -i password



# Q 13 Run the kube-bench CIS benchmark against the control plane node (control124) and remediate the following common findings: 
1) Anonymous authentication must be disabled on the API server. 
2) The AlwaysAdmit admission plugin must not be enabled. 
3) The NodeRestriction admission plugin must be enabled. After making changes, re-run kube-bench to confirm the checks pass.


`Run kubebench and grep the errors on control124`


kube-bench run --targets master > kubebench_results.txt

grep -i -E "anonymous|alwaysadmin|noderestriction" kubebench_results.txt -C 5

Edit kube-apiserver.yaml file 
  Change
    Anonymous authentication --> false
    AlwaysAdmit --> true
    NodeRestriction --> Enabled

The Rules have the descriptions to change these values.

Once the changes are saved the kube-apiserver is restarted.

`Verification`

kube-bench run --targets master > kubebench_results.txt

grep -i -E "anonymous|alwaysadmin|noderestriction" kubebench_results.txt -C 5



# Q 14 A container named suspicious in namespace incident is exhibiting unusual behaviour. Without restarting or killing it: 
1) Find which node it is running on and its container ID. 
2) List all processes running inside the container. 
3) Check what files the main process has open. 
4) Examine the container's filesystem for unexpected binaries in /tmp. 
5) Capture the container's current state to /tmp/forensics-report.txt.

1)  
k get pod suspicious -n incident -o wide

ssh to worker
crictl ps | grep suspicious

This will show the container ID for the pod

ps  aux | grep <containerID>

This will give main process id

2) 
k exec -it suspicious -n incident -- bash
cat /proc

3) 

lsof -p pid

4)
k exec -it suspicious -n incident -- bash
ls -l /tmp

5) 

k get pod suspicious -n incident -o yaml | grep state -A 3 > /tmp/forensics-report.txt


