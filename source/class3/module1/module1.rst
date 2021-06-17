Exercise 1: Basic Content-Based Routing
#############################################################

.. contents:: Contents
    :local:
    :depth: 1


Description of the Environment for the Exercise
***********************************************************

- For that use case, the **application named cafeapp** has been deployed
- The application cafe is composed of **2 services**: **cofee-svc** and **tea-svc**
- The application has been deployed in the **NameSpace lab2-cafeapp**
- For TLS, a certificate and keys have been deployed into the namespace lab2-cafeapp under the name of **cafeapp-secret-tls**
- The Custom Resource Definitions (CRD) **VirtualServer** and **VirtualServerRoute** have been installed

.. note::
    | The **VirtualServer** and **VirtualServerRoute** resources are new resources for load balancing configuration, introduced in release 1.5 as an alternative to the Ingress resource.
    | These new CRD enable use cases not supported with the Ingress resource, such as traffic splitting and advanced content-based routing.
    |



Objectives
***********************************************************

- Use the new resource **VirtualServer** to deploy the setup below:
    - listens for hostname cafe.example.com
    - TLS activated and uses a specified cert and key from a K8S secret resource
    - request for /tea are sent to service tea
    - request for /coffee are sent to service coffee




Check the Environment is up and running
*******************************************************

- Step 1: Check namespace **lab2-cafeapp** has been deployed and is **Active**:


*input:*

.. code-block:: bash

        kubectl get namespaces


*output:*

.. code-block:: bash
    :emphasize-lines: 10

        NAME                          STATUS   AGE
        debug                         Active   30d
        default                       Active   39d
        external-ingress-controller   Active   39d
        infra-kibana                  Active   39d
        kube-node-lease               Active   39d
        kube-public                   Active   39d
        kube-system                   Active   39d
        lab1-arcadia                  Active   36d
        lab2-cafeapp                  Active   12h
        lab3-cafe                     Active   27d
        lab4-sentence-api             Active   26d
        lab4-sentence-front           Active   26d


- Step2: You should have **4 running pods** in the namespace lab2-cafeapp (2 pods for the service coffee and 2 Pods for service tea):

*input:*

.. code-block:: bash

        kubectl get pods -n lab2-cafeapp | grep v1


*output:*

.. code-block:: bash

        NAME                         READY   STATUS    RESTARTS   AGE
        coffee-v1-6f4b79b975-pxjxp   1/1     Running   0          21s
        coffee-v1-6f4b79b975-xpfvr   1/1     Running   0          21s
        tea-v1-6fb46d899f-j2mqs      1/1     Running   0          21s
        tea-v1-6fb46d899f-df5ge      1/1     Running   0          21s


- Step3: You should have the services tea-svc and coffee-svc deployed:

*input:*

.. code-block:: bash

        kubectl get services -n lab2-cafeapp


*output:*

.. code-block:: bash

        NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
        coffee       ClusterIP   10.200.0.91   <none>        80/TCP    16m
        tea          ClusterIP   10.200.0.88   <none>        80/TCP    16m



- Step 4: Verify a certificate and keys have been deployed into the namespace lab2-cafeapp

*input*:

.. code-block:: bash

        kubectl describe secret cafeapp-secret-tls -n lab2-cafeapp


*output:*

.. code-block:: bash

        Name:         cafeapp-secret-tls
        Namespace:    lab2-cafeapp
        Labels:       <none>
        Annotations:  <none>

        Type:  kubernetes.io/tls

        Data
        ====
        tls.crt:  1164 bytes
        tls.key:  1675 bytes


Deploy a Virtual Server for TLS with Content-Based Routing
*************************************************************************

- Step 1: Copy/Paste the manifest below into a new file named **cafe-virtual-server-lab2-ex1.yaml**.

