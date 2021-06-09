Exercise 1: Basic Content-Based Routing
#############################################################

.. contents:: Contents
    :local:
    :depth: 1


Description of the Environment for the Exercise
***********************************************************

- For that use case, the **application named cafe** has been deployed
- The application cafe is composed of **2 services**: **cofee-svc** and **tea-svc**
- The application has been deployed in the **NameSpace cafe-ns**
- For TLS, a certificate and keys have been deployed into the namespace cafe-ns under the name of **cafe-secret**
- The Custom Resource Definitions **VirtualServer** and **VirtualServerRoute** have been installed

.. note::
    | The **VirtualServer** and **VirtualServerRoute** resources are new resources for load balancing configuration, introduced in release 1.5 as an alternative to the Ingress resource.
    | These custom resources enable use cases not supported with the Ingress resource, such as traffic splitting and advanced content-based routing.
    | These resources are implemented as Custom Resource Definitions.
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

- Step 1: Check namespace **cafe-ns** has been deployed and is **Active**:


*input:*

.. code-block:: bash

        kubectl get namespaces


*output:*

.. code-block:: bash
    :emphasize-lines: 3

        NAME                          STATUS   AGE
        arcadia                       Active   2d3h
        cafe-ns                       Active   13s
        default                       Active   2d7h
        external-ingress-controller   Active   2d6h
        internal-ingress-controller   Active   2d6h
        kube-node-lease               Active   2d7h
        kube-public                   Active   2d7h
        kube-system                   Active   2d7h


- Step2: You should have **3 running pods** in the namespace cafe-ns (2 pods for the service coffee  and 1 Pod for service tea):

*input:*

.. code-block:: bash

        kubectl get pods -n cafe-ns


*output:*

.. code-block:: bash

        NAME                      READY   STATUS    RESTARTS   AGE
        coffee-6f4b79b975-pxjxp   1/1     Running   0          21s
        coffee-6f4b79b975-xpfvr   1/1     Running   0          21s
        tea-6fb46d899f-j2mqs      1/1     Running   0          21s


- Step3: You should have the services tea-svc and coffee-svc deployed:

*input:*

.. code-block:: bash

        kubectl get services -n cafe-ns


*output:*

.. code-block:: bash

        NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
        coffee-svc   ClusterIP   10.200.0.91   <none>        80/TCP    16m
        tea-svc      ClusterIP   10.200.0.88   <none>        80/TCP    16m



- Step 4: Verify a certificate and keys have been deployed into the namespace cafe-ns

*input*:

.. code-block:: bash

        kubectl describe secret cafe-secret -n cafe-ns


*output:*

.. code-block:: bash

        Name:         cafe-secret
        Namespace:    cafe-ns
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

| That manifest uses the custom resources **VirtualServer**.
|
| The deployment configure the **external NGINX+ Ingress Controller** via the usage of the Ingress Class Name **nginx-external**.
|
| For that first deployment, the setup is very simple :
|
|    - listens for hostname cafe.example.com
|    - TLS is activated and use the cert and key from cafe-secret
|    - Simple Path Routing is done :
|        - request for /tea are sent to service tea
|        - request for /coffee are sent to service coffee
|
| A lot of features are available via the utilisation of the custom resources *VirtualServer*.
|
| The list is quite long and is available in the `on-line manual <https://docs.nginx.com/nginx-ingress-controller/installation/building-ingress-controller-image/>`_. Some of those advanced features will be used later in the workshop.
|
| For this first deployment, we're going to use the specifications below:
|    - tls: allows to attach a secret with a TLS certificate and key. The secret must belong to the same namespace as the VirtualServer.
|    - route: defines rules for matching client requests to actions like passing a request to an upstream.

.. code-block:: yaml

        apiVersion: k8s.nginx.org/v1
        kind: VirtualServer
        metadata:
          name: app-cafe
          namespace: cafe-ns
        spec:
          ingressClassName: nginx-external
          host: cafe.example.com
          tls:
            secret: cafe-secret
          upstreams:
          - name: tea
            service: tea-svc
            port: 80
          - name: coffee
            service: coffee-svc
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

        virtualserver.k8s.nginx.org/app-cafe configured


- step 3: Check the compilation status of the VirtualServer with the command below:

*input*:

.. code-block:: bash

        kubectl describe virtualserver app-cafe -n cafe-ns


Test the setup
*************************************************************


- Check the Public IP address attached to the NGINX Ingress Controller

.. code-block:: bash

    kubectl get services -n external-ingress-controller

*output:*

.. code-block:: bash

        NAME                         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
        elb-nap-ingress-controller   LoadBalancer   10.200.0.15   52.167.14.0   80:31613/TCP,443:31094/TCP   30d



.. note::
        | **Notice the EXTERNAL-IP address and write it somewhere.**
        | **It will be used for the commands below and in the others exercises.**


- Run the curl command below.

- Replace {{EXTERNAL_IP_NIC}} by the IP address of the NGINX Ingress Controller you've noticed above.

*input*:

.. code-block:: bash

        curl https://cafe.example.com/coffee --resolve cafe.example.com:443:{{EXTERNAL_IP_NIC}} --insecure


*output*:

.. code-block:: bash

        Server address: 10.22.1.55:8080
        Server name: coffee-6f4b79b975-5vmrd
        Date: 05/May/2021:14:01:43 +0000
        URI: /coffee
        Request ID: 197d3c08b40fea8ba4428ab7d53440de


*input*:

.. code-block:: bash

        curl https://cafe.example.com/tea --resolve cafe.example.com:443:{{EXTERNAL-IP}} --insecure



*output*:

.. code-block:: bash

        Server address: 10.22.1.31:8080
        Server name: tea-6fb46d899f-k2sfc
        Date: 05/May/2021:14:01:57 +0000
        URI: /tea
        Request ID: a7874c6a4389b72e75f608ce9ed0075b


*input*:

.. code-block:: bash

        curl https://cafe.example.com/ --resolve cafe.example.com:443:{{EXTERNAL-IP}} --insecure



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

    **2b.1 What kind of K8S Resource Definition can be used with NGINX+ for simplicity and advanced configuration of load balancing?**
Custom/custom/Customs/customs/crd/CRD/crds/CRDS

    **2b.2 What is the name of the Custom Resource used for Advanced load balancing configuration and used as an alternative to the Ingress resource?**
VirtualServers/virtualservers/Virtual Servers/virtual servers/VirtualServer/virtualserver/Virtual Server/virtual server

    **2b.3 What is the name of the field (in the specification of the VirtualServer CRD) which is used to select a certificate/key for decrypting TLS traffic?**
secret/Secret

    **2b.4 What is the name of the field (in the specification of the VirtualServer CRD) which is used to defines rules content-based load balancing?**
routes/Routes

