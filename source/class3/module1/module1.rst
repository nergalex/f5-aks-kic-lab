
Architecture of the Kubernetes Cluster
#######################################

.. contents:: Contents
    :local:

For simplification, for that workshop, we use the Azure Kubernetes Service aka AKS.

    .. note::
        | The goal of the workshop is not to learn how to install NGINX+ as an Ingress Controller.
        | So to gain time, we have already done the installation (see `on-line manual <https://docs.nginx.com/nginx-ingress-controller/installation/building-ingress-controller-image/>`_ for step by step).


Description of the Kubernetes Cluster
####################################################

- Name of the K8S Cluster: **CloudBuilder**
- 3 NameSpaces have already been added:

    - **external-ingress-controller** -> contains 1 pod for NIC. Will be used for external traffic.
    - **internal-ingress-controller** -> contains 1 pod for NIC. Will be used for internal traffic.
    - **arcadia** -> contains the pods for the application named arcadia

- Some Custom Resource Definitions have been added and will be used for the use cases of the workshop:

    - **virtualservers and virtualserverroutes** -> enable use cases not supported with the Ingress resource, such as traffic splitting and advanced content-based routing.
    - **Policies** -> allows to configure features like access control and rate-limiting.
    - **TransportServer** -> allows to configure TCP, UDP, and TLS Passthrough load balancing.
    - **AppProtect CRDs** -> used to configure security policies, custom attack signatures and security logs for NGINX AppProtect (F5 WAF).

- The external Ingress controller is linked to an Azure Public Load Balancer -> we will see the public IP later on.

- The coexistence of multiple Ingress Controllers in one cluster is provided by the support of Ingress Class Name:

    - External NIC has been deployed with argument **ingress-class=nginx-external**.
    - Internal NIC has been deployed with argument **ingress-class=nginx-internal**.



LAB 2A: Exploring and understanding the K8S cluster
####################################################

    .. note::
        | In order to be independant of a specific K8S distribution, standard tools will be used for managing the cluster.
        | The tool ``kubectl`` will be used during that workshop.


- Step 1: Open you browser and go to the `Azure portal <https://portal.azure.com>`_

- Step 2: Use the credentials which have been provided to you.

- Step 3: On the window, open the cli window to access a shell (the red arrow below):

    .. image:: ./images/_01_AzurePortalOpenBash.png
        :align: center
        :width: 800


- Step 4: You should see a page which looks like the one below

    .. image:: ./images/_02_bash_opened.png
        :align: center
        :width: 800

- Step 5: Configure kubectl to connect to your Kubernetes cluster using the command ``az aks get-credentials``.

    - The name of the K8S Cluster is ``CloudBuilder``.
    - Use the resource group name which has been assigned to you. For instance ``rg-aksdistrict2``.

.. code-block:: bash

    harry@Azure:~$ az aks get-credentials --resource-group rg-aksdistrict2 --name CloudBuilder
    Merged "CloudBuilder" as current context in /home/harry/.kube/config

- Step 6: Let's verify the CRDs installed:

.. code-block:: bash

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

- Step 7: Let's check the NameSpaces of the cluster:

.. code-block:: bash

        harry@Azure:~$ kubectl get ns
        NAME                          STATUS   AGE
        arcadia                       Active   30d
        default                       Active   30d
        external-ingress-controller   Active   30d
        internal-ingress-controller   Active   30d
        kube-node-lease               Active   30d
        kube-public                   Active   30d
        kube-system                   Active   30d

Step 8: Look at the pods in each NameSpaces with the command ``kubectl get pods``:

.. code-block:: bash

        harry@Azure:~$ kubectl get pods -n default
        No resources found in default namespace.

.. code-block:: bash

        harry@Azure:~$ kubectl get pods -n arcadia
        NAME                       READY   STATUS    RESTARTS   AGE
        app2-6dcf6d5845-crpv6      1/1     Running   0          30d
        app2-6dcf6d5845-wdxds      1/1     Running   0          30d
        app3-b989dc6dc-6klxk       1/1     Running   0          30d
        app3-b989dc6dc-9vpfm       1/1     Running   0          30d
        backend-56c9b667d5-4x4w2   1/1     Running   0          30d
        backend-56c9b667d5-zfgvc   1/1     Running   0          30d
        main-84cf4949b9-f5x5t      1/1     Running   0          30d
        main-84cf4949b9-pnkwt      1/1     Running   0          30d

