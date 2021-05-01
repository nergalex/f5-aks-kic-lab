Overview
##########################################################

In this class, we will deploy a modern application (Arcadia Finance app) with modern tools in a modern environment.

What are modern tools:
   - `Red Hat Ansible Tower <https://www.ansible.com/products/tower>`_ as Automation Platform
   - Terraform [ADC] non
   - Gitlab and Gitlab CI [ADC] non

What is a modern environment:
   - `Azure Kubernetes Service <https://azure.microsoft.com/en-us/services/kubernetes-service/>`_
   - `Azure Container Registry <https://azure.microsoft.com/en-us/services/container-registry/>`_
   - `Github as code repository <https://azure.microsoft.com/en-us/products/github/>`_
   - `Kibana<https://www.elastic.co/kibana>`_ + `Elasticsearch<https://www.elastic.co/elastic-stack>`_ + `Fluentd<https://www.fluentd.org/>`_  +  as a logging platform
   - `Docker<https://www.docker.com/>`_ to build images

.. note:: Don't be afraid if you don't know those tools. The goal of the lab is not to learn how to deploy them, but how to use them.

**First of all, this is Arcadia Finance application**

.. image:: ./pictures/arcadia-app.png
   :align: center

|

**Class 1 - All sections**

.. toctree::
   :maxdepth: 1
   :glob:

   module*/module*


