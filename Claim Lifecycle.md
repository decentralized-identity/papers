# Claim Lifecycle

To begin with an end in mind, the following examples detail various scenarios our technical components, standards, and implementations should enable for users, issuers, and relying parties.

### Basic Claim Use

1. Alice is traveling in a foreign country and doesn't have cell service turned on to save money. She walks up to a high tech beer and wine vending machine and decides to buy a drink, which will not sell to her without verifying her age.

2. The machine prompts Alice for 21+ claim presentation via a Bluetooth transmission. The transmission includes an object that specifies what claims it will accept, and when her phone locates a set of probable claims, asks her to choose whether to disclose one to the machine - in this case, the Proof of Age claim.

3. Alice discloses a 21+ claim to the machine, it authenticates that she is the Subject and is a valid signature from the Issuer (via keys fetched from the Universal Resolver), then proceeds to dispense her drink.

<img src="https://i.imgur.com/4UXvjYe.png" style="display: table; margin: 3em auto"/>

```sequence
participant Alice's User Agent as UA
participant Vending Machine as VM
participant Universal Resolver as UR

VM-->UA: 1. Prompts for 21+ proof via RequestClaimAction
UA-->VM: 2. Presents claim via PresentClaimAction
VM-->UR: 3. Fetches keys from UR and validates claim
VM-->VM: 4. On success, dispenses the beverage
```

### Mutli-Step, Mutli-Party Claim Use

While some claims are relatively simple exchanges where the Subject passes data to the Issuer that is deterministic and can return with claim issuance or rejection, other claims have complex multi-step and/or asynchronous processes that may require task-based flows with a persistent state that last for an indeterminate duration until the task is completed.

1. To illustrate a more advanced case, consider the case of a home purchase offer flow. The initial bid comes from the seller, and be a signed offer object that includes the price and a contingency that a licensed inspector can review and clear the property of any issues.

2. The seller may respond with a signature that approves the offer, and agrees to the contingency, or can respond with a different price. This may require several accrued offer/counter loops, all of which are accrued in the auditable offer object that is modified on each leg.

3. Once an offer price is signed by both parties, the task is frozen until the inspection is completed. The inspector arrives at the property, finds nothing wrong, and signs the offer object with his own DID, which can even sign into the offer object proof that he is licensed.

4. All parties involved in the sale can now leverage a high-precision verifiable proof of all that transpired. This may be used at other points during the sale, or at any future time when the details of the exchange may be relevant for verification or review.

<img src="https://i.imgur.com/ZixLc46.png" style="display: table; margin: 3em auto"/>

---

## How do I find claims I to acquire?

Intro: To model the lifecycle of claims, we begin with acquisition. Acquiring a claim may frequently require self-attested data input, claims from third-parties, and even multi-step asynchronous flows to complete issuance.

The Identity Hub datastore scheme allows for deterministic location of all types of semantic data objects, via its Collections interface.

1. The first thing we need in claim issuance is a data format that allows an issuer to describe what raw data, prerequisite claims, and other details are required for the issuer to successfully process a request - like a manifest for what a claim requires.

2. Because a Claim Manifest is just a type of semantic data object, Identity Hubs can expose them just as they would any other data type. This means you can crawl DIDs and make requests to their Hubs to find all publicly available Claim Manifests they expose. There is no requirement to crawl all DIDs to locate claims; you can create more direct claim registration-style services, where DID owners directly request inclusion of the claims they offer (which are fetched from their Hub using the same set of interfaces).

3. The way to best understand how finding claim manifests works (as with any other semantic object its owner publicly exposes), is to imagine the DID Resolver (which provides a directory-style cache of all known DIDs) as the white pages, wherein the DIDs are phone numbers you can call. At the other end of the line is the Identity Hub, which you can ask for specific semantic objects. In this way, you can call every number in the white pages and ask the owner "Hey, do you have any claim manifests you'd like to share?"

<img src="https://i.imgur.com/XpXG7AZ.png" style="display: table; margin: 3em auto"/>

```sequence
participant Claim Crawler/Registry as CC
participant Universal Resolver as UR
participant Identity Hubs as IH

CC-->UR: 1. Request all public DIDs
UR-->CC: 2. Return stream of DIDs
CC-->IH: 3. Request ClaimManifests from each DID's Hub
IH-->CC: 4. Respond with any ClaimManifests the DID wants to share
```

### Comparing Claim Manifest Options

Intro: There are two options under consideration for describing what an individual claim is and codifying all that is required to generate it: DIF's Claim Manifest format and use of modified and extended OIDC claim formats/properties. The two approach the problem from two different angles:

| | <img src="https://i.imgur.com/MVN3vNS.png" style="display: block; height: 3em; margin: 1em auto;" /> <p style="text-align: center;">Extended OIDC Aggregated Claims</p> | <img src="https://i.imgur.com/ikF8cAk.png" style="display: block; height: 3em; margin: 1em auto;" /> <p style="text-align: center;">DIF Claim Manifests</p> |
| :-: | - | - |
| **Description** | OIDC has some basic claim description formats/properties, and the thinking is that it might be easier for devs familiar with OIDC to digest something that looks similar. | The description and requirements for generating a claim can be quite complex, and various requirements in a description may have options/relationships that are difficult to express within an OIDC format that was never designed for this part of the claim lifecycle. |
| **Spec** | https://openid.net/specs/openid-connect-core-1_0.html#Claims | https://hackmd.io/q-selsVyQQ-1p1LK7mHJxw |

