Exercises
#################################################################

.. contents:: Contents
    :local:
    :depth: 2

----------------------------------------------------------------

This lab demonstrates the value added feature of NGINX App Protect managed by Controller:

- **Life Cycle Management**

    - **Scaling Policy**: manual or auto Scale In / Scale Out using native Kubernetes features
    - **Upgrade**: Do Rolling Upgrade of NGINX App Protect instances using native Kubernetes feature
    - **Security Update**: Update the signatures and threats using Kubernetes Rolling Upgrade

- **Multi-Tenancy**

    - **Segregation**: Isolate user roles (DevOps, NetOps, SecOps)
    - **DevOps Self-Service**: Deploy an application consuming a Security policy available in a catalog
    - **Visibility**: Define Service Level Indicators (SLI) and Service Level Objectives (SLO)

- **False Positive Management**

    - **Simple starting point**: Start with a basic WAF policy and be reactive to handle False Positive
    - **Standard and App specific policies**: Bring your own policies and publish it in catalog
    - **Update a policy widely**: Update your policy and update automatically all Applications that reference it

Life Cycle Management
*********************************************

Exercise 1: Scaling Policy
============================================
- In Lens, edit Deployment of NGINX App Protect instances:  ``Workloads`` **>** ``Deployments`` **>** ``NameSpace: waap-managed`` **>** ``nginx-appprotect``

.. image:: ./_pictures/Lens_Deployment_show.png
   :align: center
   :width: 1000
   :alt: Show instances

- Scroll to see the environment variable when a POD starts

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
- In `NGINX Controller <https://nginxctrl2.eastus2.cloudapp.azure.com>`_, login as SuperNetOps

    - email:  supernetops@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- User Role of a SuperNetOps allows a user to view, create and delete any instance-groups (FULL authorization)

- Click on ``N`` on the top left in order to switch to menu ``infrastructure``

.. image:: ./_pictures/Controller_switch_menu.png
   :align: center
   :width: 900
   :alt: Switch menu

- Click ``Instance Groups``
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

- In Lens, you COULD manually scale out Deployment of NGINX App Protect instances BUT do not do that because an auto-scaling policy is in place

.. image:: ./_pictures/Lens_Deployment_Scale_3.png
   :align: center
   :width: 800
   :alt: Scale Out

- *Horizontal Pod Autoscaler* (HPA) manage the auto-scaling policy. Here, HPA is configured to maintain an average CPU utilization across all Pods less than 80%. See `here <https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#algorithm-details>`_ for more details on the algorithm.
- In Lens, go to ``Configuration`` **>** ``HPA`` (*Horizontal Pod Autoscaler*)

.. image:: ./_pictures/Lens_HPA.png
   :align: center
   :width: 900
   :alt: Scaling Policy

- Set ``minReplicas`` value to `3` then click on *Save & Close*

- HPA will increase the number of replicas (via the deployment)

.. image:: ./_pictures/Lens_HPA_scale_in.png
   :align: center
   :width: 900
   :alt: Scale In event

- In Controller, see the new instance registered in instance-group ``lab_k8s_{{site_ID}}``

_______________________________________________________________________

**Capture The Flag**

    **1.1 What is the threshold of CPU usage that raises a Scale Out?**

_______________________________________________________________________

- Go back after 300 seconds, you will see 2 running replicas. See `here <https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#default-behavior>`_ for more details on the default behavior.

Exercise 2: Upgrade Signatures
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

    cat /var/log/app_protect/compile_error_msg.json | python -m json.tool

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

- Go to ``Workloads`` **>** ``Deployments`` **>** ``NameSpace: waap-managed`` **>** ``nginx-appprotect`` **>** *Edit*
- Update specification using latest image of NGINX App Protect (use CTRL+F on word *image*)

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

- View `rolling upgrade <https://kubernetes.io/docs/concepts/workloads/controllers/deployment/>`_ of PODs by clicking on ``nginx-appprotect`` deployment
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

Multi-Tenancy
*********************************************

Exercise 3: Super NetOps
=============================================

