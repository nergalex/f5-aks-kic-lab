Architecture of Arcadia Application
###################################

.. note:: This application is available in GitLab in case you want to build your own lab : 

First of all, it is important to understand how Arcadia app is split between micro-services


**This is what Arcadia App looks like when the 4 microservices are up and running, and you can see how traffic is routed based on URI**

.. image:: ../pictures/module1/arcadia-api.png
   :align: center

**But you can deploy Arcadia Step by Step**

If you deploy only ``Main App`` and ``Back End`` services.

.. image:: ../pictures/module1/MainApp.png
   :align: center

.. note:: You can see App2 (Money Transfer) and App3 (Refer Friend) are not available. There is dynamic content showing a WARNING instead of a 404 or blank frame.

|

If you deploy ``Main App``, ``Back End`` and ``Money Tranfer`` services.

.. image:: ../pictures/module1/app2.png
   :align: center

|

If you deploy ``Main App``, ``Back End``, ``Money Tranfer`` and ``Refer Friend`` services.

.. image:: ../pictures/module1/app3.png
   :align: center


harry@Azure:~$ kubectl get crds
NAME                                 CREATED AT
aplogconfs.appprotect.f5.com         2021-03-08T10:00:03Z
appolicies.appprotect.f5.com         2021-03-08T10:00:03Z
apusersigs.appprotect.f5.com         2021-03-08T10:00:03Z
globalconfigurations.k8s.nginx.org   2021-03-08T10:00:03Z
policies.k8s.nginx.org               2021-03-08T10:00:03Z
transportservers.k8s.nginx.org       2021-03-08T10:00:03Z
virtualserverroutes.k8s.nginx.org    2021-03-08T10:00:03Z
virtualservers.k8s.nginx.org         2021-03-08T10:00:04Z




harry@Azure:~/lab1/Deploy_Cofee$ kubectl create namespace cafe
namespace/cafe created
harry@Azure:~/lab1/Deploy_Cofee$ kubectl get namespaces
NAME                          STATUS   AGE
arcadia                       Active   2d3h
cafe                          Active   13s
default                       Active   2d7h
external-ingress-controller   Active   2d6h
internal-ingress-controller   Active   2d6h
kube-node-lease               Active   2d7h
kube-public                   Active   2d7h
kube-system                   Active   2d7h
harry@Azure:~/lab1/Deploy_Cofee$ vi cafe.yaml
harry@Azure:~/lab1/Deploy_Cofee$ kubectl create -f cafe.yaml
deployment.apps/coffee created
service/coffee-svc created
deployment.apps/tea created
service/tea-svc created
harry@Azure:~/lab1/Deploy_Cofee$ kubectl get pods -n cafe
NAME                      READY   STATUS    RESTARTS   AGE
coffee-6f4b79b975-pxjxp   1/1     Running   0          21s
coffee-6f4b79b975-xpfvr   1/1     Running   0          21s
tea-6fb46d899f-j2mqs      1/1     Running   0          21s
harry@Azure:~/lab1/Deploy_Cofee$


harry@Azure:~/lab1/Deploy_Cofee$ kubectl create -f cafe-secret.yaml
secret/cafe-secret created

harry@Azure:~/lab1/Deploy_Cofee$ kubectl describe secret cafe-secret -n cafe
Name:         cafe-secret
Namespace:    cafe
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/tls

Data
====
tls.crt:  1164 bytes
tls.key:  1675 bytes
harry@Azure:~/lab1/Deploy_Cofee$