.. code-block:: bash

        harry@Azure:~$ kubectl get pods -n external-ingress-controller
        NAME                                               READY   STATUS    RESTARTS   AGE
        nap-external-ingress-controller-54db45d656-fg4tq   1/1     Running   0          30d

.. code-block:: bash

        harry@Azure:~$ kubectl get pods -n internal-ingress-controller
        NAME                                               READY   STATUS    RESTARTS   AGE
        nap-internal-ingress-controller-55fdb8cd95-2dz77   1/1     Running   0          30d


Step 9: Let's check the Ingress Class Name attached to each NIC:

.. code-block:: bash

        harry@Azure:~$ kubectl describe pod nap-external-ingress-controller-54db45d656-fg4tq -n external-ingress-controller
        Name:         nap-external-ingress-controller-54db45d656-fg4tq
        Namespace:    external-ingress-controller
        .......
        .......
        Containers:
          external-nginx-plus-ingress-nginx-ingress:
            .......
            .......
            Ports:         80/TCP, 443/TCP, 9113/TCP, 8081/TCP
            Host Ports:    0/TCP, 0/TCP, 0/TCP, 0/TCP
            Args:
              -nginx-plus=true
              -nginx-reload-timeout=0
              -enable-app-protect=true
              .......
              .......
              -ingress-class=nginx-external        ****INGRESS CLASS NAME is nginx-external****
              .......
              .......


.. code-block:: bash

        harry@Azure:~/lab2$ kubectl describe pod nap-internal-ingress-controller-55fdb8cd95-2dz77 -n internal-ingress-controller
        Name:         nap-internal-ingress-controller-55fdb8cd95-2dz77
        Namespace:    internal-ingress-controller
        .......
        .......
            Ports:         80/TCP, 443/TCP, 9113/TCP, 8081/TCP
            Host Ports:    0/TCP, 0/TCP, 0/TCP, 0/TCP
            Args:
              -nginx-plus=true
              -nginx-reload-timeout=0
              -enable-app-protect=true
              .......
              .......
              -ingress-class=nginx-internal         ****INGRESS CLASS NAME is nginx-internal****
              .......
              .......


- Step 10: Let's check the Public IP address attached to the external Ingress Controller:

.. code-block:: bash

        harry@Azure:~$ kubectl get services -n external-ingress-controller
        NAME                         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
        elb-nap-ingress-controller   LoadBalancer   10.200.0.15   52.167.14.0   80:31613/TCP,443:31094/TCP   30d

.. note::
        | Notice the EXTERNAL-IP address and write it somewhere.
        | It will be used later in our labs.


LAB 2B: Simple Traffic Splitting and Content-Based Routing
###########################################################

| For that use case, a new application named cafe will be deployed.
| The application cafe is composed of 2 services: cofee and tea.
| The custom resource **VirtualServer** will be used.
| That first deployment is simple. We will complete it later on with more complex actions.
| For now, the use case is:
    - listens for hostname cafe.example.com
    - TLS activated and uses a specified cert and key from K8S secret resource
    - Simple Path Routing is done :
        - request for /tea are sent to service tea
        - request for /coffee are sent to service coffee
|

- Step 1: Create the directory Lab2 and move into it

.. code-block:: bash

        harry@Azure:~$ mkdir lab2
        harry@Azure:~$ cd lab2/
        harry@Azure:~/lab2$
|
- Step 2: Create a new NameSpace called cafe-ns. We will deploy the application into it.

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl create namespace cafe-ns
        namespace/cafe created

|
- Step 3: copy and paste the manifest below into a new file named cafe.yaml.

That manifest will be used to deploy the application into the cluster.
The application cafe is composed of 2 micro services: cofee and tea.


.. code-block:: bash

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

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl create -f cafe.yaml
        deployment.apps/coffee created
        service/coffee-svc created
        deployment.apps/tea created
        service/tea-svc created

|
- Step 5: Let's check everything is ok.

