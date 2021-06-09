Exercise 3: Canary or A/B Testing
###############################

.. contents:: Contents
    :local:
    :depth: 1

Description of the Environment for the Exercise
***********************************************************

- A new version of the application cafe has been deployed.
- That application has been deployed into the namespace cafe-ns.
- That application has two versions of the service coffee-svc:
    - service **coffee-v1-svc**
    - service **coffee-v2-svc**


Objectives
*******************************

- Deploy the setup below by using the field **splits** available in the custom resource **VirtualServer**
    - Pass 80% of requests to the coffee-v1-svc
    - Pass the remaining 20% to coffee-v2-svc



Check the Environment is up and running
*******************************************************

- Check the pods coffee-v1 and coffee-v2 are correctly deployed:

*input*:

.. code-block:: bash

        kubectl get pods -n cafe-ns


*output*:

.. code-block:: bash
    :emphasize-lines: 4,5

        NAME                         READY   STATUS    RESTARTS   AGE
        coffee-6f4b79b975-4gqzg      1/1     Running   0          140m
        coffee-6f4b79b975-5vmrd      1/1     Running   0          140m
        coffee-v1-75869cf7ff-4jxc2   1/1     Running   0          13m
        coffee-v2-67499ff985-7h88c   1/1     Running   0          13m
        tea-6fb46d899f-k2sfc         1/1     Running   0          140m


- Check the services coffee-v1-svc and coffee-v2-svc are correctly deployed:

*input*:

.. code-block:: bash

        kubectl get services -n cafe-ns


*output*:

.. code-block:: bash
    :emphasize-lines: 3,4

        NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
        coffee-svc      ClusterIP   10.200.0.100   <none>        80/TCP    63m
        coffee-v1-svc   ClusterIP   10.200.0.32    <none>        80/TCP    9m57s
        coffee-v2-svc   ClusterIP   10.200.0.76    <none>        80/TCP    9m56s
        tea-svc         ClusterIP   10.200.0.51    <none>        80/TCP    63m



Step 1: Create a new manifest for the 80/20 traffic splitting
**************************************************************************

- Create/Edit a new file named **cafe-virtual-server-lab2-ex3.yaml**

Copy/past the manifest below into the file **cafe-virtual-server-lab2-ex3.yaml**

.. code-block:: yaml

        apiVersion: k8s.nginx.org/v1
        kind: VirtualServer
        metadata:
          name: app-cafe
          namespace: cafe-ns
        spec:
          ingressClassName: nginx-external
          host: cafe.example.com
          upstreams:
          - name: coffee-v1
            service: coffee-v1-svc
            port: 80
          - name: coffee-v2
            service: coffee-v2-svc
            port: 80
          routes:
          - path: /coffee
            splits:
            - weight: 80
              action:
                pass: coffee-v1
            - weight: 20
              action:
                pass: coffee-v2



Step 2: Deploy the new manifest
************************************

*input*:

.. code-block:: bash

        kubectl apply -f cafe-virtual-server-lab2-ex3.yaml


*output*:

.. code-block:: bash

        virtualserver.k8s.nginx.org/app-cafe configured


Step 3: Check the status of the VirtualServer Resource
*******************************************************

*input*:

.. code-block:: bash

    kubectl describe virtualserver app-cafe -n cafe-ns


 *output*:

.. code-block:: bash

        Events:
          Type    Reason          Age   From                      Message
          ----    ------          ----  ----                      -------
          Normal  AddedOrUpdated  5s    nginx-ingress-controller  Configuration for cafe-ns/app-cafe was added or updated


Step 4: Test the setup
**************************

- Send 10 connections with curl:

Replace {{EXTERNAL_IP_NIC}} by the IP address of the NGINX Ingress Controller you've noticed on the last step of Exercise 1.


.. code-block:: bash

        curl http://cafe.example.com/coffee --resolve cafe.example.com:{{EXTERNAL_IP_NIC}}


- Check that you have around:
    - 8 connections to Server name: coffee-v1
    - 2 connections to Server name: coffee-v2


|
|
|
**Capture The Flag**

    **2d.1 What is the name of the field (in the specification of the VirtualServer CRD) which is used to setup canary or A/B testing?**


    **2d.2 What is the name of the field which allow to define a percentage of traffic as part of the splits configuration?**
