Infrastructure
##############################################################

.. contents:: Contents
    :local:

Exercise 1: NGINX Ingress Controller
************************************

In this lab, *NGINX Ingress Controller* (IC) is hosted in namespaces owned by Infra Ops,
this kind of namespace is called an **infrastructure namespace**.
An *infrastructure namespace* can host one or several ICs.
An IC is associated to a unique **Ingress Class** name.
Therefore an *infrastructure namespace* can propose to Applications several *Ingress Classes*,
for example "silver", "gold" and "platinium".

Why different *Ingress Class*?
Because an *Ingress Class* could offer different service level or be managed by different Infra Ops team,
for example: ADC, API GW, WAF, open source, supported, SRE team for digital factory etc.

In order to published, an Application have to select an *Ingress Class*.
However an Application, that is hosted in its namespace, cannot use every existing *Ingress Class*
because an *Ingress Class* can **watch** all or some *applicative namespace*.
For example an *infrastructure namespace* dedicated for non-production should only watch non-production *applicative namespace*.

Two ICs instances, *App Protect* module embedded,
have been already build on Jumphost following this `guide <https://docs.nginx.com/nginx-ingress-controller/installation/building-ingress-controller-image/#building-the-image-and-pushing-it-to-the-private-registry>`_
and deployed using `Helm <https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/>`_

- Show Arcadia application published by IC facing Internet

.. code-block:: bash

    kubectl get ingresses -n lab1-arcadia

_______________________________________________________________________

**Capture The Flag**

    **1.1 What is the version of deployed IC?**
    | Tips: *NGINX Ingress Controller* image's tag contains: {{IC version}}-{{last update of WAF signature}}. Use `docker commands <https://docs.docker.com/engine/reference/commandline/image_ls/>`_ to show images

    **1.2 What is the ingress-class name of the IC instance accessible from Internet?**

    **1.3 What is the Helm configuration parameter to limit Namespace(s) to watch?**
    | Tips: Configuration parameter for *NGINX Ingress Controller* `here <https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/#configuration>`_

Exercise 2: Kibana
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

- Download CA CRT `ca_demo.crt <https://raw.githubusercontent.com/nergalex/f5-aks-kic-lab/master/source/class2/module1/_files/ca_demo.crt>`_
- Open ``ca_demo.crt`` >> ``Install Certificate...`` >> ``Current user`` >> Place certificate in ``Trusted Root Certification Authorities``

.. image:: ./_pictures/CA_CRT.png
   :align: center
   :width: 700
   :alt: CA CRT

- Using your web browser, try to reach ELK UI ``https://kibana{{site_ID}}.f5app.dev``... Damn it's DOWN!
- Restart the container using `docker commands <https://docs.docker.com/engine/reference/commandline/docker/>`_

.. code-block:: bash

    docker ps

*output*

.. code-block:: bash

    CONTAINER ID   IMAGE          COMMAND                  CREATED      STATUS      PORTS                                                                                                                                                 NAMES
    3c87d89ab528   sebp/elk:742   "/usr/local/bin/starâ€¦"   5 days ago   Up 3 days   0.0.0.0:5144->5144/tcp, :::5144->5144/tcp, 0.0.0.0:5601->5601/tcp, :::5601->5601/tcp, 5044/tcp, 9300/tcp, 0.0.0.0:9200->9200/tcp, :::9200->9200/tcp   f5-waf-elk-dashboards_elasticsearch_1

- Note your {{CONTAINER_ID}} and restart it

.. code-block:: bash

    docker restart {{CONTAINER_ID}}

- Wait 3mn then browse ELK UI ``https://kibana{{site_ID}}.f5app.dev`` >> Dashboard >> Overview and scroll to ``All Requests``

Exercise 3: NGINX+ API
*****************************************
NGINX Plus includes a `real‑time activity monitoring <https://www.nginx.com/products/nginx/live-activity-monitoring/>`_ interface that provides key load and performance metrics.
Using a simple RESTful JSON interface, it’s very easy to connect these stats to live dashboards and third‑party monitoring tools.

- On Jumphost, get a IC POD's name`

.. code-block:: bash

    kubectl get pods -n external-ingress-controller

*output*

.. code-block:: bash
    :emphasize-lines: 2

    NAME                                              READY   STATUS    RESTARTS   AGE
    nap-external-ingress-controller-7576b65b4-ps4ck   1/1     Running   0          8d

- Get a IC POD's IP. Replace ``{{POD_name}}`

.. code-block:: bash

    kubectl describe pod -n external-ingress-controller ``{{POD_name}}`` | grep IP

*output*

.. code-block:: bash

    IP:           10.1.1.18

- On Jumphost, browse `NGINX Plus REST API <http://demo.nginx.com/swagger-ui/>`_. Replace ``{{POD_ip}}``

.. code-block:: bash

    $ curl ``{{POD_ip}}``:8080/api/6/nginx/

_______________________________________________________________________

**Capture The Flag**

    **3.1 Which build of NGINX is used by IC?**

Extra time 1: Cryptonice
*****************************************
`Cryptonice <https://github.com/F5-Labs/cryptonice>`_ collects data on a given domain
and performs a series of tests to check TLS configuration and supporting protocols such as HTTP2 and DNS.

- On Jumphost, evaluate SSL security for ``https://kibana{{site_ID}}.f5app.dev``

.. code-block:: bash

    docker run -v `pwd`:`pwd` -w `pwd` -i -t f5labs/cryptonice kibana{{site_ID}}.f5app.dev --json_out --no_console

*output*

.. code-block:: bash

    Pre-scan checks
    -------------------------------------
    Scanning kibana1.f5app.dev on port 443...
    Analyzing DNS data for kibana1.f5app.dev
    Fetching additional records for kibana1.f5app.dev
    kibana1.f5app.dev resolves to 20.75.112.65
    20.75.112.65:443: OPEN
    TLS is available: True
    Connecting to port 443 using HTTPS
    Reading HTTP headers for kibana1.f5app.dev
    Queueing TLS scans (this might take a little while...)
    Looking for HTTP/2

    Scans complete
    -------------------------------------
    Total run time: 0:00:03.059256

    Outputting data to ./kibana1.f5app.dev.json

- View evaluation results

.. code-block:: bash

    cat kibana{{site_ID}}.f5app.dev.json | jq .

_______________________________________________________________________

**Capture The Flag**

    **extra 2.1 What is the supported cipher suite?**

Extra time 2: NGINX+ dashboard
*****************************************

- For Windows Users:
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


- For Mac Users:
    - Run the command below
    - Replace {{IC_POD_IP}} with the IP address found in exercise 6 above
    - Replace {{site_ID} and {{region}} with your allocated site ID and Azure region.


    .. code-block:: bash

        ssh -L 8090:{{IC_POD_IP}}:8080 -i jumphost.pem cyber@jumphost-aksdistrict{{site_ID}}.{{region}}.cloudapp.azure.com


    - On your browser, connect to ``http://127.0.0.1:8090/dashboard.html``


    .. image:: ../../_pictures/nginx_plush_dashboard_2.png
       :align: center
       :width: 900
       :alt: SecureCRT





