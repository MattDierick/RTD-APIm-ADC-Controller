Module 2 - Publish API with OAS3 spec file via API
##################################################

.. note :: We will clean up the previous lab in order to configure exactly the same outcomes but with API calls only.

Clean up the APIm configuration
*******************************

#. In the controller GUI, go to ``APIm`` menu and click on the existing API definintion ``arcadia-api-dev`` 
#. On the right panel

   #. Delete the ``Production API`` but clicking the ``trash`` button

      .. image:: ../pictures/module2/delete-published.png
         :align: center
         :scale: 50%

   #. Delete the API definition ``arcadia-api-def``

      .. image:: ../pictures/module2/delete-definition.png
         :align: center

.. note :: Your API Definitions environment is clean.


Create and publish an API Definition with the Controller API
************************************************************

#. Connect to the Jumphost (user / user)
#. Launch ``Postman``

   #. Open ``Arcadia OAS`` collection
   #. And ``run`` all calls

   .. note :: For every call, check what is happening in the controller GUI

#. At then end, you should have the same results as the previous lab.

.. warning :: Check the call ``Import API Definition OAS3``, we imported an OAS3 YAML File directly in the Controller with all the definitions and documentations

.. note :: In a near future, we will learn more on API definition versions




.. toctree::
   :maxdepth: 1
   :glob:

   lab*

