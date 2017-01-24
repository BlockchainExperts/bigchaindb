The WebSockets Events Stream API
================================

BigchainDB provides real-time event streams over the WebSockets protocol with
the Events Stream API.

Connecting to a Events Stream from your application enables a BigchainDB node
to notify you as events are processed, such as new `validated transactions <#valid-transactions>`_.


Determining Support for the Events Stream API
---------------------------------------------

In practice, it's a good idea to make sure that the node you're connecting with
has support for the Events Stream API. To do so, send a HTTP GET request to the
node's :ref:`Root URL <bigchaindb-root-url>` and check that the response
contains a ``streams_<version>`` property in ``_links``::

    {
      "_links": {
         "streams_v1": "ws://example.com:9984/api/v1/streams/"
      }
    }


Connection Keep Alive
~~~~~~~~~~~~~~~~~~~~~

The Events Stream API requires clients to signal that they'd like their
connection to stay open by sending "pings" across the open connection.
BigchainDB nodes will automatically close any connections that haven't sent a
ping in the last three minutes.

.. note::

    While three minutes is the limit before a BigchainDB server will terminate
    a connection, we suggest sending a ping every 30 seconds for better
    reliability.

A "ping" consists of a message containing only the string ``"ping"``, for example
in JavaScript:

.. code-block:: javascript

    new WebSocket("...").send("ping")

If the BigchainDB node received the ping, it'll respond back with a message
containing only the string ``"pong"``.


Streams
-------

Each stream is meant as a unidirectional communication channel, where the
BigchainDB node is the only party sending messages (except for `keep-alive
pings <#connection-keep-alive>`_). Any messages sent to the BigchainDB node
(except the keep-alive pings) will be ignored.

Streams will always be under the WebSocket protocol (so ``ws://`` or ``wss://``)
and accessible as extensions to the ``/api/v<version>/streams/`` API root URL
(for example, `validated transactions <#valid-transactions>`_ is accessible
under ``/api/v1/streams/valid_tx``). If you're running your own BigchainDB
instance and need help determining its root URL, you can look :ref:`here <determining-the-api-root-url>`.

All messages sent in a stream are in the JSON format.

.. note::

    For simplicity, BigchainDB initially only provides a stream for all
    validated transactions. In the future, we may provide streams for other
    information, such as new blocks, new votes, or invalid transactions. We may
    also provide the ability to filter the stream for specific qualities, such
    as a specific ``output``'s ``public_key``.

Valid Transactions
~~~~~~~~~~~~~~~~~~

``/valid_tx``

Streams the IDs of any newly validated transactions. Message bodies will also
contain the block ID containing the validated transaction.

Example message::

    {
        "txid": "<sha3-256 hash>",
        "blockid": "<sha3-256 hash>"
    }


.. note::

    Transactions in BigchainDB are validated in batches ("blocks") and may,
    therefore, be streamed in batches. Each block can contain up to a 1000
    transactions, which may all be sent at once without any required ordering.


Tips and Tricks
---------------

Demoing the API
~~~~~~~~~~~~~~~

You may be interested in demoing the Events Stream API with the `WebSocket echo test <http://websocket.org/echo.html>`_
to familiarize yourself before attempting an integration.
