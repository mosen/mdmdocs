InTune Compliance Policy API
============================

This document outlines the implementation of the JAMF Pro Intune Integration.

Most of the integration revolves around sending device attributes to the Intune or Azure AD Graph API in order to make
a compliance decision. There are some API's that exist outside of Graph which seem to suggest there is a "Partner Tenant"
on JAMFs side providing some non-Graph API's.

Authentication
--------------

Authentication is performed via OAuth2 in the same manner as any other Azure AD Application wishing to access the
Microsoft Graph API.

The client application must be registered in Azure AD with the InTune permission: "Send device attributes to InTune"
selected, as per the `Integrating with Intune Whitepaper <http://docs.jamf.com/technical-papers/jamf-pro/microsoft-intune/10.1.0/Configure_the_Connection_Between_Jamf_Pro_and_Microsoft_Intune.html>`_.

The Microsoft Tenant ID may be extracted from the **tid** claim in the JWT token.


Web Service Discovery
---------------------

The JSS uses a discovery service to figure out the endpoints to use for its services. The endpoints vary by region eg.
global, china or germany and by their production or beta status *(needs verification)*.

To get endpoints for a service:

Determine the correct Graph API endpoint for the region. There are several regions but GLOBAL is the only selectable
region in the JAMF Pro UI. Details for GLOBAL are as follows:

:Authority Base URL: https://login.microsoftonline.com/
:Native Client ID: ``55d611b0-7d33-4d38-8df3-6041868930d7``
:Resource URL: https://api.manage.microsoft.com/
:Resource Name: https://graph.windows.net/


.. http::get:: https://graph.windows.net/(str:tenantName)/servicePrincipalsByAppId/0000000a-0000-0000-c000-000000000000/serviceEndpoints?api-version=1.6

    Get service endpoints for the JAMF InTune Compliance Integration.

    The ``tenantName`` is the tenant domain. You may use the registered **<tenant>.onmicrosoft.com** domain or any other domain
    registered with Azure AD.

    **Example request**:

    .. sourcecode:: http

        GET /(str:tenantName)/servicePrincipalsByAppId/0000000a-0000-0000-c000-000000000000/serviceEndpoints?api-version=1.6 HTTP/1.1
        Accept: application/json
        Authorization: Bearer (JWT TOKEN)

    :reqheader Authorization: JWT Bearer Token

    **Example response**:

    .. sourcecode:: http

        HTTP/1.1 200 OK
        Content-Type: application/json

        {
            "value": [
                {
                    "serviceName": "",
                    "uri": ""
                }
                ... more endpoints ...
            ]
        }


There wasn't much documentation available on these endpoints, but these seemed to be similar:

- `Microsoft Graph API: servicePrincipalsByAppId <https://msdn.microsoft.com/en-us/library/azure/ad/graph/api/functions-and-actions#servicePrincipalsByAppId>`_.
- `Microsoft Graph API: serviceEndpoint <https://msdn.microsoft.com/en-us/library/azure/ad/graph/api/entity-and-complex-type-reference#serviceendpoint-entity>`_.


Web Service Architecture
------------------------

There are 3 web services provided to JAMF by the Graph API:

- **Heartbeat**: Used to establish whether the JSS is still sending device information to Intune.
- **Provisioning**: Used to enable or disable the integration. If the integration is enabled, devices will be redirected
    to JAMF Pro to enroll with MDM when they perform a Workplace Join *(verify)*.
- **Upload**: This service uploads the inventory information from JAMF Pro to Intune for both compliance and inventory
    sharing of non-compliance related attributes.


Heartbeat Endpoint
------------------

.. http:put:: https://graph.windows.net/(str:tenantName)/PartnerTenantHeartbeat(guid'<aad tenant uuid>')?api-version=1.3

    Heartbeat service

    **Example request**:

    .. sourcecode:: http

        PUT /heartbeat HTTP/1.1
        Accept: application/json

        { "Timestamp": "datetimestring" }

    :reqheader Authorization: JWT Bearer Token
    :reqheader Prefer: return-content
    :reqheader Content-Type: application/json

    **Example response**:

    .. sourcecode:: http

        HTTP/1.1 200 OK
        Content-Type: application/json

        {
            "odata.metadata": "",
            "odata.id": "",
            "Key": "",
            "TenantId": "",
            "ClientType": "",
            "Timestamp": "",
            "ResyncTimestamp": "",
            "HttpStatusCode": "",
            "ErrorDetail": "",
            "MessageErrors": [
                ... errors
            ]
        }


Provisioning Endpoint
---------------------

.. http:put:: https://graph.windows.net/(str:tenantName)/PartnerTenants(guid'<aad tenant uuid>')?api-version=1.4

    Enable or disable provisioning. This basically enables or disables the integration for the current JSS instance.

    Note that in JAMF the PartnerEnrollmentUrl is calculated like this::

        https://jamf.hostname:port/DeviceRegistration.html

    This page is also known as the *Workplace Join* page, it seems to redirect the client on to the MDM for the MDM
    enrolment process after a Workplace Join using Company Portal?

    **Example request**:

    .. sourcecode:: http

        PUT /<tenant domain>/PartnerTenants(guid'<aad tenant uuid>')?api-version=1.4
        Accept: application/json

        {
            "Provisioned": 1,
            "PartnerEnrollmentUrl": "<partner enrollment url>"
        }

    :reqheader Authorization: JWT Bearer Token
    :reqheader Prefer: return-content
    :reqheader Content-Type: application/json

    **Example response**:

    .. sourcecode:: http

        HTTP/1.1 200 OK
        Content-Type: application/json

        {
            "Provisioned": 1,
            "HttpStatusCode": 200,
            "ErrorDetail": "... error ..."
        }


