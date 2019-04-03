---
title: "Openshift Deployment"
date: 2018-08-13T22:41:32-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In this tutorial, we are going to deploy the four APIs to an Openshift project. Openshift is built on top of Kubernetes, but there are still a lot of differences. It has a different command line `oc` than `kubectl` with similar syntax. 

### Assmption

Let's assume that you have an Openshift project available and you have access to the OC command line to deploy applications. 

### Overview

This is a snapshot of a sample app created by to demonstrate microservices service registration/discovery with Consul. Source repo can be found at 

https://github.com/networknt/light-example-4j

More detailed designs of service discovery and consul see below.

https://doc.networknt.com/tutorial/common/discovery/
https://doc.networknt.com/tutorial/common/discovery/consul/
https://doc.networknt.com/tutorial/common/discovery/consul-tls/
https://doc.networknt.com/tutorial/common/discovery/http-health/

Docker images used for the demo

https://hub.docker.com/r/networknt/com.networknt.apia-1.0.0/tags/
https://hub.docker.com/r/networknt/com.networknt.apib-1.0.0/tags/
https://hub.docker.com/r/networknt/com.networknt.apic-1.0.0/tags/
https://hub.docker.com/r/networknt/com.networknt.apid-1.0.0/tags/

#### Steps to deploy

* Prepare projects (namespaces) for the demo app in openshift, e.g.<br/>
    oc new-project lightdemo<br/>
    oc new-project lightdemo-dev1<br/>
    ##### Tie the project to the 3 nodes labeled with "lob=to" which are designated for the demo api deployments
    oc patch namespace lightdemo-dev1 -p '{"metadata":{"annotations":{"openshift.io/node-selector": "lob=to"}}}'<br/>
    ##### Set security context constrain (SCC) 'hostnetwork' to the project so that demo api containers can listen on hostNetwork interface.
    oc adm policy add-scc-to-user hostnetwork system:serviceaccount:lightdemo-dev1:default<br/>

   _Note:_ the api deployment requires hostNetwork for port binding and will allocate ports from a pre-defined range (30000~30499 in this case). Not only the SCC has to be granted to the serviceaccount but also iptables firewall rule needs to be added. <br/>

    iptables -I INPUT 6 -p tcp -m multiport --dports 30000:30499 -j ACCEPT 

* Push the api container images to the above openshift registry

    docker push docker-registry-default.apps-np.caas.example.com/lightdemo/com.networknt.apia-1.0.0:1.5.18-redhat<br/>
    docker push docker-registry-default.apps-np.caas.example.com/lightdemo/com.networknt.apib-1.0.0:1.5.18-redhat<br/>
    docker push docker-registry-default.apps-np.caas.example.com/lightdemo/com.networknt.apic-1.0.0:1.5.18-redhat<br/>
    docker push docker-registry-default.apps-np.caas.example.com/lightdemo/com.networknt.apid-1.0.0:1.5.18-redhat<br/>

* Login to openshift and set project (namespace) <br/>

    oc login<br/>
    oc project lightdemo-dev1<br/>

* Prepare the client.truststore with Consul CA cert

   Import the cert to one of the client.truststore files (such as api_d) with keytool -import command line, then copy the file over to other api folders.

* Edit secret.yml files with the Consul token

* Edit consul.yml file with Consul hostname and enable httpCheck to true

* Create secret objects in openshift with create_secrets.sh

    ./create_secrets.sh

* Create the deployments with attached dc yaml files

    oc create -f ./api_a/apia-dc.yaml
    oc create -f ./api_b/apib-dc.yaml
    oc create -f ./api_c/apic-dc.yaml
    oc create -f ./api_d/apid-dc.yaml

* Verify the logs of all 4 pods and make a note the listen address/port of each (they're all dynamically assigned at runtime). 

  You can test the API's by accessing https://LISTEN_IP:LISTEN_PORT/v1/data, e.g. https://172.26.152.232:30000

  Output will look like below

    ["API D: Message 1 from port 7444","API D: Message 2 from port 7444","API B: Message 1","API B: Message 2"]

* The services can also be seen on the Consul UI.

