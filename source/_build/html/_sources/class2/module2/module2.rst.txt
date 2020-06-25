Step 2 - DevOps deploy Money Transfer application
#################################################

In this module, we will deploy the Money Tranfer container for Arcadia Bank application and we will publish it.

.. note :: At the end of this module, Arcadia Bank application will look like this.


.. image:: ../pictures/module2/app2.png
   :align: center

.. note :: In this lab, we will automate some tasks in the controller. As you noticed in the previous lab, it is long to create and you can make mistakes. We will deploy a new component with Postman

|

Step 1 - Deploy Arcadia App2 with a CI/CD pipeline like a DevOps
****************************************************************

#. In Gitlab, click on Administrator / Arcadia-App2
   
   #. Click on file ``deploy``
   #. Click ``edit`` and make a modification - like YES !!!!!
   #. Click ``Commit changes``

   .. note :: At this moment, you simulate a commit like a DevOps. This ``commit`` will trigger a ``webhook`` to ``Jenkins``, so that Jenkins execute a ``pipeline``.

#. In Jenkins, click on ``DeployApp2`` pipeline
#. A pipeline is running, click on it
#. You can follow the steps

.. image:: ../pictures/module2/pipeline.png
   :align: center

.. note :: At this stage, App2 (Money Transfer app) is deployed un K8S. But you need to publish it via the controller.

Step 2 - Publish Money Transfer App with NGINX+ and Controller
**************************************************************

#. In the ``Jumphost`` open ``Postman``
#. Open collection ``Deploy Component App2``
   #. Send the first call ``Log in NGINX Controller``
   #. Send the second call ``Create App2 Component``

   .. note :: with one click, you created the component. Fast and no human mistake.

#. Connect to Controller GUI and check the new component in ``web application arcadia``

   .. image:: ../pictures/module2/components.png
   :align: center

   .. note :: You can notice the new ``Money Transfer`` component

#. In ``Chrome`` refresh the page. You can see the new App ``Money Transfer``
#. Transfer some money to your friends in order to populate analytics

.. image:: ../pictures/module2/app2.png
   :align: center


.. toctree::
   :maxdepth: 1
   :glob:

   lab*

