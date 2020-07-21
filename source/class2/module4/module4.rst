Protect Arcadia Application
###########################

In this module, we will deploy a WAF policy to protect Arcadia Bank application and we will publish it. With v16.0 (and in draft in v15.1), the WAF policy can be deployed via a declarative call, and the WAF policy itself is a JSON file.

.. note :: We use the new v15.1/v16.0 Declarative WAF policy. You can retrieve the JSON Policy in the GitLab repo and below.
.. note :: You can learn more on the Declarative WAF policy here : https://f5.sharepoint.com/sites/EMEASystemsEngineering/SitePages/Adv.-WAF-v16.0-Declarative-API.aspx

.. code-block:: json

    {
        "policy": {
            "name": "policy-fund-1",
            "description": "Policy Example - Rapid Deployment",
            "template": {
                "name": "POLICY_TEMPLATE_RAPID_DEPLOYMENT"
            },
            "enforcementMode": "blocking",
            "server-technologies": [
                {
                    "serverTechnologyName": "MySQL"
                },
                {
                    "serverTechnologyName": "Unix/Linux"
                },
                {
                    "serverTechnologyName": "MongoDB"
                }
            ],
            "signature-settings": {
                "signatureStaging": false
            },
            "policy-builder": {
                "learnOnlyFromNonBotTraffic": false
            },
            "response-pages": [
                {
                    "responsePageType": "ajax",
                    "ajaxEnabled": true,
                    "ajaxPopupMessage": "My customized popup message! Your support ID is: <%TS.request.ID()%>"
                }
                ]
        }
    }

.. note :: You can notice this JSON policy is based in Rapid Deployment template and we added few things like Server-Technologies, Signature Staging, Policy Buidler and Response Page.


|

Step 1 - Send an attack
***********************

#. In ``Chrome``, in Arcadia web application, refer a friend
    
    #. Refer bob@sponge.com

    .. image:: ../pictures/module4/bobthesponge.png
        :align: center

#. Send an attack with the below payload in the ``refer friend`` field

    .. code :: json

        {\"$ne\":\"michael@gmail.com\"}

    .. note :: This attacks means return everything not equals to ``michael@gmail.com``

#. Attack succeed and you can get the DB content

    .. image:: ../pictures/module4/attack.png
        :align: center


Step 2 - Push AS3 declaration to deploy the WAF policy
******************************************************

.. note :: It is important to understand what we are doing here. We are leveraging all the new v16.0 Adv. WAF Declarative policy features. With one API call (done by Jenkins and Ansible), we will deploy a new AS3 declaration with a WAF policy.

Check the files used here
=========================


#. In ``Gitlab``, open ``Administrator / as3-waf`` project
#. You can see several files, but the most important are 

    #. playbook-v16.yaml
    #. as3-v16.yaml
    #. policy-fund-1.json

#. Open ``policy-fund-1.json``

    .. code :: json

        {
            "policy": {
                "name": "policy-fund-1",
                "description": "Policy Example - Rapid Deployment",
                "template": {
                    "name": "POLICY_TEMPLATE_RAPID_DEPLOYMENT"
                },
                "enforcementMode": "blocking",
                "server-technologies": [
                    {
                        "serverTechnologyName": "MySQL"
                    },
                    {
                        "serverTechnologyName": "Unix/Linux"
                    },
                    {
                        "serverTechnologyName": "MongoDB"
                    }
                ],
                "signature-settings": {
                    "signatureStaging": false
                },
                "policy-builder": {
                    "learnOnlyFromNonBotTraffic": false
                },
                "response-pages": [
                    {
                        "responsePageType": "ajax",
                        "ajaxEnabled": true,
                        "ajaxPopupMessage": "My customized popup message! Your support ID is: <%TS.request.ID()%>"
                    }
                    ]
            }
        }

    .. note :: This is our declarative JSON WAF policy

#. Open ``as3-v16.json``

    .. code :: json

        {
            "class": "AS3",
            "action": "deploy",
            "persist": true,
            "declaration": {
                "class": "ADC",
                "schemaVersion": "3.2.0",
                "id": "Prod_Web_AS3",
                "Web-Prod": {
                    "class": "Tenant",
                    "defaultRouteDomain": 0,
                    "arcadia": {
                        "class": "Application",
                        "template": "generic",
                        "VS_WebApp": {
                            "class": "Service_HTTPS",
                            "remark": "Accepts HTTPS/TLS connections on port 443",
                            "virtualAddresses": ["10.1.10.26"],
                            "redirect80": false,
                            "pool": "pool_NGINX_WebApp",
                            "policyWAF": {
                                "use": "Arcadia_WAF_policy"
                            },
                            "securityLogProfiles": [{
                                "bigip": "/Common/Log all requests"
                            }],
                            "profileTCP": {
                                "egress": "wan",
                                "ingress": { "use": "TCP_Profile" } },
                            "profileHTTP": { "use": "custom_http_profile" },
                            "serverTLS": { "bigip": "/Common/arcadia_client_ssl" }
                        },
                        "Arcadia_WAF_policy": {
                            "class": "WAF_Policy",
                            "url": "http://10.1.20.4/root/as3-waf/-/raw/master/policy-fund-1.json",
                            "ignoreChanges": true
                        },
                        "pool_NGINX_WebApp": {
                            "class": "Pool",
                            "monitors": ["http"],
                            "members": [{
                                "servicePort": 8080,
                                "serverAddresses": ["10.1.20.10"]
                            }]
                        },
                        "custom_http_profile": {
                            "class": "HTTP_Profile",
                            "xForwardedFor": true
                        },
                        "TCP_Profile": {
                            "class": "TCP_Profile",
                            "idleTimeout": 60 }
                    }
                }
            }
        }

    .. note :: In this AS3 declaration, you can notice the new v16.0 Adv. WAF Reference section (``Arcadia_WAF_policy``). This section refers to our external JSON policy file, and will upload, import and apply the policy in the BIG-IP.

#. Open ``playbook-v16.yaml```

    .. code :: yaml

        ---

        - hosts: bigip
        connection: local
        gather_facts: false
        vars:
            my_admin: "admin"
            my_password: "admin"
            bigip: "10.1.1.12"

        tasks:
        - name: Deploy AS3 WebApp
            uri:
            url: "https://{{ bigip }}/mgmt/shared/appsvcs/declare"
            method: POST
            headers:
                "Content-Type": "application/json"
                "Authorization": "Basic YWRtaW46YWRtaW4="
            body: "{{ lookup('file','as3-v16.json') }}"
            body_format: json
            validate_certs: no
            status_code: 200


    .. note :: You can see the playbook is very simple in v16.0 thanks to the AS3 call. It will do all the job for us. This playbook is just sending an AS3 declaration call to the BIGIP.

|

Run the CI/CD pipeline
======================

#. In ``Jenkins``, click on ``DeployWAF`` pipeline
#. Run the ``pipeline``
#. In ``Chrome``, launch an incognito window, and retry the attack

    .. code :: json

        {\"$ne\":\"michael@gmail.com\"}

#. Attack fails and you can notice the ``AJAX blocking page`` set in the JSON declarative WAF policy

    .. image:: ../pictures/module4/ajax.png
        :align: center

#. Check logs in the BIG-IP

    .. image:: ../pictures/module4/logs.png
        :align: center

.. toctree::
   :maxdepth: 1
   :glob:

   lab*
