Over-the-air Enrollment
=======================

Non-DEP enrollment is also known as OTA Enrollment. You may also install a ``com.apple.mdm`` profile directly to enroll
with an MDM, however this is largely not a supported/documented method.

OTA Enrollment starts with a configuration profile containing a single payload of the type ``Profile Service``.

It may contain the following keys:

- ``URL``, a URL to the endpoint for next phase(s) of enrollment.
- ``ConfirmInstallation`` (Undocumented)
- ``Challenge``, a Data (base64) or String based challenge
- ``DeviceAttributes`` which can be made up of:
	- ``UDID``: Universal Device Identifier
	- ``IMEI``: IMEI (if a cellular device)
	- ``ICCID``:
	- ``VERSION``: The build version of the operating system
	- ``PRODUCT``:
	- ``BuildVersion``
	- ``Model``
	- ``DeviceName``
	- ``SerialNumber``


TODO
----

These may be valid device attributes:

- ``PRODUCT``
- ``DEVICE_NAME``
- ``SERIAL``
- ``MODEL``
- ``MAC_ADDRESS_EN0``
- ``IMEI``
- ``MEID``
- ``ICCID``
- ``COMPROMISED``
- ``DeviceID``
- ``SPIROM``
- ``MLB``

- ``UserID`` ?
- ``UserShortName`` ?
- ``UserLongName`` ?

- OTA Phases can throw 301 redirects which are handled using LSOpenFromURLSpec?

- OTA Enrollment respects header field ``X-Enrollment-Failure`` to signify failure?
	- The string value of this is logged, check whether UI shows anything?


Links
-----

- `Official Documentation <https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/iPhoneOTAConfiguration/OTASecurity/OTASecurity.html>`_.