User Role "SuperNetOps" allows user:

    - to manage NGINX App Protect instances (FULL authorization)
    - to view all Services (READ authorization)

- In `NGINX Controller <https://nginxctrl2.eastus2.cloudapp.azure.com>`_, login as SuperNetOps

    - email:  supernetops@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- Go to ``Platform`` **>** ``User Role`` then see configuration PATHs and attached authorization levels

.. image:: ./_pictures/Controller_platform_user_role_supernetops.png
   :align: center
   :width: 900
   :alt: User Role NetOps

- Go to ``Services`` **>** ``Gateway``. You can see all gateways but edit none.

Exercise 4: Super SecOps
============================================

User Role "SuperSecOps" allows user:

    - to manage all WAF policies (FULL authorization)
    - to view all Services (READ authorization)


- In `NGINX Controller <https://nginxctrl2.eastus2.cloudapp.azure.com>`_, login as SuperSecOps in a different web browser if you can

    - email:  supersecops@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- Go to ``Platform`` **>** ``User Role`` then see configuration PATHs and attached authorization levels

.. image:: ./_pictures/Controller_platform_user_role_secops.png
   :align: center
   :width: 900
   :alt: User Role SecOps

- Go to ``Services`` **>** ``Security Strategies``
- See all Security Strategies available for consumption by DevOps

Exercise 5: DevOps
============================================

User Role "DevOps" allows user:

    - to manage (FULL authorization) his applications inside his own environment {{ site_ID }} only
    - to view all WAF policies (READ authorization)

- In `NGINX Controller <https://nginxctrl2.eastus2.cloudapp.azure.com>`_, login as DevOps owner of your site

    - email:  devops{{ site_ID }}@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- Go to ``Platform`` **>** ``User Role`` then see configuration PATHs and attached authorization levels

.. image:: ./_pictures/Controller_platform_user_role_devops.png
   :align: center
   :width: 900
   :alt: User Role DevOps

--------------------------------------------

- DevOps configure an Application using logical objects:

    - **Environment**: a Tenant.
        A DevOps user is assigned into one or multiple Tenants aka *Environment*

    - **Gateway**: a listener on TCP/UDP service or on HTTP(s) host(s).
        A *Gateway* is a configuration object within an *Environment*,
        often used as the top level for defining how applications are delivered to customers – settings such as hostname, protocol, and TLS/SSL behavior –
        though such settings can also be made at a lower level.
        Gateways also employ the concept of *Placement* which is how you link Controller and the NGINX App Protect instances that receive the configurations and do the actual work (data-plane).

    - **App**: a logical group of *Components*
        App configuration object is where you begin to model applications and group traffic‑shaping behaviors.
        You can use as many or as few Apps as needed to meet the needs of your organization.
        The only requirement is that an App must be unique within an Environment.

    - **Component**: a Traffic Management policy with a Security strategy attached.
        Within an App, Components describe the desired traffic‑shaping behavior for the App.
        In the simplest case, all the traffic for a given pathname is sent to the same group of servers.
        But Components also control more advanced shaping like header manipulation, URL rewriting, backend load‑balancing behaviors, cookie handling, and other settings.

    - **Analytics**: auto-generated or custom dashboard offer observability (metrics, security event) at each object level
        Application Insights offer a clear visibility into the number, performance, and ownership costs of your apps
        With per‑app analytics, you gain new insights into app performance and reliability so you can pinpoint performance issues before they impact production.

.. image:: ./_pictures/Controller-App-Security-topology-for-WAF-policies.svg
   :align: center
   :width: 800
   :alt: User Role DevOps

- In case of an Application based on micro-services, teams works on different Components. For example, Sentence API has different micro-services:

.. image:: ./_pictures/Controller-object-relationship.svg
   :align: center
   :width: 800
   :alt: Visual representation of the relationship between the configuration objects

- Go to ``Services`` **>** ``Gateways`` **>** ``sentence-front-managed{{ site_ID }}.f5app.dev``
- ``Edit Gateway`` **>** ``Placements``: show attached ``instance-group``

.. image:: ./_pictures/Controller_service_gateway_placement_instance-group.png
   :align: center
   :width: 900
   :alt: User Role DevOps

