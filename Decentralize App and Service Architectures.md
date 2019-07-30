# Decentralized App and Service Architectures

An ambition of many developers and organizations in the decentralized identity ecosystem is to create new app and service models by leveraging foundational technical components of the DID stack. Identity is a critical bit for any application or service, which DIDs and DID Methods themselves provide, but when paired with personal datastores (e.g. DIF Identity Hubs) and other components, one can create new classes of apps and service that introduce new capabilities and novel properties.

## DWAs - DID-based PWAs

Progressive Web Apps (PWAs) is a W3C Web standard that is now becoming ubiquitous across devices and platforms. PWAs enable the developer of a web-based application to 'install' that app from the app's domain/subdomain. PWAs have the ability to store data locally, work offline (via Service Workers), and provide the sort of app UI experience one typically associates with a native, platform-specific app.

PWAs are inherently centralized, given the root of their trust and existence is based on a DNS domain, furthermore, PWA developers generally use their own specific infrastructure to store data generated in the app by users. PWAs and their standard installation, offline capabilities, and UI integration with existing platforms is desireable, and if possible, we should build on, not in conflict with, this valuable standard.

There are two primary ways we can extend the PWA model into 'DWAs', or Decentralized Web Apps:

1. Anchor the trust and existence of an app in the app's global DID, not DNS. This will allow the app and users of the app to continue interacting with it, even if the app's DNS domain is ever lost or interdicted in some way.

2. By integrating DIDs into the PWA model, apps will inherently have access to other components of the DID stack, but most notably: Identity Hubs. With a connection to a user's Identity Hub, an app can now store its data with the user themselves, vs spreading data across countless servers that each are subject to different environments of lesser security and privacy.

### Setting up DID/Domain linkage for a DWA

```sequence
participant Resolver as UR
participant User Hub as UH
participant Wallet as UA
participant Browser/PWA as WEB
participant PWA Server as PWA

Note right of PWA: 1. Dev signs domain assertion
Note right of PWA: 2. Dev places claim at /.well-known/did
WEB->PWA: 3. Alice installs the PWA
WEB->WEB: 4. Instance creates a DID
WEB->UA: 5. Instance requests permissions
UA->PWA: 6. Fetch /.well-known/did claim
UA->UR: 7. Resolve DID Doc
UA->UA: 8. Validate claim
UA->UA: 9. Display permission prompt
UA->UH: 10. Sets permission in Hub
UA->WEB: 11. Returns user to PWA experience
```

### DWA writing to a user's Identity Hub

```sequence
participant User Hub as UH
participant Browser/PWA as WEB

WEB->WEB: 1. App instance creates a Hub request
WEB->UH: 2. Sends request to user's Hub
UH->UH: 3. Checks previously created DID and permissions
UH->UH: 4. Stores the data from the instance
```