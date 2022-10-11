In Progress - Architecture
#################################################################

.. image:: ./_pictures/functional_view_3_pillars.png
   :align: center
   :width: 700
   :alt: Architecture 3 pillars

.. contents:: Contents
    :local:
    :depth: 1

Components of API Management
***************************************************************
The F5 API Management solution includes 3 components:

1) Data-Plane | API Gateways (GW)
=============================================
An API GW is a NGINX Plus instance that delivers microservices with reliability, speed and security across multiple environments (baremetal, VM, K8S container).

2) Data-Plane | Developer Portal
=============================================
A Developer Portal is a NGINX Plus instance that delivers API Documentation and Self Service management of API access (API Keys).
A Developer Portal enables internal and external Producers to rapidly discover, onboard and use APIs in their projects with consistent documentation and versioning.

3) Control-Plane & Management-Plane | Central Manager
========================================================
A Central Manager provides uniform and consistent oversight for platform environments, API specifications and configurations from a single pane of glass.
Central Manager is the product `NGINX Management Suite <https://www.nginx.com/blog/connect-scale-secure-apps-apis-with-f5-nginx-management-suite/>`_ that includes 2 modules:

    1. **Control-Plane | Instance Manager**: represents the core functionality of NGINX Management Suite. Operating within the control plane, Instance Manager simplifies configuration and monitoring of your NGINX fleet.
    2. **Management-Plane | API Connectivity Manager**: simplifies governance of API in your organization, delivers API across your NGINX fleet and enforce consistent API Security.

