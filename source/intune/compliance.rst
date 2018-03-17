InTune Compliance Policy API
============================


Authentication
--------------

Authentication is performed via OAuth2 in the same manner as any other Azure AD Application wishing to access the
Microsoft Graph API.

The Microsoft Tenant ID may be extracted from the **tid** claim in the JWT token.


Web Service Discovery
---------------------

The URL endpoints should be dynamically discovered in order to find the correct domain corresponding to the country
that the tenant resides in.

To get endpoints for a service:

Determine the correct Graph API endpoint for the region:

Default
    https://graph.windows.net/

PPE
    https://graph.ppe.windows.net/

Germany
    https://graph.cloudapi.de/

China
    https://graph.chinacloudapi.cn/


The ``tenantName`` is the tenant domain. You may use the registered <tenant>.onmicrosoft.com domain or any other domain
registered with Azure AD.

- `Microsoft Graph API: servicePrincipalsByAppId <https://msdn.microsoft.com/en-us/library/azure/ad/graph/api/functions-and-actions#servicePrincipalsByAppId>`_.
- `Microsoft Graph API: serviceEndpoint <https://msdn.microsoft.com/en-us/library/azure/ad/graph/api/entity-and-complex-type-reference#serviceendpoint-entity>`_.


.. http::get:: https://graph.windows.net/(str:tenantName)/servicePrincipalsByAppId/0000000a-0000-0000-c000-000000000000/serviceEndpoints?api-version=1.6

    Get service endpoints for the JAMF InTune Compliance Integration

    :reqheader Authorization: OAuth token usually ``Bearer <JWT>``


Sovereign Domains
-----------------

You must make requests to the sovereign domain in which the tenant resides.

The sovereign domains are as follows:

Global
    https://login.microsoftonline.com/

China
    https://login.chinacloudapi.cn/

Germany
    https://login.microsoftonline.de/

Self Hosted
    https://login.microsoftonline.com/

US Government
    https://login.microsoftonline.us/

PPE
    https://login.windows-ppe.net/

CTIP
    https://login.microsoftonline.com/


Web Service Architecture
------------------------

There are 3 web services provided to JAMF by the Graph API:

- Heartbeat ``/PartnerTenantHeartbeat?api-version=1.3``.
- Provisioning ``/PartnerTenants(guid)?api-version=1.4``.
- Upload ``/DataUploadMessages?api-version=1.1``. Used for device compliance data upload


There are two types of inventory payload sent to InTune:

- A compliance inventory payload, which contains only items that are normally used to establish compliance such as:
    - Gatekeeper status
    - SIP status
    - FileVault status etc.

- A non-compliance inventory payload, which contains items that are extraneous to InTune compliance such as:
    - Number of CPUs
    - Operating System Version
    - Serial Number etc.


Heartbeat Endpoint
------------------

.. http:put:: https://graph.windows.net/<tenant domain>/PartnerTenantHeartbeat(guid'<aad tenant uuid>')?api-version=1.3

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

.. http:put:: https://graph.windows.net/<tenant domain>/PartnerTenants(guid'<aad tenant uuid>')?api-version=1.4

    Enable or disable provisioning. This basically enables or disables the integration for the current JSS instance.

    Note that in JAMF the PartnerEnrollmentUrl is calculated like this::

        https://jamf.hostname:port/DeviceRegistration.html

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

Common keys
^^^^^^^^^^^

These keys are common to each payload

.. json:object:: Common Payload Keys

    :property integer EntityType:
    :property string TenantId: The Azure AD Tenant ID
    :property string DeviceId: The Azure AD Device ID
    :property string UserId: The Azure AD User ID
    :property string LastUpdateTime: Last update time formatted as "yyyy-MM-dd'T'HH:mm:ss"


Compliance Inventory Payload
----------------------------

The compliance inventory payload JSON representation is as follows:

.. json:object:: Compliance Inventory Payload

    Properties that affect InTunes compliance access policy

    :property boolean Encrypted: Whether the device has full disk encryption enabled
    :property string Gatekeeper: GateKeeper status
    :property string SIP: SIP status
    :property boolean PreventAutoLogin:
    :property string PasswordExpirationDays:
    :property string PasswordType:
    :property string MinimumPasswordCharSets:
    :property string PasscodeLength:
    :property string PasswordHistoryLength:


Non-Compliance Inventory Payload
--------------------------------

This payload represents attributes that seem to be used for inventory on the AAD device side only.
This may have nothing to do with enforcing compliance policies.

.. json:object:: Non-Compliance Inventory Payload

   :property string CPUArchitecture:
   :property integer RAMOpenSlots:
   :property string BatteryCapacity:
   :property string BootROM:
   :property string BusSpeed:
   :property string CacheSize:
   :property string DeviceName:
   :property string ADDomainJoined:
   :property integer JamfId:
   :property string State:
   :property string MACAddress:
   :property string Vendor:
   :property string Model:
   :property string ModelId:
   :property string NICSpeed:
   :property integer CPUCoreCount:
   :property integer CPUCount:
   :property string Os:
   :property string OsVersion:
   :property string Platform:
   :property string CPUSpeed:
   :property string CPUType:
   :property string MACAddress2:
   :property string SerialNumber:
   :property string SMCVersion:
   :property string RAM:
   :property string UDID:
   :property string UPN: User Principal which is normally the user e-mail address


Inventory Request
-----------------

