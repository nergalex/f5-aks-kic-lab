Architecture
##################################################

.. contents:: Table of Contents
   :local:

Application Services in Kubernetes
=========================================

As discussed in this `blog <https://www.nginx.com/blog/deploying-application-services-in-kubernetes-part-2/>`_,
in a Kubernetes environment, there are several locations where you might deploy application services:

.. image:: ./_pictures/app-services-Kubernetes-pt2_four-locations.png
   :align: center
   :width: 700
   :alt: All layers

Let’s take web application firewall (WAF) as an example.
WAF policies implement advanced security measures to inspect and block undesirable traffic,
but these policies often need to be fine‑tuned for specific applications in order to minimize the number of false positives.

WAF on the Ingress Controller
=========================================

.. image:: ./_pictures/app-services-Kubernetes-pt2_Ingress-controller.png
   :align: center
   :width: 600
   :alt: Edge Proxy

In those labs, it was considered to do not have any security service on Front Door, only an Load Balancer.
The WAF application service is deployed on Ingress controller in order to met the following customer context:

- Baseline of the **WAF policy definition** is owned by **SecOps** team and they make it available as a catalog for DevOps consumption
- The **implementation** of the WAF policies (baseline + modifications) are under the direction of the **DevOps** team
- Customer wants to **centralize WAF policies** at the infrastructure layer, rather than delegating them to individual applications.
- DevOps make extensive use of **Kubernetes APIs** to manage the **deployment and operation** of applications.

This approach still allows for a central **SecOps** team to **define the WAF policies**.
They can define the policies in a manner that can be easily imported into Kubernetes,
and the **DevOps** team **responsible for the Ingress controller** can then assign the WAF policies to specific applications.

The NGINX App Protect WAF module is deployed directly on the Ingress Controller.
All WAF configuration is managed using *Ingress* resources [lab #3] or *VirtualServer(Route)* resources [lab #3-4], configured through the Kubernetes API.

API GW on a Per‑Service Basis
=========================================

.. image:: ./_pictures/app-services-Kubernetes-pt2_per-service.png
   :align: center
   :width: 600
   :alt: Micro Proxy

In lab #4, an API GW is also deployed as a proxy tier within Kubernetes,
in front of one or more specific services published publicly on **Internet**,
therefore that require an **API GW** with authentication based on **oAuth / OpenID Connect** (OIDC) and apply **rate limiting**.

This API GW is also used to publish one or more specific services **internally**, i.e. published to other PODs hosted into Kubernetes,
that do not require a security policy but require **advanced delivery** features.



