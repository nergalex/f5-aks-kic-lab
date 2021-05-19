Sentence
#################################################################

.. contents:: Contents
    :local:
    :depth: 1

Web UI access
*********************************************

.. image:: ./_pictures/sentence-flow.svg
   :align: center
   :width: 900
   :alt: Web UI flows

Exercise 1: Flow 2 | Client >> Frontend
============================================

- Using your web browser, try to access to ``https://sentence{{site_ID}}.f5app.dev``

*output:*

.. image:: ./_pictures/Sentence_front_output.png
   :align: center
   :width: 800
   :alt: Sentence Web

- Check that web frontend access is well protected by NGINX App Protect: ``https://sentence{{site_ID}}.f5app.dev?<script>``

*output:*

.. code-block:: html

    <html><head><title>Request Rejected</title></head><body>The requested URL was rejected.
    Please consult with your administrator.<br><br>
    Your support ID is: 4096465330496922252
    <br><br><a href='javascript:history.back();'>[Go Back]</a></body></html>

Exercise 2: Flow 3 | Frontend >> Internal API
=============================================

- Install ``curl``

.. code-block:: bash

    kubectl create ns debug
    kubectl run multitool --image=praqma/network-multitool -n debug
    kubectl exec -it multitool -n debug -- sh

- Connect to `Internal API endpoint /name <https://github.com/fchmainy/nginx-aks-demo/blob/main/Docker/frontend-namespace-via-apigw/index.php#L25>`_

.. code-block:: bash

    curl http://apigw-microapigw.lab4-sentence-api/name

*output:*

.. code-block:: json

    {
      "adjectives": "calm",
      "animals": "whale",
      "colors": "green",
      "locations": "valley"
    }


Exercise 3: Flow 4 | Generator >> Internal API
=============================================

- Connect to `Internal API endpoint /adjectives <https://github.com/fchmainy/nginx-aks-demo/blob/main/Docker/generator-via-api-gw/generator.py#L57>`_

.. code-block:: bash

    curl http://apigw-microapigw.lab4-sentence-api/adjectives

*output:*

.. code-block:: json

    [
      {
        "id": 1,
        "name": "kind"
      },
      {
        "id": 2,
        "name": "proud"
      },
      {
        "id": 3,
        "name": "calm"
      }
    ]

Partner API access
*********************************************

.. image:: ./_pictures/sentence-flow_api.svg
   :align: center
   :width: 900
   :alt: API flows

Flow 2-7: Functional view
=============================================

Partner API access enable OpenID Connect integration for NGINX Plus as described `here <https://github.com/fchmainy/nginx-aks-demo/blob/main/Docker/generator-via-api-gw/generator.py#L57>`_.

.. image:: ./_pictures/OIDC_overview.svg
   :align: center
   :width: 600
   :alt: OIDC

Implementation done for this lab:
    - Okta as an identity provider (IdP)
    - authorization code flow
    - NGINX Ingress Controller is configured as a relying party
    - Okta knows our internal Ingress Controller as a confidential client or a public client using PKCE

In this lab, both the client and NGINX Ingress Controller communicate directly with the IdP at different stages during the initial authentication event.

.. image:: ./_pictures/OIDC_details.svg
   :align: center
   :width: 700
   :alt: OIDC

Exercise 1: Flow 3 | Client >> Okta
=============================================





