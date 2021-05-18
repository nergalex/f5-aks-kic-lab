Architecture
#################################################################

.. image:: ./_pictures/OIDC_overview.svg
   :align: center
   :width: 600
   :alt: OIDC overview

.. contents:: Contents
    :local:
    :depth: 1

Edge proxy
********************************

In previous labs, we worked on an Ingress Controller hosted in an infra namespace ``external-ingress-controller`` with Ingress Class Name ``nginx-external``.
This Ingress Controller is an **Edge Proxy** that:
    - is managed by Infrastructure Provider team
    - shares his service configuration with multiple Applications
    - protect published Application with a core WAF policy defined at Enterprise level

**Edge Proxy** is used to manage North/South traffic and to protect our published API based Application outside the K8S cluster.

Micro proxy - API GW
********************************

One of the defining characteristics of an API is that it should be reusable,
i.e. an API endpoint can be consumed by another micro-service, partner application or anybody on Internet.
`To reduce attack surface of APIs <https://www.f5.com/labs/articles/education/securing-apis--10-best-practices-for-keeping-your-data-and-infra>`_,
a threat model and according protection is defined per API consumer profile:

- **private API**: consumed within an organization and not intended to be exposed to consumers outside of the organization
- **partner API**: consumed by specific partners of your organization, meaning that there is an established relationship and some framework.
- **public API**: intended to be consumed by anybody interested in the API, and does not depend on or establish a close relationship; the goal is to make consumption easy, and to attract as many consumers as possible.

**Private API** generates Est/West traffic



Each


an API GW is also deployed as a proxy tier within Kubernetes, in front of one or more specific services published publicly on Internet, therefore that require an API GW with authentication based on oAuth / OpenID Connect (OIDC) and apply rate limiting.

This API GW is also used to publish one or more specific services internally, i.e. published to other PODs hosted into Kubernetes, that do not require a security policy but require advanced delivery features.












