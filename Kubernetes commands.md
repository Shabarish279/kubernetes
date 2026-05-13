# ☸️ Kubernetes Command Reference

> A comprehensive quick-reference for Kubernetes commands across all core and advanced topics.

---

## 📑 Table of Contents

- [Cluster Info & Context](#1-cluster-info--context)
- [Namespaces](#2-namespaces)
- [Pods](#3-pods)
- [Deployments](#4-deployments)
- [Services](#5-services)
- [ConfigMaps & Secrets](#6-configmaps--secrets)
- [Storage — PV & PVC](#7-storage--pv--pvc)
- [Apply & Manage Manifests](#8-apply--manage-manifests)
- [RBAC](#9-rbac)
- [Helm](#10-helm)
- [Ingress](#11-ingress)
- [StatefulSets](#12-statefulsets)
- [Networking & DNS](#13-networking--dns)
- [Jobs & CronJobs](#14-jobs--cronjobs)
- [Horizontal Pod Autoscaler (HPA)](#15-horizontal-pod-autoscaler-hpa)
- [Security — ServiceAccounts & PSA](#16-security--serviceaccounts--psa)
- [Resource Management](#17-resource-management)
- [etcd & Cluster Operations](#18-etcd--cluster-operations)
- [Observability & Troubleshooting](#19-observability--troubleshooting)

---

## 1. Cluster Info & Context

```bash
# Cluster info
kubectl cluster-info
kubectl version --short
kubectl api-resources
kubectl api-versions

# Nodes
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node <node-name>

# Contexts
kubectl config current-context
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl config set-context --current --namespace=<namespace>
kubectl config view
```

---

## 2. Namespaces

```bash
kubectl get namespaces
kubectl create namespace <name>
kubectl describe namespace <name>
kubectl delete namespace <name>

# Get all resources in a namespace
kubectl get all -n <namespace>

# Set default namespace for current context
kubectl config set-context --current --namespace=<name>
```

---

## 3. Pods

```bash
# Listing
kubectl get pods
kubectl get pods -A                                          # all namespaces
kubectl get pods -o wide                                     # with node/IP info
kubectl get pods -l app=<label>                              # filter by label
kubectl get pods --field-selector=status.phase!=Running -A   # non-running pods
kubectl get pods --field-selector spec.nodeName=<node> -A    # pods on a node
kubectl get pods -w                                          # watch live

# Inspect
kubectl describe pod <pod-name>
kubectl get pod <pod-name> -o yaml

# Logs
kubectl logs <pod-name>
kubectl logs <pod-name> --tail=100
kubectl logs <pod-name> --since=10m
kubectl logs <pod-name> -f                          # follow live
kubectl logs <pod-name> -c <container>              # specific container
kubectl logs <pod-name> --previous                  # last crashed container
kubectl logs -l app=<name> -f --all-containers      # all pods with label

# Execute & Copy
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -c <container> -- /bin/sh
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file

# Run a temporary debug pod
kubectl run tmp --image=busybox --rm -it -- sh
kubectl run tmp --image=curlimages/curl --rm -it -- curl http://<svc>

# Delete
kubectl delete pod <pod-name>
kubectl delete pod <pod-name> --grace-period=0 --force      # force delete stuck pod
```

---

## 4. Deployments

```bash
# Create & List
kubectl create deployment <name> --image=<image>
kubectl get deployments
kubectl get deployments -A
kubectl describe deployment <name>
kubectl get deployment <name> -o yaml

# Scale
kubectl scale deployment <name> --replicas=3

# Update
kubectl set image deployment/<name> <container>=<image>:<tag>
kubectl edit deployment <name>

# Rollouts
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>                       # rollback one version
kubectl rollout undo deployment/<name> --to-revision=2       # rollback to specific
kubectl rollout pause deployment/<name>
kubectl rollout resume deployment/<name>
kubectl rollout restart deployment/<name>

# Delete
kubectl delete deployment <name>
```

---

## 5. Services

```bash
# List & Inspect
kubectl get services
kubectl get services -A
kubectl describe service <name>
kubectl get endpoints <svc-name>                            # check pod mapping

# Create
kubectl expose deployment <name> --port=80 --type=ClusterIP
kubectl expose deployment <name> --port=80 --type=NodePort
kubectl expose deployment <name> --port=80 --type=LoadBalancer

# Port Forwarding
kubectl port-forward svc/<name> 8080:80
kubectl port-forward pod/<name> 8080:80

# Delete
kubectl delete service <name>
```

> **Service Types:**
> - `ClusterIP` — internal only (default)
> - `NodePort` — exposes on each node's IP at a static port
> - `LoadBalancer` — provisions external load balancer (cloud)
> - `ExternalName` — maps to a DNS name

---

## 6. ConfigMaps & Secrets

```bash
# ConfigMaps
kubectl get configmaps
kubectl describe configmap <name>
kubectl get configmap <name> -o yaml
kubectl create configmap <name> --from-literal=key=value
kubectl create configmap <name> --from-file=<file>
kubectl create configmap <name> --from-env-file=<.env-file>
kubectl delete configmap <name>

# Secrets
kubectl get secrets
kubectl describe secret <name>
kubectl get secret <name> -o yaml
kubectl create secret generic <name> --from-literal=key=value
kubectl create secret tls <name> --cert=tls.crt --key=tls.key
kubectl create secret docker-registry regcred \
  --docker-server=<url> \
  --docker-username=<user> \
  --docker-password=<pass>

# Decode a secret value
kubectl get secret <name> -o jsonpath='{.data.<key>}' | base64 -d
```

> ⚠️ Secrets are **base64-encoded, not encrypted** by default. Enable `EncryptionConfiguration` on the API server for encryption at rest.

---

## 7. Storage — PV & PVC

```bash
# PersistentVolumes
kubectl get pv
kubectl describe pv <name>
kubectl delete pv <name>

# PersistentVolumeClaims
kubectl get pvc
kubectl get pvc -A
kubectl describe pvc <name>
kubectl delete pvc <name>

# StorageClasses
kubectl get storageclass
kubectl describe storageclass <name>

# Check which pods are using a PVC
kubectl get pods -A -o json | jq '.items[] | select(.spec.volumes[]?.persistentVolumeClaim.claimName=="<pvc-name>") | .metadata.name'
```

> ⚠️ Deleting a StatefulSet or Deployment does **not** delete PVCs — clean them up manually.

---

## 8. Apply & Manage Manifests

```bash
# Apply
kubectl apply -f <file.yaml>
kubectl apply -f ./directory/          # all YAMLs in a dir
kubectl apply -k ./kustomization/      # kustomize

# Preview & Diff
kubectl apply -f <file.yaml> --dry-run=client
kubectl apply -f <file.yaml> --dry-run=server
kubectl diff -f <file.yaml>

# Delete
kubectl delete -f <file.yaml>
kubectl delete -f ./directory/

# Edit & Patch
kubectl edit deployment <name>
kubectl patch deployment <name> -p '{"spec":{"replicas":2}}'
kubectl patch deployment <name> --type=json \
  -p='[{"op":"replace","path":"/spec/replicas","value":3}]'

# Generate YAML without applying
kubectl create deployment <name> --image=<image> --dry-run=client -o yaml > deploy.yaml
kubectl expose deployment <name> --port=80 --dry-run=client -o yaml > svc.yaml
```

---

## 9. RBAC

```bash
# Roles & ClusterRoles
kubectl get roles -n <namespace>
kubectl get clusterroles
kubectl describe role <name> -n <namespace>
kubectl describe clusterrole <name>

# RoleBindings & ClusterRoleBindings
kubectl get rolebindings -n <namespace>
kubectl get clusterrolebindings
kubectl describe rolebinding <name> -n <namespace>

# Create
kubectl create role <name> --verb=get,list --resource=pods -n <namespace>
kubectl create clusterrole <name> --verb=get,list,watch --resource=pods
kubectl create rolebinding <name> --role=<role> --user=<user> -n <namespace>
kubectl create clusterrolebinding <name> --clusterrole=<role> --serviceaccount=<ns>:<sa>

# Check permissions
kubectl auth can-i create pods
kubectl auth can-i create pods --as=<user>
kubectl auth can-i --list --as=<user> -n <namespace>
kubectl auth can-i --list --as=system:serviceaccount:<ns>:<sa>
```

---

## 10. Helm

```bash
# Repos
helm repo add <name> <url>
helm repo update
helm repo list
helm repo remove <name>

# Search & Inspect
helm search repo <keyword>
helm search hub <keyword>
helm show values <chart>
helm show chart <chart>

# Install & Manage
helm install <release> <chart>
helm install <release> <chart> -f values.yaml
helm install <release> <chart> --set key=value
helm install <release> <chart> -n <namespace> --create-namespace
helm list -A
helm status <release>
helm get values <release>
helm get manifest <release>

# Upgrade & Rollback
helm upgrade <release> <chart> -f values.yaml
helm upgrade --install <release> <chart> -f values.yaml   # install if not exists
helm rollback <release> <revision>
helm history <release>

# Uninstall
helm uninstall <release>
helm uninstall <release> --keep-history

# Development
helm template <release> <chart> -f values.yaml            # render templates locally
helm lint ./mychart
helm package ./mychart
helm diff upgrade <release> <chart> -f values.yaml        # requires helm-diff plugin
```

> 💡 Use `--dry-run` with `helm install/upgrade` to simulate without applying.
> ⚠️ `helm uninstall` removes K8s resources but **not PVCs**.

---

## 11. Ingress

```bash
# Ingress Controller (nginx via Helm)
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx                           # get external IP

# Ingress Resources
kubectl get ingress -A
kubectl describe ingress <name>
kubectl get ingressclass
kubectl apply -f ingress.yaml
kubectl delete ingress <name>

# TLS secret for Ingress
kubectl create secret tls <name> --cert=tls.crt --key=tls.key
```

**Common nginx annotations:**

| Annotation | Value |
|---|---|
| `nginx.ingress.kubernetes.io/ssl-redirect` | `"true"` |
| `nginx.ingress.kubernetes.io/rewrite-target` | `/` |
| `nginx.ingress.kubernetes.io/proxy-read-timeout` | `"60"` |
| `nginx.ingress.kubernetes.io/enable-cors` | `"true"` |
| `nginx.ingress.kubernetes.io/limit-rps` | `"10"` |
| `nginx.ingress.kubernetes.io/auth-type` | `basic` |

---

## 12. StatefulSets

```bash
# List & Inspect
kubectl get statefulsets -A
kubectl describe statefulset <name>
kubectl get statefulset <name> -o yaml

# Scale & Update
kubectl scale statefulset <name> --replicas=3
kubectl rollout status statefulset/<name>
kubectl rollout restart statefulset/<name>
kubectl rollout history statefulset/<name>
kubectl rollout undo statefulset/<name>

# Debug
kubectl exec -it <name>-0 -- /bin/bash                    # exec into first pod
kubectl logs <name>-0
kubectl get pvc | grep <statefulset-name>                 # list associated PVCs

# Delete
kubectl delete statefulset <name>
kubectl delete statefulset <name> --cascade=orphan        # keep pods running
```

> 💡 Pod DNS: `<pod-name>.<headless-svc>.<namespace>.svc.cluster.local`
> ⚠️ Deleting a StatefulSet does NOT remove PVCs — delete them separately.

---

## 13. Networking & DNS

```bash
# DNS resolution
kubectl exec -it <pod> -- nslookup <service-name>
kubectl exec -it <pod> -- nslookup <svc>.<namespace>.svc.cluster.local
kubectl exec -it <pod> -- cat /etc/resolv.conf

# CoreDNS
kubectl get pods -n kube-system | grep coredns
kubectl get configmap coredns -n kube-system -o yaml
kubectl logs -n kube-system -l k8s-app=kube-dns

# Network Policies
kubectl get networkpolicies -A
kubectl describe networkpolicy <name>
kubectl apply -f netpol.yaml
kubectl delete networkpolicy <name>

# Connectivity testing
kubectl exec -it <pod> -- curl http://<svc>.<namespace>
kubectl exec -it <pod> -- wget -qO- http://<svc>
kubectl run test --image=curlimages/curl --rm -it -- curl http://<svc>

# Service endpoints
kubectl get endpoints <svc-name>
kubectl get pods -l <label-selector>                      # verify selector matches

# kube-proxy
kubectl logs -n kube-system -l k8s-app=kube-proxy
```

**DNS patterns:**

| Resource | DNS Format |
|---|---|
| Service | `<svc>.<namespace>.svc.cluster.local` |
| Pod | `<pod-ip-dashes>.<namespace>.pod.cluster.local` |
| StatefulSet Pod | `<pod>.<headless-svc>.<namespace>.svc.cluster.local` |

---

## 14. Jobs & CronJobs

```bash
# Jobs
kubectl get jobs -A
kubectl describe job <name>
kubectl logs job/<name>
kubectl logs -l job-name=<name> -f
kubectl create job <name> --image=<image> -- <command>
kubectl delete job <name>
kubectl delete jobs --field-selector status.successful=1   # clean completed jobs

# Create job from a CronJob (manual trigger)
kubectl create job <name> --from=cronjob/<cronjob-name>

# CronJobs
kubectl get cronjobs -A
kubectl describe cronjob <name>
kubectl get cronjob <name> -o yaml

# Suspend / Resume
kubectl patch cronjob <name> -p '{"spec":{"suspend":true}}'
kubectl patch cronjob <name> -p '{"spec":{"suspend":false}}'

# Delete
kubectl delete cronjob <name>
```

> 💡 Add `.spec.ttlSecondsAfterFinished` to Jobs for automatic cleanup.
> ⚠️ CronJob schedule is in **UTC**. Use `.spec.timeZone` (K8s 1.25+) for local time.

---

## 15. Horizontal Pod Autoscaler (HPA)

```bash
# Setup — metrics-server required
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl top nodes                                          # verify metrics-server works

# HPA
kubectl get hpa -A
kubectl describe hpa <name>
kubectl get hpa <name> -w                                 # watch scaling live
kubectl autoscale deployment <name> --cpu-percent=50 --min=2 --max=10
kubectl delete hpa <name>

# Current resource usage
kubectl top pods
kubectl top pods -A --sort-by=cpu
kubectl top pods -A --sort-by=memory
kubectl top nodes
```

> 💡 HPA `v2` (autoscaling/v2) supports memory and custom metrics — prefer it over v1.
> 💡 VPA (Vertical Pod Autoscaler) adjusts CPU/memory requests and can complement HPA in Recommendation mode.

---

## 16. Security — ServiceAccounts & PSA

```bash
# ServiceAccounts
kubectl get serviceaccounts -A
kubectl describe serviceaccount <name>
kubectl create serviceaccount <name>
kubectl create token <sa-name>                            # generate SA token (K8s 1.24+)
kubectl delete serviceaccount <name>

# Bind SA to a role
kubectl create rolebinding <name> --role=<role> --serviceaccount=<ns>:<sa> -n <ns>
kubectl create clusterrolebinding <name> --clusterrole=<role> --serviceaccount=<ns>:<sa>

# Pod Security Admission (K8s 1.23+)
# Profiles: privileged | baseline | restricted
# Modes: enforce | warn | audit

kubectl label ns <name> pod-security.kubernetes.io/enforce=restricted
kubectl label ns <name> pod-security.kubernetes.io/warn=baseline
kubectl label ns <name> pod-security.kubernetes.io/audit=restricted
kubectl get ns <name> -o yaml | grep pod-security

# Check permissions for a SA
kubectl auth can-i --list --as=system:serviceaccount:<ns>:<sa>
```

> ⚠️ Avoid using the `default` ServiceAccount — create dedicated SAs with least-privilege RBAC.

---

## 17. Resource Management

```bash
# View usage
kubectl top pods -A --sort-by=cpu
kubectl top pods -A --sort-by=memory
kubectl top nodes
kubectl describe node <name> | grep -A10 Allocatable
kubectl describe node <name> | grep -A10 "Allocated resources"

# Pod resource details
kubectl describe pod <name> | grep -A5 Limits
kubectl describe pod <name> | grep -A5 Requests

# LimitRange (defaults per pod/container in a namespace)
kubectl get limitrange -A
kubectl describe limitrange <name> -n <namespace>

# ResourceQuota (caps per namespace)
kubectl get resourcequota -A
kubectl describe resourcequota <name> -n <namespace>

# PodDisruptionBudget
kubectl get pdb -A
kubectl describe pdb <name>
```

---

## 18. etcd & Cluster Operations

```bash
# etcd health (run inside etcd pod)
kubectl exec -it etcd-<node> -n kube-system -- \
  etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt \
          --cert=/etc/kubernetes/pki/etcd/server.crt \
          --key=/etc/kubernetes/pki/etcd/server.key \
          endpoint health

kubectl exec -it etcd-<node> -n kube-system -- etcdctl member list

# etcd Backup & Restore
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot.db
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore

# Cluster upgrade (kubeadm)
kubeadm upgrade plan
kubeadm upgrade apply v1.XX.X

# Node maintenance
kubectl cordon <node>                                     # stop scheduling new pods
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <node>                                   # re-enable scheduling

# Post-upgrade kubelet
apt-get install -y kubelet=1.XX.X-* kubectl=1.XX.X-*
systemctl daemon-reload && systemctl restart kubelet

# Verify
kubectl get nodes
kubectl version --short
```

> ⚠️ Always backup etcd before upgrades. Store snapshots off-cluster.
> ⚠️ Upgrade **one minor version at a time** — skipping versions is unsupported.

---

## 19. Observability & Troubleshooting

```bash
# Events
kubectl get events -n <namespace> --sort-by=.lastTimestamp
kubectl get events -A --sort-by=.lastTimestamp
kubectl get events -A -w                                  # watch live

# Pod diagnostics
kubectl describe pod <name>                               # check Events section
kubectl get pod <name> -o yaml | grep -A5 status
kubectl get pods -A --field-selector=status.phase!=Running

# Node diagnostics
kubectl describe node <name> | grep -A10 Conditions
kubectl describe node <name> | grep -A10 "Allocated resources"

# Resource hogs
kubectl top pods -A --sort-by=cpu
kubectl top pods -A --sort-by=memory

# Check selector → endpoints mapping
kubectl get endpoints <svc>
kubectl get pods -l <label>=<value>

# Raw metrics endpoint
kubectl get --raw /metrics | head -50

# Get all image versions running in cluster
kubectl get pods -A -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}' | sort -u

# Find pods without resource limits
kubectl get pods -o json | jq '.items[] | select(.spec.containers[].resources.limits == null) | .metadata.name'

# Count pods per node
kubectl get pods -A -o wide | awk '{print $8}' | sort | uniq -c | sort -rn

# Component status
kubectl get componentstatuses
kubectl get pods -n kube-system

# Useful aliases to add to ~/.bashrc or ~/.zshrc
alias k='kubectl'
alias kgp='kubectl get pods -A'
alias kgs='kubectl get svc -A'
alias kgn='kubectl get nodes'
alias kdp='kubectl describe pod'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'
alias kns='kubectl config set-context --current --namespace'
```

---

## 🧰 Handy One-liners

```bash
# Delete all completed/failed pods in a namespace
kubectl delete pods --field-selector=status.phase=Succeeded -n <namespace>
kubectl delete pods --field-selector=status.phase=Failed -n <namespace>

# Restart all deployments in a namespace
kubectl rollout restart deployment -n <namespace>

# Get all resource types in a namespace
kubectl api-resources --verbs=list --namespaced -o name | \
  xargs -n1 kubectl get -n <namespace> --ignore-not-found 2>/dev/null

# Watch pod restarts live
kubectl get pods -A -w | grep -v "0   0"

# Force delete a namespace stuck in Terminating
kubectl get namespace <name> -o json | \
  jq '.spec.finalizers = []' | \
  kubectl replace --raw "/api/v1/namespaces/<name>/finalize" -f -

# Base64 encode a secret value
echo -n "mypassword" | base64

# Port-forward multiple services at once (run in background)
kubectl port-forward svc/app 8080:80 &
kubectl port-forward svc/db 5432:5432 &
```

---

## 📌 Quick Flags Reference

| Flag | Meaning |
|---|---|
| `-n <namespace>` | Target a specific namespace |
| `-A` | All namespaces |
| `-o wide` | Extra columns (node, IP) |
| `-o yaml` | Full YAML output |
| `-o json` | Full JSON output |
| `-o jsonpath='...'` | Extract a specific field |
| `-l key=value` | Filter by label |
| `--field-selector` | Filter by field (status, node, etc.) |
| `-w` | Watch for changes live |
| `--dry-run=client` | Preview without applying |
| `--force --grace-period=0` | Force delete immediately |
| `--all-namespaces` | Same as `-A` |
| `--show-labels` | Display labels in output |
