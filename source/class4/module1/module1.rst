Arcadia
##############################################################

.. contents:: Contents
    :local:

Exercise 1: NGINX Configuration
*********************

- Get a IC's POD name

.. code-block:: bash

    kubectl get pods -n external-ingress-controller

*output:*

.. code-block:: bash
    :emphasize-lines: 2

    NAME                                              READY   STATUS    RESTARTS   AGE
    nap-external-ingress-controller-7576b65b4-ps4ck   1/1     Running   0          8d

- Log into IC's container

.. code-block:: bash

    kubectl exec --namespace external-ingress-controller -it nap-external-ingress-controller-7576b65b4-ps4ck bash

Package
=====================

- See installed App Protect software

.. code-block:: bash

    apt list --installed | grep protect

*output:*

.. code-block:: bash
    :emphasize-lines: 1

    app-protect-attack-signatures/now 2021.04.29-1~buster amd64 [installed,local]
    app-protect-compiler/now 6.3.6-1~buster amd64 [installed,local]
    app-protect-engine/now 6.3.6-1~buster amd64 [installed,local]
    ...

**Capture The Flag**

    **2.1 What is the name of Threat Campaigns package?**
    | app-protect-threat-campaigns

    **2.2 How many signature attack update were released in April?**
    | 10
    | Tip: ``yum list --showduplicates {{ package name }}``

Directive
=====================

- Show App Protect directives in Arcadia configuration

.. code-block:: bash

    grep protect /etc/nginx/conf.d/lab1-arcadia-arcadia-ingress-external-master.conf

*output:*

.. code-block:: nginx

    app_protect_enable on;
    app_protect_policy_file /etc/nginx/waf/nac-policies/external-ingress-controller_generic-security-level-low;
    app_protect_security_log_enable on;
    app_protect_security_log /etc/nginx/waf/nac-logconfs/external-ingress-controller_naplogformat syslog:server=10.1.0.10:5144;

Annotation
=====================

- On Jumphost, show App Protect annotations in Arcadia ingress resource

.. code-block:: bash

    kubectl describe ingress -n lab1-arcadia arcadia-ingress-external-master | grep protect

*output:*

.. code-block:: bash
    :emphasize-lines: 2

    Annotations:  appprotect.f5.com/app-protect-enable: True
                  appprotect.f5.com/app-protect-policy: external-ingress-controller/generic-security-level-low
                  appprotect.f5.com/app-protect-security-log: external-ingress-controller/naplogformat
                  appprotect.f5.com/app-protect-security-log-destination: syslog:server=10.1.0.10:5144
                  appprotect.f5.com/app-protect-security-log-enable: True

Exercise 2: Security Policy
*********************

APPolicy
=====================

- Show App Protect policy resource

.. code-block:: bash

    kubectl describe appolicy -n external-ingress-controller generic-security-level-low | grep -A 100 Spec

*output:*

.. code-block:: yaml
    :emphasize-lines: 10

    Spec:
      Policy:
        Application Language:  utf-8
        Blocking - Settings:
          Violations:
            Alarm:         true
            Block:         true
            Name:          VIOL_HTTP_RESPONSE_STATUS
        Enforcement Mode:  blocking
        Name:              generic-security-level-low
        Signatures:
          Enabled:       false
          Signature Id:  200000128
        Template:
          Name:  POLICY_TEMPLATE_NGINX_BASE

Declarative policy
=====================

- On IC, show App Protect `declarative policy <https://docs.nginx.com/nginx-app-protect/policy/>`_

.. code-block:: bash

    cat /etc/nginx/waf/nac-policies/external-ingress-controller_generic-security-level-low

*output:*

