Arcadia
##############################################################

.. image:: ./_pictures/arcadia-ui.png
   :align: center
   :width: 800
   :alt: App design

.. contents:: Contents
    :local:

App Design
*********************

Arcadia app is split between 4 micro-services.
More details `here <https://rtd-nginx-app-protect-udf.readthedocs.io/en/latest/class1/module1/module1.html>`_

.. image:: ./_pictures/arcadia-api.png
   :align: center
   :width: 800
   :alt: App design

K8S resources
*********************

Arcadia is published through ingress class External.

.. image:: ./_pictures/arcadia_resources.svg
   :align: center
   :width: 900
   :alt: App resources

Page rendering generates requests to each micro-service routed by Ingress Controller.

.. image:: ./_pictures/arcadia_flow.svg
   :align: center
   :width: 900
   :alt: App flow

- Connect to ``https://arcadia{{site_ID}}.f5app.dev/``
- Login with user ``matt`` and password ``ilovef5``

Master / Minions
===================

Arcadia is published using `mergeable Ingress <https://github.com/nginxinc/kubernetes-ingress/tree/v1.11.1/examples/mergeable-ingress-types>`_.

Ingress configuration a spread for host ``arcadia{{site_ID}}.f5app.dev`` across multiple Ingress resources using Mergeable Ingress resources.
Here all resources belong to a same namespace ``lab1-arcadia`` but it could be different namespaces.
This enables easier management when using a large number of paths.

- Connect to a IC container

.. code-block:: bash

    $ kubectl get pods -n external-ingress-controller
    NAME                                              READY   STATUS    RESTARTS   AGE
    nap-external-ingress-controller-7576b65b4-ps4ck   1/1     Running   0          8d
    nap-external-ingress-controller-7576b65b4-w599m   1/1     Running   0          8d

    $ kubectl exec --namespace external-ingress-controller -it nap-external-ingress-controller-7576b65b4-ps4ck bash

- Show Arcadia configuration

.. code-block:: bash

    more /etc/nginx/conf.d/lab1-arcadia-arcadia-ingress-external-master.conf

- Check that configuration of Arcadia is a merge results of a master and minions

.. code-block:: nginx

    server {
            # configuration for lab1-arcadia/arcadia-ingress-external-master

            location /api {
                    # location for minion lab1-arcadia/arcadia-ingress-external-minion-app2
            }

            location /app3 {
                    # location for minion lab1-arcadia/arcadia-ingress-external-minion-app3
            }

            location /files {
                    # location for minion lab1-arcadia/arcadia-ingress-external-minion-backend
            }

            location / {
                    # location for minion lab1-arcadia/arcadia-ingress-external-minion-main
            }
    }

- Show ingress resources for Arcadia

.. code-block:: bash
    $ kubectl get ingresses -n lab1-arcadia

    NAME                                      CLASS            HOSTS                ADDRESS        PORTS     AGE
    arcadia-ingress-external-master           nginx-external   arcadia1.f5app.dev   20.75.112.65   80, 443   5d15h
    arcadia-ingress-external-minion-app2      nginx-external   arcadia1.f5app.dev   20.75.112.65   80        5d15h
    arcadia-ingress-external-minion-app3      nginx-external   arcadia1.f5app.dev   20.75.112.65   80        5d15h
    arcadia-ingress-external-minion-backend   nginx-external   arcadia1.f5app.dev   20.75.112.65   80        5d15h
    arcadia-ingress-external-minion-main      nginx-external   arcadia1.f5app.dev   20.75.112.65   80        5d15h

- Show Master's ingress resources for Arcadia

.. code-block:: bash
    $ kubectl describe ingress -n lab1-arcadia arcadia-ingress-external-master

    Name:             arcadia-ingress-external-master
    Namespace:        lab1-arcadia
    Address:          {{site_ID}}
    Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
    TLS:
      arcadia-secret-tls terminates arcadia1.f5app.dev
    Rules:
      Host        Path  Backends
      ----        ----  --------
      *           *     default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
    Annotations:  appprotect.f5.com/app-protect-enable: True
                  appprotect.f5.com/app-protect-policy: external-ingress-controller/generic-security-level-low
                  appprotect.f5.com/app-protect-security-log: external-ingress-controller/naplogformat
                  appprotect.f5.com/app-protect-security-log-destination: syslog:server=10.{{site_ID}}.0.10:5144
                  appprotect.f5.com/app-protect-security-log-enable: True
                  ingress.kubernetes.io/ssl-redirect: true
                  nginx.org/mergeable-ingress-type: master
                  nginx.org/server-snippets:
                    proxy_ignore_headers X-Accel-Expires Expires Cache-Control;
                    proxy_cache_valid any 30s;

.. note:: **Capture The Flag**
    | **What is the cookie name that allow a login user to persist his session on "Money Transfer" micro-service of Arcadia across multiple ICs?**
    | arcadia_app2
    | Tip:  `Session Persistence <https://github.com/nginxinc/kubernetes-ingress/tree/v1.11.1/examples/session-persistence>`_

Advanced Configuration
======================
**Annotation**

The Ingress resource only allows you to use basic NGINX features – host and path-based routing and TLS termination.
For more advanced features like rewriting the request URI or inserting additional response headers,
annotations `here <https://docs.nginx.com/nginx-ingress-controller/configuration/ingress-resources/advanced-configuration-with-annotations/#summary-of-annotations>`_
can be applied to an Ingress resource that allow to use advanced NGINX features and customize/fine tune NGINX behavior for that Ingress resource.

.. code-block:: yaml

    annotations:
      nginx.org/mergeable-ingress-type: "master"

**Snippets**

One annotation available is Snippets.
Snippets allow you to insert raw NGINX config into different contexts of the NGINX configurations that the Ingress Controller generates.
These should be used as a last-resort solution in cases where annotations entries cannot help.

.. code-block:: yaml

    annotations:
      nginx.org/server-snippets: |
        proxy_ignore_headers X-Accel-Expires Expires Cache-Control;
        proxy_cache_valid any 30s;

.. note:: **Capture The Flag**
    | **What is the nginx directive seen in configuration for snippet 'proxy_ignore_headers'?**
    | proxy_ignore_headers

Disadvantages
=============
Annotation and Snippets have the following disadvantages:

- **Complexity**
    - Annotation can not reference an unique object, therefore:
        - all your configurations is flat and become quickly difficult to read
        - part of configuration is copied for each Master or Minion, for example jwt annotations. A simple change will impact a lot of resources and become risky.
    - To use snippets, you will need to:
        - Understand NGINX configuration primitives and implement a correct NGINX configuration.
        - Understand how the IC generates NGINX configuration so that a snippet doesn’t interfere with the other features in the configuration.
- **Decreased robustness**
    - An incorrect snippet makes the NGINX config invalid, which causes reload failures. This will prevent any new configuration updates, including updates for the other Ingress resources, until the snippet is fixed.
- **Security implications**
    - Snippets give access to NGINX configuration primitives and those primitives are not validated by the Ingress Controller. For example, a snippet can configure NGINX to serve the TLS certificates and keys used for TLS termination for Ingress resources.

**Note**: If the NGINX config includes an invalid snippet, NGINX will continue to operate with the latest valid configuration





