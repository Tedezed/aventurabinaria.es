---
layout: post
title: "External Load Balancer for Kubernetes - HAProxy"
date: 2017-06-10 16:25:06 -0700
comments: true
---

- Inspired by [service-loadbalancer](https://github.com/kubernetes/contrib/tree/master/service-loadbalancer)
- Source [Celtic-Kubernetes](https://github.com/Tedezed/Celtic-Kubernetes/tree/master/external_loadbalancer_hap)


#### You need:

* Cluster Kubernetes
* New node for HAProxy

#### Instalation in node HAProxy

Install basic sowftware

	yum install epel-release
	yum install haproxy git socat python-pip
	pip install jinja2
	pip install deepdiff

Clone repository in / or other route for dinamic configuration of HAProxy

	git clone https://github.com/Tedezed/Celtic-Kubernetes.git

Create errors html for service HAProxy

	mkdir /etc/haproxy/errors/
	cp /Celtic-Kubernetes/external_loadbalancer_hap/errors/* /etc/haproxy/errors/
	cp /Celtic-Kubernetes/external_loadbalancer_hap/system/haproxy.cfg /etc/haproxy/

Create state global

	mkdir -p /var/state/haproxy/
	touch  /var/state/haproxy/global

Enable Haproxy

	systemctl enable haproxy

#### Test

	python hap_manager_daemon.py start
	python hap_manager_daemon.py stop
	sh haproxy_reload

&nbsp;

## HAP Manager


You need the repository [https://github.com/Tedezed/Celtic-Kubernetes.git](https://github.com/Tedezed/Celtic-Kubernetes.git)

Modify configuration.json for hap_manager
	
	{
	"kube_api": "morrigan:8080",
	"version": "v1",
	"file_conf": "template.cfg",
	"stats": true,
	"sleep": 3
	}

* Kube API master
	
		"kube_api": "ip_kube_api_server:port_http"

#### Unit for systemd of hap_manager

Copy file hap_manager.service

	cp /Celtic-Kubernetes/external_loadbalancer_hap/system/hap_manager.service /lib/systemd/system/hap_manager.service

Modify permissions for hap_manager

	chmod 644 /lib/systemd/system/hap_manager.service

Reload daemon systemctl for reload configuration of units

	systemctl daemon-reload

Start hap_manager.service

	systemctl start hap_manager.service

	systemctl enable hap_manager.service

See settings

	cat /etc/haproxy/haproxy.cfg | grep acl

&nbsp;

#### Define services

Example rc

	apiVersion: v1
	kind: ReplicationController
	metadata:
	 name: nginx-controller
	spec:
	 replicas: 2
	 selector:
	   name: nginx
	 template:
	   metadata:
	     labels:
	       name: nginx
	   spec:
	     containers:
	       - name: nginx
	         image: nginx
	         ports:
	           - containerPort: 80

Example svc, you need `NodePort`

	apiVersion: v1
	kind: Service
	metadata:
	  name: nginx-service-domain
	  labels:
	    app: nginx
	spec:
	  type: NodePort
	  ports:
	  - port: 80
	    protocol: TCP
	    name: http
	  selector:
	    name: nginx

Enter with [http://IP-SERVER-HAP/NAME-SERVICE/](http://IP-SERVER-HAP/NAME-SERVICE/)

You need domain for the service, no problem, you can use the label "domain"

Example svc with domain

	apiVersion: v1
	kind: Service
	metadata:
	  name: nginx-service-domain
	  labels:
	    app: nginx
	    domain: www.test-domain.com
	spec:
	  type: NodePort
	  ports:
	  - port: 80
	    protocol: TCP
	    name: http
	  selector:
	    name: nginx

&nbsp;

#### Not repeat the domain name

You can use [manager_tools.py](https://github.com/Tedezed/Celtic-Kubernetes/blob/master/external_loadbalancer_hap/manager_tools.py) (function **constraint_domain**) for not to repeat the domain name. If return True domain name is in use.

Example 
```
constraint_domain("morrigan:8080","v1","www.test-domain.com")
```