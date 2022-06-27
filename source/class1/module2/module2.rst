Ingress Controller
##################################################

.. image:: ./_pictures/dia-MH-2021-04-06-NIC-for-KIC-03-no-legend-padding-LP-1400x515-1.svg
   :align: center
   :width: 800
   :alt: All layers

.. contents:: Contents
    :local:

API gateway
**********************************************************
Infrastructure
=====================================
API gateways are a general microservices design pattern.
An API gateway sits between external clients and the microservices.
It acts as a reverse proxy, routing requests from clients to microservices.
It may also perform various cross-cutting tasks such as authentication, SSL termination, Web Application Firewall, anti-DoS and rate-limiting.

In Kubernetes, the functionality of an API gateway is primarily handled by an Ingress controller.

This reference implementation `here <https://github.com/mspnp/microservices-reference-implementation>`_, shown in the picture below, shows a set of best practices for building and running a microservices architecture on Microsoft Azure, using Kubernetes.

.. image:: ./_pictures/aks_ref_architecture.png
   :align: center
   :width: 800
   :alt: AKS ref design

Resource management
=====================================
Current standard to configure an API gateway is `Ingress <https://docs.nginx.com/nginx-ingress-controller/configuration/ingress-resources/>`_.

NGINX is part of the Kubernetes Gateway API community,
adding its leadership and vision to the future of Kubernetes Ingress,
as explained by Mark Church (Product Manager at Google) in `this video <https://youtu.be/p51b6t5okCI?t=5888>`_.

NGINX already implements `concepts of the Gateway API <https://gateway-api.sigs.k8s.io/>`_ in  `VirtualServer(Route) resources <https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/>`_.
VirtualServer(Route) aims to improve upon current standards like Ingress:

    - `Multi-Tenancy and Namespace Isolation in Kubernetes <https://www.nginx.com/blog/enabling-multi-tenancy-namespace-isolation-in-kubernetes-with-nginx/>`_: Gateway is composed of API resources which model organizational roles that use and configure Kubernetes service networking.
    - `Expressive declaration <https://www.nginx.com/blog/migrating-from-community-ingress-controller-to-f5-nginx-ingress-controller/>`_: - Gateway API resources support core functionality for things like header-based matching, traffic weighting, and other capabilities that were only possible in Ingress through custom annotations.
    - `Load Balancing TCP and UDP Traffic <https://www.nginx.com/blog/load-balancing-tcp-and-udp-traffic-in-kubernetes-with-nginx/>`_: - Gateway API resources support core functionality for things like header-based matching, traffic weighting, and other capabilities that were only possible in Ingress through custom annotations.

.. image:: ./_pictures/api-model.png
   :align: center
   :width: 600
   :alt: API gateway model

Why does a role-oriented API matter?¶
=====================================
Whether it’s roads, power, data centers, or Kubernetes clusters, infrastructure is built to be shared.
However, shared infrastructure raises a common challenge - how to provide flexibility to users of the infrastructure while maintaining control by owners of the infrastructure?

VirtualServer(Route) accomplishes this through a role-oriented design for Kubernetes service networking that strikes a balance between distributed flexibility and centralized control.
It allows shared network infrastructure (hardware load balancers, cloud networking, cluster-hosted proxies etc) to be used by many different and non-coordinating teams, all bound by the **policies** and constraints set by cluster operators.
The following example shows how this works in practice.

.. image:: ./_pictures/gateway-roles.png
   :align: center
   :width: 800
   :alt: API gateway roles

Which Ingress Controller?
**********************************************************
One great place to start your high‑level feature comparison is `learnk8s <https://learnk8s.io/>`_, which provides a `free comparison table <https://docs.google.com/spreadsheets/d/191WWNpjJ2za6-nbG4ZoUMXMpUK8KlCIosvQB0f-oq3k/>`_ of the Ingress controllers they’ve evaluated.

NGINX Ingress Controller (NIC) comes in three different implementations:

    #. `NGINX Plus Ingress Controller <https://www.nginx.com/products/nginx-ingress-controller>`_ maintained by NGINX
    #. `NGINX Open Source (OSS) Ingress controller <https://github.com/nginxinc/docker-nginx>`_ maintained by NGINX
    #. `Community Ingress controller <https://kubernetes.github.io/ingress-nginx/>`_ maintained by the Kubernetes community and based on **NGINX Open Source**

Check out this `article <https://www.nginx.com/blog/guide-to-choosing-ingress-controller-part-1-identify-requirements/>`_ to help you choosing an Ingress Controller.

Why NGINX Plus?
**********************************************************
This workshop enables NGINX Plus Ingress Controller because it provides a robust feature set to secure, strengthen, and scale your containerized apps, including:

Additional features
*********************

.. image:: ./_pictures/dia-JG-2020-10-30-NIC-LP-Advanced-App-Centric-Config-1024x500-1.svg
   :align: center
   :width: 600
   :alt: Production‑Grade Features

