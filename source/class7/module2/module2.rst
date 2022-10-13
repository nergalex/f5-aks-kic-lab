In Progress - Exercises
#################################################################

.. contents:: Contents
    :local:
    :depth: 2

----------------------------------------------------------------


NGINX Management Suite >> Instance Manager
*********************************************

Exercise 1: Inventory
============================================
- Connect to NMS `here <https://nms.f5dc.dev>`_
- Check that instances are UP: ``Instance Manager`` **>** ``Instances``
- In Lens, connect to your vK8S
- Edit Deployment of NGINX API GW:  ``Workloads`` **>** ``Deployments`` **>** ``NameSpace: {{student_id}}`` **>** ``nginx-apigw``
- Note the ``Instance Group`` value for ``spec.template.spec.containers[0].env.name = ENV_CONTROLLER_INSTANCE_GROUP``
- Scale Out by modifying the value of ``spec.replicas`` to `2` per Site, then ``Save & Close``
- Review POD creation: ``Workloads`` **>** ``PODs``
- In NMS, check that new instances are well automatically discovered: ``Instance Manager`` **>** ``Instances``
- Check that new instances are well automatically attached the Instance Group name noted before: ``Instance Manager`` **>** ``Instance Group``

**Scale Out demo**

.. raw:: html

    <a href="http://www.youtube.com/watch?v=p-xUuhJ7tBg"><img src="http://img.youtube.com/vi/p-xUuhJ7tBg/0.jpg" width="600" height="300" title="Scale Out" alt="Scale Out"></a>

- In Lens, Scale In by modifying the value of ``spec.replicas`` to `1` per Site, then ``Save & Close``
- In NMS, check that new instances are well automatically unregistered after 150 seconds

**Scale In demo**

.. raw:: html

    <a href="http://www.youtube.com/watch?v=9MR91T3YDwg"><img src="http://img.youtube.com/vi/9MR91T3YDwg/0.jpg" width="600" height="300" title="Scale In" alt="Scale In"></a>

_______________________________________________________________________

**Capture The Flag**

    **1.1 What is the value of terminationGracePeriodSeconds?**

_______________________________________________________________________

Exercise 2: Authentication | oAuth OIDC + PKCE
==============================================
- Connect to NMS `here <https://nms.f5dc.dev>`_
- Check that instances are UP: ``API Connectivity Manager`` **>** ``Infrastructure`` **>** ``Sentence{{ studentID }}``
- In Lens, connect to your vK8S
- Edit Deployment of NGINX API GW:  ``Workloads`` **>** ``Deployments`` **>** ``NameSpace: {{student_id}}`` **>** ``nginx-apigw``
- Note the ``Instance Group`` value for ``spec.template.spec.containers[0].env.name = ENV_CONTROLLER_INSTANCE_GROUP``
- Scale Out by modifying the value of ``spec.replicas`` to `2` per Site, then ``Save & Close``
- Review POD creation: ``Workloads`` **>** ``PODs``
- In NMS, check that new instances are well automatically discovered: ``Instance Manager`` **>** ``Instances``
- Check that new instances are well automatically attached the Instance Group name noted before: ``Instance Manager`` **>** ``Instance Group``

**Scale Out demo**

.. raw:: html

    <a href="http://www.youtube.com/watch?v=p-xUuhJ7tBg"><img src="http://img.youtube.com/vi/p-xUuhJ7tBg/0.jpg" width="600" height="300" title="Scale Out" alt="Scale Out"></a>