NameSpace cafe should have been created and should be in status Active:

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl get namespaces
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

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl get pods -n cafe-ns
        NAME                      READY   STATUS    RESTARTS   AGE
        coffee-6f4b79b975-pxjxp   1/1     Running   0          21s
        coffee-6f4b79b975-xpfvr   1/1     Running   0          21s
        tea-6fb46d899f-j2mqs      1/1     Running   0          21s

You should have the services tea-svc and coffee-svc deployed:

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl get services -n cafe-ns
        NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
        coffee-svc   ClusterIP   10.200.0.91   <none>        80/TCP    16m
        tea-svc      ClusterIP   10.200.0.88   <none>        80/TCP    16m

|
- Step 6: Copy and Past the manifest below into a new file called **cafe-secret.yaml**.

That manifest deploys a certificate and keys that will be used later for TLS traffic.

.. code-block:: bash

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
- Step 7: Deploy the manifest cafe-secret.

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl create -f cafe-secret.yaml
        secret/cafe-secret created

|
- Step 8: Verify the certificate and keys have been deployed into the namespace cafe

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl describe secret cafe-secret -n cafe-ns
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
- Step 9: Copy/Paste the manifest below into a new file named **cafe-virtual-server.yaml** and deploy it.

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

.. code-block:: bash

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


- Deploy the manifest:

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl apply -f cafe-virtual-server.yaml
        virtualserver.k8s.nginx.org/app-cafe configured


- Check the compilation status of the VirtualServer with the command below:

.. code-block:: bash
        kubectl describe virtualserver app-cafe -n cafe-ns


- Step 10: Test the setup
|
| If you have the rights to modify the hosts file of your client:
|
|    - Edit the host file of your client
|    - Add a line with hostname *cafe.example.com* and the EXTERNAL-IP address of the cluster you've seen on step 11 of LAB 2A above.

.. code-block:: bash

        52.167.14.0         cafe.example.com

|    - Open a browser and test some connections :
    | https://cafe.example.com/tea and https://cafe.example.com/coffee should be successfull
    | https://cafe.example.com/ should receive an HTTP 404 Not Found page. -> This is because the path / hasn't been defined into the Routes field of the manifest above.


| If you don't have the rights on the hosts file of your client then you can use the curl command with the EXTERNAL-IP address of the cluster you've seen on step 11 of LAB 2A above:

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



LAB 2C: Canary and A/B Testing
###########################################################

| For that use case, we're going to use a new cafe application with two versions of the service coffee.
|
| The aim is to pass 80% of requests to the coffee-v1 and the remaining 20% to coffee-v2.
|
| The custom resource **VirtualServer** will be used with the field **splits**.
|
| The split defines a weight for an action as part of the splits configuration.
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


- Step 2: Create/Edit a new file named **cafe-virtual-server-lab-2C.yaml** and copy/past the manifest below.

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

        harry@Azure:~/lab2$ kubectl apply -f cafe-virtual-server-lab-2C.yaml
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



LAB 2D: Advanced Traffic Splitting and Content-Based Routing
##############################################################

LAB 2D: Advanced Traffic Splitting and Content-Based Routing
*************************************************************

| For that use case, a new application named cafe will be deployed.
| The application cafe is composed of 2 services: cofee and tea.
| The custom resource **VirtualServer** will be used.
| That first deployment is simple. We will complete it later on with more complex actions.
| For now, the use case is:
    - listens for hostname cafe.example.com
    - TLS activated and uses a specified cert and key from K8S secret resource
    - Simple Path Routing is done :
        - request for /tea are sent to service tea
        - request for /coffee are sent to service coffee
|


11. Let's modify the deployment with a more complex setup

- Copy and Paste the manifest below into a new file named cafe-virtual-server-2.yaml and deploy it.
- For that new deployment we use a lot more features available in the CRDs VirtualServer.
    - if path is */redirect* then the action is **redirect** to http://www.nginx.com.
    - if path is */proxy* then the action is **proxy** to add/rewrite/ignore some headers.
    - if path is */return_page* then the action is **return** to reply with a custom web page.
    - in each action, variables could be used like: $request_uri, $request_method, $request_body, $scheme, $host, $request_time, $request_length, $connection, $remote_addr, $remote_port, $ssl_cipher, $ssl_client_cert, etc


