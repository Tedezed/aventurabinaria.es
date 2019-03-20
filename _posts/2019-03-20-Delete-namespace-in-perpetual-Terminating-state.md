---
layout: post
title: "Delete namespace in perpetual Terminating state"
date: 2019-03-20 17:00:06 -0700
comments: true
---

Error, perpetual Terminating state:
```
NAME             STATUS        AGE
cert-manager     Terminating   3h
default          Active        1y
kube-public      Active        1y
kube-system      Active        1y
```

Clean namespace:
```
kubectl delete all -n cert-manager --all --force --grace-period=0
kubectl delete ns cert-manager --force --grace-period=0
```

Variables for configuration:
```
export NAMESPACE_TO_DELETE="cert-manager"
export CLUSTER_NAME="gke_PRO-ID_ZONE-GCP_NAME-CLUSTER"
```

Create service account with permissions:
```
kubectl create -f - -o yaml << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tmpadmin
EOF
```

Save the namespace to edit it `kubectl get namespace $NAMESPACE_TO_DELETE -o json > tmp.json`

Edit:
```
    "spec": {
        "finalizers": [
            "kubernetes"
        ]
    },
```
To:
```
    "spec": {
        "finalizers": []
    },
```

Create the following variables
```
APISERVER=$(kubectl config view -o jsonpath="{.clusters[?(@.name==\"$CLUSTER_NAME\")].cluster.server}")
TOKEN=$(kubectl get secrets -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='tmpadmin')].data.token}"|base64 -d)
```

Test token: `curl -X GET $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure`

Update namespace:
```
curl -X PUT $APISERVER/api/v1/namespaces/$NAMESPACE_TO_DELETE/finalize -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" --data-binary @tmp.json  --insecure
```

After this the namespace is erased.

Clean the service account:
```
kubectl delete sa tmpadmin
```