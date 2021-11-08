Exercises
#################################################################

By the end of the lab you will be able to:

- **Life Cycle Management**

    - Scale Out / Scale In
    - Upgrade NGINX App Protect version
    - Update the signatures and threats on NGINX App Protect

- **False Positive Management**

    - Deploy an application
    - Start with a basic WAF policy and be reactive to handle False Positive
    - Create your standard policy and publish it in catalog
    - Update your standard policy and all attached applications

- **Visibility**

    - Detect which Components have not a policy attached to it

.. contents:: Contents
    :local:
    :depth: 2

Life Cycle Management
*********************************************

Exercise 1: Scale Out
============================================
- In Lens, edit Deployment of NGINX App Protect instances:  ``Workloads`` **>** ``Deployments`` **>** ``NameSpace: waap-managed`` **>** ``nginx-appprotect``

.. image:: ./_pictures/Lens_Deployment_show.png
   :align: center
   :width: 900
   :alt: Show instances

- Scroll to see the environment variable used to startup PODs

.. code-block:: yaml
    :emphasize-lines: 3

          env:
            - name: ENV_CONTROLLER_INSTANCE_GROUP
              value: lab_k8s_{{site_ID}}
            - name: ENV_CONTROLLER_LOCATION
              value: site{{site_ID}}_waap_managed
            - name: ENV_CONTROLLER_API_URL
              value: '10.0.0.12:443'

- Note the value of Controller's Instance-Group to be registered
- In NGINX Controller, login as SuperNetOps

    - email:  supernetops@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- User Role of a SuperNetOps allow a user to view, create and delete any instance-groups (FULL authorization)
- Check if you see POD names of your Kubernetes cluster {{site_ID}}

.. image:: ./_pictures/Controller_instances_show.png
   :align: center
   :width: 900
   :alt: Show instances in instance-group

- Click on an instance name

.. image:: ./_pictures/Controller_instance_show.png
   :align: center
   :width: 900
   :alt: Show instance

- Scroll down to see Services running in this Instance

.. image:: ./_pictures/Controller_instance_show_services.png
   :align: center
   :width: 900
   :alt: Show instance > Services

- In Lens, scale out Deployment of NGINX App Protect to 3 replicas (instances)

.. image:: ./_pictures/Lens_Deployment_Scale_3.png
   :align: center
   :width: 800
   :alt: Scale Out

- In Controller, see the instance registered in instance-group ``lab_k8s_{{site_ID}}``

Exercise 2: Upgrade Software and Signatures
============================================

Oh my god, deployed image of NGINX App Protect contains a very old signature package.
Upgrade with a fresh image that have been already built and uploaded to Azure Container Registry linked to your AKS cluster.

- In Lens, go to ``Workloads`` **>** ``PODs`` **>** ``NameSpace: waap-managed`` **>** ``nginx-appprotect``
- Open a shell

.. image:: ./_pictures/Lens_POD_shell.png
   :align: center
   :width: 900
   :alt: Scale Out

- Show signature packages

.. code-block:: bash

    cat /var/log/app_protect/compile_error_msg.json

*output*

.. code-block:: json
    :emphasize-lines: 2,3,4

    {
        "user_signatures_packages": [],
        "threat_campaigns_package": {},
        "attack_signatures_package":
        {
            "revision_datetime": "2019-07-16T12:21:31Z"
        },
        "completed_successfully": true
    }

_______________________________________________________________________

**Capture The Flag**

    **2.1 What is the last update of signature package?**

    **2.2 What is the last update of Threat Campaign?**

_______________________________________________________________________

- Show the last update of signature package

    .. code-block:: bash

        sudo yum --showduplicates list app-protect-attack-signatures

- Show the last update of Threat Campaign

    .. code-block:: bash

        sudo yum --showduplicates list app-protect-threat-campaigns

- Go to ``Workloads`` **>** ``Deployments`` **>** ``NameSpace: waap-managed`` **>** ``nginx-appprotect``
- Update specification using latest image of NGINX App Protect

.. code-block:: json
    :emphasize-lines: 8

    spec:
      ...
      template:
        ...
        spec:
          containers:
            - name: nginx-agent
              image: 'aksdistrict2.azurecr.io/nginx-agent:latest'

-  View `rolling upgrade <https://kubernetes.io/docs/concepts/workloads/controllers/deployment/>`_ of PODs by clicking on ``nginx-appprotect`` deployment
- Open a shell on a fresh POD
- Show signature packages and see that packages are now up-to-date

.. code-block:: bash

    cat /var/log/app_protect/compile_error_msg.json

*output*

.. code-block:: json
    :emphasize-lines: 7,13

    {
        "user_signatures_packages":
        [],
        "attack_signatures_package":
        {
            "revision_datetime": "2021-11-04T19:18:42Z",
            "version": "2021.11.04"
        },
        "completed_successfully": true,
        "threat_campaigns_package":
        {
            "revision_datetime": "2021-11-03T07:51:14Z",
            "version": "2021.11.03"
        }
    }

False Positive Management
*********************************************

Exercise 3: View all applications as a NetOps
=============================================

- In NGINX Controller, login as SuperNetOps

    - email:  supernetops@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- User Role of a SuperNetOps allow user to view all Services (READ authorization)
- Go to ``Platform`` **>** ``User Role`` then see configuration PATHs and attached authorization levels

.. image:: ./_pictures/Controller_platform_user_role_supernetops.png
   :align: center
   :width: 900
   :alt: User Role NetOps

- Go to ``Services`` **>** ``Gateway``. You can see all gateways but edit none.

Exercise 4: View application configuration
============================================

- In NGINX Controller, login as DevOps owner of your site

    - email:  devops{{ site_ID }}@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- User Role of a Devops allow user to create, update and delete Services (FULL authorization) of its own environment {{ site_ID }}
- Go to ``Platform`` **>** ``User Role`` then see configuration PATHs and attached authorization levels

.. image:: ./_pictures/Controller_platform_user_role_devops.png
   :align: center
   :width: 900
   :alt: User Role DevOps

- Go to ``Services`` **>** ``Gateways`` **>** ``sentence-front-managed{{ site_ID }}.f5app.dev``
- ``Edit Gateway`` **>** ``Placements``: show attached ``instance-group``

.. image:: ./_pictures/Controller_service_gateway_placement_instance-group.png
   :align: center
   :width: 900
   :alt: User Role DevOps

    **Note**
    NGINX Controller v4: specifics``instance-groups`` could be assign to a User Role

- In your web browser, go to ``https://sentence-front-managed{{ site_ID }}.f5app.dev`` and generate some traffic by refreshing 10 times the page
- Go to ``Services`` **>** ``Apps`` **>** ``sentence-front-managed{{ site_ID }}.f5app.dev``
- Update filter to ``Last 15 minutes``
- Scroll down to ``Web (HTTP) Components`` and see graphs
- Switch tab to ``Latency metrics``, click on compare to ``Prev week`` and see graphs