**Note:** NGINX Controller v4: specific ``instance-groups`` could be assign to a User Role

- In your web browser, go to ``https://sentence-front-managed{{ site_ID }}.f5app.dev`` and generate some traffic by refreshing 10 times the page
- Go to ``Services`` **>** ``Apps`` **>** ``sentence-front-managed{{ site_ID }}.f5app.dev``
- Update filter to ``Last 5 minutes``
- Switch tab to ``Latency metrics``, click on compare to ``Prev week`` and see graphs

.. image:: ./_pictures/Controller_service_app_metrics.png
   :align: center
   :width: 900
   :alt: App metrics

- Go to ``Components`` **>** ``frontend``
- Update filter to ``Last 5 minutes``
- Switch tab to ``Latency metrics``, click on compare to ``Prev week`` and see graphs

.. image:: ./_pictures/Controller_service_component_metrics.png
   :align: center
   :width: 900
   :alt: Component metrics

False Positive Management
*********************************************

Exercise 6: Simple starting point
============================================
If SecOps doesn't have time to specify a standard WAF policy, a good way is to

    1. use the pre-defined ``balanced_default`` WAF policy in Monitoring mode,
    2. disable matched Signatures to prevent False Positives during QA tests and then in Production,
    3. enable policy in Blocking mode after few weeks in Production
    4. disable matched Signatures in case of complain from legitimate user

- In `NGINX Controller <https://nginxctrl2.eastus2.cloudapp.azure.com>`_, login as DevOps owner of your site

    - email:  devops{{ site_ID }}@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- Go to ``Services`` **>** ``Apps`` **>** ``sentence-front-managed{{ site_ID }}.f5app.dev`` **>** ``Components`` **>** ``frontend``
- Click on ``Edit component`` **>** ``Security``
- Choose Security Strategy proposed by default: ``balanced_default`` then ``Submit``

.. image:: ./_pictures/Controller_service_component_security_balanced_default_monitoring.png
   :align: center
   :width: 700
   :alt: Set default Security strategy in Monitoring mode

- Choose Strategy proposed by default: ``balanced_default`` then ``Submit``
- On your web browser, do an attack:

.. code-block:: bash

    https://sentence-front-managed{{ site_ID }}.f5app.dev?a=<script>alert('This is an attack');</script>

- In Controller, show related Security event

.. image:: ./_pictures/Controller_service_component_security_event.png
   :align: center
   :width: 700
   :alt: Show Security event

- Disable related signatures, set Security Strategy to blocking mode then *Submit*

.. image:: ./_pictures/Controller_service_component_security_balanced_default_blocking.png
   :align: center
   :width: 700
   :alt: Set default Security strategy in Blocking mode

- On your web browser, do an attack:

.. code-block:: bash

    https://sentence-front-managed{{ site_ID }}.f5app.dev/?a='1=1;cat /etc/password'

- Note the *support ID* on the blocking page
- In Controller, show related Security event by filtering on *support ID*

.. image:: ./_pictures/Controller_service_component_security_event_filter.png
   :align: center
   :width: 900
   :alt: Show Security event

- Append related signatures to disabled list of signatures then *Submit*

.. image:: ./_pictures/Controller_service_component_security_balanced_default_blocking2.png
   :align: center
   :width: 700
   :alt: Set default Security strategy in Blocking mode

- On your web browser, repeat the attack and see that is not blocked anymore

.. code-block:: bash

    https://sentence-front-managed{{ site_ID }}.f5app.dev/?a='1=1;cat /etc/password'

Exercise 7: Standard and App specific policy
============================================

A good approach is to define few WAAP policies in order to cover Risk Prevalence by App Criticality.
For example:

**Tier1**

    Less Critical App with no user's persistent privacy data
    Protect from software vulnerabilities and common attack vectors
    Tier1 is is a generic policy

**Tier2**

    Medium Critical App with user's persistent privacy data or generate a loss of revenue if App is out of service
    Protect from targeted attacks and advanced threat actors
    Tier2 should start using a generic policy and then be customized if needed

