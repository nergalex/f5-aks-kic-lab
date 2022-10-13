Exercises
#################################################################

.. contents:: Contents
    :local:
    :depth: 2

----------------------------------------------------------------

1. Inventory in NGINX Instance Manager
*********************************************

1.1. Scale Out
============================================
- Connect to NMS `here <https://nms.f5dc.dev>`_
- Check that instances are UP: ``Instance Manager`` **>** ``Instances``
- In Lens, connect to your vK8S
- Edit Deployment of NGINX API GW:  ``Workloads`` **>** ``Deployments`` **>** ``NameSpace: {{ site_ID }}`` **>** ``nginx-apigw``
- Note the ``Instance Group`` value for ``spec.template.spec.containers[0].env.name = ENV_CONTROLLER_INSTANCE_GROUP``
- Scale Out by modifying the value of ``spec.replicas`` to `2` per Site, then ``Save & Close``
- Review POD creation: ``Workloads`` **>** ``PODs``
- In NMS, check that new instances are well automatically discovered: ``Instance Manager`` **>** ``Instances``
- Check that new instances are well automatically attached the Instance Group name noted before: ``Instance Manager`` **>** ``Instance Group``

**demo**

.. raw:: html

    <a href="http://www.youtube.com/watch?v=p-xUuhJ7tBg"><img src="http://img.youtube.com/vi/p-xUuhJ7tBg/0.jpg" width="600" height="300" title="Scale Out" alt="Scale Out"></a>

1.2. Scale In
============================================
- In Lens, Scale In by modifying the value of ``spec.replicas`` to `1` per Site, then ``Save & Close``
- In NMS, check that new instances are well automatically unregistered after 150 seconds

**demo**

.. raw:: html

    <a href="http://www.youtube.com/watch?v=9MR91T3YDwg"><img src="http://img.youtube.com/vi/9MR91T3YDwg/0.jpg" width="600" height="300" title="Scale In" alt="Scale In"></a>

_______________________________________________________________________

**Capture The Flag**

    **1. What is the value of terminationGracePeriodSeconds?**

_______________________________________________________________________

2. Authentication | oAuth OIDC + PKCE
**********************************************
- Connect to NMS `here <https://nms.f5dc.dev>`_
- Check that instances are UP: ``API Connectivity Manager`` **>** ``Infrastructure``  **>** ``Workspaces`` **>** ``sentence{{ site_ID }}`` **>** ``Environment`` **>** ``sentence{{ site_ID }}-non-prod`` **>** ``API Gateway Cluster`` **>** ``Sentence{{ site_ID }}-non-prod`` **>** ``Instances``
- Check that oAuth Policy is configured to return the ID token to the client: ``Policies`` **>** ``Manage`` **>** ``OpenID Connect Relying Party`` **>** ``...`` **>** ``Edit policy`` **>** ``Select the token to return to client upon login``
- Connect to ``sentence{{ site_ID }}-non-prod.f5dc.dev`` using your web browser
- Click on login button on the top right of the page
- Use user credential: producer{{ site_ID }}@acme.com / F5-NGINX-lab!
- Click on the `+` button under the *adjective* of the sentence
- A success should appear

**demo**

.. raw:: html

    <a href="http://www.youtube.com/watch?v=f3jQuDWSaxQ"><img src="http://img.youtube.com/vi/f3jQuDWSaxQ/0.jpg" width="600" height="300" title="End User Authentication" alt="End User Authentication"></a>

_______________________________________________________________________

**Capture The Flag**

    **2.1. What is the parameter name returned by the API GW after login flow?**
    **2.2. What are the cookie name added by the application?**
    **2.3. What are the cookie names added by the API GW?**

_______________________________________________________________________

3. Authentication | Client / Single Page Application / Mobile App
*************************************************************************

3.1 Create an Application Credential
============================================

- Connect to NMS `here <https://nms.f5dc.dev>`_
- Check that *APIKey Authentication* is enabled on *Colors* micro-service: ``API Connectivity Manager`` **>** ``Services``  **>** ``Workspaces`` **>** ``sentence{{ site_ID }}`` **>** ``API Proxies`` **>** ``colors-non-prod`` **>** ``...`` **>** ``Edit proxy`` **>** ``Policies`` **>** ``APIKey Authentication``
- Check that there is no APIKey registered: ``APIKey Authentication`` **>** ``Policies`` **>** ``Credentials``
- Note the Developer Portal name where *colors*'s API is published: ``API Proxies`` **>** ``colors-non-prod`` **>** ``Edit proxy`` **>** ``Configuration`` **>** ``Developer Portal`` **>** ``Portal Proxy Hostname``
- Check that the status of the Developer Portal is UP: ``API Connectivity Manager`` **>** ``Infrastructure``  **>** ``Workspaces`` **>** ``sentence{{ site_ID }}`` **>** ``Environment`` **>** ``sentence{{ site_ID }}-non-prod`` **>** ``Developer Portal Clusters`` **>** ``Sentence{{ site_ID }}-non-prod-devportal`` **>** ``Instances``
- Click on the top link in order to be redirected to ``sentence{{ site_ID }}-non-prod-devportal.f5dc.dev``
- Login with user credential ``producer{{ site_ID }}@acme.com`` / ``F5-NGINX-lab!``
- Click on ``App Credentials`` **>** ``Create organization`` **>** ``Create credential``
- ACM is reconfiguring API GW, please wait 30s. Click on ``APIs`` and browse available APIs. After 30s, go back to your ``App Credential`` in your ``Organization``.
- Click on the arrow of your ``App name`` and copy your ``API Key``

3.2. Set your Application Credential in the Single Page Application
=====================================================================

- Connect to ``sentence{{ site_ID }}-non-prod.f5dc.dev`` using your web browser
- Click on login button on the top right of the page
- Use user credential: producer{{ site_ID }}@acme.com / F5-NGINX-lab!
- Open Dev Tools in your browser or press ``CTRL + SHIFT + i`` then open the sheet ``network``
- Click on the `+` button under the *colors* of the sentence
- Create a new *color* name
- You will receive a ``403`` HTTP code

.. image:: ./_pictures/api_key_403.png
   :align: center
   :width: 800
   :alt: 403 Forbidden

- On Dev Tools, open the sheet ``Sources``
- In ``script.js`` line 245, replace ``myApiKey`` with your API Key then **SAVE** (``CTRL + s``)
- Open the sheet ``network``
- Test to create a new *color*
- You will receive a ``200`` HTTP code

.. raw:: html

    <a href="http://www.youtube.com/watch?v=OWLFoDRHMS0"><img src="http://img.youtube.com/vi/OWLFoDRHMS0/0.jpg" width="600" height="300" title="App credential" alt="App credential"></a>

_______________________________________________________________________

**Capture The Flag**

    **3. What is the name of the API token header in the request?**

_______________________________________________________________________




