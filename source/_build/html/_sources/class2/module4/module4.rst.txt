Protect Arcadia Application
###########################

In this module, we will deploy a WAF policy to protect Arcadia Bank application and we will publish it.

.. note :: We use the new v15.1 Declarative WAF policy. You can retrieve the JSON Policy in the GitLab repo and below.

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
            }
        }
    }

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

#. In ``Jenkins``, click on DeployWAF pipeline
#. Run the ``pipeline``
#. In ``Chrome``, launch an incognito window, and retry the attack
#. Attack fails
#. Check logs in the BIG-IP

    .. image:: ../pictures/module4/logs.png
        :align: center


.. warning :: In the next version of this lab, we will upload the OAS3 file directly into the BIG-IP v16.0, instead of pushing declarative WAF policy. Benn Alp is working on such Blueprint.

.. toctree::
   :maxdepth: 1
   :glob:

   lab*
