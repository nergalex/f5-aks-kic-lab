Infrastructure
##############################################################

.. contents:: Contents
    :local:

Jumphost
*********************

- Download SSH key `jumphost.key <https://f5-my.sharepoint.com/:f:/r/personal/al_dacosta_f5_com/Documents/Lab/f5-aks-kic-lab?csf=1&web=1&e=PYcBdc>`_
- Ask F5 for your ``{{site_ID}}`` and your Azure ``{{region}}``
- Open an SSH session to ``jumphost-aksdistrict{{site_ID}}.{{region}}.cloudapp.azure.com``. Log in as user ``cyber`` authenticated with private key ``jumphost.key``.

.. code-block:: bash

    ssh -i jumphost.key cyber@jumphost-aksdistrict{{site_ID}}.{{region}}.cloudapp.azure.com

Kubernetes cluster
*********************
Azure Kubernetes Service (AKS) is used as a Managed Kubernetes Services.
However all labs and setups of the workshop will work with any other K8S distribution.

On Jumphost, communicate with K8S API using kubectl

.. code-block:: bash

    $ kubectl get namespaces

K8S resources
*********************

Two *NGINX Ingress Controller* instances, *App Protect* module embedded, have been already deployed using `Helm <https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/>`_

.. warning::
    **Capture The Flag**

    What is the ingress-class name of the *NGINX Ingress Controller* instance accessible from Internet

ELK UI ``https://kibana{{site_ID}}.f5app.dev`` is published by Ingress Controller.

.. image:: ./_pictures/infra_resources.svg
   :align: center
   :width: 900
   :alt: infra

ELK UI is protected by NGINX App Protect embedded in Ingress Controller.

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













