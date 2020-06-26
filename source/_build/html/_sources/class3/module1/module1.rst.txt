Module 1 - Publish API with OAS3 spec file from the Controller GUI
##################################################################

.. note :: In this section we will push OAS3 specification file into the controller GUI in order to create the API 

**Connect to Controller GUI** via your laptop's browser or the jumphost

   login: admin@nginx-udf.internal
   
   password: admin123!


Step 1 - Create an API Application
**********************************

#. Click on top left right corner icon, and click on ``Apps``
#. Click ``Create``
#. Create a new Application

   #. name : ``app_api``
   #. display name : ``API Application Arcadia``
   #. Environment : ``Production Environment``
#. Click ``Summit``

.. image:: ../pictures/module1/apps.png
   :align: center

Step 2 - Create an API Definition
**********************************

#. Click on the left menu ``APIs```
#. Click ``Create API Definition``

   #. Name : ``arcadia-api-def``
   #. Display Name : ``Arcadia APi Definition``
   #. Version : ``v1``
   #. Select ``OpenAPI specification`` 
   
      #. and ``Copy and paste specification text`` **if your not connected in the jumphost**
      #. or ``Import file`` **if your are connected in the jumphost**, the file is located in Downloads and its name is ``arcadia-swagger3.yaml``

      .. image:: ../pictures/module1/oas_paste.png
         :align: center

   #. Click ``Next```
   #. You can see all the resources have been imported from the ``swagger file`` and you can open one resource to check its parameters

      .. image:: ../pictures/module1/resource_list.png
         :align: center
         :scale: 70%

      .. image:: ../pictures/module1/resource_sell_stocks.png
         :align: center
         :scale: 70%

   #. Click ``Summit``


Step 3 - Publish the API
************************

.. note :: At this stage, the API definition is created. So the controller knows the differents URI but doesn't know yet where to forward the traffic to.

#. Click on the API definition raw, and on the right frame, click on ``+ Add Published API``
#. Configure the mandatory settings 
   
   #. Name: prod-api
   #. Display Name: Production API

      .. image:: ../pictures/module1/published-api-1.png
         :align: center

#. Click ``Next``
#. Configure the ``deployment``

   #. Environment: ``Production Environment``
   #. App: ``API Application Arcadia``
   #. Gateways: ``Gateway API``

      .. image:: ../pictures/module1/published-api-2.png
         :align: center

#. It is time to configure the ``Routing``. It is similar to the ``components`` in the WebApp configuration
#. Create a new component routing to the ``MainApp``

   #. Click ``Add New`` in the ``Components section`` and configure it as below

      .. image:: ../pictures/module1/component_1.png
         :align: center
         :scale: 70%

      .. image:: ../pictures/module1/component_2.png
         :align: center
         :scale: 70%

      .. warning :: Don't forget to click on ``done`` 2 times. One ``done`` for the URI and one ``done`` for the workload
   
   #. You shouls see the workload

      .. note :: We only configure one workload as the API we will test is hosted in the main app pod (sell stocks and buy stocks)

   #. Click ``Next`` til the end and click ``Submit``

#. Now, drag and drop the 3 URI starting by ``/trading`` to the right ``Component MainApp``

      .. image:: ../pictures/module1/routing.png
         :align: center
         :scale: 70%

#. Click ``Next`` and ``Submit``

   .. image:: ../pictures/module1/api-definition.png
      :align: center
      :scale: 50%


Step 4 - Test your API
**********************

**RDP to the jumphost**

   login: user
   
   password: user

#. Open ``Postman``
#. Open up the collection ``Arcadia API``
#. Make 2 calls

   #. Last transactions
   #. POST Buy Stocks

#. Both works and are routed to the ``MainApp pod`` in K8S thanks to the NIGNX+ API GW.
#. You can check in the Web Application in ``Chrome`` if your Buy Stock call passed. It should appears in the last transaction GUI.

.. image:: ../pictures/module1/webapp.png
   :align: center
   :scale: 50%

.. toctree::
   :maxdepth: 1
   :glob:

   lab*