**Tier3**

    Most Critical App that need an external communication of your company if an incident or a breach is encountered
    Protect from advanced fraud and highly specialized techniques
    Tier3 is clearly an App specific Policy

3 ways to define a policy:

**1. Prepare in BIG-IP UI**

if SecOps used to define WAF policy on BIG-IP, he can still continue to define it using BIG-IP UI and import it in Controller by following `this guide <https://www.nginx.com/blog/bringing-f5-and-nginx-waf-policies-into-controller-app-security/#Preparing-F5-WAF-Policies-for-Controller-App-Security>`_

.. image:: ./_pictures/bring-WAF-policy-Controller-App-Sec_convert-policy.svg
   :align: center
   :width: 700
   :alt: Preparing F5 WAF Policies for Controller App Security

- Go to Jumphost

.. code-block:: bash

    mkdir /tmp/converter
    cp /root/source_images/f5-nap-policies/policy/owasp_rdp.xml /tmp/converter/
    docker run -v /tmp/converter/:/tmp/converter/ aksdistrict{{ site_ID }}.azurecr.io/nap_converter_tool /opt/app_protect/bin/convert-policy -i /tmp/converter/owasp_rdp.xml -o /tmp/converter/policy.json | jq

- The output warns you about features not implemented yet in NGINX App Protect

**2. Prepare in WAFFLER**

Use `this tool <https://waffler.dev/prod/>`_ to discover how to create a basic Declarative Policy through an UI

- allow only ``server-technologies``: ``Nginx``, ``Node.js`` and ``Python``
- In ``Blocking Settings``, alarm and block on violation ``Illegal file type``
- In ``File Types``, add filetype ``md``
- In ``Blocking Settings``, enable violation ``Illegal URL``
- In ``URLs``, add wildcard url ``/admin*``
- In ``Blocking Settings``, enable violation ``IP is in the deny list``
- In ``whitelist-ips``, add a public ip address and ``always`` block it

*output*

.. code-block:: json

    {
      "policy": {
        "name": "policy_name",
        "template": {
          "name": "POLICY_TEMPLATE_NGINX_BASE"
        },
        "applicationLanguage": "utf-8",
        "enforcementMode": "blocking",
        "server-technologies": [
          {
            "serverTechnologyName": "Nginx"
          },
          {
            "serverTechnologyName": "Node.js"
          },
          {
            "serverTechnologyName": "Python"
          }
        ],
        "bot-defense": {
          "mitigations": {},
          "settings": {
            "isEnabled": false
          }
        },
        "urls": [
          {
            "name": "/admin*",
            "wildcardOrder": 0,
            "protocol": "https",
            "type": "wildcard",
            "attackSignaturesCheck": true,
            "metacharsOnUrlCheck": true
          }
        ],
        "filetypes": [
          {
            "name": "md",
            "wildcardOrder": 0
          }
        ],
        "whitelist-ips":
        [
            {
                "blockRequests": "always",
                "ipAddress": "1.1.1.1",
                "ipNetmask": "255.255.255.255"
            }
        ],
        "blocking-settings": {
          "violations": [
            {
              "name": "VIOL_FILETYPE",
              "alarm": true,
              "block": true
            },
            {
              "name": "VIOL_URL",
              "block": true,
              "alarm": true
            },
            {
              "name": "VIOL_BLACKLISTED_IP",
              "block": true,
              "alarm": true
            }
          ]
        }
      }
    }

**3. Advanced tuning**

SecOps can tune his policy directly in the JSON file.
More explanation in `this guide <https://docs.nginx.com/nginx-app-protect/configuration/>`_
and all details in the `Schema reference <https://docs.nginx.com/nginx-app-protect/policy/>`_

- go to `specification of filetypes key <https://docs.nginx.com/nginx-app-protect/policy/#policy/filetypes>`_
- disallow filetype using ``allowed`` key with value ``false``
- go to `specification of urls key <https://docs.nginx.com/nginx-app-protect/policy/#policy/urls>`_
- disallow url using ``allowed`` key with value ``false``
- go to `specification of response page <https://docs.nginx.com/nginx-app-protect/policy/#policy/response-pages>`_
- define you own ``responseContent`` as shown at the end dof the JSON example below

