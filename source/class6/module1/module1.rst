Architecture
#################################################################

.. image:: ./_pictures/logical_data_path.svg
   :align: center
   :width: 900
   :alt: Application design

.. contents:: Contents
    :local:
    :depth: 1


Application design
*********************************************
*Sentence* application is a monolithic application that has been elected to be `Refactored <https://www.clouddirect.net/five-rs-of-application-modernisation/>`_.

*Sentence* will serve differently two kinds of edge computing, as shown in the picture above:

    - **traditional Web Browser** that access frontend web servers
    - **Mobile App and modern Web Browser** that are computing the page rendering on end-user device and make lightweight API calls directly to API micro-services (colors, adjectives...)

Frontend web servers make an API call to a micro-service named "generator" that then make API call to API micro-services (colors, adjectives...)

Distributed strategic points of control, One controller
***************************************************************
Security architect decided to distribute security across all subscriptions owned by each business unit.
Regarding Modern Application hosted on Kubernetes cluster, security is embedded inside the Kubernetes cluster.
This design allows to published application directly on Internet in a secured manner.

Those *strategic point of control* are controlled centrally by a Controller, as a Out of Band Control-Plane.

.. image:: ./_pictures/oob_controller.svg
   :align: center
   :width: 900
   :alt: Architecture Security

This design allows Operational Security team (SecOps) to have across subscription:

    - **Consistency**: standard security policies
    - **Simplicity**: standard security policies available in a catalog consumed by Operational Developer team (DevOps) responsible to publish applications
    - **Visibility** of Security events in order to understand the threat that targets widely the company or a specific application
    - **Reactivity**: when SecOps `Decides <https://en.wikipedia.org/wiki/OODA_loop>`_ to mitigate an `Observed <https://en.wikipedia.org/wiki/OODA_loop>`_ threat, stop a lateral movement that exploit a CVE for example, SecOps have to act quickly and widely by updating a standard seciurity policy that will apply to all protected applications

This design was approved by DevOps and `SRE team <https://sre.google/>`_ because they also find value added propositions:

    - **Multi-Tenancy**: each Business Unit have its own tenant with only their published application on different environments (sandbox, QA, Prod)
    - **Self-Service**: DevOps are autonomous to manage Traffic Management and attach a security policy available in a catalog, using UI or API of Controller in CI/CD pipelines
    - **Observability - SLI**: SRE can consume metrics from *strategic points of control* as `SLIs <https://sre.google/sre-book/service-level-objectives/>`_ on their own tool by forwarding data centrally from Controller
    - **Observability - SLO**: for small Business Unit, SREs do not have their own tool and so they are free to use Analytics engine available to define SLOs + Alerts directly on Controller






-----------------------------------------------------------------------

- **High Resiliency**: WAAP containerized instances are managed across all of your K8S clusters... as well as VM instances
- **Consistency of Security**: Applications reference WAF policies published in a central catalog
    - The Controller App Security add‑on eliminates friction by enforcing security throughout the application lifecycle without slowing developers down.
    - How? Our “WAF-as-a-Service” is an app‑centric approach to enabling WAF in automated pipelines, along with traffic services managed with the Controller Application Delivery Module, for app protection and visibility.
- **True Forecast Cost**: A simple economic model - based on Application - allows you to project on your budget with certainty
    - Application Insights offer a clear visibility into the number, performance, and ownership costs of your apps
    - With per‑app analytics, you gain new insights into app performance and reliability so you can pinpoint performance issues before they impact production.
- **Granular RBAC**: Aspire to improve collaboration across your modern app teams to unleash their productivity and efficiency.
    - All your teams – DevOps, NetOps, SecOps, and AppDev – can enjoy self‑service management and monitoring of their own apps through role‑based access control (RBAC).
