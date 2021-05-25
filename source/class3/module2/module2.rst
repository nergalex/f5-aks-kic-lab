LAB 2B: TLS Traffic with Content-Based Routing
#############################################################

.. contents:: Contents
    :local:
    :depth: 1


    .. note::
        | * For that use case, a new application named cafe will be deployed.
        |
        | * The application cafe is composed of 2 services: **cofee-svc** and **tea-svc**.
        |
        | * The custom resource **VirtualServer** will be used.
        |
        | * The setup we want to deploy is :
        |
        |    - listens for hostname cafe.example.com
        |    - TLS activated and uses a specified cert and key from a K8S secret resource
        |    - request for /tea are sent to service tea
        |    - request for /coffee are sent to service coffee



Deployment of a new application named cafe
***********************************************************

- Step 1: Create the directory Lab2 and move into it

.. code-block:: bash

        $ mkdir lab2
        $ cd lab2/
|

- Step 2: Create a new NameSpace called cafe-ns. We will deploy the application into it.

*input:*

.. code-block:: bash

        kubectl create namespace cafe-ns

*output:*

.. code-block:: bash

        namespace/cafe created

|
- Step 3: copy and paste the manifest below into a new file named cafe.yaml.

That manifest will be used to deploy the application into the cluster.
The application cafe is composed of 2 micro services: coffee and tea.


.. code-block:: yaml
    :linenos:

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: coffee
          namespace: cafe-ns
        spec:
          replicas: 2
          selector:
            matchLabels:
              app: coffee
          template:
            metadata:
              labels:
                app: coffee
            spec:
              containers:
              - name: coffee
                image: nginxdemos/nginx-hello:plain-text
                ports:
                - containerPort: 8080
        ---
        apiVersion: v1
        kind: Service
        metadata:
          name: coffee-svc
          namespace: cafe-ns
        spec:
          ports:
          - port: 80
            targetPort: 8080
            protocol: TCP
            name: http
          selector:
            app: coffee
        ---
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: tea
          namespace: cafe-ns
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: tea
          template:
            metadata:
              labels:
                app: tea
            spec:
              containers:
              - name: tea
                image: nginxdemos/nginx-hello:plain-text
                ports:
                - containerPort: 8080
        ---
        apiVersion: v1
        kind: Service
        metadata:
          name: tea-svc
          namespace: cafe-ns
        spec:
          ports:
          - port: 80
            targetPort: 8080
            protocol: TCP
            name: http
          selector:
            app: tea

|
- Step 4: Deploy the application cafe

*input:*

.. code-block:: bash

        kubectl create -f cafe.yaml


*output:*

.. code-block:: bash

        deployment.apps/coffee created
        service/coffee-svc created
        deployment.apps/tea created
        service/tea-svc created

|
- Step 5: Let's check everything is ok.

NameSpace cafe should have been created and should be in status Active:

*input:*

.. code-block:: bash

        kubectl get namespaces


*output:*

.. code-block:: bash

        NAME                          STATUS   AGE
        arcadia                       Active   2d3h
        cafe-ns                       Active   13s
        default                       Active   2d7h
        external-ingress-controller   Active   2d6h
        internal-ingress-controller   Active   2d6h
        kube-node-lease               Active   2d7h
        kube-public                   Active   2d7h
        kube-system                   Active   2d7h

The services of the application cafe should have been deployed in the NameSpace cafe-ns and should be in status Running.
You should have 2 Pods for the coffee service and 1 Pod for tea service


*input:*

.. code-block:: bash

        kubectl get pods -n cafe-ns


*output:*

.. code-block:: bash

        NAME                      READY   STATUS    RESTARTS   AGE
        coffee-6f4b79b975-pxjxp   1/1     Running   0          21s
        coffee-6f4b79b975-xpfvr   1/1     Running   0          21s
        tea-6fb46d899f-j2mqs      1/1     Running   0          21s
|

You should have the services tea-svc and coffee-svc deployed:

*input:*

.. code-block:: bash

        kubectl get services -n cafe-ns


*output:*

