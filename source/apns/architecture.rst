
apsd message store
    /Library/Application Support/ApplePushService/aps.db



apsd service
------------



Initial device activation
-------------------------

Each client device needs its own identity certificate in order to talk to the Push Notification service.

This is a rough outline of that process.

.. note:: This is a breakdown of ``-[APSCertificateProvisioner generateClientIdentity]:``.

- Generate a private key via selector ``createUserKeyPair:privKey:keychain:algorithm:size:userName:accessRef:``.
- Generate a CSR ``csrForPublicKey:privateKey:error:``.
- Create a dict containing the keys **DeviceCertRequest** and **ActivationRandomness**.
- **ActivationRandomness** seems to be derived from the machine serial.
- (Further payload information TBD)
- **POST** to https://albert.apple.com/deviceservices/deviceActivation?device=MacOS with ``application/x-www-form-urlencoded``
    field ``activation-info``, which contains the previous data.
- (?) receive signed identity for APNS client.

