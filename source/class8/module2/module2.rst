Secure Access
#################################################################

Powered by the most popular data plane in the world,
NGINX's *Secure Access* PaaS solution improves your security posture:
    - **Minimize attack surface**: Reduce the accessible attack surface through **access control**
    - **Prevent unauthorized activity**: Automatically block ongoing attacks through constant authentication, identity validation and detection of behavioral anomalies coupling with `F5 XC Malicious User <https://docs.cloud.f5.com/docs/how-to/advanced-security/malicious-users>`_ feature based on IA/ML.
    - **Single Sign-On**: enable Single Sign-On (SSO) for your proxied applications by integrating with your Identity Provider and OpenID Connect

.. image:: ./_pictures/illo-SolutionsZeroTrust-450x400-1.svg
   :align: center
   :width: 300
   :alt: Secure Access

.. contents:: Contents
    :local:
    :depth: 1

Access control
***************************************************************
With *NGINX Secure Access*,
both the client and NGINX Plus communicate directly with the IdP at different stages during the initial authentication event.

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

*NGINX Secure Access* PaaS solution is built using the reference implementation of NGINX Plus as relying party for OpenID Connect authentication,
please refer to it `here <https://github.com/nginxinc/nginx-openid-connect>`_ for more details.

For more information on OpenID Connect and JWT validation with NGINX Plus, see `Authenticating Users to Existing Applications with OpenID Connect and NGINX Plus <https://www.nginx.com/blog/authenticating-users-existing-applications-openid-connect-nginx-plus/>`_.

Malicious User
***************************************************************
Combined with `F5 XC Malicious User <https://docs.cloud.f5.com/docs/how-to/advanced-security/malicious-users>`_ feature,
IT and security operations teams can enforce strong access policies from login to logout,
including step-up challenges for suspect behavior.

----------------------------------------------------

*demo video:*

.. raw:: html

    <a href="http://www.youtube.com/watch?v=-F-QV-IJI9o"><img src="http://img.youtube.com/vi/-F-QV-IJI9o/0.jpg" width="600" height="300" title="XC - Malicious User + Client Side Defense"></a>

Detection
=============================================
*Malicious User* is part of the `F5 XC User Behavior Analysis <https://docs.cloud.f5.com/docs/how-to/app-security/user-behavior-analysis>`_ that detects suspect behavior based on watching several dimensions:

.. image:: ./_pictures/Malicious_User_detection.png
   :align: center
   :width: 500
   :alt: Detection

Mitigation
=============================================
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
=============================================
By default, the identifier for a malicious user is Client IP address.
If end user is already logged in, the user identifier is the NGINX session cookie (see step 11 in § *Access control*).
Therefore, even if the client changes his IP address, his behavior will be tracked, based on its session cookie.

.. image:: ./_pictures/User_Identifier.png
   :align: center
   :width: 500
   :alt: User Identifier

Supported Identity Providers
***************************************************************
Any Identity Provider that supports OpenID Connect 1.0.

Implementation of *NGINX Secure Access* assumes that your IdP knows F5 XC as a confidential client or a public client using PKCE.
By following the steps in `this guide <https://docs.nginx.com/nginx/deployment-guides/single-sign-on/okta/>`_,
you will learn how to set up SSO using OpenID Connect as the authentication mechanism with well known IdPs and NGINX Plus as the relying party.

PaaS configuration files can be edited in F5 XC Console directly.

.. image:: ./_pictures/paas_configuration_files.png
   :align: center
   :width: 500
   :alt: Configuration files

See bellow some configuration examples for the use of
`auth code flow paired with Proof Key for Code Exchange (PKCE) and OpenID Connect (OIDC) <https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-auth-code-flow#request-an-authorization-code>`_
to get access tokens and ID tokens in these types of apps:
    - Single Page Application (SPA)
    - Standard (server-based) web application
    - Desktop and mobile apps

Azure AD
=============================================

*demo video:*

