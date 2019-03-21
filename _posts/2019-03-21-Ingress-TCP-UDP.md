---
layout: post
title: "Ingress TCP/UDP"
date: 2019-03-21 17:00:06 -0700
comments: true
---

Source: https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/exposing-tcp-udp-services.md

Install ingress nginx with helm
```
helm install stable/nginx-ingress \
	--name nginx-ingress-tcp-udp \
	--namespace nginx-ingress-tcp-udp \
	--set controller.ingressClass=nginx-ingress-tcp-udp
```
Create configmap with format: `namespace/service:port`
```
kubectl apply -f - -o yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: tcp-services
  namespace: nginx-ingress-tcp-udp
data:
  2000: "demo-test/mysql-demo-test:3306"
  2001: "demo-test/sftp-server:22"
  2002: "demo-test/postgres-server:5432"
EOF
```

Edit service nginx-ingress `kubectl edit svc nginx-ingress-tcp-udp-controller -n nginx-ingress-tcp-udp`
```
  - name: proxied-tcp-2000
    port: 2000
    targetPort: 2000
    protocol: TCP
  - name: proxied-tcp-2001
    port: 2001
    targetPort: 2001
    protocol: TCP
  - name: proxied-tcp-2002
    port: 2002
    targetPort: 2002
    protocol: TCP
```

Edit deployment nginx-ingress `kubectl edit deploy nginx-ingress-tcp-udp-controller --namespace nginx-ingress-tcp-udp`
Edit:
```
    spec:
      containers:
      - args:
        - /nginx-ingress-controller
        - --tcp-services-configmap=nginx-ingress-tcp-udp/tcp-services
```

Access through the port previously assigned to the services configured in the configmap.