Module 3 - Protect Arcadia API with Adv. Waf and APM (Bearer SSO)
#################################################################

In this lab we will deploy a BIG-IP security policy based on Adv. WAF and APM, in front of the NGINX+ API GW.
In order to make life better and simple for DevOps, we will delegate all the Authentication layer to APM. 
APM will authenticate JWT tokens coming from different providers with different keys, and we will use APM Bearer SSO in order to share a unique JWT key with the API gateways.

.. note :: APM will download keys from external providers automatically (by using OIDC discovery process) and will use another an unique key for internal SSO with NGINX API Gateways. This will allow DevOps to know only one key for all their deployments. And SecOps will manage the external providers.

.. image:: ../pictures/module3/archi.png
   :align: center

|

Configure NGINX Controller with a new Identity Provider
*******************************************************

#. In the left menu, click on ``Identity Provider`` icon
#. Create a new Identity Provider as below. Use the JSON code below for the JWK

   .. code :: json

      {
      "keys": [
            {
            "k": "aWxvdmVuZ2lueA",
            "kid": "9876543210",
            "kty": "oct"
            }
         ]
      }

   .. note :: I invite you to decode the "k" value to know what is the ``key``. As you can notice, we don't use a RSA key, but a secret (just to simplify the lab). This secret is BASE64 encoded.

   .. image:: ../pictures/module3/identity-provider.png
      :align: center

#. Assign this ``Identity Provider`` with your API Definition

   #. Get back to your ``API definition`` and ``edit`` the ``Published API`` 

      .. image:: ../pictures/module3/edit-published.png
         :align: center
         :scale: 50%

   #. Click on ``routing`` and edit the ``Security Settings``

      .. image:: ../pictures/module3/edit-security.png
         :align: center
         :scale: 75%

   #. Click on ``Add Authentication``

      .. image:: ../pictures/module3/add-auth.png
         :align: center
         :scale: 75%

   #. Select the provider created previouly ``JWT Bearer SSO`` and ``Bearer``

      .. image:: ../pictures/module3/auth.png
         :align: center
         :scale: 75%

   #. Click ``Done`` and ``Submit``

   #. Click ``Submit`` again

#. Make a quick test with ``Postman`` by sending a request to the Arcadia API like ``Last Transactions`` or ``Buy stocks``

   #. You can see a ``401 Unauthorized`` 

      .. image:: ../pictures/module3/401.png
         :align: center
         :scale: 75%

.. note :: As you don't include any JWT token in your request, the API GW rejected the request. It is time to configure APM to inject this JWT Bearer SSO



Configure Adv. WAF and APM
**************************

.. note :: In this lab we will use Access Guided Configuration and we will do some custom tuning in the policies. There are several ways to protect API with BIG-IP, but at the moment, we will focus on AGC so that you can understand how it works. GSA team is working on a dedicated UDF Blueprint for API Declarative WAF policy with v16.0

#. Delete the existing ``vs-arcadia-api`` Virtual Server in the BIG_IP. We are going to create a new one from the Guided Configuration.

#. Connect to the Jumhost (user / user)
#. Open ``Chrome`` and connect to the BIG-IP (admin / admin)
#. First of all, create a JWK Bearer SSO key. If you remember below, the key (encoded64) was ``aWxvdmVuZ2lueA``, and decoded64 ``ilovenginx``

      #. Click ``Access`` > ``Federation`` > ``JSON Web Token`` > ``Key Configuration``
      #. Create a new key as below with the value ``ilovenginx`` as Shared Secret

         .. image:: ../pictures/module3/bearer-key.png
            :align: center
            :scale: 75%     

      .. warning :: Don't forget to set an ID. It is mandatory in order to use this key in the Bearer SSO profile

#. In ``Access``, click on ``Guided Configuration`` and select the template ``API Protection Proxy`` in ``API Protection`` group

   .. image:: ../pictures/module3/AGC-1.png
      :align: center
      :scale: 50%


#. Configure the template as below.

   .. warning :: The AGC template does not support yet OpenAPI spec file Version 3. But only Version 2. We will use another version of the OAS file.

   .. note ::  The OAS file is located in ``Downloads`` directory and its name is ``swaggerArcadia2.json``

   #. Check the boxes ``Use Rate Limiting`` and OAuth 2.0``

      .. image:: ../pictures/module3/AGC-2.png
         :align: center
         :scale: 50%

      - Select the default Servrer at the bottom of the screen

      .. image:: ../pictures/module3/AGC-3.png
         :align: center
         :scale: 50%


      .. note :: You can notice the URI and the back server have been imported from the OAS2 file


      .. image:: ../pictures/module3/AGC-4.png
         :align: center
         :scale: 50%

      .. image:: ../pictures/module3/AGC-5.png
         :align: center
         :scale: 75%

   #. Select ``AzureAD`` ``AAD-F5Sales`` as provider

      .. warning :: Due to a bug in AGC, we can't add more providers here. We will modify the list later on directly in the APM configuraiton (ID 835509)

      .. image:: ../pictures/module3/AGC-6.png
         :align: center
         :scale: 50%

   #. Configure ``Signle Sign-On Settings`` as below

      .. image:: ../pictures/module3/AGC-7.png
         :align: center
         :scale: 75%

      .. note :: We will focus on Claims later on


   #. Configure ``Rate Limiting`` as below. We will limit request per user based on their Email address extracted from the JWT token. The value used for the ``User ID Key`` is ``subsession.oauth.scope.last.jwt.Email``

      .. image:: ../pictures/module3/AGC-8.png
         :align: center
         :scale: 75%