.. code-block:: bash

        NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
        coffee-svc   ClusterIP   10.200.0.91   <none>        80/TCP    16m
        tea-svc      ClusterIP   10.200.0.88   <none>        80/TCP    16m

|

Deployment of a Certificate and Keys for TLS Traffic
*********************************************************

- Step 1: Copy and Past the manifest below into a new file called **cafe-secret.yaml**.

That manifest deploys a certificate and keys that will be used later for TLS traffic.

.. code-block:: yaml
    :linenos:

        apiVersion: v1
        kind: Secret
        metadata:
          name: cafe-secret
          namespace: cafe-ns
        type: kubernetes.io/tls
        data:
          tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURMakNDQWhZQ0NRREFPRjl0THNhWFdqQU5CZ2txaGtpRzl3MEJBUXNGQURCYU1Rc3dDUVlEVlFRR0V3SlYKVXpFTE1Ba0dBMVVFQ0F3Q1EwRXhJVEFmQmdOVkJBb01HRWx1ZEdWeWJtVjBJRmRwWkdkcGRITWdVSFI1SUV4MApaREViTUJrR0ExVUVBd3dTWTJGbVpTNWxlR0Z0Y0d4bExtTnZiU0FnTUI0WERURTRNRGt4TWpFMk1UVXpOVm9YCkRUSXpNRGt4TVRFMk1UVXpOVm93V0RFTE1Ba0dBMVVFQmhNQ1ZWTXhDekFKQmdOVkJBZ01Ba05CTVNFd0h3WUQKVlFRS0RCaEpiblJsY201bGRDQlhhV1JuYVhSeklGQjBlU0JNZEdReEdUQVhCZ05WQkFNTUVHTmhabVV1WlhoaApiWEJzWlM1amIyMHdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFDcDZLbjdzeTgxCnAwanVKL2N5ayt2Q0FtbHNmanRGTTJtdVpOSzBLdGVjcUcyZmpXUWI1NXhRMVlGQTJYT1N3SEFZdlNkd0kyaloKcnVXOHFYWENMMnJiNENaQ0Z4d3BWRUNyY3hkam0zdGVWaVJYVnNZSW1tSkhQUFN5UWdwaW9iczl4N0RsTGM2SQpCQTBaalVPeWwwUHFHOVNKZXhNVjczV0lJYTVyRFZTRjJyNGtTa2JBajREY2o3TFhlRmxWWEgySTVYd1hDcHRDCm42N0pDZzQyZitrOHdnemNSVnA4WFprWldaVmp3cTlSVUtEWG1GQjJZeU4xWEVXZFowZXdSdUtZVUpsc202OTIKc2tPcktRajB2a29QbjQxRUUvK1RhVkVwcUxUUm9VWTNyemc3RGtkemZkQml6Rk8yZHNQTkZ4MkNXMGpYa05MdgpLbzI1Q1pyT2hYQUhBZ01CQUFFd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFLSEZDY3lPalp2b0hzd1VCTWRMClJkSEliMzgzcFdGeW5acS9MdVVvdnNWQTU4QjBDZzdCRWZ5NXZXVlZycTVSSWt2NGxaODFOMjl4MjFkMUpINnIKalNuUXgrRFhDTy9USkVWNWxTQ1VwSUd6RVVZYVVQZ1J5anNNL05VZENKOHVIVmhaSitTNkZBK0NuT0Q5cm4yaQpaQmVQQ0k1ckh3RVh3bm5sOHl3aWozdnZRNXpISXV5QmdsV3IvUXl1aTlmalBwd1dVdlVtNG52NVNNRzl6Q1Y3ClBwdXd2dWF0cWpPMTIwOEJqZkUvY1pISWc4SHc5bXZXOXg5QytJUU1JTURFN2IvZzZPY0s3TEdUTHdsRnh2QTgKN1dqRWVxdW5heUlwaE1oS1JYVmYxTjM0OWVOOThFejM4Zk9USFRQYmRKakZBL1BjQytHeW1lK2lHdDVPUWRGaAp5UkU9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
          tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBcWVpcCs3TXZOYWRJN2lmM01wUHJ3Z0pwYkg0N1JUTnBybVRTdENyWG5LaHRuNDFrCkcrZWNVTldCUU5semtzQndHTDBuY0NObzJhN2x2S2wxd2k5cTIrQW1RaGNjS1ZSQXEzTVhZNXQ3WGxZa1YxYkcKQ0pwaVJ6ejBza0lLWXFHN1BjZXc1UzNPaUFRTkdZMURzcGRENmh2VWlYc1RGZTkxaUNHdWF3MVVoZHErSkVwRwp3SStBM0kreTEzaFpWVng5aU9WOEZ3cWJRcCt1eVFvT05uL3BQTUlNM0VWYWZGMlpHVm1WWThLdlVWQ2cxNWhRCmRtTWpkVnhGbldkSHNFYmltRkNaYkp1dmRySkRxeWtJOUw1S0Q1K05SQlAvazJsUkthaTAwYUZHTjY4NE93NUgKYzMzUVlzeFR0bmJEelJjZGdsdEkxNURTN3lxTnVRbWF6b1Z3QndJREFRQUJBb0lCQVFDUFNkU1luUXRTUHlxbApGZlZGcFRPc29PWVJoZjhzSStpYkZ4SU91UmF1V2VoaEp4ZG01Uk9ScEF6bUNMeUw1VmhqdEptZTIyM2dMcncyCk45OUVqVUtiL1ZPbVp1RHNCYzZvQ0Y2UU5SNThkejhjbk9SVGV3Y290c0pSMXBuMWhobG5SNUhxSkpCSmFzazEKWkVuVVFmY1hackw5NGxvOUpIM0UrVXFqbzFGRnM4eHhFOHdvUEJxalpzVjdwUlVaZ0MzTGh4bndMU0V4eUZvNApjeGI5U09HNU9tQUpvelN0Rm9RMkdKT2VzOHJKNXFmZHZ5dGdnOXhiTGFRTC94MGtwUTYyQm9GTUJEZHFPZVBXCktmUDV6WjYvMDcvdnBqNDh5QTFRMzJQem9idWJzQkxkM0tjbjMyamZtMUU3cHJ0V2wrSmVPRmlPem5CUUZKYk4KNHFQVlJ6NWhBb0dCQU50V3l4aE5DU0x1NFArWGdLeWNrbGpKNkY1NjY4Zk5qNUN6Z0ZScUowOXpuMFRsc05ybwpGVExaY3hEcW5SM0hQWU00MkpFUmgySi9xREZaeW5SUW8zY2czb2VpdlVkQlZHWTgrRkkxVzBxZHViL0w5K3l1CmVkT1pUUTVYbUdHcDZyNmpleHltY0ppbS9Pc0IzWm5ZT3BPcmxEN1NQbUJ2ek5MazRNRjZneGJYQW9HQkFNWk8KMHA2SGJCbWNQMHRqRlhmY0tFNzdJbUxtMHNBRzR1SG9VeDBlUGovMnFyblRuT0JCTkU0TXZnRHVUSnp5K2NhVQprOFJxbWRIQ2JIelRlNmZ6WXEvOWl0OHNaNzdLVk4xcWtiSWN1YytSVHhBOW5OaDFUanNSbmU3NFowajFGQ0xrCmhIY3FIMHJpN1BZU0tIVEU4RnZGQ3haWWRidUI4NENtWmlodnhicFJBb0dBSWJqcWFNWVBUWXVrbENkYTVTNzkKWVNGSjFKelplMUtqYS8vdER3MXpGY2dWQ0thMzFqQXdjaXowZi9sU1JxM0hTMUdHR21lemhQVlRpcUxmZVpxYwpSMGlLYmhnYk9jVlZrSkozSzB5QXlLd1BUdW14S0haNnpJbVpTMGMwYW0rUlk5WUdxNVQ3WXJ6cHpjZnZwaU9VCmZmZTNSeUZUN2NmQ21mb09oREN0enVrQ2dZQjMwb0xDMVJMRk9ycW40M3ZDUzUxemM1em9ZNDR1QnpzcHd3WU4KVHd2UC9FeFdNZjNWSnJEakJDSCtULzZzeXNlUGJKRUltbHpNK0l3eXRGcEFOZmlJWEV0LzQ4WGY2ME54OGdXTQp1SHl4Wlp4L05LdER3MFY4dlgxUE9ucTJBNWVpS2ErOGpSQVJZS0pMWU5kZkR1d29seHZHNmJaaGtQaS80RXRUCjNZMThzUUtCZ0h0S2JrKzdsTkpWZXN3WEU1Y1VHNkVEVXNEZS8yVWE3ZlhwN0ZjanFCRW9hcDFMU3crNlRYcDAKWmdybUtFOEFSek00NytFSkhVdmlpcS9udXBFMTVnMGtKVzNzeWhwVTl6WkxPN2x0QjBLSWtPOVpSY21Vam84UQpjcExsSE1BcWJMSjhXWUdKQ2toaVd4eWFsNmhZVHlXWTRjVmtDMHh0VGwvaFVFOUllTktvCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==

