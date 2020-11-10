Module 5 - Developer Portal
###########################

.. note :: If you remember, we deployed 3 instances. One for the WebApp, one for the APIs and another one for the DevPortal. We will use the latest in this lab.

When we uploaded the OAS3 file, this file included the API documentation as well. There is only one step to publish the documentation into the DevPortal instance.

Step 1 - Create a Dev Portal object
***********************************

#. In ``APIs``, then ``Dev Portals`` create a new Dev Portal object

    .. image:: ../pictures/module5/create.png
      :align: center  

#. Configure the object as below 
    #. Name: ``devportal``
    #. Display Name: ``Dev Portal Arcadia``
    #. Environment: ``Production Environment``
    #. Gateway: ``Gateway Dev Portal``
    #. Published API: ``prod-api``

        .. image:: ../pictures/module5/config.png
          :align: center 

    #. Click ``Next``

#. Give a ``Brand name`` like ``API for Arcadia Application``, and upload any logo if you want
#. Click ``Next`` and ``Submit``

Step 2 - Navigate to the Developer Portal
*****************************************

#. RDP to ``Jumphost`` as ``user``/``user``
#. Open ``Chrome`` and click on bookmark ``Dev Portal APIm``
    #. Click on ``Explore API`` and ``Get Started``

        .. image:: ../pictures/module5/devportal.png
          :align: center       


.. note :: Navigate in the Developer Portal. As you can notice, this has been populated automatically thanks to the OAS file. As a reminder, the OAS file looks like that (this is an extract for the ``buy stock`` API).

.. code :: YAML

  /trading/rest/buy_stocks.php:
    post:
      summary: Add stocks to your portfolio
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/buy'
            example:
              trans_value: '312'
              qty: '16'
              company: MSFT
              action: buy
              stock_price: '198'
      responses:
        '200':
          description: 200 response
          content:
            application/json:
              example:
                status: success
                name: Microsoft
                qty: '16'
                amount: '312'
                transid: '855415223'

.. toctree::
   :maxdepth: 1
   :glob:

   lab*
