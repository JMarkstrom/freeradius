.. Updated by Jonas Markström on May 18, 2021

===========================================
FreeRADIUS Agent for SafeNet Trusted Access
===========================================

.. toctree::
   :hidden:
   :maxdepth: 3


   index





Overview
^^^^^^^^

The **SafeNet agent for FreeRADIUS** extends the open-source FreeRADIUS project to offer a highly secure container-based service that enables legacy RADIUS clients such as VPN gateways and routers to perform authentication and authorization against SafeNet Authentication Service (SAS-PCE) or SafeNet Trusted Access (STA).

While abstracted from the customer through containerization it may be interesting to understand that the SafeNet agent for FreeRADIUS installs as a module to the open-source based FreeRADIUS server. In the SafeNet agent bundle, the FreeRADIUS server configuration has been configured on behalf of the customer to call the :abbr:`sasagent (SafeNet Authentication Service Agent)` module for supported RADIUS protocol requests.



