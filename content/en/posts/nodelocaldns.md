---
title: "Improving DNS Reliability with NodeLocalDNS in Kubernetes"
description: "How to fix intermittent DNS resolution issues in local Kubernetes clusters by deploying NodeLocalDNS as a caching layer on each node. Includes full manifest setup and configuration steps."
date: 2025-04-08
tags: ["kubernetes", "dns", "nodelocaldns", "coredns", "devops", "ubuntu", "networking"]
categories: ["DevOps"]
seo_keywords: ["NodeLocalDNS setup", "kubernetes dns troubleshooting", "kubernetes node-local dns caching", "ubuntu kubernetes dns fix", "coredns node caching"]
weight: 1
draft: false
---


---

# Improving DNS Reliability with NodeLocalDNS in Kubernetes

Occasionally in local Kubernetes clusters (specifically on Ubuntu 24.04), I noticed **intermittent DNS resolution failures**—pods would fail to resolve names even though the host itself could do so just fine.

The fix? Adding `NodeLocalDNS` as a **caching DNS layer** on each node. This setup reduces reliance on external DNS lookups by caching frequently queried results locally. It’s especially useful in lab environments, homelabs, or clusters where DNS latency or flakiness causes issues with microservice resolution.

This post walks through deploying NodeLocalDNS as a `DaemonSet` using an official manifest template and customizing it with your cluster’s values.

---

## Symptoms

- Sporadic DNS failures inside pods
- Host-level resolution works, but `nslookup` or app service discovery fails inside Kubernetes
- Resolved by restarting `CoreDNS` but reappears randomly

---

## Step 1: Save the NodeLocalDNS Manifest

Save the following manifest template as `nodelocaldns.yaml`. This includes the `ServiceAccount`, `ConfigMap`, `DaemonSet`, and the headless service for Prometheus metrics.

> 🛑 **Note:** The manifest contains placeholder variables (`__PILLAR__LOCAL__DNS__`, etc.) which you’ll replace in the next steps.

<details>
<summary>Click to view full NodeLocalDNS manifest</summary>

```yaml
# Copyright 2018 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

apiVersion: v1
kind: ServiceAccount
metadata:
  name: node-local-dns
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
---
apiVersion: v1
kind: Service
metadata:
  name: kube-dns-upstream
  namespace: kube-system
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/name: "KubeDNSUpstream"
spec:
  ports:
  - name: dns
    port: 53
    protocol: UDP
    targetPort: 53
  - name: dns-tcp
    port: 53
    protocol: TCP
    targetPort: 53
  selector:
    k8s-app: kube-dns
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: node-local-dns
  namespace: kube-system
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
data:
  Corefile: |
    __PILLAR__DNS__DOMAIN__:53 {
        errors
        cache {
                success 9984 30
                denial 9984 5
        }
        reload
        loop
        bind __PILLAR__LOCAL__DNS__ __PILLAR__DNS__SERVER__
        forward . __PILLAR__CLUSTER__DNS__ {
                force_tcp
        }
        prometheus :9253
        health __PILLAR__LOCAL__DNS__:8080
        }
    in-addr.arpa:53 {
        errors
        cache 30
        reload
        loop
        bind __PILLAR__LOCAL__DNS__ __PILLAR__DNS__SERVER__
        forward . __PILLAR__CLUSTER__DNS__ {
                force_tcp
        }
        prometheus :9253
        }
    ip6.arpa:53 {
        errors
        cache 30
        reload
        loop
        bind __PILLAR__LOCAL__DNS__ __PILLAR__DNS__SERVER__
        forward . __PILLAR__CLUSTER__DNS__ {
                force_tcp
        }
        prometheus :9253
        }
    .:53 {
        errors
        cache 30
        reload
        loop
        bind __PILLAR__LOCAL__DNS__ __PILLAR__DNS__SERVER__
        forward . __PILLAR__UPSTREAM__SERVERS__
        prometheus :9253
        }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-local-dns
  namespace: kube-system
  labels:
    k8s-app: node-local-dns
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 10%
  selector:
    matchLabels:
      k8s-app: node-local-dns
  template:
    metadata:
      labels:
        k8s-app: node-local-dns
      annotations:
        prometheus.io/port: "9253"
        prometheus.io/scrape: "true"
    spec:
      priorityClassName: system-node-critical
      serviceAccountName: node-local-dns
      hostNetwork: true
      dnsPolicy: Default  # Don't use cluster DNS.
      tolerations:
      - key: "CriticalAddonsOnly"
        operator: "Exists"
      - effect: "NoExecute"
        operator: "Exists"
      - effect: "NoSchedule"
        operator: "Exists"
      containers:
      - name: node-cache
        image: registry.k8s.io/dns/k8s-dns-node-cache:1.25.0
        resources:
          requests:
            cpu: 25m
            memory: 5Mi
        args: [ "-localip", "__PILLAR__LOCAL__DNS__,__PILLAR__DNS__SERVER__", "-conf", "/etc/Corefile", "-upstreamsvc", "kube-dns-upstream" ]
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
        ports:
        - containerPort: 53
          name: dns
          protocol: UDP
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
        - containerPort: 9253
          name: metrics
          protocol: TCP
        livenessProbe:
          httpGet:
            host: __PILLAR__LOCAL__DNS__
            path: /health
            port: 8080
          initialDelaySeconds: 60
          timeoutSeconds: 5
        volumeMounts:
        - mountPath: /run/xtables.lock
          name: xtables-lock
          readOnly: false
        - name: config-volume
          mountPath: /etc/coredns
        - name: kube-dns-config
          mountPath: /etc/kube-dns
      volumes:
      - name: xtables-lock
        hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
      - name: kube-dns-config
        configMap:
          name: kube-dns
          optional: true
      - name: config-volume
        configMap:
          name: node-local-dns
          items:
            - key: Corefile
              path: Corefile.base
---
# A headless service is a service with a service IP but instead of load-balancing it will return the IPs of our associated Pods.
# We use this to expose metrics to Prometheus.
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/port: "9253"
    prometheus.io/scrape: "true"
  labels:
    k8s-app: node-local-dns
  name: node-local-dns
  namespace: kube-system
spec:
  clusterIP: None
  ports:
    - name: metrics
      port: 9253
      targetPort: 9253
  selector:
    k8s-app: node-local-dns
```

