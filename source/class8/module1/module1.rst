Architecture
#################################################################

.. contents:: Contents
    :local:
    :depth: 1

Components
***************************************************************
Solution architecture is based on 3 components:

.. image:: ./_pictures/components.png
   :align: center
   :width: 400
   :alt: Architecture 3 pillars

1) Data-Plane | Reverse Proxy
=============================================
NGINX+ is installed with OPSWAT dynamic module.
NGINX+ instance forwards all HTTP request and response to MetaDefender ICAP Server for malware analysis.

.. image:: ./_pictures/flow_clean.png
   :align: center
   :width: 400
   :alt: Flow clean

MetaDefender ICAP server can sanitized the file, the flow is unchanged.

.. image:: ./_pictures/flow_sanitized.png
   :align: center
   :width: 400
   :alt: Flow sanitized

MetaDefender ICAP server can sanitized the file, the flow is unchanged.

.. image:: ./_pictures/flow_sanitized.png
   :align: center
   :width: 400
   :alt: Flow clean

If a malware is present, a blocked page is answered back to client.

.. image:: ./_pictures/flow_blocked.png
   :align: center
   :width: 400
   :alt: Flow clean

NGINX configuration is generic, therefore it does not need to be touched:

.. code-block:: bash

    http {
        upstream internal-gw {
            server internal-gw:80;
        }
        server {
            listen       80  default_server;

            location / {
                ometascan_pass http://md-icapsrv-rest-icap;
                ometascan_methods POST PUT;
                ometascan_read_timeout 1d;
                proxy_set_header Host $host;
                proxy_pass http://internal-gw;
            }
        }
    }


2) Data-Plane | MetaDefender ICAP server
=============================================
MetaDefender ICAP Server provides HTTP interface, a NGINX connector, on top of MetaDefender Core.

3) Data-Plane | MetaDefender CORE
=============================================
MetaDefender Core prevents malicious file uploads on web applications that bypass sandboxes and other detection-based security solutions.
It also helps protect confidential data, minimize data breaches, and prevent privacy violations with Proactive DLP.

3) Control-Plane & Management-Plane | Central Manager
========================================================
A Central Manager provides provides consistent configuration and oversight for NGINX+ instances hosted anywhere.
Central Manager is the product `NGINX Management Suite <https://www.nginx.com/blog/connect-scale-secure-apps-apis-with-f5-nginx-management-suite/>`_ that includes only this module for handling this use case:

    **Instance Manager**: represents the core functionality of NGINX Management Suite. Operating within the control plane, Instance Manager manages configuration and monitoring of your NGINX fleet.

.. image:: ./_pictures/NSM-Instance-Mgr_topology.png
   :align: center
   :width: 600
   :alt: NIM


Infrastructure & flows
***************************************************************
Components of the solution, described above, are deployed as containers and are hosted in a distributed Kubernetes cluster managed by F5 Distributed Cloud (XC),
named *Virtual Kubernetes* (vK8S).
Nodes of vK8S are grouped by *Customer Edge* (CE).
Two types of functional site are deployed for this solution:

    **1. Administration & SRE tooling**: CE located in Administration zone or Out of Band zone
    **2. Application workload**: CE located in applicative landing zone(s)

.. image:: ./_pictures/Architecture_overview.png
   :align: center
   :width: 800
   :alt: Overview

For this lab, one CE of each type described below are deployed on Microsoft Azure and in 2 different regions.

XC Global Infrastructure
========================================================
F5 Distributed Cloud (XC) Global Infrastructure is a `Multi-Cloud Networking Software <https://blogs.gartner.com/andrew-lerner/2022/04/21/multicloud-networking-software-mcns/>`_
that creates a virtual Backbone between ``sites``:
    - **Regional Edges (RE)** or `F5 POPs <https://www.f5cloudstatus.com/>`_ that interconnect Internet Service Providers, Cloud Service Providers and Customer Private Links
    - **Customer Edges (CE)** that are hosted in customer landing zones (On Premise, Private or Public Cloud) and are interconnected to REs via secured VPN tunnels

.. image:: ./_pictures/app_traffic.png
   :align: center
   :width: 800
   :alt: App Traffic

For this lab, CEs are named ``cloudbuilder-x``.

XC App Stack
========================================================
Each ``site`` (RE or CE) is a local Kubernetes clusters of 1 or 3+ nodes.
A ``virtual site`` is a logical group of ``sites`` with same label(s) set.
F5 Distributed Cloud (XC) AppStack is a Virtual Kubernetes (vK8S) cluster deployed across a ``virtual site``.
Each Node of a site is seen as a node of vK8S cluster.

