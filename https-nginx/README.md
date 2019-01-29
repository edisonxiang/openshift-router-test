# https-nginx

Test Router and Routes which refer to the backend https nginx application in the OpenShift Origin.

In this example, we try to get the client X-Forward-For IP Address in the https ngixin application.

The workflow is like below:
```
client => Elastic Load Balancer(ELB) => OpenShift Router(HAProxy) => OpenShift Route(Reencrypt) => Nginx(PHP)
```

Before you start to test this example, please install the OpenShift Origin environment whose version is greater than 3.9.

* Add privilege for serviceaccount
```
  oc adm policy add-scc-to-user privileged system:serviceaccount:default:service-nginx
  oc adm policy add-scc-to-group anyuid system:authenticated
```

* Create https-nginx Service
```
  oc create -f statefulset.yaml
```

* Create routes exposed on the Router
```
  oc create route reencrypt --service=service-nginx  --dest-ca-cert=tls.crt
```

If you are using Elastic Load Balancer(ELB) on the Cloud, please finish the following steps:

* Create Elastic Load Balancer(ELB) on the Cloud
* Add Listener for Protocol:Port(TCP:443) and point to the OpenShift Router Host IP Address
* Enable the TCP Source IP Address Transit for your Elastic Load Balancer(ELB)

DNS Zone and Record Set are necessary Since we are using the OpenShift Router
which is based on Domain Name to finish the Load Balancer feature.
* Create DNS Private Zone

  Name: openshift.example.com
  
* Create DNS Record Set

  Name: service-nginx-default.apps.openshift.example.com
  
  IP Address: Elastic Load Balancer(ELB) IP Address

At last, you can try to run the following command to test it:
```
  curl -v -k https://service-nginx-default.apps.openshift.example.com/test.php
```

