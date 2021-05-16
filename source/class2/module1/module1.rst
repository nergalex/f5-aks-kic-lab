Infrastructure
##############################################################

.. contents:: Contents
    :local:

Exercise 1: Jumphost
*********************
Logging into your lab System:

- Download SSH key `jumphost.key <https://f5-my.sharepoint.com/:f:/r/personal/al_dacosta_f5_com/Documents/Lab/f5-aks-kic-lab?csf=1&web=1&e=NcV9gX>`_
- Ask F5 for your ``{{site_ID}}`` and your Azure ``{{region}}``
- Open an SSH session to ``jumphost-aksdistrict{{site_ID}}.{{region}}.cloudapp.azure.com``. Log in as user ``cyber`` authenticated with private key ``jumphost.key``.

.. code-block:: bash

    $ ssh -i jumphost.key cyber@jumphost-aksdistrict{{site_ID}}.{{region}}.cloudapp.azure.com

Exercise 2: Kubernetes cluster
*********************
Logging into your Azure Kubernetes Service (AKS), a Managed Kubernetes Services:

- On Jumphost, communicate with K8S API using kubectl

.. code-block:: bash

    $ kubectl get namespaces

Exercise 3: NGINX Ingress Controller
************************************

Two *NGINX Ingress Controller* (IC) instances, *App Protect* module embedded,
have been already build on Jumphost following this `guide <https://docs.nginx.com/nginx-ingress-controller/installation/building-ingress-controller-image/#building-the-image-and-pushing-it-to-the-private-registry>`_
and deployed using `Helm <https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/>`_

    .. note:: **Capture The Flag**
        | What is the ingress-class name of the IC instance accessible from Internet?
        | nginx-external

    .. note:: **Capture The Flag**
        | **What is the version of deployed IC?**
        | Tips: *NGINX Ingress Controller* image's tag contains: {{IC version}}-{{last update of WAF signature}}. `docker commands <https://docs.docker.com/engine/reference/commandline/docker/>`_
        | 1.11.1

Exercise 4: Kibana
*****************************************

Kibana is published by Ingress Controller.

.. image:: ./_pictures/infra_resources.svg
   :align: center
   :width: 900
   :alt: infra

Kibana is protected by NGINX App Protect embedded in Ingress Controller.

.. image:: ./_pictures/infra_resources_elk.svg
   :align: center
   :width: 900
   :alt: ELK

Security events logs are sent to ELK.

.. image:: ./_pictures/infra_resources_nap_log.svg
   :align: center
   :width: 900
   :alt: NAP logs

Security dashboards are available on Kibana. Mode details `here <https://github.com/f5devcentral/f5-waf-elk-dashboards#screenshots>`_

.. image:: ./_pictures/dashboard1.png
   :align: center
   :width: 900
   :alt: NAP logs

- Download CA CRT `ca_demo.crt <https://f5-my.sharepoint.com/:f:/r/personal/al_dacosta_f5_com/Documents/Lab/f5-aks-kic-lab?csf=1&web=1&e=NcV9gX>`_
- Open ``ca_demo.crt`` >> ``Install Certificate...`` >> ``Current user`` >> Place certificate in ``Trusted Root Certification Authorities``

.. image:: ./_pictures/CA_CRT.png
   :align: center
   :width: 700
   :alt: CA CRT

- Using your web browser, try to reach ELK UI ``https://kibana{{site_ID}}.f5app.dev``... Damn it's DOWN!
- Restart the container using `docker commands <https://docs.docker.com/engine/reference/commandline/docker/>`_

.. code-block:: bash

    $ docker ps
    $ docker stop {{CONTAINER_ID}}
    $ docker ps --all
    $ docker start {{CONTAINER_ID}}

- Browse ELK UI ``https://kibana{{site_ID}}.f5app.dev`` >> Dashboard >> Overview and scroll to ``All Requests``

Exercise 5: Cryptonice
*****************************************
`Cryptonice <https://github.com/F5-Labs/cryptonice>`_ collects data on a given domain and performs a series of tests to check TLS configuration and supporting protocols such as HTTP2 and DNS.

- On Jumphost, evaluate SSL security for ``https://kibana{{site_ID}}.f5app.dev``

.. code-block:: bash

    $ docker run -v `pwd`:`pwd` -w `pwd` -i -t f5labs/cryptonice kibana{{site_ID}}.f5app.dev --json_out --no_console
    $ cat kibana{{site_ID}}.f5app.dev.json | jq .


    .. note:: **Capture The Flag**
        | **What is the cipher suite supported?**
        | ECDHE-RSA-AES256-GCM-SHA384

Exercise 6: NGINX+ API
*****************************************
NGINX Plus includes a `real‑time activity monitoring <https://www.nginx.com/products/nginx/live-activity-monitoring/>`_ interface that provides key load and performance metrics.
Using a simple RESTful JSON interface, it’s very easy to connect these stats to live dashboards and third‑party monitoring tools.

- On Jumphost, get a IC POD's IP

.. code-block:: bash
    :emphasize-lines: 3,6

    $ kubectl get pods -n external-ingress-controller
    NAME                                              READY   STATUS    RESTARTS   AGE
    nap-external-ingress-controller-7576b65b4-ps4ck   1/1     Running   0          8d

    $ kubectl describe pod -n external-ingress-controller nap-external-ingress-controller-7576b65b4-ps4ck | grep IP
    IP:           10.1.1.18

- On Jumphost, browse `NGINX Plus REST API <http://demo.nginx.com/swagger-ui/>`_

.. code-block:: bash

    $ curl 10.1.1.18:8080/api/6/nginx/

    .. note:: **Capture The Flag**
        | **Which build of NGINX is used by IC?**
        | nginx-plus-r23

Extra time: NGINX+ dashboard
*****************************************

- On your ssh client, configure ssh port forwarding on Jumphost session as described `here <https://blog.devolutions.net/2017/4/how-to-configure-an-ssh-tunnel-on-putty>`_

.. image:: ./_pictures/securecrt.png
   :align: center
   :width: 900
   :alt: SecureCRT

- On your web browser, connect to ``http://127.1.1.1/dashboard.html``

.. image:: ./_pictures/nginx_plush_dashboard.png
   :align: center
   :width: 900
   :alt: SecureCRT