<div style="text-align: center;">

<!-- **Comparison:** https://hackmd.io/5xlPH_Q3Q_iAqDupZDxPug?view -->

</div>


### Claim Property/Value Definitions

There are no existing claim definition formats that approach the level of detail needed for flexible generation of custom claim schemas with Issuer-defined properties and values.

It is advisable to use an existing format/specification, such as JSON Schema, to precisely define the properties and values of claim schemas.

| | <img src="https://i.imgur.com/NHpvQJl.png" style="display: block; height: 3em; margin: 1em auto;" /> <p style="text-align: center;">JSON Schema</p> |
| :-: | - |
| **Description** | JSON Schema is a vocabulary that allows you to annotate and validate JSON documents. |
| **Spec** | https://cswr.github.io/JsonSchema/spec/introduction/ |
| **Example** | https://hackmd.io/5xlPH_Q3Q_iAqDupZDxPug?view#2-Claim-Definition |

### DID Proof of Control (Auth)

In all the following examples presented, the initial exchanges between the entities may depend on authenticating that the user is the controller of a DID.

To do this, we have built an OIDC-compatible authentication flow/library that allow entities to prove control of a DID in a way that works with the existing standards in use today: https://github.com/decentralized-identity/did-auth-jose

---

## How do I acquire a claim from an issuer?

1. User Agent apps present users with claims they can request from Issuers, which are represented by Claim Manifests that are found by crawling Identity Hubs or explicit ingestion into a UA's directory (as noted above). Alice, a DID user, chooses to initiate the claim request with the Issuer. 

2. The Issuer's ClaimManifest specifies a set of data and prerequisite claims Alice needs to submit in order for the Issuer to process her claim request. Alice's UA presents her with an interface that helps her input the right data and select relevant prerequisite claims.

3. When Alice has filled out all the fields to meet the requirements, her request is sent to the Identity Hub of the Issuer for processing, which the Enterprise Agent handles. The EA is a headless agent with code that handles business logic and activities specific to an Issuer, Assuming Alice filled out the fields correctly and sent acceptable prerequisite claims, the Issuer will return her an MDL claim.

<img src="https://i.imgur.com/iBmIuX5.png" style="display: table; margin: 3em auto"/>

```sequence
participant Alice's Hub
participant Alice's User Agent
participant Issuer Enterprise Agent
participant Issuer Hub

Alice's User Agent->Alice's User Agent: Alice chooses the MDL claim
Alice's User Agent->Alice's User Agent: Alice assembles required data
Alice's User Agent->MH: UA sends an IssueClaimAction
Issuer Hub-->Issuer Enterprise Agent: EA acts on the request 
Issuer Enterprise Agent-->MH: EA stores the claim in its Hub
Issuer Enterprise Agent-->Alice's Hub: EA returns the claim to Alice
```

### Claim Delivery Targets: User Agent vs Identity Hub

There are two options issuers have for delivering the claims they generate for users: the User Agent wallet app on a user's device, or one of the user's Identity Hub instances. There are different scenarios where one or the other make more sense, depending on the constraints of the use case. Regardless of which is selected, the response should be a DeliverClaimAction, so that it can be handled the same way by either of the targets.

| User Agent | Identity Hub |
| - | - |
| An issuer who is delivering a claim as a final act in a direct, sustained negotiation flow between its Identity Hub/Enterprise Agent and the User Agent app of a user will most likely want to deliver the claim directly to the User Agent already involved in the flow. This would provide more immediate deliver and user awareness of completion, without having to make a needless hop from the Hub to the UA of the user. | In other cases, a claim may be generated as part of a multi-step, long-lived claim generation flow where there is no sustained connection between the UA and the Hub/EA of the Issuer. In these cases, the Issuer has no simple way to reestablish connection with the UA (likely a mobile device) when the claim generation process is completed. In this situation, the best option is to use the DID of the user to locate their Identity Hub instance(s) and deliver the claim there, which the UA will gain awareness of when the two perform their next sync exchange. |

### Claim Negotiation/Storage: Action Exchange vs Collections Write

Much like the targets for deliver (UAs and Hubs), there are two ways a claim data object can be added to a user's Hub: the  Action interface or acquisition of a permission from the user to directly write to their  Collection object storage. It's helpful to understand each interface and in what situations you might use one or the other:

| | Collections | Actions |
| :-: | - | - |
| **Description** | Collections storage is the end destination for all objects that are persisted in Hubs, but outside of specific flows (e.g. Actions) an external entity must acquire permissions to write data directly to the schema/object-specific areas of Collections. Collections is just data storage, with no negotiation or interactive exchange capabilities. This is generally the interface you'd use if your use case involved the need for direct, non-interactive, long-lasting write ability, or you wanted to write/update data multiple times without the user being prompted to allow the activity. | The Actions interface can be thought of as an email inbox of sorts, where the objects an external entity sends to it are semantically typed and intended to invoke specific task/activity flows in response. In the case of claims, claim-specific Action objects allow an entity to negotiate claim acquisition, which may have differing steps and long, asynchronous idle periods between each, depending on the type of claim and its lifecycle. Actions are also used to deliver claims, all without having to acquire a permission for deeper, persistent access to the Collections CRUD interfaces. |
| **Spec** | [Collections Interface](https://github.com/decentralized-identity/identity-hub/blob/master/explainer.md#collections) | [Actions Interface](https://github.com/decentralized-identity/identity-hub/blob/master/explainer.md#actions) |
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTAwNDc0MDA5OF19
-->