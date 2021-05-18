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

.. thumbnail:: /images/freeradius/freeRADIUSArchitecture.png
  :width: 100%
  :title: Figure: High level deployment architecture.
  :show_caption: true
|
FreeRADIUS takes in a standardized RADIUS request over UDP on port 1812 (configurable) and if the client (e.g. the VPN gateway) is authorized based on the configured RADIUS clients list, the sasagent module will forward end-user authentication and authorization to either SafeNet Authentication Service (SAS) or SafeNet Trusted Access (STA).

This 'forwarding' as well as the return decision (accept/reject) is done using SafeNet proprietary encryption over TLS (TCP port 443) facilitated through the use of a key file (the :file:`Agent.bsidkey`) as well as explicit authentication node ("auth node") authorization in the SAS or STA virtual server.