*output*

.. code-block:: json
    :emphasize-lines: 32,40,72-80

    {
        "policy":
        {
            "name": "policy_name",
            "template":
            {
                "name": "POLICY_TEMPLATE_NGINX_BASE"
            },
            "applicationLanguage": "utf-8",
            "enforcementMode": "blocking",
            "server-technologies":
            [
                {
                    "serverTechnologyName": "Nginx"
                },
                {
                    "serverTechnologyName": "Node.js"
                },
                {
                    "serverTechnologyName": "Python"
                }
            ],
            "urls":
            [
                {
                    "name": "/admin*",
                    "wildcardOrder": 0,
                    "protocol": "https",
                    "type": "wildcard",
                    "attackSignaturesCheck": true,
                    "metacharsOnUrlCheck": true,
                    "isAllowed": false
                }
            ],
            "filetypes":
            [
                {
                    "name": "md",
                    "wildcardOrder": 0,
                    "allowed": false
                }
            ],
            "whitelist-ips":
            [
                {
                    "blockRequests": "always",
                    "ipAddress": "1.1.1.1",
                    "ipNetmask": "255.255.255.255"
                }
            ],
            "blocking-settings":
            {
                "violations":
                [
                    {
                        "name": "VIOL_FILETYPE",
                        "alarm": true,
                        "block": true
                    },
                    {
                        "name": "VIOL_URL",
                        "block": true,
                        "alarm": true
                    },
                    {
                        "name": "VIOL_BLACKLISTED_IP",
                        "block": true,
                        "alarm": true
                    }
                ]
            },
            "response-pages":
            [
                {
                    "responseContent": "<html><head><title>NGINX workshop</title></head><body><h1>Blocking page</h1><br><br>Support = <%TS.request.ID()%></body></html>",
                    "responseHeader": "HTTP/1.1 200 OK\r\nConnection: close",
                    "responseActionType": "custom",
                    "responsePageType": "default"
                }
            ]
        }
    }

- save this policy in a file locally

- In `NGINX Controller <https://nginxctrl2.eastus2.cloudapp.azure.com>`_, login as SuperSecOps

    - email:  supersecops@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- Go to ``Services`` **>** ``Security Strategies``
- Create a strategy named ``lab5_site{{site_ID}}`` that reference a policy named also ``lab5_site{{site_ID}}``
- Import your file and check if JSON syntax is *green*

.. image:: ./_pictures/Controller_security_policy_import.png
   :align: center
   :width: 500
   :alt: Import OK

- In `NGINX Controller <https://nginxctrl2.eastus2.cloudapp.azure.com>`_, login as DevOps owner of your site

    - email:  devops{{ site_ID }}@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- Go to ``Services`` **>** ``Apps`` **>** ``sentence-front-managed{{ site_ID }}.f5app.dev`` **>** ``Components`` **>** ``frontend``
- Click on ``Edit component`` **>** ``Security``
- Choose Security Strategy: ``lab5_site{{site_ID}}``  then ``Submit``
- On your browser, check that URLs below are blocked and retrieve ``support ID`` in Security events on Controller:

.. code-block:: bash

    https://sentence-front-managed{{site_ID}}.f5app.dev/README.md
    https://sentence-front-managed{{site_ID}}.f5app.dev/admin

.. image:: ./_pictures/Controller_service_component_security_event_logs.png
   :align: center
   :width: 1100
   :alt: Import OK

Exercise 8: Update a policy widely
============================================

SecOps needs **Enhance Security** by enable bot defense due to a Web Scrapping attack done from fake Google bot

- On Jumphost, try to impersonated the search engine googlebot:

.. code-block:: bash

    curl -k --user-agent "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://sentence-front-managed{{site_ID}}.f5app.dev

