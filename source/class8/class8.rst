Demo | XC Fast PaaS
##############################################################

`F5 Distributed Cloud (XC) <https://www.f5.com/cloud>`_ is a SaaS-based platform
that enables Network and Security services in data path to your application.
If customer desires to add also custom Security services in the data path,
customer will deploy Platform as a Services (PaaS), a container based solution (NGINX you said?),
that could be hosted in F5 Points of Presence (POPs).

This folder shares some typical customer use case of PaaS:

.. toctree::
   :maxdepth: 1
   :glob:

   module*/module*

First, a quick introduction to F5 XC:

.. contents:: Contents
    :local:
    :depth: 1

Application Delivery Network (ADN)
***************************************************************

F5 XC Application Delivery Network (ADN) is a Virtual Backbone that interconnects clients and services to be consumed by them.
Edges of F5 XC ADN are:
    - F5 Regional Edges (RE): F5's Points of Presence, based hardware managed by F5
    - or/and Customer Edges (CE): Customer's Points of Presence, based hardware managed by customer or by a Cloud Service Provider

Interconnections between Edges are:
    - `F5 private backbone <https://docs.cloud.f5.com/docs/services/mesh/secure-backbone>`_: REs are interconnected using a multi-terabit, dedicated and redundant private backbone for maximum performance. These PoPs are densely peered and connected with multiple Tier1 transit providers to deliver high-quality internet access for applications and consumers. REs are directly connected to multiple cloud providers from these locations to provide a reliable and predictable experience across cloud providers.
    - `Tunneling <https://docs.cloud.f5.com/docs/how-to/advanced-networking/tunneling>`_: REs-CEs and CEs-CEs are interconnected using tunnels IP over IP, IPsec or TLS depending on the trust of the underlay network.

Edges have the same software that provides Network, Security and K8S services managed in F5 XC Console.

.. image:: ./_pictures/virtual_backbone.png
   :align: center
   :width: 800
   :alt: OPSWAT

Virtual Load Balancer
***************************************************************




