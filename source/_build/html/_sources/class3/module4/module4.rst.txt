Module 4 - Fine granted access with NGINX Controller APIm module
################################################################

In this lab, we will allow access to the Arcadia API, only for Manager Role. To do so, we will first check the JWT token claims and understand how to forward a claims from a provider into the Bearer SSO.

Step 1 - Understand the JWT token claims
****************************************

In the previous lab, we used 2 tokens:
   .. code :: bash 

         Partner 1
         eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwYXJ0bmVyMSIsImlhdCI6MTU5MzQ1NTk4NSwiZXhwIjoxNjg4MDYzOTg1LCJhdWQiOiJhcGkuYXJjYWRpYS1maW5hbmNlLmlvIiwic3ViIjoiYXBpLmFyY2FkaWEtZmluYW5jZS5pbyIsIkdpdmVuTmFtZSI6IkpvaG5ueSIsIlN1cm5hbWUiOiJSb2NrZXQiLCJFbWFpbCI6Impyb2NrZXRAZXhhbXBsZS5jb20iLCJSb2xlIjoiTWFuYWdlciJ9.JRboDfKWvSLVU3md6OULGifoVxJ-ryx7y-0DKrOlPOM

   .. code :: bash

         Partner 2
         eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwYXJ0bmVyMiIsImlhdCI6MTU5MzQ1NTk4NSwiZXhwIjoxNjg4MDYzOTg1LCJhdWQiOiJhcGkuYXJjYWRpYS1maW5hbmNlLmlvIiwic3ViIjoiYXBpLmFyY2FkaWEtZmluYW5jZS5pbyIsIkdpdmVuTmFtZSI6IkJvYiIsIlN1cm5hbWUiOiJUaGUgU3BvbmdlIiwiRW1haWwiOiJib2JAc3BvbmdlLmNvbSIsIlJvbGUiOiJDb250cmFjdG9yIn0.aqTxd6X4z7EFijJsyiuq8mZAKMLG519Bmjz1ra24L-s

#. Navigate to https://jwt.io/ and paste ``Partner1`` JWT token into the website. And check the Role claim. ``Partner 1`` is a manager.

   .. image:: ../pictures/module4/jwt-partner1.png
      :align: center

#. Do the them with Partner2 JWT token. ``Partner 2`` is a contractor

   .. image:: ../pictures/module4/jwt-partner2.png
      :align: center

Step 2 - Create a new Claim in APM in order to forward this Role claim into the Bearer SSO
******************************************************************************************

.. note :: The providers will inject a Claim into the JWT. This claim is ``Role``. We need to re-inject this claim into the Bearer SSO token so that NGINX GW only access request from user belonging to Manager Role.

#. In the ``BIG-IP`` > ``Federation`` > ``Oauth Authorization Server`` > ``Claim```
#. Click ``Create``
#. Create this ``Claim``

   #. Name ``Claim_Role``
   #. Claim Type ``String``
   #. Claim Name ``Role```
   #. Claim Value ``%{subsession.oauth.scope.last.jwt.Role}```
#. Click ``Save```

Step 3 - Modify the Bearer SSO in order to inject this new Claim
****************************************************************

.. note :: Now, it is time to tell to the Bearer SSO profile to inject this claim in the JWT SSO token

#. In ``Single Sign-on`` > ``Oauth Bearer`` > ``arcadia-api-sso``
#. Add the previous created ``Claim`` into the ``Selected`` list
#. Click ``Update``

.. image:: ../pictures/module4/claim.png
   :align: center

Step 4 - Update the authorization component setting in the controller
*********************************************************************

.. note :: Now, the BIG-IP APM is injecting the Claim in the JWT Bearer SSO token. It is time to tell to the NGINX GW, to only grant access if the Role contains Manager

#. In the controller GUI
#. In ``APIs`` > ``arcadia-api-def``, edit the ``published API`` ``prod-api``

   .. image:: ../pictures/module4/edit-def.png
      :align: center
      :scale: 75%

#. Edit ``Authentication``

   .. image:: ../pictures/module4/edit-auth.png
      :align: center
      :scale: 75%

#. Click on ``Enable Conditional Access``
#. And enter the values below. This will allow access only if the ``Claim`` contains ``Manager``. Select ``401`` as Failure Reposonse as ``403`` is not allowed as reponse type by Adv. Waf (by default)

   .. image:: ../pictures/module4/conditional.png
      :align: center
      :scale: 75%

#. Click ``Submit`` and ``Submit`` 


Step 5 - Make a test
********************

#. In ``Postman``, send a request with ``Partner1`` JWT token. As a reminder, he is ``Manager``. Request passes.

   .. code :: bash 

         Partner 1
         eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwYXJ0bmVyMSIsImlhdCI6MTU5MzQ1NTk4NSwiZXhwIjoxNjg4MDYzOTg1LCJhdWQiOiJhcGkuYXJjYWRpYS1maW5hbmNlLmlvIiwic3ViIjoiYXBpLmFyY2FkaWEtZmluYW5jZS5pbyIsIkdpdmVuTmFtZSI6IkpvaG5ueSIsIlN1cm5hbWUiOiJSb2NrZXQiLCJFbWFpbCI6Impyb2NrZXRAZXhhbXBsZS5jb20iLCJSb2xlIjoiTWFuYWdlciJ9.JRboDfKWvSLVU3md6OULGifoVxJ-ryx7y-0DKrOlPOM

#. Then, send the same request with ``Partner2`` JWT token. As a reminder, he is ``Contractor``. Request fails.

   .. code :: bash

         Partner 2
         eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwYXJ0bmVyMiIsImlhdCI6MTU5MzQ1NTk4NSwiZXhwIjoxNjg4MDYzOTg1LCJhdWQiOiJhcGkuYXJjYWRpYS1maW5hbmNlLmlvIiwic3ViIjoiYXBpLmFyY2FkaWEtZmluYW5jZS5pbyIsIkdpdmVuTmFtZSI6IkJvYiIsIlN1cm5hbWUiOiJUaGUgU3BvbmdlIiwiRW1haWwiOiJib2JAc3BvbmdlLmNvbSIsIlJvbGUiOiJDb250cmFjdG9yIn0.aqTxd6X4z7EFijJsyiuq8mZAKMLG519Bmjz1ra24L-s


.. note :; As you can notice, APM is collecting the different claims and only forward the relevant claims to the internal API GW. Then, API GW grant access based on the claim values.


.. toctree::
   :maxdepth: 1
   :glob:

   lab*