* **Visibility and performance monitoring**: A number of metrics about how NGINX Plus and applications are performing are available through the API or a built-in dashboard. Optionally, the metrics can be exported to Prometheus.
* **Additional load balancing methods**: additional methods are available for HTTP, TCP, and UDP load balancing: least_time and random two with least_time.
* **Session persistence**: In addition to the hash‑based session persistence supported by NGINX Open Source (the Hash and IP Hash load‑balancing methods), NGINX Plus supports cookie‑based session persistence
* **Active health checks**: By default NGINX performs basic checks on responses from upstream servers, retrying failed requests where possible. NGINX Plus adds out-of-band application health checks (also known as synthetic transactions) and a slow‑start feature to gracefully add new and recovered servers into the load‑balanced group. for HTTP and Health checks can also be enabled for non-HTTP protocols, such as FastCGI, memcached, SCGI, uwsgi, and also for TCP and UDP..
* **JWT-Based and OpenID Connect Authentication**: fetch JWK file, JWT validation, authorization code flow
* **WAF**: enable add-on Web Application Firewall module based on F5 WAF engine, named NGINX App Protect (NAP)


Dynamic Deployment
*****************************************************
For each rolling upgrade or scaling on an existing Deployment, IP of PODs are modified.
The Ingress controller need to updates associated configuration : ``upstream`` configuration must reflect up to date ``endpoint``.

**NGINX OSS and NGINX Community**

NGINX configuration is updated and reloaded in response to the changing endpoints for the back‑end application.

As discussed in `Timeout and Error Results for the Dynamic Deployment <https://www.nginx.com/blog/performance-testing-nginx-ingress-controllers-dynamic-kubernetes-cloud-environment/#timeout-error-results/>`_ ,
the latency experienced with the community and NGINX Open Source Ingress Controllers is caused by errors and timeouts that occur after NGINX configuration reload.

**NGINX Plus**

NGINX Plus Ingress Controller uses the `NGINX Plus API <https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/>`_  to dynamically update the NGINX configuration when endpoints change.
This prevents increase of memory usage:

* during reloads, especially with a high volume of client requests,
* when load balancing applications with long-lived connections (WebSocket, applications with file uploading/downloading or streaming).

As described `here <https://www.nginx.com/blog/performance-testing-nginx-ingress-controllers-dynamic-kubernetes-cloud-environment/>`_ ,
the performance results show that to completely eliminate timeouts and errors in a dynamic Kubernetes cloud environment,
the Ingress controller must dynamically adjust to changes in back‑end endpoints without event handlers or configuration reloads.

.. image:: ./_pictures/NGINX-IC-Kubernetes-cloud_dynamic.png
   :align: center
   :width: 600
   :alt: Latency Results for the Dynamic Deployment

Here’s a finer‑grained view of the results for the community and NGINX Plus Ingress Controllers in the same test condition as the previous graph.
The NGINX Plus Ingress Controller introduces virtually no latency until the 99.9999th percentile, where it hits 254ms.

.. image:: ./_pictures/NGINX-IC-Kubernetes-cloud_dynamic-zoomed.png
   :align: center
   :width: 600
   :alt: Latency Results for the Dynamic Deployment - zoom

Based on the results, NGINX Plus API is the optimal solution for dynamically reconfiguring NGINX in a dynamic environment.
NGINX Plus Ingress Controller achieved the flawless performance in highly dynamic Kubernetes environments that you need to keep your users satisfied.


Quick and Easy Mitigation of Security Vulnerabilities
*******************************************************
In today’s ever‑more competitive business landscape,
software teams are under pressure to deliver new and updated code as fast as possible to deliver the most innovative services.
At the same time, the rise of sophisticated attacks on infrastructure and applications make it vital to track vulnerabilities and mitigate them as soon as possible,
a tedious and time‑consuming practice.

As discussed in this `blog <https://www.nginx.com/blog/mitigating-security-vulnerabilities-quickly-easily-nginx-plus/>`_, an NGINX Plus subscription offloads that burden,
enabling application teams to concentrate on their main mission – delivering code faster – while the organization remains protected from security vulnerabilities.

NGINX Plus makes it so much faster and easier to mitigate CVEs and other security threats beceause:

* **F5 SIRT Provides Real-Time Help to NGINX Plus Subscribers Under Attack** to provide real‑time assistance
* **NGINX Proactively Informs NGINX Plus Subscribers About Patches**, proactively our customers are informed about it by email.
* **NGINX Plus Subscribers Get Patches Immediately**, generally a patch is released on the day of the CVE disclosure (or within a few days in some cases).


**A significant lag between NGINX OSS vs Plus release of a patch**

As discussed `here <https://www.nginx.com/blog/mitigating-security-vulnerabilities-quickly-easily-nginx-plus/#immediate-patches>`_,
there is sometimes quite a significant lag between our release of a patch for NGINX Open Source and the release of a corresponding patch for third‑party technologies:

.. image:: ./_pictures/mitigating-vulnerabilities-NGINX-Plus_patch-lag.png
   :align: center
   :width: 600
   :alt: NGINX Plus Subscribers Get Patches Immediately

Users of NGINX Open Source benefit from the fact that NGINX Plus is based on it,
because we are committed to enterprise‑level support and process standards for NGINX Plus customers.
These standards include service level agreements (SLAs) with our customers that dictate bug‑fix and testing procedures with which patches must comply,
for the core software, dependencies, and supported modules alike.
The SLAs help customers achieve compliance with regulations, minimizing the risk of exposure to vulnerability exploits.

**What about OSS third‑party modules?**

An added wrinkle for NGINX Open Source is that many third‑party technologies leverage and embed it in their products.
The providers of those technologies have their own processes for disclosing and patching software vulnerabilities when they are discovered.