| That manifest uses the CRD **VirtualServer**.
|
| The deployment configure the **external NGINX+ Ingress Controller** via the usage of the Ingress Class Name **nginx-external**.
|
| For that first deployment, the setup is very simple :
|
|    - listens for hostname cafe.example.com
|    - TLS is activated and use the cert and key from cafeapp-secret-tls
|    - Simple Path Routing is done :
|        - request for /tea are sent to service tea
|        - request for /coffee are sent to service coffee
|
| A lot of features are available via the utilisation of the CRD *VirtualServer*.
|
| The list is quite long and is available in the `on-line manual <https://docs.nginx.com/nginx-ingress-controller/installation/building-ingress-controller-image/>`_. Some of those advanced features will be used later in the workshop.
|
| For this first deployment, we're going to use the specifications below:
|    - tls: allows to attach a secret with a TLS certificate and key. The secret must belong to the same namespace as the VirtualServer.
|    - route: defines rules for matching client requests to actions like passing a request to an upstream.


**REPLACE {{SITE_ID}} in the field host by your allocated site ID before saving and applying the manifest below in cafe-virtual-server-lab2-ex1.yaml.**

.. code-block:: yaml
    :emphasize-lines: 8

    apiVersion: k8s.nginx.org/v1
    kind: VirtualServer
    metadata:
      name: cafeapp
      namespace: lab2-cafeapp
    spec:
      ingressClassName: nginx-external
      host: cafeapp{{SITE_ID}}.f5app.dev
      tls:
        secret: cafeapp-secret-tls
        redirect:
          enable: true
          code: 301
          basedOn: scheme
      upstreams:
      - name: tea
        service: tea
        port: 80
      - name: coffee
        service: coffee
        port: 80
      routes:
      - path: /tea
        action:
          pass: tea
      - path: /coffee
        action:
          pass: coffee


- step 2: Deploy the manifest:

*input*:

.. code-block:: bash

        kubectl apply -f cafe-virtual-server-lab2-ex1.yaml

*output*:

.. code-block:: bash

        virtualserver.k8s.nginx.org/cafeapp configured


- step 3: Check the compilation status of the VirtualServer with the command below:

*input*:

.. code-block:: bash

        kubectl describe virtualserver cafeapp -n lab2-cafeapp


Test the setup
*************************************************************

- Run the curl command below.

- Replace {{SITE_ID}} by your allocated site ID.

*input*:

.. code-block:: bash

        curl https://cafeapp{{SITE_ID}}.f5app.dev/coffee --insecure


*output*:

.. code-block:: bash

        Server address: 10.22.1.55:8080
        Server name: coffee-6f4b79b975-5vmrd
        Date: 05/May/2021:14:01:43 +0000
        URI: /coffee
        Request ID: 197d3c08b40fea8ba4428ab7d53440de


*input*:

.. code-block:: bash

        curl https://cafeapp{{SITE_ID}}.f5app.dev/tea --insecure



*output*:

.. code-block:: bash

        Server address: 10.22.1.31:8080
        Server name: tea-6fb46d899f-k2sfc
        Date: 05/May/2021:14:01:57 +0000
        URI: /tea
        Request ID: a7874c6a4389b72e75f608ce9ed0075b


*input*:

.. code-block:: bash

        curl https://cafeapp{{SITE_ID}}.f5app.dev/



*output*:

.. code-block:: html

        <html>
        <head><title>404 Not Found</title></head>
        <body>
        <center><h1>404 Not Found</h1></center>
        <hr><center>nginx/1.19.5</center>
        </body>
        </html>


|
|
**Capture The Flag**

    **1.1 What kind of K8S Resource Definition can be used with NGINX+ for simplicity and advanced configuration of load balancing?**
    | Tips: Answer is an acronym in 3 letters

    **1.2 What is the name of the CRD used for Advanced load balancing configuration and used as an alternative to the Ingress resource?**
    | Tips: `here <https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/#virtualserver-specification>`_

    **1.3 What is the name of the field (in the specification of the VirtualServer CRD) which is used to implement SSL Offloading?**
    | Tips: `here <https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/#virtualserver-tls>`_

    **1.4 What is the name of the field (in the specification of the VirtualServer CRD) which is used to defines a rule of content-based load balancing?**
    | Tips: `here <https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/#virtualserver-route>`_

