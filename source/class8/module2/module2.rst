Secure Access
#################################################################

JSON Web Tokens (JWTs, pronounced “jots”) are a compact and highly portable means of exchanging identity information.
The JWT specification has been an important underpinning of OpenID Connect,
providing a single sign‑on token for the OAuth 2.0 ecosystem.
JWTs can also be used as authentication credentials in their own right
and are a better way to control access to web‑based APIs than traditional API keys.

Powered by the most popular data plane in the world,
F5 Distributed Cloud (XC) *Secure Access* improves your security posture:
    - **Reduce the attack surface**: Require **access control** for Web and API based Application
    - **Least Privilege**: Validate the claimed Scope against per App or micro-service security policy
    - **Prevent malicious activity**: Track identified user activities, detect behavioral anomalies and auto-mitigate with `F5 XC Malicious User <https://docs.cloud.f5.com/docs/how-to/advanced-security/malicious-users>`_ feature.

.. image:: ./_pictures/illo-SolutionsZeroTrust-450x400-1.svg
   :align: center
   :width: 300
   :alt: Secure Access

.. contents:: Contents
    :local:
    :depth: 2

What is an OAuth 2.0 Grant Type?
***************************************************************
In OAuth 2.0, the term “grant type” refers to the way an application gets an access token.
OAuth 2.0 defines several grant types, including the authorization code flow.
Each grant type is optimized for a particular use case,
whether that’s a web app, a native app, a device without the ability to launch a web browser,
or server-to-server applications.

oAuth/OIDC authentication for user access [Authorization Code]
***************************************************************
*Secure Access* implementation assumes the following environment:
    - The Identity Provider (IdP) supports OpenID Connect 1.0
    - The **authorization code** flow is in use
    - F5 XC is seen as a relying party
    - The IdP knows F5 XC as a confidential client or a public client using PKCE

The Authorization Code Flow
================================================================
The Authorization Code grant type is used by web and mobile apps.
It differs from most of the other grant types by first requiring the app launch a browser to begin the flow.
At a high level, the flow has the following steps:
    - The application opens a browser to send the user to the OAuth server
    - The user sees the authorization prompt and approves the app’s request
    - The user is redirected back to the application with an authorization code in the query string
    - The application exchanges the authorization code for an access token

All these steps above are managed by F5 XC *secure access*, exactly by the NGINX gateway as a relying party,
that free up the application about implementing this authentication feature.

Relying party
================================================================
With F5 XC *Secure Access*,
both the client and the NGINX gateway communicate directly with the IdP at different stages during the initial authentication event.

.. image:: ./_pictures/high_level_flow.svg
   :align: center
   :width: 400
   :alt: Flow high level

NGINX Plus is configured to perform OpenID Connect authentication:
    - **1.** Upon a first visit to a protected resource,
    - **2.** NGINX Plus initiates the OpenID Connect authorization code flow and redirects the client to the OpenID Connect provider (IdP).
    - **3-5.** The IdP verifies the identity of the end-user.
    - **6.** When the client returns to NGINX Plus with an authorization code,
    - **7-8.** NGINX Plus exchanges that code for a set of tokens by communicating directly with the IdP.
    - **9.** The ID Token received from the IdP is validated. NGINX Plus then stores the ID token in the key-value store, issues a session cookie to the client using a random string, (which becomes the key to obtain the ID token from the key-value store)
    - **10.** and redirects the client to the original URI requested prior to authentication.
    - **11.** Subsequent requests to protected resources are authenticated by exchanging the session cookie for the ID Token in the key-value store.
    - **12.** JWT validation is performed on **each request**, as normal, so that the ID Token validity period is enforced.

.. image:: ./_pictures/flow.svg
   :align: center
   :width: 700
   :alt: Flow

----------------------------------------------------

*demo video:*

.. raw:: html

    <a href="http://www.youtube.com/watch?v=_SOcDFslFl4"><img src="http://img.youtube.com/vi/_SOcDFslFl4/0.jpg" width="600" height="300" title="XC - PaaS NGINX Secure Access"></a>

The *Secure Access* PaaS is built using the reference implementation of NGINX Plus as relying party for OpenID Connect authentication,
please refer to it `here <https://github.com/nginxinc/nginx-openid-connect>`_ for more details.

For more information on OpenID Connect and JWT validation with NGINX Plus, see `Authenticating Users to Existing Applications with OpenID Connect and NGINX Plus <https://www.nginx.com/blog/authenticating-users-existing-applications-openid-connect-nginx-plus/>`_.

Secure Access as a Shared Service
================================================================
F5 Distributed Cloud (XC), as a Multi-Cloud Networking Software (MCNS),
allows to insert any container based service in the data-path,
named **Platform as a Service** (PaaS).