.. raw:: html

    <a href="http://www.youtube.com/watch?v=0OfIQUqWw6g"><img src="http://img.youtube.com/vi/0OfIQUqWw6g/0.jpg" width="600" height="300" title="XC Secure GW - Azure AD"></a>

A guide `here <https://learn.microsoft.com/en-us/azure/active-directory/develop/scenario-spa-app-registration>`_ to configure a SPA with Azure AD.
After registering your 'PaaS Secure Access' instance as an Application in Azure AD, you will obtain a *Client ID*,
or ``$oidc_client``, a Public identifier for the client that is required for all OAuth flows.

*Secure Access* configuration file ``openid_connect_configuration.conf``:

.. code-block:: nginx
    :emphasize-lines: 1-15

        map $host $oidc_authz_endpoint {
            default "https://login.microsoftonline.com/MyAzureTenantID/oauth2/v2.0/authorize";
        }
        map $host $oidc_token_endpoint {
            default "https://login.microsoftonline.com/MyAzureTenantID/oauth2/v2.0/token";
        }
        map $host $oidc_jwt_keyfile {
            default "https://login.microsoftonline.com/MyAzureTenantID/discovery/keys";
        }
        map $host $oidc_client {
            default "MyClientID";
        }
        map $host $oidc_pkce_enable {
            default 1;
        }

------------------------------------------------------------------

*Secure Access* configuration file ``openid_connect.server_conf``:

.. code-block:: nginx
    :emphasize-lines: 1-16

    location = /_jwks_uri {
        proxy_set_header Host "login.microsoftonline.com";
        proxy_ssl_name        "login.microsoftonline.com";
        ...
    }
    location = /_token {
        proxy_set_header      Origin $host;
        proxy_set_header Host "login.microsoftonline.com";
        proxy_ssl_name        "login.microsoftonline.com";
        ...
   }
    location = /_refresh {
        proxy_set_header      Origin $host;
        proxy_set_header Host "login.microsoftonline.com";
        proxy_ssl_name        "login.microsoftonline.com";
        ...
    }


Then, for each Application to protect by your 'PaaS Secure Access',
allows the App's FQDN in *Redirect URIs* using the suffix ``/_codexch``.
A wildcard can be used as described `here <https://learn.microsoft.com/en-us/azure/active-directory/develop/reply-url#restrictions-on-wildcards-in-redirect-uris>`_.

Example:

.. image:: ./_pictures/azure_ad_login_uri.png
   :align: center
   :width: 500
   :alt: User Identifier

Okta
=============================================

A guide `here <https://github.com/nginx-openid-connect/nginx-oidc-core/tree/main/docs/oidc-pkce#pkce-setup-with-okta>`_ to configure a SPA with Okta.
After registering your 'PaaS Secure Access' instance as an Application in Okta, you will obtain a *Client ID*,
or ``$oidc_client``, a Public identifier for the client that is required for all OAuth flows.

*Secure Access* configuration file ``openid_connect_configuration.conf``:

.. code-block:: nginx
    :emphasize-lines: 1-15

        map $host $oidc_authz_endpoint {
            default "https://MyOktaTenant.okta.com/oauth2/MyAuthorizationServerID/v1/authorize";
        }
        map $host $oidc_token_endpoint {
            default "https://MyOktaTenant.okta.com/oauth2/MyAuthorizationServerID/v1/token";
        }
        map $host $oidc_jwt_keyfile {
            default "https://MyOktaTenant.okta.com/oauth2/MyAuthorizationServerID/v1/keys";
        }
        map $host $oidc_client {
            default "MyClientID";
        }
        map $host $oidc_pkce_enable {
            default 1;
        }


Then, for each Application to protect by your 'PaaS Secure Access',
allows the App's FQDN in *Sign-in redirect URIs* using the suffix ``/_codexch`` and *Sign-out redirect URIs* lists in Okta.
A wildcard can be used.

Example:

.. image:: ./_pictures/okta_login_uri.png
   :align: center
   :width: 500
   :alt: User Identifier