|
- Step 2: Deploy the manifest cafe-secret.

*input:*

.. code-block:: bash

        kubectl create -f cafe-secret.yaml


*output:*

.. code-block:: bash

        secret/cafe-secret created

|
- Step 3: Verify the certificate and keys have been deployed into the namespace cafe-ns

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

|

Deployment of a Virtual Server for TLS with Content-Based Routing
*************************************************************************

- Step 1: Copy/Paste the manifest below into a new file named **cafe-virtual-server-lab-2B.yaml** and deploy it.

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
| For this first deployment, we're going to use the features below:
|    - tls: allows to attach a secret with a TLS certificate and key. The secret must belong to the same namespace as the VirtualServer.
|    - route: defines rules for matching client requests to actions like passing a request to an upstream.

.. code-block:: yaml
    :linenos:

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

        kubectl apply -f cafe-virtual-server-lab-2B.yaml

*output*:

.. code-block:: bash

        virtualserver.k8s.nginx.org/app-cafe configured


- step 3: Check the compilation status of the VirtualServer with the command below:

*input*:

.. code-block:: bash
        kubectl describe virtualserver app-cafe -n cafe-ns


- Step 4: Test the setup
|
| If you have the rights to modify the hosts file of your client:
|
|    - Edit the host file of your client
|    - Add a line with hostname *cafe.example.com* and the EXTERNAL-IP address of the cluster you've seen on step 11 of LAB 2A above.