</details>

---

## Step 2: Set Your Variables

Populate required values using the commands below. Replace placeholder domains and addresses with ones that match your environment.

```bash
kubedns=$(kubectl get svc kube-dns -n kube-system -o jsonpath={.spec.clusterIP})
domain=<cluster-domain>             # usually 'cluster.local'
localdns=<node-local-address>       # e.g. 169.254.20.10
upstreamdns="<dns-ip-1> <dns-ip-2>" # replace with valid reachable upstream DNS servers
```

If upstream DNS servers aren’t available from the pod network, consider using the host loopback address:

```bash
upstreamdns="127.0.0.1:53"
```

---

## Step 3: Replace Placeholders in the Manifest

Run the following commands to replace template variables in the manifest:

```bash
sed -i "s/__PILLAR__LOCAL__DNS__/$localdns/g; \
s/__PILLAR__DNS__DOMAIN__/$domain/g; \
s/__PILLAR__DNS__SERVER__/$kubedns/g" nodelocaldns.yaml

sed -i "s/__PILLAR__UPSTREAM__SERVERS__/$upstreamdns/g" nodelocaldns.yaml
```

Double-check that **no `__PILLAR__` placeholders remain** in the file.

---

## Step 4: Apply the Manifest

Deploy NodeLocalDNS:

```bash
kubectl create -f nodelocaldns.yaml
```

---

## Step 5: Restart CoreDNS

Force CoreDNS pods to restart so they can detect the NodeLocalDNS layer:

```bash
kubectl delete pods -n kube-system -l k8s-app=kube-dns
```

---

## Step 6: Test DNS Resolution

Run a simple BusyBox pod and test DNS queries:

```bash
kubectl run -i --tty busybox --image=busybox --restart=Never -- sh
```

Inside the pod:

```sh
nslookup google.com
```

You should see a quick response without resolution failures.

---

## References

- 📘 [NodeLocalDNS Kubernetes Docs](https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/)
- 🔧 [CoreDNS Forward Plugin](https://coredns.io/plugins/forward/)

---

Let me know if you'd like a follow-up post on integrating this into GitOps workflow using FluxCD.