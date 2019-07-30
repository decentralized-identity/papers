# DID Trust Establishment

A list of ideas and proposals for establishing trust in and between DIDs.

## Domain-to-DID Linkage

One common form of trust we would like to support is the ability of a DID to prove ownership of a DNS domain. There are a few possible options for doing this, but the following two are relatively straightforward to accomplish without any complex protocol code:

### DNS URI Record

The URI record type is arguably the closest fit for declaring a DID relationship in a DNS Zone File. In talking with Markus, there was some exploration done in this area that participated in with a few folks from DNS working groups. You can read their proposal here: https://datatracker.ietf.org/doc/draft-mayrhofer-did-dns/?include_text=1

Here is an example of what they currently propose:

    _did.example.net.  IN URI 100 10 "did:ion:1234abcd"

The group that worked on this proposal assumed that you would resolve the DID referenced in the URI Record and locate a matching domain in its DID Document, to create a two-way verification of the linkage. The issue with this approach is that validation based on inclusion of a domain entry in a DID Document forces a potentially significant amount of unnecessary data into the DID Document, consuming the DID network's scarce resources. Instead, I propose adding a signature to the URI record's DID value that allows an observer to prove the DID owner signed the matching domain string to authorize the association.

Here is an example of my proposed modification:

    _did.example.net.  IN URI 100 10 "did:ion:1234abcd?sig=3Yj4De#key-1"
    
The signature is over the value of the domain name string specified in the URI record's Name string (at the start of the record, on the left), and the DID Document key descriptor ID of the key it was signed with.

### `Well-Known` URI

Another option is to register a new `/.well-known` URI with IANA. Here is the current list of IANA-registered Well-Known URIs: https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml

We could register a URI of `/.well-known/did`, which could be a JSON file of the following format:

```jsonld=
{
    "did:ion:1234abcd": {
        "key": "key-1",
        "signature": "3Yj4De..."
    },
    "did:ion:5678efgh": { ... }
}
```

## DID-to-DID Trust Expression

Another area of interest is how DIDs communicate trust or distrust of other DIDs. There are many possible ways to accomplish the conveyance of trust in another entity, but whatever we pick must be _universally_ knowable, so that you can locate trust information quickly and easily across all entities in the ecosystem.

### Exposing TrustLists via Identity Hubs

One way to expose and digest the trust/distrust of DIDs is to allow DIDs a way to expose publicly resolvable data that describes how they view another entity. To do this, we can leverage another piece of decentralized identity infrastructure: Identity Hubs.

Identity Hubs feature an interface for exposing public data via semantic formats, called Collections. The Collections interface allows a DID owner to store any type of data within it, the only requirement is that the data be semantically typed and declared to the Hub, for proper segmentation within the datastore.

#### `TrustList` Schema

The first thing we need to establish for using an Identity Hub as the means of conveying trust information, is what schema the information with be encoded in. The following proposes a schema for a `TrustList`, which allows a DID owner to declare a list of DIDs and trust context details that scope the trust intent.

```jsonld=
{
    "@context": "schema.identity.foundation/hub",
    "@type": "TrustList",
    "scope": "known-entity",
    "entities": {
        "did:foo:123": {
            "sentiment": "positive"
        },
        "did:foo:456": {
            "sentiment": "negative"
        }
    }
}
```

