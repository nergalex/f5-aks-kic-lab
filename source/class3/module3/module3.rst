Exercise 3: Advanced Content-Based Routing
##############################################################

.. contents:: Contents
    :local:
    :depth: 1

Objectives
*************************************************************
|
- For that new deployment we use a lot more features available in the Custom Resource VirtualServer.
    - if path is */redirect* then the action is **redirect** to http://www.nginx.com.
    - if path is */proxy* then the action is **proxy** to add/rewrite/ignore some headers.
    - if path is */return_page* then the action is **return** to reply with a custom web page.
    - in each action, variables could be used like: $request_uri, $request_method, $request_body, $scheme, $host, $request_time, $request_length, $connection, $remote_addr, $remote_port, $ssl_cipher, $ssl_client_cert, etc
|
|

- Step 1: Modify the deployment done in the previous lab.

Copy and Paste the manifest below into a new file named cafe-virtual-server-Lab-2C.yaml and deploy it.

.. code-block:: yaml

        apiVersion: k8s.nginx.org/v1
        kind: VirtualServer
        metadata:
          name: app-cafe
          namespace: cafe-ns
        spec:
          ingressClassName: nginx-external
          host: cafe.example.com
          tls:
            secret: cafe-secret
          upstreams:
          - name: tea
            service: tea-svc
            port: 80
          - name: coffee
            service: coffee-svc
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


- Step 2: Deploy the manifest:

*input*:

.. code-block:: bash

        kubectl apply -f cafe-virtual-server-lab-2C.yaml

*output*:

.. code-block:: bash

        virtualserver.k8s.nginx.org/app-cafe configured


- Step 3: Check the compilation status of the VirtualServer with the command below:

.. code-block:: bash

        kubectl describe virtualserver app-cafe -n cafe-ns


- Step 4: Test the setup

- Test the pages below using the following curl command:

    - https://cafe.example.com/coffee         -> request is sent to the service coffee

    - https://cafe.example.com/tea            -> request is sent to the service tea

    - https://cafe.example.com/redirect       -> client is redirected to www.nginx.com

    - https://cafe.example.com/return_page    -> custom page Hello World is returned

    - https://cafe.example.com/proxy          -> requests go to coffee you should see custom headers in the responses

.. code-block:: bash

        curl https://cafe.example.com/<PATH> --resolve cafe.example.com:443:<EXTERNAL_IP_Cluster> --insecure

|
|
|
**Capture The Flag**

    **2c.1 What is the name of the action which forwards requests to an upstream?**


    **2c.2 What is the name of the action which replies a preconfigured response?**


    **2c.3 What is the name of the action which passes requests to an upstream with the ability to modify the request/response (for example, rewrite the URI or modify the headers)?**


    **2c.4 What is the name of the action which redirects requests to a provided URL?**