.. code-block:: bash

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
          - path: /redirect
            action:
              redirect:
                url: http://www.nginx.com
                code: 301
          - path: /proxy
            action:
              proxy:
                upstream: coffee
                requestHeaders:
                  pass: true
                  set:
                  - name: My-Header
                    value: Value
                  - name: Client-Cert
                    value: ${ssl_client_escaped_cert}
                responseHeaders:
                  add:
                  - name: My-Header
                    value: Hello_this_your_value
                  - name: IC-Nginx-Version
                    value: ${nginx_version}
                    always: true
                  hide:
                  - x-internal-version
                  ignore:
                  - Expires
                  - Set-Cookie
                  pass:
                  - Server
          - path: /return_page
            action:
              return:
                code: 200
                type: text/plain
                body: "Hello World\n\n\n\nRequest is ${request_uri}\nRequest Method is ${request_method}\nRequest Scheme is ${scheme}\nRequest Host is ${host}\nRequest Lengthis ${request_length}\nNGINX Version is ${nginx_version}\nClient IP address is ${remote_addr}\nClient Port is : ${remote_port}\nLocal Time is ${time_local}\nServer IP Address is ${server_addr}\nServer Port is ${server_port}\nProtocol is ${server_protocol}\n"


- Deploy the manifest:

.. code-block:: bash

        harry@Azure:~/lab2$ kubectl apply -f cafe-virtual-server-2.yaml
        virtualserver.k8s.nginx.org/app-cafe configured


- Check the compilation status of the VirtualServer with the command below:

.. code-block:: bash
        kubectl describe virtualserver app-cafe -n cafe-ns


12. Test the setup

[ADC] Check compilation status of VS: kubectl describe virtualserver cafe -n cafe
[HK]  CHECK COMPILATION ADDED IN PRECEDENT POINT ABOVE

[ADC] Check compilation status of VSR: kubectl describe virtualserverroute coffee -n cafe
[HK] VSR are done in steps 13+


Open a browser and test some connections on:

https://cafe.example.com                -> works like previous configuration
https://ccafe.example.com/redirect      -> client is redirected to www.nginx.com
https://cafe.example.com/return_page    -> custom page Hello World is returned
https://cafe.example.com/proxy          -> requests go to coffee you should see custom headers in the responses



13. Let's modify the setup to use the CRD VirtualServerRoute.

In this example we use the VirtualServer and VirtualServerRoute resources to configure load balancing.
We put the load balancing configuration as well as the deployments and services into multiple namespaces.
Instead of one namespace, we now use three: tea, coffee, and cafe.

    In the tea namespace, we create the tea deployment, service, and the corresponding load-balancing configuration.
    In the coffee namespace, we create the coffee deployment, service, and the corresponding load-balancing configuration.
    In the cafe namespace, we create the cafe secret with the TLS certificate and key and the load-balancing configuration for the cafe application.


- copy and paste the manifests below:

namespaces.yaml:

.. code-block:: bash

        apiVersion: v1
        kind: Namespace
        metadata:
          name: cafe-ns
        ---
        apiVersion: v1
        kind: Namespace
        metadata:
          name: tea-ns
        ---
        apiVersion: v1
        kind: Namespace
        metadata:
          name: coffee-ns


tea.yaml:

.. code-block:: bash

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: tea
          namespace: tea-ns
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
          namespace: tea-ns
        spec:
          ports:
          - port: 80
            targetPort: 8080
            protocol: TCP
            name: http
          selector:
            app: tea


cofee.yaml:

.. code-block:: bash

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: coffee
          namespace: coffee-ns
        spec:
          replicas: 1
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
          namespace: coffee-ns
        spec:
          ports:
          - port: 80
            targetPort: 8080
            protocol: TCP
            name: http
          selector:
            app: coffee


coffee-virtual-server-route.yaml:

.. code-block:: bash

        apiVersion: k8s.nginx.org/v1
        kind: VirtualServerRoute
        metadata:
          name: coffee-vsr
          namespace: coffee-ns
        spec:
          host: cafe.example.com
          upstreams:
          - name: coffee
            service: coffee-svc
            port: 80
          subroutes:
          - path: /coffee
            action:
              pass: coffee


