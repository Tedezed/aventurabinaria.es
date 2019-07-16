---
layout: post
title: "Massive editor json for Kubernetes"
date: 2019-07-15 20:00:06 -0700
comments: true
---

In this post we will see the use of a custom bash (kmae and kselect) function that can save you time in your Kuberentes environments.

The idea is to interact with Kubernetes to be able to quickly edit a large amount of resources using grep filters; external (kubectl get pod) and internal (kubectl get pod -o json).

Code: [https://gist.github.com/Tedezed/a8abece3507296c4fa1eb0ea70cc15e5](https://gist.github.com/Tedezed/a8abece3507296c4fa1eb0ea70cc15e5)

> **Kmae**: It can be very dangerous if it is not used with caution, it saves three days of json edited to be able to recover them in the case of needing them in `$HOME/.kmae`.
> You can create backup of your cluster in json using: [https://github.com/Tedezed/kubernetes-resources/tree/master/kubebackup](https://github.com/Tedezed/kubernetes-resources/tree/master/kubebackup)
&nbsp;

To test it you can create an environment with several nginx with the following for:
```
for i in $(seq 1 6); do
  kubectl create namespace nginx-$i;
  kubectl create deploy nginx-$i -n nginx-$i --image=nginx;
done
```
&nbsp;
<img src="https://www.aventurabinaria.es/images/posts/tty-kmae.gif" width="99%" alt="Gif tty" />
&nbsp;

We will use kselect to test your filters in the first place:

**Format:**
```
kselect (ENTITY) (GREP_EXTERNAL_KUBECTL) (GREP_INTERNAL_JSON) (JSON_PATH) (NAMESPACE OR --all-namespaces)
```
**Example:** Get json path spec.strategy.rollingUpdate of the nginx with name nginx-1 to 3 and generation equal to 1 in all namespaces
```
kselect deploy "(nginx-[1-3])" '"observedGeneration": 1' .spec.strategy.rollingUpdate --all-namespaces
```
&nbsp;

In the second place, modify the spec.strategy.rollingUpdate of the three deployments at the same command:

**Format:**
```
kmae (ENTITY) (GREP_EXTERNAL_KUBECTL) (GREP_INTERNAL_JSON) (JSON_PATH_TO_UPDATE) (NAMESPACE OR --all-namespaces)
```
**Example:**
```
kmae deploy "(nginx-[1-3])" '"observedGeneration": 1' '{"spec":{"strategy":{"rollingUpdate":{"maxUnavailable": "50%", "maxSurge": "45%"}}}}' --all-namespaces
```
&nbsp;

We tested the previous kselect that does not return anything because the generation number changed:
```
kselect deploy "(nginx-[1-3])" '"observedGeneration": 1' .spec.strategy.rollingUpdate --all-namespaces
```

If it works if we change the generation:
```
kselect deploy "(nginx-[1-3])" '"observedGeneration": 2' .spec.strategy.rollingUpdate --all-namespaces
```
&nbsp;

One last test with modification of replicas:
```
kselect deploy "(nginx-[1-3])" '"observedGeneration": 2' .spec.replicas --all-namespaces
kmae deploy "(nginx-[1-3])" '"observedGeneration": 2' '{"spec":{"replicas":2}}' --all-namespaces
kselect deploy "(nginx-[1-3])" '"observedGeneration": 3' .spec.replicas --all-namespaces
```
&nbsp;

Clean enviroment:
```
for i in $(seq 1 6); do 
  kubectl delete deploy nginx-$i -n nginx-$i --force --grace-period=0;
  kubectl delete namespace nginx-$i --force --grace-period=0; 
done
```