- go to `specification of anti-bot key <https://docs.nginx.com/nginx-app-protect/policy/#policy/bot-defense>`_
- in ``settings``, enable anti-bot
- in ``mitigations``, raise an ``alarm`` for bots that belong to  ``classes`` named ``trusted-bot`` and ``untrusted-bot`` ;
- ``block`` bots that does not succeed the validation challenge, i.e. the  class named ``malicious-bot`` ;

*output*

.. code-block:: json
    :emphasize-lines: 23-47

    {
        "policy":
        {
            "name": "policy_name",
            "template":
            {
                "name": "POLICY_TEMPLATE_NGINX_BASE"
            },
            "applicationLanguage": "utf-8",
            "enforcementMode": "blocking",
            "server-technologies":
            [
                {
                    "serverTechnologyName": "Nginx"
                },
                {
                    "serverTechnologyName": "Node.js"
                },
                {
                    "serverTechnologyName": "Python"
                }
            ],
            "bot-defense":
            {
                "settings":
                {
                    "isEnabled": true
                },
                "mitigations":
                {
                    "classes":
                    [
                        {
                            "name": "trusted-bot",
                            "action": "alarm"
                        },
                        {
                            "name": "untrusted-bot",
                            "action": "alarm"
                        },
                        {
                            "name": "malicious-bot",
                            "action": "block"
                        }
                    ]
                }
            },
            "urls":
            [
                {
                    "name": "/admin*",
                    "wildcardOrder": 0,
                    "protocol": "https",
                    "type": "wildcard",
                    "attackSignaturesCheck": true,
                    "metacharsOnUrlCheck": true,
                    "isAllowed": false
                }
            ],
            "filetypes":
            [
                {
                    "name": "md",
                    "wildcardOrder": 0,
                    "allowed": false
                }
            ],
            "whitelist-ips":
            [
                {
                    "blockRequests": "always",
                    "ipAddress": "1.1.1.1",
                    "ipNetmask": "255.255.255.255"
                }
            ],
            "blocking-settings":
            {
                "violations":
                [
                    {
                        "name": "VIOL_FILETYPE",
                        "alarm": true,
                        "block": true
                    },
                    {
                        "name": "VIOL_URL",
                        "block": true,
                        "alarm": true
                    },
                    {
                        "name": "VIOL_BLACKLISTED_IP",
                        "block": true,
                        "alarm": true
                    }
                ]
            },
            "response-pages":
            [
                {
                    "responseContent": "<html><head><title>NGINX workshop</title></head><body><h1>Blocking page</h1><br><br>Support = <%TS.request.ID()%></body></html>",
                    "responseHeader": "HTTP/1.1 200 OK\r\nConnection: close",
                    "responseActionType": "custom",
                    "responsePageType": "default"
                }
            ]
        }
    }

- save this policy in a file locally

- In `NGINX Controller <https://nginxctrl2.eastus2.cloudapp.azure.com>`_, login as SuperSecOps

    - email:  supersecops@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- Go to ``Services`` **>** ``Security Strategies`` **>** ``Policies``
- Edit ``lab5_site{{site_ID}}``, import your file and ``Submit``
- All Components that reference this policy are automatically updated
- On Jumphost, check that the Googlebot request is now blocked:

.. code-block:: bash

    curl -k --user-agent "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://sentence-front-managed{{site_ID}}.f5app.dev

- Retrieve the support ID and see details in Security events

.. image:: ./_pictures/Controller_service_component_security_event_log_bot.png
   :align: center
   :width: 1100
   :alt: Fake googlebot blocked

Exercise 9: False Positive > Update a specific policy
=====================================================

SecOps needs to update a policy for a Specific App due to a **False Positive**.
During functional test in QA environment, a signature were raised.
SQUAD asked to their "Security Coach" to tag them as False Positive.
Application Developers will update their code in next dev Sprint, update task is added in backlog.
For this current Sprint, the new App feature must be deployed in Production as is.

- Check `here <https://clouddocs.f5.com/cloud-services/latest/f5-cloud-services-Essential.App.Protect-Details.html#attack-signatures>`_ the risk level associate to signature 200009283

