Web Application Firewall
#####################################

ToDo

-	DS-NGINX-NGINXAppProtectVsAzure: an official comparison of WAF features with Azure WAF
-	XLS matrix: documents done with 2 customers for a WAF comparison
-	Gigaom-Report: a performance report done by an independent company. It’s important to determine if the WAF engine is based on ModSecurity (AVI for example), because performance is lower than F5 WAF engine (named “App Protect”) and therefore require more compute (CPU).



Lower rate of False Positive and more protection
One big benefit of F5 WAF is that False Positive are reduced as described here:
-	Violation rating
F5 knows that some signatures are not sufficiently accurate (accuracy score) and so could generates False Positive. So mitigation can’t be raised if the signature is match alone. However if the low accurate signature is matched with other signature, it generates globally a Violation Rating and the mitigation is raised

-	Threat Campaigns
Because attackers understood this mechanism, their goal is to generate an attack under the radar, i.e. that match only the low accurate signature. F5 deployed a honey pot infrastructure and F5 Labs team develop very accurate signatures to block the specified attack. This set of signatures, updated up to several times a day, is named “Threat campaigns”

Best Practices
A WAF policy includes 2 parts, I share with you some Best Practice to onboard them easier:

1.	Positive security policy
›	Objective
ê	reduce the surface attack to publish only expected request by the Application (URI, method, parameter, JSON schema (key and vaue types), file types, header, cookies)
›	Owner
ê	Application knowledge is owned by App Developpers.
›	How to configure
ê	For API based application, App Dev consolidate their knowledge in a specification file in a standard format (OpenAPI 3.x, swagger 2.x). This file is imported in F5 WAF and F5 WAF auto-reconfigure its positive security policy. Because this spec evolves each App release (2-4 weeks), my customer allow DevOps to upload this file directly to F5 WAF.
ê	For non-API based application, the effort to get the knowledge of the App from/with App Dev could be simple or huge, it depends on your organization.

2.	Negative security policy
›	Objective
Enable protection from:
ê	software vulnerabilities & common web exploits: non-legitimate request
ê	fraud & abuse: legitimate request but the intent is bad (DoS, Credential Stuffing, Brute Force, Web Scraping…)
›	Owner
ê	Security knowledge is owned by SecOps.
›	How to configure
›	Start with base F5 WAF template here that includes already OWASP top 10 recommendation
›	Enable WAF policy in transparent mode for an application (FQDN)
›	For False Positive, disable signature per entity (URI path, page, header) here
›	Enable WAF policy in blocking mode for an application (FQDN). Keep WAF policy in transparent mode for URI paths where False Positive are still encountered.
›	Define Server Technologies here to improve performance by enabling only signatures linked to Application frameworks (Apache, IIS, MySQL…)
›	Enable advanced protections, step by step, and monitor False Positive:
ê	Bot Defense here
ê	Data Guard here
ê	Custom signature here
ê	Clickjacking here
ê	…
5.	In parallel, repeat step 3 and 4 until False Positive disappeared on an URI path
6.	In parallel, review modification created by False Positive:
ê	Change your base line policy: for example, integrate a signature disabled because the CVE is already patched on server side and this signature generates False Positive
ê	Modify Application to be compliant with your base line policy



Design overview
=========================================

.. image:: ./_pictures/global_design.png
   :align: center
   :width: 800
   :alt: Design overview

