App Protect DDoS
#################################################################

*App Protect DDoS*,
a F5 Distributed Cloud (XC) Platform as a Service (PaaS),
defends, adapts, and mitigates against Layer 7 denial-of-service attacks on your apps and APIs.

.. image:: ./_pictures/intro.png
   :align: center
   :width: 1000
   :alt: Introduction

.. contents:: Contents
    :local:
    :depth: 1

Modern Denial of Service Attacks become Complex to detect and mitigate without impacting legitimate users
*********************************************************************************************************

Denial of Service attacks continue to be ubiquitous and have remained in the top spot of incidents for several years now.
Source: `DBIR 2023 <https://www.verizon.com/business/fr-fr/resources/reports/dbir/>`_

.. image:: ./_pictures/DBIR_DDOS.png
   :align: center
   :width: 600
   :alt: Patterns over time in incidents

This `blog <https://www.nginx.com/blog/defending-applications-complex-modern-attacks/>`_ describes
volumetric denial-of-service (DoS) and distributed DoS (DDoS) attacks at the network and transport levels (Layers 3 and 4),
which exhaust servers’ available bandwidth by flooding them with TCP/UDP connection requests.
Now attackers have added a new tool to their arsenal – DoS and DDoS attacks that use HTTP requests
or API calls to exhaust resources at the application level (Layer 7).

Layer 7 attacks are more complex to design than network attacks,
and many tools that can handle Layer 3/4 attacks are ineffective at protecting modern application architectures.
Layer 7 DDoS attacks are more difficult to detect because bots and automation allow attackers to disguise themselves as legitimate traffic,
especially when they’re using sophisticated security penetration tools.
When a hacker can assemble a botnet – thousands of compromised machines under the hacker’s control – it’s easy to initiate attacks on a huge scale.
With bad bot traffic stealthily hiding among legitimate customer traffic, Layer 7 attacks create a new challenge.

Overview of the XC Multi-Layered DDoS protection
***************************************************************

.. image:: ./_pictures/oignon-multi-layered-ddos-protection.png
   :align: center
   :width: 1000
   :alt: Onion

F5 Distributed Cloud (XC) offers a multi-Layered DDoS protection:
    - **1. Volumetric network DDoS protection**: leveraging IP anycast, `highly BGP peered to metro networks <https://bgp.he.net/report/peers>`_ and F5 XC private global network, XC "Transit DDoS" mitigates volumetric DDoS attacks up to 20 Tbps.
    - **2. TLS based DDoS protection**: when you publish an application on XC, using an HTTP LB object, a native L7 DDoS protection detects attacks leveraging Machine Learning and automatically mitigates attacks identified by the botnet's TLS fingerprint.
    - **2. HTTP headers based DDoS protection**: *App Protect DDoS* analyzes client behavior and application health to model normal traffic patterns, uses unique algorithms to create a dynamic statistical model that provides the most accurate protections, and deploys dynamic signatures to automatically mitigate attacks.


L7 DDoS protection, a Modern Solution for a Modern Problem
***************************************************************
What are the key components that protect against Layer 7 attackers?

On a basic level,
you need a tool that recognizes when your site is under attack
– something that’s able to distinguish between “good guys” (legitimate traffic) and “bad guys” (malicious traffic).

PaaS *App Protect DDoS* is a **fire and forget** solution,
that can be **deployed everywhere** on the globe
and blocks traffic **closest to attackers**
and **only requests that cause harm**.

Seamless integration
=============================================
*App Protect DDoS* is a native PaaS deployed in F5 XC Edges:
    - in F5 managed infrastructure (Regional Edge)
    - or/and in Customer Private / Public Clouds (Customer Edge)

Components of the solution are:

.. image:: ./_pictures/components.png
   :align: center
   :width: 1000
   :alt: components

    1. **Data-plane / gateways** workload: gateways learn traffic, enforce security rules and notify the arbitrator of ongoing attacks
    2. **Control-plane / the arbitrator** workload: the arbitrator is aware of all ongoing attacks and synchronise gateways to enforce security rules of all ongoing attacks
    3. **Analytics plane / log collector / SIEM**: customer is free to forward metrics and security event from gateways to a his own log collector or SIEM solution. `Here <https://github.com/f5devcentral/nap-dos-elk-dashboards>`_ a simple ELK dashboard used for the demo.

.. raw:: html

    <a href="http://www.youtube.com/watch?v=cO2bbAVtTdw"><img src="http://img.youtube.com/vi/cO2bbAVtTdw/0.jpg" width="600" height="300" title="XC PaaS App Protect DDoS - components"></a>

Monitoring of the solution
=============================================

F5 XC offers native *Service Mesh* dashboards:
    **1. Data-plane**: health of the data-plane service per location and key metrics of users traffic

.. raw:: html

    <a href="http://www.youtube.com/watch?v=WtT1W45Oid4"><img src="http://img.youtube.com/vi/WtT1W45Oid4/0.jpg" width="600" height="300" title="XC PaaS App Protect DDoS - Service Mesh - data-plane"></a>

    **2. Control-plane**: health of the control-plane services per location and key metrics of operational traffic

.. raw:: html

    <a href="http://www.youtube.com/watch?v=yDj0WBGCVDE"><img src="http://img.youtube.com/vi/yDj0WBGCVDE/0.jpg" width="600" height="300" title="XC PaaS App Protect DDoS - Service Mesh - control-plane"></a>



Cost-effective
=============================================



----------------------------------------------------

In Progress

----------------------------------------------------


Lowest False Positive
=============================================



Boundless scaling
=============================================
Better the attack is distributed,
more difficult will be the detection, based on a small portion of the traffic,
and the mitigation too, because it requires a synchronization between all security sensors.
your protection



Closest to the attackers


No-touch policy configuration
a **zero touch** solution