.. code-block:: json

    {
      "policy": {
        "applicationLanguage": "utf-8",
        "blocking-settings": {
          "violations": [
            {
              "alarm": true,
              "block": true,
              "name": "VIOL_HTTP_RESPONSE_STATUS"
            }
          ]
        },
        "enforcementMode": "blocking",
        "name": "generic-security-level-low",
        "signatures": [
          {
            "enabled": false,
            "signatureId": 200000128
          }
        ],
        "template": {
          "name": "POLICY_TEMPLATE_NGINX_BASE"
        }
      }
    }

**Capture The Flag**

    **2.3 Which request type are logged by App Protect for Arcadia application?**
    | all
    | Tip: `App Protect Logs <https://docs.nginx.com/nginx-ingress-controller/app-protect/configuration/#app-protect-logs>`_

Exercise 3: Monitoring
*********************

Support ID
=====================

- To test that the site is protected, on Jumphost, append a script to the end of the curl statement:

.. code-block:: bash

    curl -k -s "https://arcadia1.f5app.dev/?a=<script>"

*output:*

.. code-block:: html
    <html><head><title>Request Rejected</title></head><body>The requested URL was rejected.
    Please consult with your administrator.<br><br>
    Your support ID is: 4096465330496922252
    <br><br><a href='javascript:history.back();'>[Go Back]</a></body></html>

Analytics
=====================

- Connect to Kibana ``https://kibana{{site_ID}}.f5app.dev`` >> Dashboard >> Overview
- Add a filter ``vs_name is *arcadia1.f5app.dev*``

.. image:: ./_pictures/kibana_filter.png
   :align: center
   :width: 300
   :alt: SecureCRT

- Add another filter ``support_id is {{support_ID}}`` and replace {{support_ID}} by previous blocked request

- Review log

**Capture The Flag**

    **3.1 What is the policy name?**
    | generic-security-level-low

    **3.2 What is the client_class for curl?**
    | Untrusted Bot

    **3.3 Which violations are raised?**
    | Illegal meta character in value, Attack signature detected, Violation Rating Threat detected, Bot Client Detected

    **3.4 Which attack signatures are detected?**
    | 200001475, 200000098

Exercise 4: Modifications
*************************

By default, App Protect minimize false positives :
    - block requests that are declared as threats their Violation Rating is 4 or 5.
    - if the violation rating is 4-5 the request is blocked using the VIOL_RATING_THREAT violation.
    - other requests which have a lower violation rating are not blocked, except for some specific violations described `here <https://docs.nginx.com/nginx-app-protect/configuration/#basic-configuration-and-the-default-policy>`_ .

App Developers assume that matched signatures are a False Positive.
They added modifications of security policy `here <https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/modifications/arcadia.f5app.dev.json>`_.

Now, a new security policy for Arcadia must be applied to allow this request.

APPolicy - Manifest
=====================

- On Jumphost, create a manifest of App Protect Policy reusing `the current policy <https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/core/secure_low.yaml>`_ and referencing modifications set by AppDev

.. code-block:: bash

    vi lab3-arcadia_appolicy.yaml

*input:*

.. code-block:: yaml
    :linenos:
    :emphasize-lines: 25

    apiVersion: appprotect.f5.com/v1beta1
    kind: APPolicy
    metadata:
      name: arcadia
      namespace: external-ingress-controller
      labels:
        app: arcadia
        policy-version: 1.0.0
    spec:
      policy:
        applicationLanguage: utf-8
        blocking-settings:
          violations:
          - alarm: true
            block: true
            name: VIOL_HTTP_RESPONSE_STATUS
        enforcementMode: blocking
        name: arcadia
        signatures:
        - enabled: false
          signatureId: 200000128
        template:
          name: POLICY_TEMPLATE_NGINX_BASE
      modificationsReference:
          link: https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/modifications/arcadia.f5app.dev.json

APPolicy - APPLY
=====================

- Apply manifest APPolicy

.. code-block:: bash

    kubectl apply -f lab3-arcadia_appolicy.yaml

*output:*

.. code-block:: bash

    appolicy.appprotect.f5.com/arcadia created

