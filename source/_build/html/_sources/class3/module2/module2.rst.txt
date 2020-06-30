Module 2 - Publish API with OAS3 spec file via API
##################################################

.. note :: We will clean up the previous lab in order to configure exactly the same objetcts but with API calls only.

Clean up the APIm configuration
*******************************

#. In the controller GUI, go to ``APIs`` left menu and click on the existing API definintion ``arcadia-api-def`` 
#. On the right panel

   #. Delete the ``Production API`` by clicking the ``trash`` button

      .. image:: ../pictures/module2/delete-published.png
         :align: center
         :scale: 50%

   #. Delete the API definition ``arcadia-api-def``

      .. image:: ../pictures/module2/delete-definition.png
         :align: center

.. note :: Your API Definitions environment is clean.


Create and publish an API Definition with the Controller API
************************************************************

.. note :: We will execute exactly the same job but by using the NGINX Controller control plane only. No GUI.

#. Connect to the Jumphost (user / user)
#. Launch ``Postman``

   #. Open ``Arcadia OAS`` collection
   #. And ``run`` all calls from top to bottom

   .. note :: For every call, check what is happening in the controller GUI

#. At then end, you should have the same results as the previous lab.
#. Edit the ``Published API``

      .. image:: ../pictures/module2/edit-published.png
         :align: center
         :scale: 50%

#. Check the ``Routing``. You can see the routes are imported from the OAS3 file and the mapping is done with the components.

      .. image:: ../pictures/module2/routing.png
         :align: center

#. Make a quick test with the ``Arcadia API`` postman collection

.. warning :: Check the call ``Import API Definition OAS3``, we imported an OAS3 YAML File directly in the Controller with all the definitions and documentations

.. note :: In a near future, we will learn more on API definition versions

.. warning :: As you can notice, there is no security applied on the ``Component``. Let's move to the next lab to assign a BIG-IP and Controller security policy

.. toctree::
   :maxdepth: 1
   :glob:

   lab*

