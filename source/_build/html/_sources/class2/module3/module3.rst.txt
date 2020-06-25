DevOps deploy Refer Friends Application
#######################################

In this module, we will deploy the Refer Friends container for Arcadia Bank application and we will publish it.

.. note :: At the end of this module, Arcadia Bank application will look like this.


.. image:: ../pictures/module3/app3.png
   :align: center

|

.. note :: In this lab, all tasks will be automated. DevOps will deploy the app in K8S, and NetOps will create the new component at the same time.

|

Step 1 - Deploy Arcadia App3 and the new componenent with a CI/CD pipeline
**************************************************************************

#. In Gitlab, click on Administrator / Arcadia-App3
   
   #. Click on file ``deploy``
   #. Click ``edit`` and make a modification - like YES !!!!!
   #. Click ``Commit changes``

   .. note :: At this moment, you simulate a commit like a DevOps. This ``commit`` will trigger a ``webhook`` to ``Jenkins``, so that Jenkins execute a ``pipeline``.

#. In Jenkins, click on ``DeployApp3`` pipeline
#. A pipeline is running, click on it
#. You can follow the steps

   .. image:: ../pictures/module3/pipeline.png
      :align: center

#. Connect to Controller GUI and check the new component in ``web application arcadia``

   .. image:: ../pictures/module3/components.png
   :align: center

#. In ``Chrome`` refresh the page. You can see the new App ``Refer friends`
#. Transfer some money to your friends in order to populate analytics

.. image:: ../pictures/module3/app3.png
   :align: center

.. note :: Congrats, as you can notice, with one ``commit`` in Gitlab, you triggered a webhook that deployed the ``app`` and the ``infra``

.. warning :: Now, it's time to protect Arcadia Finance application with a BIG-IP.

.. toctree::
   :maxdepth: 1
   :glob:

   lab*