APPolicy - CHECK
=====================

- Check apply status

.. code-block:: bash
    :emphasize-lines: 6

    kubectl describe appolicy -n external-ingress-controller arcadia

*output:*

.. code-block:: log

    Events:
      Type    Reason          Age   From                      Message
      ----    ------          ----  ----                      -------
      Normal  AddedOrUpdated  36s   nginx-ingress-controller  AppProtectPolicy external-ingress-controller/arcadia was added or updated

Ingress - Manifest
=====================

- Create a manifest reusing current Arcadia's ingress resource and reference newly created APPolicy

.. code-block:: bash

    vi lab3-arcadia_ingress.yaml

*input:*

    .. note:: Replace {{ site_ID }} in Manifest file, see highlighted lines below

.. code-block:: yaml
    :linenos:
    :emphasize-lines: 19,24,27

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: "arcadia-ingress-external-master"
      namespace: "lab1-arcadia"
      labels:
        app: "arcadia"
        policy_target: external
      annotations:
        nginx.org/mergeable-ingress-type: "master"
        nginx.org/server-snippets: |
          proxy_ignore_headers X-Accel-Expires Expires Cache-Control;
          proxy_cache_valid any 30s;
        ingress.kubernetes.io/ssl-redirect: "true"
        appprotect.f5.com/app-protect-policy: "external-ingress-controller/arcadia"
        appprotect.f5.com/app-protect-enable: "True"
        appprotect.f5.com/app-protect-security-log-enable: "True"
        appprotect.f5.com/app-protect-security-log: "external-ingress-controller/naplogformat"
        appprotect.f5.com/app-protect-security-log-destination: "syslog:server=10.{{ site_ID }}.0.10:5144"
    spec:
      ingressClassName: "nginx-external"
      tls:
      - hosts:
        - "arcadia{{ site_ID }}.f5app.dev"
        secretName: "arcadia-secret-tls"
      rules:
      - host: "arcadia{{ site_ID }}.f5app.dev"

Ingress - APPLY
=====================

- Apply manifest Ingress

.. code-block:: bash

    kubectl apply -f lab3-arcadia_ingress.yaml

*output:*

.. code-block:: bash

    ingress.networking.k8s.io/arcadia-ingress-external-master configured

Ingress - CHECK
=====================

- Check apply status

.. code-block:: bash

    kubectl describe ingress -n lab1-arcadia arcadia-ingress-external-master

*output:*

.. code-block:: bash
    :emphasize-lines: 4

    Events:
      Type    Reason          Age                   From                      Message
      ----    ------          ----                  ----                      -------
      Normal  AddedOrUpdated  45s (x20 over 7d23h)  nginx-ingress-controller  Configuration for lab1-arcadia/arcadia-ingress-external-master was added or update

- Check that request is not block by WAF

.. code-block:: bash

    curl -k -s "https://arcadia1.f5app.dev/?a=<script>"

Exercise 5: Anti Automation
***************************
Anti Automation provides basic bot protection by detecting bot signatures and clients that falsely claim to be browsers or search engines.
The bot-defense section in the policy is enabled by default.

The default actions for bot classes are:

    - detect for trusted-bot
    - alarm for untrusted-bot
    - block for malicious-bot

Now, core policy is updated by SecOps to block ``untrusted-bot`` class.

APPolicy - Manifest
=====================

- On Jumphost, create a new manifest of App Protect Policy using `new core policy <https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/core/secure_medium.yaml>`_ and still referencing modifications set by AppDev

.. code-block:: bash

    vi lab3-arcadia_appolicy_bot.yaml

*input:*

