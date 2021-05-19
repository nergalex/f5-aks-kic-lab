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
   :width: 600
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
   :width: 500
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

- Using a Web Browser, connect to `oidcdebugger <https://oidcdebugger.com/>`_
- Fill the form:
    - Authorize URI: ``https://dev-431905.okta.com/oauth2/aus2zvhijqcz1rlq84x7/v1/authorize``
    - Redirect URI: ``https://oidcdebugger.com/debug``
    - Client ID: ``0oa2zvfps3gsoYGHm4x7``
    - Scope: ``openid``
    - State: ``France``
    - Nonce: ``39pog581mp9``
    - Response type: ``code``
    - Response mode: ``query``

.. image:: ./_pictures/okta_OIDC_debugger.png
   :align: center
   :width: 500
   :alt: OIDC debugger - query

- Authenticate user with username ``cloudbuilder@acme.com`` and password ``F5-AKS-KIC-lab!``

.. image:: ./_pictures/okta_OIDC_debugger_auth.png
   :align: center
   :width: 500
   :alt: OIDC debugger - auth

- You should receive a response code

.. image:: ./_pictures/okta_OIDC_debugger_response.png
   :align: center
   :width: 500
   :alt: OIDC debugger - response

Exercise 2: API GW - K8S configuration
=================================================

NGINX Ingress Controller is configured to perform OpenID Connect authentication.

- View OIDC configuration in VirtualServerRoute resource

.. code-block:: bash

    kubectl describe virtualserverroute -n lab4-sentence-api generator | grep -A 13 Spec

*output:*

.. code-block:: yaml
    :emphasize-lines: 8

      Host:                sentence-api1.f5app.dev
      Ingress Class Name:  sentence-api-nginx-internal
      Subroutes:
        Action:
          Pass:  generator
        Path:    /
        Policies:
          Name:       oidc-policy
          Namespace:  lab4-sentence-api
      Upstreams:
        Name:     generator
        Port:     80
        Service:  generator

- View OIDC policy resource

.. code-block:: bash

    kubectl describe policy -n lab4-sentence-api oidc-policy | grep -A 7 Spec

*output:*

.. code-block:: yaml

      Oidc:
        Auth Endpoint:   https://dev-431905.okta.com/oauth2/aus2zvhijqcz1rlq84x7/v1/authorize
        Client ID:       0oa2zvfps3gsoYGHm4x7
        Client Secret:   oidc-secret
        Jwks URI:        https://dev-431905.okta.com/oauth2/aus2zvhijqcz1rlq84x7/v1/keys
        Scope:           openid
        Token Endpoint:  https://dev-431905.okta.com/oauth2/aus2zvhijqcz1rlq84x7/v1/token

**Capture The Flag**

    **2.1 What is the URI that publish Public Key used by IC to check signature of JWT?**
    | https://dev-431905.okta.com/oauth2/aus2zvhijqcz1rlq84x7/v1/keys

    **2.2 What is the secret length in bytes?**
    | 40

Exercise 3: API GW - NGINX configuration
=================================================

- Connect to internal IC as API GW
- View configuration in Location block

.. code-block:: bash

    grep oidc /etc/nginx/conf.d/vs_lab4-sentence-api_sentence-api-internal.conf

*output:*

.. code-block:: nginx
    :emphasize-lines: 1

    include oidc/oidc.conf;
    set $oidc_pkce_enable 0;
    set $oidc_logout_redirect "/_logout";
    set $oidc_hmac_key "sentence-api-internal";
    set $oidc_authz_endpoint "https://dev-431905.okta.com/oauth2/aus2zvhijqcz1rlq84x7/v1/authorize";
    set $oidc_token_endpoint "https://dev-431905.okta.com/oauth2/aus2zvhijqcz1rlq84x7/v1/token";
    set $oidc_jwt_keyfile "https://dev-431905.okta.com/oauth2/aus2zvhijqcz1rlq84x7/v1/keys";
    set $oidc_scopes "openid";
    set $oidc_client "0oa2zvfps3gsoYGHm4x7";
    set $oidc_client_secret "-lzR61blfzTcU05FEohmXZ0HwPhjZJPGAPmJpSk-";

- View configuration of OIDC code exchange request received by client

.. code-block:: bash

    grep -A 5 _codexch /etc/nginx/oidc/oidc.conf

*output:*

.. code-block:: nginx
    :emphasize-lines: 1

    #set $redir_location "/_codexch";
    location = /_codexch {
        # This location is called by the IdP after successful authentication
        status_zone "OIDC code exchange";
        js_content oidc.codeExchange;
        error_page 500 502 504 @oidc_error;
    }

- View configuration of OIDC token request sent by NGINX Ingress Controller

.. code-block:: bash

    grep -A 5 /_token /etc/nginx/oidc/oidc.conf

*output:*

.. code-block:: nginx
    :emphasize-lines: 1

    #set $redir_location "/_codexch";
    location = /_codexch {
        # This location is called by the IdP after successful authentication
        status_zone "OIDC code exchange";
        js_content oidc.codeExchange;
        error_page 500 502 504 @oidc_error;
    }


Exercise 3: API GW - OIDC configuration
=================================================

Upon a first visit to a protected resource, NGINX Plus initiates the OpenID Connect authorization code flow and redirects the client to the OpenID Connect provider (IdP). When the client returns to NGINX Plus with an authorization code, NGINX Plus exchanges that code for a set of tokens by communicating directly with the IdP.