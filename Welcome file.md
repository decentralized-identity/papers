
# Browser Needs for DID

The following encompass the initial set of modifications to the browser the DID initiative needs to maximize the value of DID technology for users across platforms and devices.

## Generic DID APIs

### Custom Protocol Handler API

The browser should provide a robust API that allows user-selected apps, origins, or entities to register handlers and hooks for custom protocols (most notably for this work: register as a handler for the `did:`, `hub:`, and `ipfs:` protocols). This includes the following min-bar features:

- Registration of custom protocol handlers (for our needs: `did:`, `hub:`, and `ipfs:`)
- Ability for the user to designate an app, origin, or entity as the default handler, or exclusive handler, for a particular protocol
- Access to all forms of requests, interactions, invocations related to use of protocol
  - Notably: observation and interdiction of `fetch()` requests, `src`/`href` resolutions, etc.

For examples of the type of hooks we seek, the closest thing in the platform currently is the Web Extension `webRequest` interface, which provides many of the interdiction points we would need for robust protocol handling: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/webRequest

Additionally, there is a parallel desire/proposal from the Distributed Web group of browser vendor representatives to add this type of API: https://arewedistributedyet.com/programmable-custom-protocol-handlers/

### DIDs as Distinct Origins

Modify origin scoping and handling components of the browser to recognize different DIDs as distinct origins, and apply the same separation and protections afforded to top-level DNS origins: https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy

A similar need has been expressed by the Distributed Web group of browser vendor representatives: https://arewedistributedyet.com/control-origin-security-context/

### CSP Additions

Update the Web's [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) declarations and APIs to include recognition of DID origins, and add a set of additional features:

- Modify the [`connect-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/connect-src) directive to allow passage of a single DID, which all request and execution interfaces are limited to.

### DID Permission Requests

DIDs and DID-based apps need a way to request permissions from the DID owner they are interacting with. We need an interface that allows the calling DID to request permissioned access to the target DID's personal datastore. The following is a strawman example to provide a rough idea of what is needed:

#### `navigator.did.requestPermissions(PROPS)`

| Property | Value | Description |
| ------ | ----------- | ----------- |
| `PROPS.did`   | String | OPTIONAL - A specific DID string to request permissions against. (by default it allows the user to select whatever DID they wish to grant permissions against) |
| `PROPS.keyFunctions`   | Object | Functions required for signing and communicating the permission request to the DID owner's Hub. |
| `PROPS.permissions` | Array | An array of DIF Identity Hub Permissions descriptor objects. |

The request should return a Promise, with the following `then`/`catch` profile:

#### *`Promise.then(PERMISSION_GRANT)`*

A `PermissionGrant` object provides details about the permission grant, including:

| Property | Value | Description |
| ------ | ----------- | ----------- |
| `GRANT.did`   | string | The DID a grant was permissioned for. |
| `GRANT.permissions` | Array | An array of the permissions that were granted - which may not have been all the permissions requested. |

#### *`Promise.catch(ERROR)`*

A `PermissionGrant` object provides details about the permission grant, including:

| Property | Value | Description |
| ------ | ----------- | ----------- |
| `ERROR.type` | string | Either a generic error, or a string signifying permission denial. |
| `ERROR.reason` | string | The reason for permission denial. |
| `ERROR.blocked` | boolean | An indication that the user will not accept further permission requests via ad hoc attempts by the requesting DID. |

### Service Worker Modifications

[Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) should, in accordance with the Origin modifications described above, be augmented to allow a DID to register a Service Worker that communicates with the DID owner's User Agent to handle and cache DID-related requests/data as it would regular HTTP/DNS-based requests.

### DID Data Disclosure Requests

DIDs and origins should be able to request identity data/information from the user, and need a way to invoke such a request. This could be facilitated via the following strawman interface:

#### `navigator.did.requestData({@type: ProofSet})`

| Property | Value | Description |
| ------ | ----------- | ----------- |
| `ProofSet`   | OBject | DIF-specified proof object that describes a set of data, information, or other proofs that the requesting entity would like the user to provide in response. |

The request should return a Promise, with a successful resolution regardless of the outcome of acceptance by the DID owner of any one (or more) permissions. Upon successful resolution the Promise value should be an object that contains two properties:

| Property | Value | Description |
| ------ | ----------- | ----------- |
| `RESOLVED.granted`   | Array | An array of the permissions that were granted from the requested set. |
| `RESOLVED.denied` | Array | An array of the permissions that were denied from the requested set. |

## Support for Personal Apps

Unlike apps of today, the combination of the DID layer's technical components make a new 'Personal App' paradigm possible: developers can create apps that store their data in a user's personal datastore, instead of a centralized application server. Additionally, for apps with compatible requirements (e.g. to-do list), a developer could write a truly serverless app where they only write clientside code and interact with the user's personal datastore, without any backend code at all. This latter variant is something of a Holy Grail for developers, which many would welcome.

DID-based apps are based on the following differences from traditional Progressive Web App model:

1. Instead of a DNS origin being the source of an app, an app developer creates a DID that represents their app, which is treated as a distinct origin.
2. Unlike a PWA install with client/local-centric permissions, the user's agent application (which could be the browser or an external app) signs a permission that allows the DID-based app to access various interfaces of the user's personal datastore.

### App Manifest Additions

Currently the [App Manifest](https://developer.mozilla.org/en-US/docs/Web/Manifest) expects a DNS origin as the source of the application. We need this modded to recognize a DID as the application source, and ensure that the code passed to the client representing that DID-based app is signed by the DID it claims to be from.

### App Execution Requirements

DID-based apps are essentially a bundle of code signed by a DID the user has permitted to access certain features of a DID they own. To ensure that the code is safe and executed properly, the browser should perform the following actions:

1. Ensure that the code bundle is signed by the DID it claims to be from
2. Provide a means for the user's DID User Agent to handle various protocol-specific requests (via its protocol handler hooks) that are raised from the app's running code. For example: ability to view responses for local caching of data.







<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MjY5MTY1MzBdfQ==
-->