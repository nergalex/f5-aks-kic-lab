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




harry@Azure:~/lab1$ vi cafe-secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: cafe-secret
  namespace: cafe
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURMakNDQWhZQ0NRREFPRjl0THNhWFdqQU5CZ2txaGtpRzl3MEJBUXNGQURCYU1Rc3dDUVlEVlFRR0V3SlYKVXpFTE1Ba0dBMVVFQ0F3Q1EwRXhJVEFmQmdOVkJBb01HRWx1ZEdWeWJtVjBJRmRwWkdkcGRITWdVSFI1SUV4MApaREViTUJrR0ExVUVBd3dTWTJGbVpTNWxlR0Z0Y0d4bExtTnZiU0FnTUI0WERURTRNRGt4TWpFMk1UVXpOVm9YCkRUSXpNRGt4TVRFMk1UVXpOVm93V0RFTE1Ba0dBMVVFQmhNQ1ZWTXhDekFKQmdOVkJBZ01Ba05CTVNFd0h3WUQKVlFRS0RCaEpiblJsY201bGRDQlhhV1JuYVhSeklGQjBlU0JNZEdReEdUQVhCZ05WQkFNTUVHTmhabVV1WlhoaApiWEJzWlM1amIyMHdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFDcDZLbjdzeTgxCnAwanVKL2N5ayt2Q0FtbHNmanRGTTJtdVpOSzBLdGVjcUcyZmpXUWI1NXhRMVlGQTJYT1N3SEFZdlNkd0kyaloKcnVXOHFYWENMMnJiNENaQ0Z4d3BWRUNyY3hkam0zdGVWaVJYVnNZSW1tSkhQUFN5UWdwaW9iczl4N0RsTGM2SQpCQTBaalVPeWwwUHFHOVNKZXhNVjczV0lJYTVyRFZTRjJyNGtTa2JBajREY2o3TFhlRmxWWEgySTVYd1hDcHRDCm42N0pDZzQyZitrOHdnemNSVnA4WFprWldaVmp3cTlSVUtEWG1GQjJZeU4xWEVXZFowZXdSdUtZVUpsc202OTIKc2tPcktRajB2a29QbjQxRUUvK1RhVkVwcUxUUm9VWTNyemc3RGtkemZkQml6Rk8yZHNQTkZ4MkNXMGpYa05MdgpLbzI1Q1pyT2hYQUhBZ01CQUFFd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFLSEZDY3lPalp2b0hzd1VCTWRMClJkSEliMzgzcFdGeW5acS9MdVVvdnNWQTU4QjBDZzdCRWZ5NXZXVlZycTVSSWt2NGxaODFOMjl4MjFkMUpINnIKalNuUXgrRFhDTy9USkVWNWxTQ1VwSUd6RVVZYVVQZ1J5anNNL05VZENKOHVIVmhaSitTNkZBK0NuT0Q5cm4yaQpaQmVQQ0k1ckh3RVh3bm5sOHl3aWozdnZRNXpISXV5QmdsV3IvUXl1aTlmalBwd1dVdlVtNG52NVNNRzl6Q1Y3ClBwdXd2dWF0cWpPMTIwOEJqZkUvY1pISWc4SHc5bXZXOXg5QytJUU1JTURFN2IvZzZPY0s3TEdUTHdsRnh2QTgKN1dqRWVxdW5heUlwaE1oS1JYVmYxTjM0OWVOOThFejM4Zk9USFRQYmRKakZBL1BjQytHeW1lK2lHdDVPUWRGaAp5UkU9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBcWVpcCs3TXZOYWRJN2lmM01wUHJ3Z0pwYkg0N1JUTnBybVRTdENyWG5LaHRuNDFrCkcrZWNVTldCUU5semtzQndHTDBuY0NObzJhN2x2S2wxd2k5cTIrQW1RaGNjS1ZSQXEzTVhZNXQ3WGxZa1YxYkcKQ0pwaVJ6ejBza0lLWXFHN1BjZXc1UzNPaUFRTkdZMURzcGRENmh2VWlYc1RGZTkxaUNHdWF3MVVoZHErSkVwRwp3SStBM0kreTEzaFpWVng5aU9WOEZ3cWJRcCt1eVFvT05uL3BQTUlNM0VWYWZGMlpHVm1WWThLdlVWQ2cxNWhRCmRtTWpkVnhGbldkSHNFYmltRkNaYkp1dmRySkRxeWtJOUw1S0Q1K05SQlAvazJsUkthaTAwYUZHTjY4NE93NUgKYzMzUVlzeFR0bmJEelJjZGdsdEkxNURTN3lxTnVRbWF6b1Z3QndJREFRQUJBb0lCQVFDUFNkU1luUXRTUHlxbApGZlZGcFRPc29PWVJoZjhzSStpYkZ4SU91UmF1V2VoaEp4ZG01Uk9ScEF6bUNMeUw1VmhqdEptZTIyM2dMcncyCk45OUVqVUtiL1ZPbVp1RHNCYzZvQ0Y2UU5SNThkejhjbk9SVGV3Y290c0pSMXBuMWhobG5SNUhxSkpCSmFzazEKWkVuVVFmY1hackw5NGxvOUpIM0UrVXFqbzFGRnM4eHhFOHdvUEJxalpzVjdwUlVaZ0MzTGh4bndMU0V4eUZvNApjeGI5U09HNU9tQUpvelN0Rm9RMkdKT2VzOHJKNXFmZHZ5dGdnOXhiTGFRTC94MGtwUTYyQm9GTUJEZHFPZVBXCktmUDV6WjYvMDcvdnBqNDh5QTFRMzJQem9idWJzQkxkM0tjbjMyamZtMUU3cHJ0V2wrSmVPRmlPem5CUUZKYk4KNHFQVlJ6NWhBb0dCQU50V3l4aE5DU0x1NFArWGdLeWNrbGpKNkY1NjY4Zk5qNUN6Z0ZScUowOXpuMFRsc05ybwpGVExaY3hEcW5SM0hQWU00MkpFUmgySi9xREZaeW5SUW8zY2czb2VpdlVkQlZHWTgrRkkxVzBxZHViL0w5K3l1CmVkT1pUUTVYbUdHcDZyNmpleHltY0ppbS9Pc0IzWm5ZT3BPcmxEN1NQbUJ2ek5MazRNRjZneGJYQW9HQkFNWk8KMHA2SGJCbWNQMHRqRlhmY0tFNzdJbUxtMHNBRzR1SG9VeDBlUGovMnFyblRuT0JCTkU0TXZnRHVUSnp5K2NhVQprOFJxbWRIQ2JIelRlNmZ6WXEvOWl0OHNaNzdLVk4xcWtiSWN1YytSVHhBOW5OaDFUanNSbmU3NFowajFGQ0xrCmhIY3FIMHJpN1BZU0tIVEU4RnZGQ3haWWRidUI4NENtWmlodnhicFJBb0dBSWJqcWFNWVBUWXVrbENkYTVTNzkKWVNGSjFKelplMUtqYS8vdER3MXpGY2dWQ0thMzFqQXdjaXowZi9sU1JxM0hTMUdHR21lemhQVlRpcUxmZVpxYwpSMGlLYmhnYk9jVlZrSkozSzB5QXlLd1BUdW14S0haNnpJbVpTMGMwYW0rUlk5WUdxNVQ3WXJ6cHpjZnZwaU9VCmZmZTNSeUZUN2NmQ21mb09oREN0enVrQ2dZQjMwb0xDMVJMRk9ycW40M3ZDUzUxemM1em9ZNDR1QnpzcHd3WU4KVHd2UC9FeFdNZjNWSnJEakJDSCtULzZzeXNlUGJKRUltbHpNK0l3eXRGcEFOZmlJWEV0LzQ4WGY2ME54OGdXTQp1SHl4Wlp4L05LdER3MFY4dlgxUE9ucTJBNWVpS2ErOGpSQVJZS0pMWU5kZkR1d29seHZHNmJaaGtQaS80RXRUCjNZMThzUUtCZ0h0S2JrKzdsTkpWZXN3WEU1Y1VHNkVEVXNEZS8yVWE3ZlhwN0ZjanFCRW9hcDFMU3crNlRYcDAKWmdybUtFOEFSek00NytFSkhVdmlpcS9udXBFMTVnMGtKVzNzeWhwVTl6WkxPN2x0QjBLSWtPOVpSY21Vam84UQpjcExsSE1BcWJMSjhXWUdKQ2toaVd4eWFsNmhZVHlXWTRjVmtDMHh0VGwvaFVFOUllTktvCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==


                                                                                                                      5,17          All


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


harry@Azure:~/lab1$ vi cafe-virtual-server.yaml

apiVersion: k8s.nginx.org/v1
kind: VirtualServer
metadata:
  name: app-cafe
  namespace: cafe
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




harry@Azure:~/lab1$ kubectl apply -f cafe-secret.yaml
secret/cafe-secret created
harry@Azure:~/lab1$
harry@Azure:~/lab1$
harry@Azure:~/lab1$ kubectl apply -f cafe-virtual-server.yaml
virtualserver.k8s.nginx.org/app-cafe created
harry@Azure:~/lab1$ kubectl get virtualserver -n cafe
NAME       STATE   HOST               IP            PORTS      AGE
app-cafe   Valid   cafe.example.com   52.167.14.0   [80,443]   48s
harry@Azure:~/lab1$




apiVersion: k8s.nginx.org/v1
kind: VirtualServer
metadata:
  name: app-cafe
  namespace: cafe
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
            value: Value
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
"cafe-virtual-server.yaml" 60L, 1614C                                                                                                                60,34         Bot