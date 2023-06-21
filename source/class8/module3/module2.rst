DDoS App Protect
#################################################################

*DDoS App Protect* defends, adapts, and mitigates against Layer 7 denial-of-service attacks on your apps and APIs.

Powered by the most popular data plane in the world,
NGINX's *Secure Access* PaaS solution improves your security posture:
    - **Minimize attack surface**: Reduce the accessible attack surface through **access control**
    - **Prevent unauthorized activity**: Automatically block ongoing attacks through constant authentication, identity validation and detection of behavioral anomalies coupling with `F5 XC Malicious User <https://docs.cloud.f5.com/docs/how-to/advanced-security/malicious-users>`_ feature based on IA/ML.
    - **Single Sign-On**: enable Single Sign-On (SSO) for your proxied applications by integrating with your Identity Provider and OpenID Connect

.. image:: ./_pictures/intro.png
   :align: center
   :width: 800
   :alt: Introduction

.. contents:: Contents
    :local:
    :depth: 1

Overview of the XC Multi-Layered DDoS protection
***************************************************************
F5 Distributed Cloud (XC) offers a multi-Layered DDoS protection:
    - **1. Volumetric network DDoS protection**: leveraging IP anycast, `highly BGP peered to metro networks <https://bgp.he.net/report/peers>`_ and F5 XC private global network, XC "Transit DDoS" mitigate volumetric DDoS attacks up to 20 Tbps.
    - **2. TLS based DDoS protection**: when you publish an application on XC, using an HTTP LB object, a native L7 DDoS protection detects attacks leveraging Machine Learning and automatically mitigate attacks identified by the botnet's TLS fingerprint.
    - **2. HTTP headers based DDoS protection**: **DDoS App Protect** analyzes client behavior and application health to model normal traffic patterns, uses unique algorithms to create a dynamic statistical model that provides the most accurate protections, and deploys dynamic signatures to automatically mitigate attacks.

.. image:: ./_pictures/oignon-multi-layered-ddos-protection.png
   :align: center
   :width: 900
   :alt: Onion

Benefits
***************************************************************
PaaS *DDoS App Protect* is a **fire and forget** solution,
that can be **deployed everywhere** on the globe
and blocks traffic **closest to attackers**
and **only requests that causes harm**.

Closest to the attackers
=============================================

Lowest False Positive
=============================================

Cost-effective
=============================================



----------------------------------------------------
Notes
----------------------------------------------------

No-touch policy configuration
a **zero touch** solution