.. image:: ./_pictures/AWAF_Risk.png
   :align: center
   :width: 700
   :alt: https://techdocs.f5.com/en-us/bigip-15-1-0/big-ip-asm-attack-and-bot-signatures/assigning-attack-signatures-to-security-policies.html

- Append a ``modification`` block in order to disable signature 200009283

.. code-block:: json
    :emphasize-lines: 107-121

    {
        "policy":
        {
            "name": "policy_name",
            "template":
            {
                "name": "POLICY_TEMPLATE_NGINX_BASE"
            },
            "applicationLanguage": "utf-8",
            "enforcementMode": "blocking",
            "server-technologies":
            [
                {
                    "serverTechnologyName": "Nginx"
                },
                {
                    "serverTechnologyName": "Node.js"
                },
                {
                    "serverTechnologyName": "Python"
                }
            ],
            "bot-defense":
            {
                "settings":
                {
                    "isEnabled": true
                },
                "mitigations":
                {
                    "classes":
                    [
                        {
                            "name": "trusted-bot",
                            "action": "alarm"
                        },
                        {
                            "name": "untrusted-bot",
                            "action": "alarm"
                        },
                        {
                            "name": "malicious-bot",
                            "action": "block"
                        }
                    ]
                }
            },
            "urls":
            [
                {
                    "name": "/admin*",
                    "wildcardOrder": 0,
                    "protocol": "https",
                    "type": "wildcard",
                    "attackSignaturesCheck": true,
                    "metacharsOnUrlCheck": true,
                    "isAllowed": false
                }
            ],
            "filetypes":
            [
                {
                    "name": "md",
                    "wildcardOrder": 0,
                    "allowed": false
                }
            ],
            "whitelist-ips":
            [
                {
                    "blockRequests": "always",
                    "ipAddress": "1.1.1.1",
                    "ipNetmask": "255.255.255.255"
                }
            ],
            "blocking-settings":
            {
                "violations":
                [
                    {
                        "name": "VIOL_FILETYPE",
                        "alarm": true,
                        "block": true
                    },
                    {
                        "name": "VIOL_URL",
                        "block": true,
                        "alarm": true
                    },
                    {
                        "name": "VIOL_BLACKLISTED_IP",
                        "block": true,
                        "alarm": true
                    }
                ]
            },
            "response-pages":
            [
                {
                    "responseContent": "<html><head><title>NGINX workshop</title></head><body><h1>Blocking page</h1><br><br>Support = <%TS.request.ID()%></body></html>",
                    "responseHeader": "HTTP/1.1 200 OK\r\nConnection: close",
                    "responseActionType": "custom",
                    "responsePageType": "default"
                }
            ]
        },
        "modifications":
        [
            {
                "entityType": "signature",
                "entity":
                {
                    "signatureId": 200001475
                },
                "entityChanges":
                {
                    "enabled": false
                },
                "action": "add-or-update"
            }
        ]
    }

*extra time* :
As describe in `Exercise 7 <https://f5-k8s-ctfd.docs.emea.f5se.com/en/latest/class6/module2/module2.html#exercise-7-standard-and-app-specific-policy>`_,
create a new strategy and attach it to component

Extra time: API Protection
*********************************************

Sentence API is wide opened on Internet, please do something to protect it!
Developpers have saved their API specification in `this repository <https://github.com/nergalex/f5-nap-policies/blob/master/policy/open-api-files/sentence-api.f5app.dev.yaml>`_

- Create a local file on your computer
- Copy paste the policy below.

**Note** Only the emphasize lines are specific for each API Protection policy, the others are generic (source: ``NGINX App Protect API Security template Policy`` file)

