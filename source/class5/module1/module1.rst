Architecture
#################################################################

.. image:: ./_pictures/sentence_overview.svg
   :align: center
   :width: 600
   :alt: Architecture overview

.. contents:: Contents
    :local:
    :depth: 1

Edge Proxy - Web Application & API Protection
*********************************************

In previous labs, we worked on an Ingress Controller hosted in an infra namespace ``external-ingress-controller`` with Ingress Class Name ``nginx-external``.
This Ingress Controller is an **Edge Proxy** that:
    - is managed by Infrastructure Provider team
    - shares his service configuration with multiple Applications
    - manage North/South traffic
    - protect published Application with a core WAF policy defined at Enterprise level

In this lab, **Edge Proxy** protects an Web Browser based and API based Application when it is consumed from outside K8S cluster.

Micro Proxy - API GW
********************************

One of the defining characteristics of an API is that it should be reusable,
i.e. an API endpoint can be consumed by another internal micro-service, partner application or anybody on Internet.
`To reduce attack surface of APIs <https://www.f5.com/labs/articles/education/securing-apis--10-best-practices-for-keeping-your-data-and-infra>`_,
a threat model and according protection are defined per API consumer profile:

    - **Private API**: consumed within an organization and not intended to be exposed to consumers outside of the organization
    - **Partner API**: consumed by specific partners of your organization, meaning that there is an established relationship and some framework.
    - **Public API**: intended to be consumed by anybody interested in the API, and does not depend on or establish a close relationship; the goal is to make consumption easy, and to attract as many consumers as possible.

In this lab, **Micro Proxy** publishes API endpoints of an Application in 2 ways:

    - **Private API** consumed by another namespace and inside the same namespace. This communication generates cross namespace, or East/West, traffic inside K8S cluster.
    - **Partner API** consumed by a authenticated user using an IDaaS. This communication generates Est/West traffic inside K8S cluster.


.. note::
    **Private API** could also generates **North/South traffic** regarding our K8S cluster point of view: for example a consumer hosted on another K8S cluster in another Cloud Service Provider


Each


an API GW is also deployed as a proxy tier within Kubernetes, in front of one or more specific services published publicly on Internet, therefore that require an API GW with authentication based on oAuth / OpenID Connect (OIDC) and apply rate limiting.

This API GW is also used to publish one or more specific services internally, i.e. published to other PODs hosted into Kubernetes, that do not require a security policy but require advanced delivery features.