The *Secure Access* PaaS can be deployed per Application namespace or as **Shared Service** consumed by any  Application namespace.
The App owner is free to leverage the *Secure Access* (oAuth/OIDC authentication) for the whole web site (per HTTP LB, per FQDN)
or per micro-service (per FQDN, per PATH, per client source). For example the private part of a web site ``/my-account`` requires to be authenticated.
How to? In the Public HTTP LB (blue) Route configuration, the selected Origin Pool is the *Secure Access PaaS* service

.. image:: ./_pictures/flow_design.svg
   :align: center
   :width: 700
   :alt: LLD Flow

Once authenticated, the *Secure Access PaaS* gateway forward the request to the internal HTTP LB (pink).

Multiple IdPs support
================================================================
*Secure Access* PaaS supports any Identity Provider that supports OpenID Connect 1.0.
Where *Secure Access* PaaS is configured to proxy requests for multiple websites or applications, or user groups,
these may require authentication by different IdPs.
Separate IdPs can be configured, with each one matching on an attribute of the HTTP request,
e.g. hostname or part of the URI path for example.

.. image:: ./_pictures/http_route.png
   :align: center
   :width: 500
   :alt: Public HTTP LB Route

And a custom header ``x-my-idp`` is added (or replaced if existing) to define the IdP name that the *Secure Access* gateway will use then to authenticate the user.

.. image:: ./_pictures/header-my-idp.png
   :align: center
   :width: 500
   :alt: Public HTTP LB Route

Grant limited access to applications
================================================================
OAuth is all about enabling users to grant limited access to applications.
The application first needs to decide which permissions it is requesting,
then send the user to a browser to get their permission.

The application defines the permission to request, **scope**, in their F5 XC Public LB.

Custom scope per App
================================================================
Same as done for IdP selection mechanism,
the App owner can define the scope to be allowed for a DNS domain or per Path.
For example: ``/admin`` for administrator scope only.
The App owner defines set the Scope value in a custom header `x-my-scope`, at the HTTP Route level:

.. image:: ./_pictures/x-my-scope.png
   :align: center
   :width: 500
   :alt: x-my-scope

This custom header ``x-my-idp`` content will be used by the *Secure Access* gateway during the authentication.

.. image:: ./_pictures/x-my-scope-n1.png
   :align: center
   :width: 500
   :alt: NGINX One dynamic scope

Secure Access gateways managed by the F5 XC SaaS console
================================================================
*Secure Access* PaaS is NGINX+ enterprise grade instances, deployed in F5 Edges
hosted anywhere (Public Cloud, Private Cloud, F5 hardware, VM),
and managed centrally in `NGINX One console <https://www.f5.com/fr_fr/products/nginx/one>`_.


.. image:: ./_pictures/nginx-one.png
   :align: center
   :width: 1200
   :alt: Configuration files


oAuth JWT validation, for API key access
***************************************************************

Behavioral based prevention, for Malicious User access and activities
************************************************************************
Combined with `F5 XC Malicious User <https://docs.cloud.f5.com/docs/how-to/advanced-security/malicious-users>`_ feature,
IT and security operations teams can enforce strong access policies from login to logout,
including step-up challenges for suspect behavior.

----------------------------------------------------

*demo video:*

.. raw:: html

    <a href="http://www.youtube.com/watch?v=-F-QV-IJI9o"><img src="http://img.youtube.com/vi/-F-QV-IJI9o/0.jpg" width="600" height="300" title="XC - Malicious User + Client Side Defense"></a>

Detection
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
*Malicious User* is part of the `F5 XC User Behavior Analysis <https://docs.cloud.f5.com/docs/how-to/app-security/user-behavior-analysis>`_ that detects suspect behavior based on watching several dimensions:

.. image:: ./_pictures/Malicious_User_detection.png
   :align: center
   :width: 500
   :alt: Detection

Mitigation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If a client tries to access resources with insufficient privileges,
NGINX gateway will return a 401 response code.
After several attempts (10 by default),
a security event will be raised and a step-up challenge will be send.
The system tags the users into threat levels High, Medium, and Low.

.. image:: ./_pictures/Malicious_User_mitigation.png
   :align: center
   :width: 800
   :alt: Mitigation

If the client succeed to resolve challenges and continues to tries to access resources with insufficient privileges,
then client will be blocked.
The system automatically reduces the score when there is no malicious behavior detected for the user for a period of time.
This is known as the ``Cooling Off Period``.
This period indicates how long it takes to reduce from High threat level to Low.
The system executes a score decay mechanism over a period of time for this to happen.

User Identifier
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
By default, the identifier for a malicious user is Client IP address.
If end user is already logged in, the user identifier is the NGINX session cookie (see step 11 in § *Access control*).
Therefore, even if the client changes his IP address, his behavior will be tracked, based on its session cookie.

.. image:: ./_pictures/User_Identifier.png
   :align: center
   :width: 500
   :alt: User Identifier



