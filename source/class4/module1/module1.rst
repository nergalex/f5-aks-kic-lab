Arcadia
##############################################################

.. contents:: Contents
    :local:

Exercise 1: NGINX Configuration
*********************

- Log into IC

.. code-block:: bash
    :emphasize-lines: 3

    $ kubectl get pods -n external-ingress-controller
    NAME                                              READY   STATUS    RESTARTS   AGE
    nap-external-ingress-controller-7576b65b4-ps4ck   1/1     Running   0          8d

    $ kubectl exec --namespace external-ingress-controller -it nap-external-ingress-controller-7576b65b4-ps4ck bash

- See installed NGINX Plus software

.. code-block:: bash
    :emphasize-lines: 3

    $ apt list --installed | grep nginx

    nginx-plus-module-appprotect/now 23+3.332.0-1~buster amd64 [installed,local]
    nginx-plus-module-njs/now 23+0.5.0-1~buster amd64 [installed,local]
    nginx-plus/now 23-1~buster amd64 [installed,local]

- Show App Protect directives in Arcadia configuration

.. code-block:: bash

    $ grep protect /etc/nginx/conf.d/lab1-arcadia-arcadia-ingress-external-master.conf

.. code-block:: nginx

    app_protect_enable on;
    app_protect_policy_file /etc/nginx/waf/nac-policies/external-ingress-controller_generic-security-level-low;
    app_protect_security_log_enable on;
    app_protect_security_log /etc/nginx/waf/nac-logconfs/external-ingress-controller_naplogformat syslog:server=10.1.0.10:5144;

- On Jumphost, show App Protect directives in Arcadia ingress resource

.. code-block:: bash
    :emphasize-lines: 4

    $ kubectl describe ingress -n lab1-arcadia arcadia-ingress-external-master | grep protect

    Annotations:  appprotect.f5.com/app-protect-enable: True
                  appprotect.f5.com/app-protect-policy: external-ingress-controller/generic-security-level-low
                  appprotect.f5.com/app-protect-security-log: external-ingress-controller/naplogformat
                  appprotect.f5.com/app-protect-security-log-destination: syslog:server=10.1.0.10:5144
                  appprotect.f5.com/app-protect-security-log-enable: True

Exercise 2: Security Policy
*********************

- Show App Protect policy resource

.. code-block:: yaml
    :emphasize-lines: 12

    $ kubectl describe appolicy -n external-ingress-controller generic-security-level-low | grep -A 100 Spec

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

- On IC, show App Protect policy

.. code-block:: json

    $ cat /etc/nginx/waf/nac-policies/external-ingress-controller_generic-security-level-low

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

.. note:: **Capture The Flag**
    | **Which request type are logged by App Protect for Arcadia application?**
    | all
    | Tip: `App Protect Logs <https://docs.nginx.com/nginx-ingress-controller/app-protect/configuration/#app-protect-logs>`_

Exercise 3: Monitoring
*********************

- To test that the site is protected, on Jumphost, append a script to the end of the curl statement:

.. code-block:: html

    $ curl -k -s https://arcadia1.f5app.dev/?a=%3Cscript%3E

    <html><head><title>Request Rejected</title></head><body>The requested URL was rejected.
    Please consult with your administrator.<br><br>
    Your support ID is: 4096465330496922252
    <br><br><a href='javascript:history.back();'>[Go Back]</a></body></html>

- Connect to Kibana ``https://kibana{{site_ID}}.f5app.dev`` >> Dashboard >> Overview
- Add a filter ``vs_name is *arcadia1.f5app.dev*``

.. image:: ./_pictures/kibana_filter.png
   :align: center
   :width: 300
   :alt: SecureCRT

- Add another filter ``support_id is {{support_ID}}`` and replace {{support_ID}} by previous blocked request

- Review log

.. note:: **Capture The Flag**
    | **What is the policy name?**
    | generic-security-level-low

.. note:: **Capture The Flag**
    | **What is the client_class for curl?**
    | Untrusted Bot

.. note:: **Capture The Flag**
    | **What are the signatures raised?**
    | 200001475, 200000098

.. note:: **Capture The Flag**
    | **What are the violations raised?**
    | Illegal meta character in value, Attack signature detected, Violation Rating Threat detected, Bot Client Detected