.. code-block:: yaml
    :linenos:
    :emphasize-lines: 23-33

    apiVersion: appprotect.f5.com/v1beta1
    kind: APPolicy
    metadata:
      name: arcadia
      namespace: external-ingress-controller
      labels:
        app: arcadia
        policy-version: 1.1.0
    spec:
      policy:
        name: arcadia
        enforcementMode: blocking
        applicationLanguage: utf-8
        template:
          name: POLICY_TEMPLATE_NGINX_BASE
        server-technologies:
          - serverTechnologyName: Unix/Linux
          - serverTechnologyName: Nginx
          - serverTechnologyName: "Apache/NCSA HTTP Server"
          - serverTechnologyName: PHP
          - serverTechnologyName: JavaScript
          - serverTechnologyName: PostgreSQL
        bot-defense:
          settings:
            isEnabled: true
          mitigations:
            classes:
            - name: trusted-bot
              action: alarm
            - name: untrusted-bot
              action: block
            - name: malicious-bot
              action: block
      modificationsReference:
          link: https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/modifications/arcadia.f5app.dev.json

APPolicy - APPLY
=====================

- Apply new Arcadia's APPolicy

.. code-block:: bash

    kubectl apply -f lab3-arcadia_appolicy_bot.yaml

*output:*

.. code-block:: bash

    appolicy.appprotect.f5.com/arcadia configured

App Protect - configuration
===========================

- On IC, see configured WAF policies

.. code-block:: bash

    grep -A 8 arcadia /opt/app_protect/config/config_set.json

*output:*

.. code-block:: json
    :emphasize-lines: 1

    "/etc/nginx/waf/nac-policies/external-ingress-controller_arcadia": {
    "import_filename": "/etc/nginx/waf/nac-policies/external-ingress-controller_arcadia",
    "src_config_line": 79,
    "vs_info": {
      "49-arcadia1.f5app.dev:12-/": {
        "vs_name": "49-arcadia1.f5app.dev:12-/",
        "logging": {
          "dest_ip": "10.1.0.10",
          "dest_port": "5144",
          "dest_filename": "",
          "$ref": "/etc/nginx/waf/nac-logconfs/external-ingress-controller_naplogformat",
          "src_config_line": 81
        }
      }
    (...)

App Protect - declarative policy
================================

- See content of Arcadia's WAF policy

.. code-block:: bash

    cat /etc/nginx/waf/nac-policies/external-ingress-controller_arcadia

App Protect - compilation status
================================

- See WAF compilation output

.. code-block:: bash

    cat /var/log/app_protect/compile_error_msg.json

*output:*

.. code-block:: json
    :emphasize-lines: 2

    {
      "completed_successfully": true,
      "user_signatures_packages": [],
      "threat_campaigns_package": {
        "revision_datetime": "2021-05-04T21:03:00Z",
        "version": "2021.05.04"
      },
      "attack_signatures_package": {
        "revision_datetime": "2021-04-29T10:41:04Z",
        "version": "2021.04.29"
      }
    }

CHECK
================================

- Test again with curl

.. code-block:: bash

    curl -k -s "https://arcadia1.f5app.dev/?a=<script>"

- Test again with your web browser ``https://arcadia1.f5app.dev/?a=<script>``

- Review log generated by curl using support ID in Kibana ``https://kibana{{site_ID}}.f5app.dev``

**Capture The Flag**

    **5.1 What is the violation rating?**
    | 3

    **5.2 What are the violations?**
    | Illegal meta character in value, Bot Client Detected

Exercise 6: Checks of the trusted bot’s authenticity
****************************************************
NAP's Search engine signatures such as googlebot are under the ``trusted_bots`` class,
but App Protect performs additional checks of the trusted bot’s authenticity
as reverse DNS for example.

- Try to  impersonated the search engine googlebot

.. code-block:: bash

    curl -k --user-agent "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://arcadia{{site_ID}}.f5app.dev

- Review log generated by curl using support ID in Kibana ``https://kibana{{site_ID}}.f5app.dev``

**Capture The Flag**

    **6.1 What is the bot anomaly?**
    | Search Engine Verification Failed

    **6.2 What is the client class?**
    | Malicious Bot