.. code-block:: bash

        52.167.14.0         cafe.example.com


|    - Open a browser and test some connections :
|
| https://cafe.example.com/tea and https://cafe.example.com/coffee should be successfull.
|
| https://cafe.example.com/ should receive an HTTP 404 Not Found page. -> This is because the path / hasn't been defined into the Routes field of the manifest above.
|
|
| If you don't have the rights on the hosts file of your client then you can use the curl command with the EXTERNAL-IP address of the cluster you've seen on last step of LAB 2A:

.. code-block:: bash

    $ curl https://cafe.example.com/coffee --resolve cafe.example.com:443:52.167.14.0 --insecure
    Server address: 10.22.1.55:8080
    Server name: coffee-6f4b79b975-5vmrd
    Date: 05/May/2021:14:01:43 +0000
    URI: /coffee
    Request ID: 197d3c08b40fea8ba4428ab7d53440de

    $ curl https://cafe.example.com/tea --resolve cafe.example.com:443:52.167.14.0 --insecure
    Server address: 10.22.1.31:8080
    Server name: tea-6fb46d899f-k2sfc
    Date: 05/May/2021:14:01:57 +0000
    URI: /tea
    Request ID: a7874c6a4389b72e75f608ce9ed0075b

    $ curl https://cafe.example.com/ --resolve cafe.example.com:443:52.167.14.0 --insecure
    <html>
    <head><title>404 Not Found</title></head>
    <body>
    <center><h1>404 Not Found</h1></center>
    <hr><center>nginx/1.19.5</center>
    </body>
    </html>