|

   #. Configure the ``Virtual Server`` as below

      - VS : 10.1.10.18
      - Log All Requests
      - Client SSL arcadia_client_ssl

         .. image:: ../pictures/module3/AGC-9.png
            :align: center
            :scale: 50%
   
   #. Click ``Deploy``

#. Now we have to add manually the 2 more providers in the APM configuration (due to the BUGID in AGC 6.0)

   #. ``Unstrict`` the configuration in AGC, by clicking on the ``lock`` icon and approve the change.

      .. image:: ../pictures/module3/unstrictness.png
         :align: center
         :scale: 50%

   #. Click ``Access`` > ``Federation`` > ``JSON Web Token`` > ``Provider List`` and ``edit`` the existing profile
   #. Add ``provider1`` and ``provider2`` into the list

      .. image:: ../pictures/module3/provider-list.png
         :align: center
         :scale: 50%      


.. note :: Congratulation, Arcadia API is protected by an Advanced WAF (you can check the policy) and APM in order to authenticate requests from 3 providers.

.. note :: I invite you to check the Access > API Protection configuration

|

Test your protected API with Authentication, WAF and Rate Limiting
******************************************************************

#. Open ``Postman`` and select the ``Arcadia API`` collection
#. Select one call, the one you want.
#. ``F5ers only - For F5 partners and customers, please jump to the next bullet point.`` In ``authentication`` select ``Oauth 2.0``. We will start with an Azure AD provider - similating a partner having an AAD subcription and wanting to use it.

   #. Click ``Get New Token``
   #. I have already set the values for the Oauth Client. As a reminder, here, Postman is the Oauth Agent - it is requesting the Access Token

      .. image:: ../pictures/module3/oauth2.png
         :align: center
         :scale: 50%    
   #. Authenticate with your Corporate F5 account. If it fails, it means you are not part of the F5 Sales Azure tenant (Open an IT Ticket)
   #. When done, click ``Use token`` and send your request.

   .. note :: It passes. Token is approved by APM, and a new token is generated by APM and sent to the NGINX API GW (Bearer SSO)

#. ``Available for F5ers, partners and customers``. Now, try with the 2 other providers (partner1 and partner2)

      #. You can find the tokens on the desktop in the file ``JWT tokens.txt``
      #. Don't use ``Oauth 2.0``, as we already have the tokens. But use ``Bearer Token`` instead. I generated these tokens from the website http://jwtbuilder.jamiekurtz.com/

      .. code :: bash 

         Partner 1:

         eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwYXJ0bmVyMSIsImlhdCI6MTU5MzQ1NTk4NSwiZXhwIjoxNjg4MDYzOTg1LCJhdWQiOiJhcGkuYXJjYWRpYS1maW5hbmNlLmlvIiwic3ViIjoiYXBpLmFyY2FkaWEtZmluYW5jZS5pbyIsIkdpdmVuTmFtZSI6IkpvaG5ueSIsIlN1cm5hbWUiOiJSb2NrZXQiLCJFbWFpbCI6Impyb2NrZXRAZXhhbXBsZS5jb20iLCJSb2xlIjoiTWFuYWdlciJ9.JRboDfKWvSLVU3md6OULGifoVxJ-ryx7y-0DKrOlPOM

      .. code :: bash

         Partner 2:
         
         eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwYXJ0bmVyMiIsImlhdCI6MTU5MzQ1NTk4NSwiZXhwIjoxNjg4MDYzOTg1LCJhdWQiOiJhcGkuYXJjYWRpYS1maW5hbmNlLmlvIiwic3ViIjoiYXBpLmFyY2FkaWEtZmluYW5jZS5pbyIsIkdpdmVuTmFtZSI6IkJvYiIsIlN1cm5hbWUiOiJUaGUgU3BvbmdlIiwiRW1haWwiOiJib2JAc3BvbmdlLmNvbSIsIlJvbGUiOiJDb250cmFjdG9yIn0.aqTxd6X4z7EFijJsyiuq8mZAKMLG519Bmjz1ra24L-s

#. Test the **Rate Limiting** by sending 4 calls with the same token. The 4th will be block. You can notice the reponse code ``429 Too Many Requests``

      .. image:: ../pictures/module3/ratelimit.png
         :align: center
         :scale: 75%   


#. Send an **attack**

   #. Select the call ``POST Buy Stocks XSS attack``
   #. Send the request and notice the ``200 OK`` response. It means the WAF didn't block the request
   #. Check why and change your policy accordingly.

.. note :: Tip : attack signatures are in Staging mode

.. toctree::
   :maxdepth: 1
   :glob:

   lab*
