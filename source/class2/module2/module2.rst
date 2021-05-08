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

Arcadia is published through ingress class External using `mergeable Ingress <https://github.com/nginxinc/kubernetes-ingress/tree/v1.11.1/examples/mergeable-ingress-types>`_.

.. image:: ./_pictures/arcadia_resources.svg
   :align: center
   :width: 800
   :alt: App resources

Ingress configuration a spread for host ``arcadia{{site_ID}}.f5app.dev`` across multiple Ingress resources using Mergeable Ingress resources.
Here all resources belong to a same namespace ``lab1-arcadia`` but it could be different namespaces.
This enables easier management when using a large number of paths.

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

Page rendering generates requests to each micro-service routed by Ingress Controller.

.. image:: ./_pictures/arcadia_flow.svg
   :align: center
   :width: 800
   :alt: App flow

- Connect to ``https://arcadia{{site_ID}}.f5app.dev/``
- Login with user ``matt`` and password ``ilovef5``
