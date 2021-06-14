Exercise 2: Advanced Content-Based Routing
##############################################################

.. contents:: Contents
    :local:
    :depth: 1


Objectives
*************************************************************
|
- Deploying the setup belowUsing by using more features available in the resource VirtualServer compared to exercise 1
    - if path is */redirect* then the action is **redirect** to http://www.nginx.com.
    - if path is */proxy* then the action is **proxy** to add/rewrite/ignore some headers.
    - if path is */return_page* then the action is **return** to reply with a custom web page.
    - in each action, some variables could be used like: $request_uri, $request_method, $request_body, $scheme, $host, etc (cf `here <https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/#action-return>`_ for a complete list of available variables).
|


Step 1: Create a new manifest for the deployment
*************************************************************

- Copy and Paste the manifest below into a new file named **cafe-virtual-server-lab2-ex2.yaml**.
- REPLACE {{SITE_ID}} in the field host by your allocated site ID before saving and applying cafe-virtual-server-lab2-ex1.yaml.

.. code-block:: yaml
    :emphasize-lines: 8

        apiVersion: k8s.nginx.org/v1
        kind: VirtualServer
        metadata:
          name: cafeapp
          namespace: lab2-cafeapp
        spec:
          ingressClassName: nginx-external
          host: cafeapp{{SITE_ID}}.com
          tls:
            secret: cafeapp-secret-tls
          upstreams:
          - name: tea
            service: tea
            port: 80
          - name: coffee
            service: coffee
            port: 80
          routes:
          - path: /tea
            action:
              pass: tea
          - path: /coffee
            action:
              pass: coffee
          - path: /redirect
            action:
              redirect:
                url: http://www.nginx.com
                code: 301
          - path: /proxy
            action:
              proxy:
                upstream: coffee
                requestHeaders:
                  pass: true
                  set:
                  - name: My-Header
                    value: Value
                  - name: Client-Cert
                    value: ${ssl_client_escaped_cert}
                responseHeaders:
                  add:
                  - name: My-Header
                    value: Hello_this_your_value
                  - name: IC-Nginx-Version
                    value: ${nginx_version}
                    always: true
                  hide:
                  - x-internal-version
                  ignore:
                  - Expires
                  - Set-Cookie
                  pass:
                  - Server
          - path: /return_page
            action:
              return:
                code: 200
                type: text/plain
                body: "Hello World\n\n\n\nRequest is ${request_uri}\nRequest Method is ${request_method}\nRequest Scheme is ${scheme}\nRequest Host is ${host}\nRequest Lengthis ${request_length}\nNGINX Version is ${nginx_version}\nClient IP address is ${remote_addr}\nClient Port is : ${remote_port}\nLocal Time is ${time_local}\nServer IP Address is ${server_addr}\nServer Port is ${server_port}\nProtocol is ${server_protocol}\n"


Step 2: Deploy the manifest
*************************************************************

*input*:

.. code-block:: bash

        kubectl apply -f cafe-virtual-server-lab2-ex2.yaml

*output*:

.. code-block:: bash

        virtualserver.k8s.nginx.org/cafeapp configured


Step 3: Check the compilation status of the VirtualServer
*************************************************************

.. code-block:: bash

        kubectl describe virtualserver cafeapp -n lab2-cafeapp


Step 4: Test the setup
*************************************************************

- Test with the curl command below.
- REPLACE {{SITE_ID}} in the field host by your allocated site ID.
- Replace {{PATH}} with:
    - https://cafeapp{{SITE_ID}}.com/coffee         -> request is sent to the service coffee
    - https://cafeapp{{SITE_ID}}.com/tea            -> request is sent to the service tea
    - https://cafeapp{{SITE_ID}}.com/redirect       -> client is redirected to www.nginx.com
    - https://cafeapp{{SITE_ID}}.com/return_page    -> custom page Hello World is returned
    - https://cafeapp{{SITE_ID}}.com/proxy          -> requests go to coffee you should see custom headers in the responses



.. code-block:: bash

        curl {{PATH}}


|
|
**Capture The Flag**

    **2.1 What is the name of the action which forwards requests to an upstream?**
    | Tips: `here <https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/#action>`_

    **2.2 What is the name of the action which replies a preconfigured response?**
    | Tips: `here <https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/#action>`_

    **2.3 What is the name of the action which passes requests to an upstream with the ability to modify the request/response (for example, rewrite the URI or modify the headers)?**
    | Tips: `here <https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/#action>`_

    **2.4 What is the name of the action which redirects requests to a provided URL?**
    | Tips: `here <https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/#action>`_