Inventory Endpoint
------------------

The inventory endpoint keeps the JSS in sync with the Azure AD Directory.

InTune then makes decisions about compliance based upon the inventory information supplied.

See `Information shared from JAMF Pro to Intune <https://docs.microsoft.com/en-us/intune/conditional-access-integrate-jamf#information-shared-from-jamf-pro-to-intune>`_
for more information about attributes that are synced to Intune.

Additionally the `Whitepaper: Integrating with Microsoft Intune <http://docs.jamf.com/technical-papers/jamf-pro/microsoft-intune/10.1.0/Appendix__Inventory_Information_Shared_with_Microsoft_Intune.html>`_ has
an appendix of example information.

There are two types of inventory payload sent to InTune:

- A compliance inventory payload, which contains only items that are normally used to establish compliance such as:
    - Gatekeeper status
    - SIP status
    - FileVault status etc.

- A non-compliance inventory payload, which contains items that are extraneous to InTune compliance such as:
    - Number of CPUs
    - Operating System Version
    - Serial Number etc.


Common keys
^^^^^^^^^^^

These keys are common to each payload

.. json:object:: Common Payload Keys

    :property integer EntityType:
    :property string TenantId: The Azure AD Tenant ID
    :property string DeviceId: The Azure AD Device ID
    :property string UserId: The Azure AD User ID
    :property string LastUpdateTime: Last update time formatted as eg. "2017-06-07T13:32:42Z"


Compliance Inventory Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The compliance inventory payload JSON representation is as follows:

.. json:object:: Compliance Inventory Payload

    Properties that affect InTunes compliance access policy

    :property boolean Encrypted: Whether the device has full disk encryption enabled. Encrypted (FileVault 2)
    :property string Gatekeeper: GateKeeper status
    :property string SIP: SIP status
    :property boolean PreventAutoLogin: Prevent Auto Login
    :property string PasswordExpirationDays: Password expiration (days)
    :property string PasswordType: Password Type - simple, alphanumeric, or unknown
    :property string MinimumPasswordCharSets: Password: minimum number of character sets
    :property string PasscodeLength: Required Passcode Length
    :property string PasswordHistoryLength: Password: number of previous passwords to prevent reuse


Non-Compliance Inventory Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This payload represents attributes that seem to be used for inventory on the AAD device side only.
This may have nothing to do with enforcing compliance policies.

.. json:object:: Non-Compliance Inventory Payload

   :property string CPUArchitecture: Architecture Type
   :property integer RAMOpenSlots: Available RAM Slots
   :property string BatteryCapacity: Battery Capacity
   :property string BootROM: Boot ROM
   :property string BusSpeed: Bus Speed
   :property string CacheSize: Cache Size
   :property string DeviceName: Device Name
   :property string ADDomainJoined: Domain Join eg. "Likewise: ad.jamf.com" "ad.jamf.com" or "FALSE"
   :property integer JamfId: General category: Jamf Computer ID
   :property string State: JAMF Inventory State (inventory state of a computer checked in with Jamf Pro within the last 24 hours) 0 = activated, 1 = deactivated, 2 = unresponsive (+24 hours)
   :property string MACAddress: MAC address
   :property string Vendor: Vendor "Apple"
   :property string Model: Model
   :property string ModelId: Model Identifier
   :property string NICSpeed: NIC Speed
   :property integer CPUCoreCount: Number of Cores
   :property integer CPUCount: Number of Processors
   :property string Os: OS
   :property string OsVersion: OS Version
   :property string Platform: Platform
   :property string CPUSpeed: Processor Speed
   :property string CPUType: Processor Type
   :property string MACAddress2: Secondary MAC Address
   :property string SerialNumber: Serial Number
   :property string SMCVersion: SMC Version
   :property string RAM: Total RAM
   :property string UDID: UDID
   :property string UPN: User Principal which is normally the user e-mail address


Inventory Request
-----------------

.. http:put:: https://graph.windows.net/(str:tenantName)/DataUploadMessages(guid'<aad tenant uuid>')?api-version=1.1

    Perform an inventory update

    **Example request**:

    .. sourcecode:: http

        PUT (str:tenantName)/DataUploadMessages(guid'<aad tenant uuid>')?api-version=1.1
        Accept: application/json; charset=utf8
        Prefer: return-content
        Authorization: Bearer (JWT Token)

        {
            "TenantId": "<Tenant UUID>",
            "UploadTime": "2017-06-07T13:32:42Z",
            "Content": [
                ... Non-Compliance payloads ...
                ... Compliance payloads, if all the compliance attributes are available, otherwise omit computer ...
            ]
        }

    :reqheader Authorization: JWT Bearer Token
    :reqheader Prefer: return-content
    :reqheader Content-Type: application/json; charset=utf8

    **Example response**:

    .. sourcecode:: http

        HTTP/1.1 200 OK
        Content-Type: application/json

        {
            "TenantId": "<AAD Tenant UUID>",
            "ErrorCode": 0,
            "ErrorDetail": "... error ...",
            "MessageId": "<message id>"
        }