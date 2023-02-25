Secure Access
#################################################################

.. image:: ./_pictures/illo-SolutionsZeroTrust-450x400-1.svg
   :align: center
   :width: 400
   :alt: Secure Access

Powered by the most popular data plane in the world,
NGINX's *Secure Access* PaaS solution improves your security posture:
    - **Minimize attack surface**: Reduce the accessible attack surface through **access control**
    - **Prevent unauthorized activity**: Automatically block ongoing attacks through constant authentication, identity validation and detection of behavioral anomalies coupling with `F5 XC Malicious User <https://docs.cloud.f5.com/docs/how-to/advanced-security/malicious-users>`_ feature based on IA/ML.
    - **Protect apps from N clouds to One edge**: Make security independent of all other variables, including environment and geography
    - **Single Sign-On**: enable Single Sign-On (SSO) for your proxied applications by integrating with your Identity Provider and OpenID Connect

.. contents:: Contents
    :local:
    :depth: 1

Access control
***************************************************************
With *NGINX Secure Access*,
both the client and NGINX Plus communicate directly with the IdP at different stages during the initial authentication event.

.. image:: ./_pictures/high_level_flow.svg
   :align: center
   :width: 500
   :alt: Flow high level

NGINX Plus is configured to perform OpenID Connect authentication:

.. image:: ./_pictures/flow.svg
   :align: center
   :width: 700
   :alt: Flow

    - **1.** Upon a first visit to a protected resource,
    - **2.** NGINX Plus initiates the OpenID Connect authorization code flow and redirects the client to the OpenID Connect provider (IdP).
    - **3-5.** The IdP verify the identity of the end-user.
    - **6.** When the client returns to NGINX Plus with an authorization code,
    - **7-8.** NGINX Plus exchanges that code for a set of tokens by communicating directly with the IdP.
    - **9.** The ID Token received from the IdP is validated. NGINX Plus then stores the ID token in the key-value store, issues a session cookie to the client using a random string, (which becomes the key to obtain the ID token from the key-value store)
    - **10.** and redirects the client to the original URI requested prior to authentication.
    - **11.** Subsequent requests to protected resources are authenticated by exchanging the session cookie for the ID Token in the key-value store.
    - **12.** JWT validation is performed on **each request**, as normal, so that the ID Token validity period is enforced.

*demo video:*

.. raw:: html

    <a href="http://www.youtube.com/watch?v=DJoYL5dgg7E"><img src="http://img.youtube.com/vi/DJoYL5dgg7E/0.jpg" width="600" height="300" title="Service Mesh view"></a>

*NGINX Secure Access* PaaS solution is built using the reference implementation of NGINX Plus as relying party for OpenID Connect authentication,
please refer to it `here <https://github.com/nginxinc/nginx-openid-connect>`_ for more details.

For more information on OpenID Connect and JWT validation with NGINX Plus, see `Authenticating Users to Existing Applications with OpenID Connect and NGINX Plus <https://www.nginx.com/blog/authenticating-users-existing-applications-openid-connect-nginx-plus/>`_.

Malicious User
***************************************************************
Your IT and security operations teams enjoy total
visibility into all user activity—even web browsing and
unsanctioned app usage—and can automate and enforce
strong access policies from login to logout, including
step-up authentication challenges for suspect behavior


Supported Identity Providers
***************************************************************
Any Identity Provider that supports OpenID Connect 1.0.

Implementation of *NGINX Secure Access* assumes that your IdP knows F5 XC as a confidential client or a public client using PKCE.
By following the steps in `this guide <https://docs.nginx.com/nginx/deployment-guides/single-sign-on/okta/>`_,
you will learn how to set up SSO using OpenID Connect as the authentication mechanism with well known IdPs and NGINX Plus as the relying party.
For Azure AD as an IdP, follow `this guide <https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-auth-code-flow#request-an-authorization-code>`_.



