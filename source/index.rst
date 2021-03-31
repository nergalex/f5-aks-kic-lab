Workshop Kubernetes Ingress Controller with NGINX+
##################################################

.. note:: For any remark or mistake in the lab, please send an email to xxxx@f5.com.


NGINX Ingress Controller, also know as NGINX Ingress Controller for Kubernetes, comes in three different versions:

    #. Commercial: NGINX Plus
    #. Opensource: NGINX official mainline build available on https://github.com/nginxinc/docker-nginx
    #. Kubernetes Community: Custom NGINX build available on https://github.com/kubernetes/ingress-nginx/tree/master/images/nginx

.. note:: Check out our `GitHub <https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/nginx-ingress-controllers.md>`_ for a comparison of the three options.


.. image:: ./images/nginx_kic.svc
   :align: center


Below are the key characteristics that NGINX Plus brings:

**Additional features**

- Real-time metrics: A number of metrics about how NGINX Plus and applications are performing are available through the API or a built-in dashboard. Optionally, the metrics can be exported to Prometheus.
- Additional load balancing methods: additional methods are available for HTTP, TCP, and UDP load balancing: least_time and random two with least_time.
- Session persistence: In addition to the hash‑based session persistence supported by NGINX Open Source (the Hash and IP Hash load‑balancing methods), NGINX Plus supports cookie‑based session persistence
- Active health checks: By default NGINX performs basic checks on responses from upstream servers, retrying failed requests where possible. NGINX Plus adds out-of-band application health checks (also known as synthetic transactions) and a slow‑start feature to gracefully add new and recovered servers into the load‑balanced group. for HTTP and Health checks can also be enabled for non-HTTP protocols, such as FastCGI, memcached, SCGI, uwsgi, and also for TCP and UDP..
- JWT validation
- Optional F5 Based Web Application Firewall module (NGINX App Protect)

**Dynamic reconfiguration**

- Every time the number of pods of services you expose via an Ingress resource changes, the Ingress controller updates the configuration of the load balancer to reflect those changes.
For NGINX, the configuration file must be changed and the configuration subsequently reloaded. For NGINX Plus, the dynamic reconfiguration is utilized, which allows NGINX Plus to be updated on-the-fly without reloading the configuration. This prevents increase of memory usage during reloads, especially with a high volume of client requests, as well as increased memory usage when load balancing applications with long-lived connections (WebSocket, applications with file uploading/downloading or streaming).

**Commercial support**

- Support from NGINX Inc is available for NGINX Plus Ingress controller.


In that Workshop we will use NGINX+ to demonstrate some of his many added values and differentiators.
Each lab will focus on a specific use case where NGINX+ can simplify or add strong values with extra features.

The use cases are:

- fssqf
- fdsq






