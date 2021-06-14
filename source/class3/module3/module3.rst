Exercise 3: Canary or A/B Testing
###############################

.. contents:: Contents
    :local:
    :depth: 1

Description of the Environment for the Exercise
***********************************************************

- A new version of the application cafeapp has been deployed.
- That application has been deployed into the namespace lab2-cafeapp.
- That application has two versions of the service coffee:
    - service **coffee-v1**
    - service **coffee-v2**


Objectives
*******************************

- Deploy the setup below by using the field **splits** available in the CRD **VirtualServer**
    - Pass 80% of requests to the coffee-v1
    - Pass the remaining 20% to coffee-v2



Check the Environment is up and running
*******************************************************

- Check the pods coffee-v1 and coffee-v2 are correctly deployed:

*input*:

.. code-block:: bash

        kubectl get pods -n lab2-cafeapp


*output*:

.. code-block:: bash
    :emphasize-lines: 4,5

        NAME                            READY   STATUS    RESTARTS   AGE
        coffee-v1-6f4b79b975-4gqzg      1/1     Running   0          140m
        coffee-v1-6f4b79b975-5vmrd      1/1     Running   0          140m
        coffee-v2-75869cf7ff-4jxc2      1/1     Running   0          140m
        coffee-v2-67499ff985-7h88c      1/1     Running   0          140m
        tea-v1-6fb46d899f-k2sfc         1/1     Running   0          140m
        tea-v1-6fb46d899f-df5ge         1/1     Running   0          140m

- Check the services coffee-v1 and coffee-v2 are correctly deployed:

*input*:

.. code-block:: bash

        kubectl get services -n lab2-cafeapp


*output*:

.. code-block:: bash
    :emphasize-lines: 2,3

        NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
        coffee-v1       ClusterIP   10.200.0.100   <none>        80/TCP    63m
        coffee-v2       ClusterIP   10.200.0.32    <none>        80/TCP    9m57s
        tea-v1          ClusterIP   10.200.0.51    <none>        80/TCP    63m



Step 1: Create a new manifest for the 80/20 traffic splitting
**************************************************************************

- Create/Edit a new file named **cafe-virtual-server-lab2-ex3.yaml**
- Copy/past the manifest below into the file **cafe-virtual-server-lab2-ex3.yaml**
- REPLACE {{SITE_ID}} in the field host by your allocated site ID before saving and applying cafe-virtual-server-lab2-ex1.yaml.

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
      - name: coffee-v1
        service: coffee-v1
        port: 80
      - name: coffee-v2
        service: coffee-v2
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

        virtualserver.k8s.nginx.org/cafeapp configured


Step 3: Check the status of the VirtualServer Resource
*******************************************************

*input*:

.. code-block:: bash

    kubectl describe virtualserver cafeapp -n lab2-cafeapp


 *output*:

.. code-block:: bash

        Events:
          Type    Reason          Age   From                      Message
          ----    ------          ----  ----                      -------
          Normal  AddedOrUpdated  5s    nginx-ingress-controller  Configuration for lab2-cafeapp/app-cafe was added or updated


Step 4: Test the setup
**************************

- Send 10 connections with curl.
- REPLACE {{SITE_ID}} by your allocated SITE ID in the curl command below.

.. code-block:: bash

        curl https://cafeapp{{SITE_ID}}.f5app.dev/coffee


- Check that you have around:
    - 8 connections to Server name: coffee-v1
    - 2 connections to Server name: coffee-v2


|
|
**Capture The Flag**

    **3.1 What is the name of the field (in the specification of the VirtualServer CRD) which is used to setup canary or A/B testing?**
    | Tips: `here <https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/#split>`_

    **3.2 What is the name of the field which allow to define a percentage of traffic as part of the splits configuration?**
    | Tips: `here <https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/#split>`_
