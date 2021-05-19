LAB 2D: Canary or A/B Testing
###############################

.. contents:: Contents
    :local:
    :depth: 1

LAB 2D: Canary or A/B Testing
*******************************

    .. note::
        | * For that use case, we're going to use a new cafe application with two versions of the service coffee-svc.
        |
        | * The aim is to pass 80% of requests to the coffee-v1-svc and the remaining 20% to coffee-v2-svc.
        |
        | * We're going to use another field named **splits** and available in the custom resource **VirtualServer**.
        |
        | * The split defines a weight for an action as part of the splits configuration.
        |


- Step 1: Deploy the new application cafe-v2:

Create/Edit a new file named **cafe-v2.yaml** and copy/past the manifest below.

.. code-block:: bash

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: coffee-v1
          namespace: cafe-ns
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: coffee-v1
          template:
            metadata:
              labels:
                app: coffee-v1
            spec:
              containers:
              - name: coffee-v1
                image: nginxdemos/nginx-hello:plain-text
                ports:
                - containerPort: 8080
        ---
        apiVersion: v1
        kind: Service
        metadata:
          name: coffee-v1-svc
          namespace: cafe-ns
        spec:
          ports:
          - port: 80
            targetPort: 8080
            protocol: TCP
            name: http
          selector:
            app: coffee-v1
        ---
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: coffee-v2
          namespace: cafe-ns
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: coffee-v2
          template:
            metadata:
              labels:
                app: coffee-v2
            spec:
              containers:
              - name: coffee-v2
                image: nginxdemos/nginx-hello:plain-text
                ports:
                - containerPort: 8080
        ---
        apiVersion: v1
        kind: Service
        metadata:
          name: coffee-v2-svc
          namespace: cafe-ns
        spec:
          ports:
          - port: 80
            targetPort: 8080
            protocol: TCP
            name: http
          selector:
            app: coffee-v2

Deploy the application:

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl apply -f cafe-v2.yaml

Verify the pods coffee-v1 and coffee-v2 are correctly deployed:

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl get pods -n cafe-ns
        NAME                         READY   STATUS    RESTARTS   AGE
        coffee-6f4b79b975-4gqzg      1/1     Running   0          140m
        coffee-6f4b79b975-5vmrd      1/1     Running   0          140m
        coffee-v1-75869cf7ff-4jxc2   1/1     Running   0          13m
        coffee-v2-67499ff985-7h88c   1/1     Running   0          13m
        tea-6fb46d899f-k2sfc         1/1     Running   0          140m

Verify the services coffee-v1-svc and coffee-v2-svc are correctly deployed:

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl get services -n cafe-ns
        NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
        coffee-svc      ClusterIP   10.200.0.100   <none>        80/TCP    63m
        coffee-v1-svc   ClusterIP   10.200.0.32    <none>        80/TCP    9m57s
        coffee-v2-svc   ClusterIP   10.200.0.76    <none>        80/TCP    9m56s
        tea-svc         ClusterIP   10.200.0.51    <none>        80/TCP    63m


- Step 2: Create/Edit a new file named **cafe-virtual-server-lab-2D.yaml** and copy/past the manifest below.

.. code-block:: bash

        apiVersion: k8s.nginx.org/v1
        kind: VirtualServer
        metadata:
          name: cafe-vs
          namespace: cafe-ns
        spec:
          ingressClassName: nginx-external
          host: cafev2.example.com
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


- Step 3: Deploy the new setup

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl apply -f cafe-virtual-server-lab-2D.yaml
        virtualserver.k8s.nginx.org/cafe-vs configured


- Step 4: Verify the status of the VirtualServer with the command below:

.. code-block:: bash

        kubectl describe virtualserver cafe-vs -n cafe-ns
        . . .
        Events:
          Type    Reason          Age   From                      Message
          ----    ------          ----  ----                      -------
          Normal  AddedOrUpdated  5s    nginx-ingress-controller  Configuration for cafe-ns/cafe-vs was added or updated

- Step 4: Test the setup

Use curl (see step 10 in Lab 2B for the command and options) to open 10 connections on https://cafe.example.com/coffee

.. code-block:: bash

        $ curl http://cafev2.example.com/coffee --resolve cafev2.example.com:80:52.167.14.0

8 connections should go to Server name: coffee-v1

2 connections should go to Server name: coffee-v2