For this lab, dataplane components are all deployed on one CE ``cloudbuilder-x`` and have only one replica on node ``master-0``.
For vK8S point of view, dataplane components are deployed on Node ``cloudbuilder-x-master-0``

.. image:: ./_pictures/vk8s_nodes.png
   :align: center
   :width: 800
   :alt: App Traffic

Service Mesh feature of F5 XC AppStack monitor traffic between components of the solution:

.. raw:: html

    <a href="http://www.youtube.com/watch?v=XInA_Hm5Xlc"><img src="http://img.youtube.com/vi/XInA_Hm5Xlc/0.jpg" width="600" height="300" title="Service Mesh view"></a>

Central Manager
========================================================
NGINX Management Suite (NMS) is hosted on a VM and accessible from API owners on Internet.
NMS UI/API is published and secured by F5 XC.

.. image:: ./_pictures/api_owner_to_nms.png
   :align: center
   :width: 800
   :alt: API owner <> NMS

API GWs
========================================================
API gateway instances are hosted on Containers and deployed on vK8S, in closest regions where consumers are present.
A consumer can be a endpoint/edge on Internet or another micro-service present in vK8S.
Application Services are published and secured by F5 XC.

.. image:: ./_pictures/api_gw_consumer.png
   :align: center
   :width: 800
   :alt: Consumer <> API GW

NMS communicates with managed API gateways through a GRPC session initiated by a ``nginx-agent`` installed on API gateways.
Communications are forwarded through F5 XC virtual backbone.

.. image:: ./_pictures/api_gw_nms.png
   :align: center
   :width: 800
   :alt: Consumer <> API GW


Developer Portal
========================================================
Developer Portal instances are hosted on Containers and deployed on vK8S, in closest regions where developers are present.
Developer Portals are published and secured by F5 XC.

.. image:: ./_pictures/devportal_developer.png
   :align: center
   :width: 800
   :alt: Developer >> DevPortal

NMS communicates with managed Developer Portals through:
    - ``nginx-agent`` sessions, as seen for API GWs,
    - and a ``nginx-devportal`` service: NMS initiates REST API calls to a ``nginx-devportal`` service that is published and secured by F5 XC.

.. image:: ./_pictures/devportal_nginx-devportal.png
   :align: center
   :width: 800
   :alt: NMS >> DevPortal

Persistent data of Developer Portals, from all regions, are stored in a PaaS DB (Azure Database for PostgreSQL) available in only one region.
Communications are forwarded through F5 XC virtual backbone.

.. image:: ./_pictures/devportal-pass_db.png
   :align: center
   :width: 800
   :alt: DevPortal >> PaaS DB

Sentence app
***************************************************************
For this lab, `Sentence application  <https://gitlab.com/sentence-app>`_ is deployed.

Components
========================================================
Sentence is based on multiple decoupled services:

    - **frontend**: Web UI that generates an HTML page and delivers a Single Page Application (JavaScript).
    - **background**: a micro-service that returns (GET) a *background* image
    - **generator**: a micro-service that generates a *Sentence* by aggregating an *adjective*, an *animal*, a *color* and a *location*
    - **adjectives**: a micro-service that returns (GET) an *adjective*
    - **animals**: a micro-service that returns (GET) an *animal*
    - **colors**: a micro-service that returns (GET) an *color*
    - **locations**: a micro-service that returns (GET) an *location*

.. image:: ./_pictures/sentence_global_view.png
   :align: center
   :width: 800
   :alt: Sentence App

End user authentication | oAuth OIDC + PKCE
========================================================
End user must be authenticated to CREATE a new *adjective*, *animal*, *color* or *location*.
Authentication is handled by API GW that leverages Okta as an Identity Provider.

.. image:: ./_pictures/sentence_POST.png
   :align: center
   :width: 800
   :alt: Sentence App

`oAuth/OIDC <https://developer.okta.com/docs/concepts/oauth-openid/>`_ flow is described in picture below and `here <https://github.com/nginxinc/nginx-openid-connect>`_.

.. image:: ./_pictures/API_GW_PKCE.png
   :align: center
   :width: 800
   :alt: Sentence App

PKCE is used to make Sentence apps more secure, see explanations `here  <https://developer.okta.com/blog/2019/08/22/okta-authjs-pkce#use-pkce-to-make-your-apps-more-secure>`_.

Application authentication | API Key
========================================================
Client, for example a mobile app, must be authenticated to CREATE a new *color*.
Authentication is handled by API GW and authentication is based on API key provided by the application.

