Exercise 1: Exploring the K8S cluster
####################################################

.. contents:: Contents
    :local:
    :depth: 1



Step 1: Login to your attributed K8S Cluster
*********************************************************

Follow `this link <https://f5-k8s-ctfd.docs.emea.f5se.com/en/latest/class2/module1/module1.html#exercise-1-jumphost>`_ to log into your lab System



Step 2: Check the installed CRDs
*********************************************************

.. code-block:: bash

    kubectl get crds


*output*:

.. code-block:: bash

    NAME                                 CREATED AT
    aplogconfs.appprotect.f5.com         2021-03-08T10:00:03Z
    appolicies.appprotect.f5.com         2021-03-08T10:00:03Z
    apusersigs.appprotect.f5.com         2021-03-08T10:00:03Z
    globalconfigurations.k8s.nginx.org   2021-03-08T10:00:03Z
    policies.k8s.nginx.org               2021-03-08T10:00:03Z
    transportservers.k8s.nginx.org       2021-03-08T10:00:03Z
    virtualserverroutes.k8s.nginx.org    2021-03-08T10:00:03Z
    virtualservers.k8s.nginx.org         2021-03-08T10:00:04Z

.. note::
    | The **VirtualServer** and **VirtualServerRoute** resources are new load balancing configuration, introduced in release 1.5 as an alternative to the Ingress resource.
    | The resources enable use cases not supported with the Ingress resource, such as traffic splitting and advanced content-based routing.
    | The resources are implemented as Custom Resource Definitions.
    | The VirtualServer Custom Resource will be used in the labs 2.
    |
    |


Step 3: Check the NameSpaces of the cluster
*********************************************************

.. code-block:: bash

        kubectl get ns

*output:*

.. code-block:: bash

        NAME                          STATUS   AGE
        arcadia                       Active   30d
        default                       Active   30d
        external-ingress-controller   Active   30d
        internal-ingress-controller   Active   30d
        kube-node-lease               Active   30d
        kube-public                   Active   30d
        kube-system                   Active   30d

.. note::
    | The namespace of interest for the Labs 2 is **external-ingress-controller**.
    | That namespace includes the NGINX Ingress Controller which will be used during the labs.
    | Some new namespaces will be created later during the labs.
    | The namespace arcadia will be used during the lab 3 WAF.
    |
    |


Step 4: Check the pods in the NameSpace used for the NIC
*********************************************************

.. code-block:: bash

        kubectl get pods -n external-ingress-controller

*output:*

.. code-block:: bash

        NAME                                               READY   STATUS    RESTARTS   AGE
        nap-external-ingress-controller-54db45d656-fg4tq   1/1     Running   0          30d



Step 5: Check the Ingress Class Name attached to the NIC
*********************************************************

.. code-block:: bash

        kubectl describe pod nap-external-ingress-controller-54db45d656-fg4tq -n external-ingress-controller

*output:*

.. code-block:: bash
    :emphasize-lines: 14

        Name:         nap-external-ingress-controller-54db45d656-fg4tq
        Namespace:    external-ingress-controller
        .......
        Containers:
          external-nginx-plus-ingress-nginx-ingress:
            .......
            Ports:         80/TCP, 443/TCP, 9113/TCP, 8081/TCP
            Host Ports:    0/TCP, 0/TCP, 0/TCP, 0/TCP
            Args:
              -nginx-plus=true
              -nginx-reload-timeout=0
              -enable-app-protect=true
              .......
              -ingress-class=nginx-external

.. note::
    | The Ingress Class Name **nginx-external**  will be used as a reference into the deployment of the manifests.
    | It allows to indicate which Ingress Controller must be used for a specific deployment.


Step 6: Check the Public IP address attached to the NIC
*********************************************************

.. code-block:: bash

    kubectl get services -n external-ingress-controller

*output:*

.. code-block:: bash

        NAME                         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
        elb-nap-ingress-controller   LoadBalancer   10.200.0.15   52.167.14.0   80:31613/TCP,443:31094/TCP   30d



.. note::
        | **Notice the EXTERNAL-IP address and write it somewhere.**
        | **It will be used later in our labs.**

|
|
|
**Capture The Flag**

    **2a.1 What kind of K8S Resource Definition can be used with NGINX+ for simplicity and advanced configuration of load balancing?**


    **2a.2 What is the name of the Custom Resource used for Advanced load balancing configuration and used as an alternative to the Ingress resource?**



