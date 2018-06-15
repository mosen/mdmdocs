CertificateList
===============

List installed certificates.

Example Responses
-----------------

OS X 10.11.x El Capitan
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: xml

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
		<dict>
			<key>CertificateList</key>
			<array>
				<dict>
					<key>CommonName</key>
					<string>com.apple.systemdefault</string>
					<key>Data</key>
					<data>
						... base64 ...
					</data>
					<key>IsIdentity</key>
					<true/>
				</dict>
				<dict>
					<key>CommonName</key>
					<string>Apple Worldwide Developer Relations Certification Authority</string>
					<key>Data</key>
					<data>
						... base64 ...
					</data>
					<key>IsIdentity</key>
					<false/>
				</dict>
			</array>
			<key>CommandUUID</key>
			<string>00000000-1111-2222-3333-444455556666</string>
			<key>RequestType</key>
			<string>CertificateList</string>
			<key>Status</key>
			<string>Acknowledged</string>
			<key>UDID</key>
			<string>00000000-1111-2222-3333-444455556666</string>
		</dict>
	</plist>