.. code-block:: json
    :emphasize-lines: 4,6,108-113

    {
        "policy":
        {
            "name": "sentence-api",
            "description": "Based on NGINX App Protect API Security template Policy",
            "enforcementMode": "blocking",
            "template":
            {
                "name": "POLICY_TEMPLATE_NGINX_BASE"
            },
            "urls":
            [
                {
                    "name": "/_codexch",
                    "type": "explicit",
                    "isAllowed": true,
                    "wildcardOrder": 0,
                    "attackSignaturesCheck": false,
                    "metacharsOnUrlCheck": false
                }
            ],
            "blocking-settings":
            {
                "violations":
                [
                    {
                        "block": true,
                        "description": "Disallowed file upload content detected in body",
                        "name": "VIOL_FILE_UPLOAD_IN_BODY"
                    },
                    {
                        "block": true,
                        "description": "Mandatory request body is missing",
                        "name": "VIOL_MANDATORY_REQUEST_BODY"
                    },
                    {
                        "block": true,
                        "description": "Illegal parameter location",
                        "name": "VIOL_PARAMETER_LOCATION"
                    },
                    {
                        "block": true,
                        "description": "Mandatory parameter is missing",
                        "name": "VIOL_MANDATORY_PARAMETER"
                    },
                    {
                        "block": true,
                        "description": "JSON data does not comply with JSON schema",
                        "name": "VIOL_JSON_SCHEMA"
                    },
                    {
                        "block": true,
                        "description": "Illegal Base64 value",
                        "name": "VIOL_PARAMETER_VALUE_BASE64"
                    },
                    {
                        "block": true,
                        "description": "Disallowed file upload content detected",
                        "name": "VIOL_FILE_UPLOAD"
                    },
                    {
                        "block": true,
                        "description": "Illegal request content type",
                        "name": "VIOL_URL_CONTENT_TYPE"
                    },
                    {
                        "block": true,
                        "description": "Illegal static parameter value",
                        "name": "VIOL_PARAMETER_STATIC_VALUE"
                    },
                    {
                        "block": true,
                        "description": "Illegal parameter value length",
                        "name": "VIOL_PARAMETER_VALUE_LENGTH"
                    },
                    {
                        "block": true,
                        "description": "Illegal parameter data type",
                        "name": "VIOL_PARAMETER_DATA_TYPE"
                    },
                    {
                        "block": true,
                        "description": "Illegal parameter numeric value",
                        "name": "VIOL_PARAMETER_NUMERIC_VALUE"
                    },
                    {
                        "block": true,
                        "description": "Illegal URL",
                        "name": "VIOL_URL"
                    },
                    {
                        "block": true,
                        "description": "Illegal parameter",
                        "name": "VIOL_PARAMETER"
                    },
                    {
                        "block": true,
                        "description": "Illegal empty parameter value",
                        "name": "VIOL_PARAMETER_EMPTY_VALUE"
                    },
                    {
                        "block": true,
                        "description": "Illegal repeated parameter name",
                        "name": "VIOL_PARAMETER_REPEATED"
                    }
                ]
            },
            "open-api-files":
            [
                {
                    "link": "https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/open-api-files/sentence-api.f5app.dev.yaml"
                }
            ]
        }
    }

- In `NGINX Controller <https://nginxctrl2.eastus2.cloudapp.azure.com>`_, login as SuperSecOps

    - email:  supersecops@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- Go to ``Services`` **>** ``Security Strategies``
- Create a strategy named ``lab5_site{{site_ID}}_api`` that reference a policy named also ``lab5_site{{site_ID}}_api``
- Import your file and check if JSON syntax is *green*

- In `NGINX Controller <https://nginxctrl2.eastus2.cloudapp.azure.com>`_, login as DevOps owner of your site

    - email:  devops{{ site_ID }}@f5cloudbuilder.dev
    - password: NGINXC0ntroller!

- Go to ``Services`` **>** ``Apps`` **>** ``sentence-api-managed{{ site_ID }}.f5app.dev`` **>** ``Components``

*ToDo* Remove all components during creation and create only one named "main" based on '/'

- Delete all components except name
- Click on ``Edit component`` **>** ``URIs`` **>** ``/name`` and rename uri as ``/``
- Click on ``Edit component`` **>** ``Security``
- Choose Security Strategy: ``lab5_site{{site_ID}}_api`` then ``Submit``
- On your browser, check that URLs below are blocked and retrieve ``support ID`` in Security events on Controller:

.. code-block:: bash

    https://sentence-front-api{{site_ID}}.f5app.dev/README.md
    https://sentence-front-api{{site_ID}}.f5app.dev/admin
