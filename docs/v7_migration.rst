.. _migrating_v6_to_v7:

Migrating your code from v6 to v7
=================================

web3.py follows `Semantic Versioning <http://semver.org>`_, which means
that version 7 introduced backwards-incompatible changes. If your
project depends on *web3.py* ``v7``, then you'll probably need to make some changes.

Breaking Changes:


Class-Based Middleware Model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The middleware model has been changed to a class-based model. This allows for
more flexibility in implementation, for asynchronous processing for example. Previously,
the middleware were functions that tightly wrapped the ``make_request`` function of the
provider, always making the request except if the middleware itself modified the
``make_request`` function. Now, the middleware logic can be separated into
``request_processor`` and ``response_processor`` functions that do pre-request and
post-response processing, respectively. This gives more flexibility for asynchronous
operations where the response may come back at a later time than the request was made.
This also paves the way for support of batch requests, which is a part of the roadmap
for web3.py.

The new middleware model is documented in the :ref:`internals__middlewares` section.


Middleware Renaming and Removals
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following middleware have been renamed for generalization or clarity:

- ``name_to_address_middleware`` -> ``ENSNameToAddressMiddleware``
- ``geth_poa_middleware`` -> ``ExtraDataToPOAMiddleware``

The following middleware have been removed:

ABI Middleware
``````````````

``abi_middleware`` is no longer necessary and was removed. All of the functionality
of the ``abi_middleware`` was already handled by the abi formatters but a bug in the
ENS name-to-address middleware would override the formatters. Fixing this bug has
removed the need for the ``abi_middleware``.

Caching Middleware
``````````````````

The following middleware have been removed:

- ``simple_cache_middleware``
- ``latest_block_based_cache_middleware``
- ``time_based_cache_middleware``

All caching middleware has been removed in favor of a decorator / wrapper around the
``make_request`` methods of providers with configuration options on the provider class.
The configuration options are outlined in the documentation in the
:ref:`request_caching` section.

If desired, the previous caching middleware can be re-created using the new class-based
middleware model overriding the ``wrap_make_request`` (or ``async_wrap_make_request``)
method in the middleware class.

Result Generating Middleware
````````````````````````````
The following middleware have been removed:

- ``fixture_middleware``
- ``result_generator_middleware``

The ``fixture_middleware`` and ``result_generator_middleware`` which were used for
testing / mocking purposes have been removed. These have been replaced internally by the
``RequestMocker`` class, utilized for testing via a ``request_mocker`` pytest fixture.

HTTP Retry Request Middleware
`````````````````````````````

The ``http_retry_request_middleware`` has been removed in favor of a configuration
option on the ``HTTPProvider`` and ``AsyncHTTPProvider`` classes. The configuration
options are outlined in the documentation in the :ref:`http_retry_requests` section.

Normalize Request Parameters Middleware
```````````````````````````````````````

The ``normalize_request_parameters`` middleware was not used anywhere internally and
has been removed.


Provider Updates
~~~~~~~~~~~~~~~~

WebSocketProvider
`````````````````

``WebsocketProviderV2``, introduced in *web3.py* ``v6``, has taken priority over the
legacy ``WebsocketProvider``. The ``LegacyWebSocketProvider`` is also deprecated in
``v7`` and will likely be slated for removal in a later major version of the library
to help with the transition to the asynchronous patterns required when using the
``WebSocketProvider``.

- ``WebsocketProvider`` -> ``LegacyWebSocketProvider`` (and deprecated)
- ``WebsocketProviderV2`` -> ``WebSocketProvider``


EthereumTesterProvider
``````````````````````

``EthereumTesterProvider`` now returns ``input`` instead of ``data`` for ``eth_getTransaction*``
calls, as expected.

AsyncIPCProvider (non-breaking feature)
```````````````````````````````````````

An asynchronous IPC provider, ``AsyncIPCProvider``, is also available in ``v7``. This
provider makes use of some of the same internals that the new ``WebSocketProvider`` does
which allow it to also support ``eth_subscription``.


Python 3.7 Support Dropped
~~~~~~~~~~~~~~~~~~~~~~~~~~

Python 3.7 support has been dropped in favor of Python 3.8+. Python 3.7 is no longer
supported by the Python core team, and we want to focus our efforts on supporting
the latest versions of Python.


Miscellaneous Changes
~~~~~~~~~~~~~~~~~~~~~

- ``LRU`` has been removed from the library and dependency on ``lru-dict`` library was
  dropped.
- ``CallOverride`` type was changed to ``StateOverride`` since more methods than
  ``eth_call`` utilize the state override params.
- ``User-Agent`` header was changed to a more readable format.
- ``BaseContractFunctions`` iterator now returns instances of ``ContractFunction`` rather
  than the function names.
- Beacon API filename change: ``beacon/main.py`` -> ``beacon/beacon.py``.
- The ``geth.miner`` namespace and methods, deprecated in ``v6``, is removed in ``v7``.
- The asynchronous version of ``w3.eth.wait_for_transaction_receipt()`` changes its
  signature to use ``Optional[float]`` instead of ``float`` since it may be ``None``.
- ``get_default_ipc_path()`` and ``get_dev_ipc_path()`` now return the path value
  without checking if the ``geth.ipc`` file exists.
- ``Web3.is_address()`` returns ``True`` for non-checksummed addresses.
