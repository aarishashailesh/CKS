# Q 1

▶  TASK

You are working on a multi-tenant cluster. A namespace called payments has been created. A pod named api-server is running in the payments namespace and should only be reachable from pods in the same namespace that carry the label role=frontend. All other ingress and all egress traffic must be blocked.

# Q 2

▶  TASK

A developer has created a ServiceAccount named ci-bot in namespace ci-cd. The account currently has cluster-admin via a ClusterRoleBinding — a serious security risk. Your task: remove the cluster-admin binding, then create a Role and RoleBinding granting ci-bot only the ability to list and get Pods and read Pod logs in the ci-cd namespace. Finally, verify that ci-bot cannot list Secrets.

# Q 3

▶  TASK

An application pod named web-app in namespace prod does not need to call the Kubernetes API. However, it is currently mounting the default ServiceAccount token, which exposes an unnecessary attack surface. 
1) Patch the default ServiceAccount in the prod namespace to disable auto-mounting. 
2) Create a new pod manifest for web-app that explicitly sets automountServiceAccountToken: false. 
3) Verify no token is mounted inside the running pod.

# Q 4

▶  TASK

On worker125 (192.168.1.125), an AppArmor profile named deny-write has been requested. Create this profile, load it on the node, then deploy a pod that enforces it. The profile must deny all file-write operations. Verify the pod is running with the profile enforced, and that a write inside the container is blocked.

# Q 5

▶  TASK

A pod named secure-app in namespace hardening is running without any seccomp profile, leaving all syscalls available. Apply the RuntimeDefault seccomp profile to restrict the container to only the syscalls needed by the container runtime. Verify the profile is active using /proc inside the pod.


# Q 6

▶  TASK

A pod named insecure-app is running with root privileges and a writable filesystem. Recreate pod insecure-app with: 

1) Container must run as UID 1000. 
2) Root filesystem must be read-only. 
3) Privilege escalation must be disabled. 
4) All Linux capabilities must be dropped. Mount an emptyDir at /tmp so the app can still write temporary files.

# Q 7

▶  TASK

Your cluster has OPA Gatekeeper v3.23.0 installed. Write a ConstraintTemplate named K8sBlockPrivileged and a Constraint that enforces it cluster-wide. Any pod that sets securityContext.privileged: true must be rejected at admission time. Verify by attempting to create a privileged pod (expect rejection) and a non-privileged pod (expect success).


# Q 8

▶  TASK

gVisor (runsc) is installed on worker126.

1) Create a RuntimeClass named sandboxed that uses the runsc handler. 
2) Deploy a pod named untrusted-app in namespace sandbox that uses this RuntimeClass. 
3) Verify the pod runs on worker126 and that it is using the gVisor runtime.


```
ON WORKER126 — add runsc stanza to containerd config
CRITICAL: use io.containerd.cri.v1.runtime (containerd 2.x), NOT grpc
```

# Q 9

▶  TASK

A team wants to deploy an old nginx image (nginx:1.19). Before allowing it, you must scan it for vulnerabilities using Trivy. 
1) Scan nginx:1.19 and output results to /tmp/nginx-scan.txt. 
2) Identify any CRITICAL severity CVEs. 
3) The team should use nginx:stable-alpine instead — scan it and compare. 
4) Document which image is safe to deploy.

# Q 10

▶  TASK

The API server on control124 is not currently configured with audit logging. Configure an audit policy that: 
1) Logs all access to Secrets at the Metadata level. 
2) Logs Pod create/delete events at the RequestResponse level. 
3) Ignores all other events (None). 
4) Write the policy to /etc/kubernetes/audit-policy.yaml and update the API server manifest to enable it. 
5) Verify logs are written to /var/log/kubernetes/audit.log.


# Q 11

▶  TASK

Falco 0.44.1 is running on your cluster using the modern_ebpf driver. 
1) Verify Falco is healthy. 
2) Find the built-in shell-in-container rule. 
3) Trigger it by exec-ing a shell. 
4) Confirm the alert. 
5) Write a custom rule: alert when /etc/passwd is read in any container at WARNING priority.

# Q 12

▶  TASK

A pod named legacy-app mounts a database password via an environment variable sourced directly from a Secret. This is insecure because env vars are visible in process listings, crash dumps, and log output. 
1) Create a Secret named db-creds in namespace apps with key password=S3cr3tPwd. 
2) Deploy a pod that mounts the Secret as a file (volume mount) rather than an env var. 
3) Verify the file is accessible at /etc/secrets/password inside the pod. 
4) Verify the password does NOT appear in the pod environment (env | grep password).


# Q 13

▶  TASK

Run the kube-bench CIS benchmark against the control plane node (control124) and remediate the following common findings: 
1) Anonymous authentication must be disabled on the API server. 
2) The AlwaysAdmit admission plugin must not be enabled. 
3) The NodeRestriction admission plugin must be enabled. After making changes, re-run kube-bench to confirm the checks pass.

# Q 14

▶  TASK

A container named suspicious in namespace incident is exhibiting unusual behaviour. Without restarting or killing it: 
1) Find which node it is running on and its container ID. 
2) List all processes running inside the container. 
3) Check what files the main process has open. 
4) Examine the container's filesystem for unexpected binaries in /tmp. 
5) Capture the container's current state to /tmp/forensics-report.txt.

