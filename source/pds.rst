Profile Domain Services
=======================

Each XPC Service (pds) contains an Info.plist with these additional keys which explain capabilities or requirements
of the pds.

:file:`/System/Library/PrivateFrameworks/ConfigurationProfiles.framework/XPCServices`

- User
- Device
- Destinations
- PreLogin
- Setup
- LockedKeychain
- DarkWake
- RespondNotNow
- PersistResponse
- RequireAccessRights
- **MDMCommands**: List of implemented MDM Commands
- **DomainsSupported**: List of implemented profile payload domains.
- **ExcludeDomainsFromManagedPrefs**: Keys in this domain will not be written to Managed Preferences