tea-virtual-server-route.yaml:

.. code-block:: bash

        apiVersion: k8s.nginx.org/v1
        kind: VirtualServerRoute
        metadata:
          name: tea-vsr
          namespace: tea-ns
        spec:
          host: cafe.example.com
          upstreams:
          - name: tea
            service: tea-svc
            port: 80
          subroutes:
          - path: /tea
            action:
              pass: tea


cafe-virtual-server.yaml:

.. code-block:: bash

        apiVersion: k8s.nginx.org/v1
        kind: VirtualServer
        metadata:
          name: cafe
          namespace: cafe-ns
        spec:
          host: cafe.example.com
          tls:
            secret: cafe-secret
          routes:
          - path: /tea
            route: tea-vsr/tea
          - path: /coffee
            route: coffee-vsr/coffee


- Step 1: Create the Namespaces

Create the required tea, coffee, and cafe namespaces:

.. code-block:: bash

        $ kubectl create -f namespaces.yaml


- Step 2: Deploy the new Cafe Application

Create the tea deployment and service in the tea-ns namespace:

.. code-block:: bash

        $ kubectl create -f tea.yaml

Create the coffee deployment and service in the coffee-ns namespace:

.. code-block:: bash

        $ kubectl create -f coffee.yaml


- Step 3: Configure Load Balancing and TLS Termination

Create the VirtualServerRoute resource for tea in the tea-ns namespace:

.. code-block:: bash

        $ kubectl create -f tea-virtual-server-route.yaml

Create the VirtualServerRoute resource for coffee in the coffee-ns namespace:

.. code-block:: bash

        $ kubectl create -f coffee-virtual-server-route.yaml


Create the VirtualServer resource for the cafe app in the cafe-ns namespace:

.. code-block:: bash

        $ kubectl create -f cafe-virtual-server.yaml


- Step 4: Test the Configuration

Check that the configuration has been successfully applied by inspecting the events of the VirtualServerRoutes and VirtualServer:

.. code-block:: bash

        $ kubectl describe virtualserverroute tea -n tea-ns
        . . .
        Events:
          Type     Reason                 Age   From                      Message
          ----     ------                 ----  ----                      -------
          Warning  NoVirtualServersFound  2m    nginx-ingress-controller  No VirtualServer references VirtualServerRoute tea/tea
          Normal   AddedOrUpdated         1m    nginx-ingress-controller  Configuration for tea-ns/tea was added or updated

.. code-block:: bash

        $ kubectl describe virtualserverroute coffee -n coffee-ns
        . . .
        Events:
          Type     Reason                 Age   From                      Message
          ----     ------                 ----  ----                      -------
          Warning  NoVirtualServersFound  2m    nginx-ingress-controller  No VirtualServer references VirtualServerRoute coffee/coffee
          Normal   AddedOrUpdated         1m    nginx-ingress-controller  Configuration for coffee-ns/coffee was added or updated

.. code-block:: bash

        $ kubectl describe virtualserver cafe -n cafe-ns
        . . .
        Events:
          Type    Reason          Age   From                      Message
          ----    ------          ----  ----                      -------
          Normal  AddedOrUpdated  1m    nginx-ingress-controller  Configuration for cafe-ns/cafe was added or updated


Access the application using curl.
We'll use curl's --insecure option to turn off certificate verification of our self-signed certificate and --resolve option to set the IP address and HTTPS port of the Ingress Controller to the domain name of the cafe application:

To get coffee:

.. code-block:: bash

        $ curl --resolve cafe.example.com:$IC_HTTPS_PORT:$IC_IP https://cafe.example.com:$IC_HTTPS_PORT/coffee --insecure
        Server address: 10.16.1.193:80
        Server name: coffee-7dbb5795f6-mltpf
        ...


    If your prefer tea:

.. code-block:: bash

        $ curl --resolve cafe.example.com:$IC_HTTPS_PORT:$IC_IP https://cafe.example.com:$IC_HTTPS_PORT/tea --insecure
        Server address: 10.16.0.157:80
        Server name: tea-7d57856c44-674b8
        ...




                                                                                                          60,34         Bot