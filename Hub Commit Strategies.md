Hub Commit Strategies
===

## Abstract

After further disucssion around our [previous storage/replication concept](https://hackmd.io/OInEIRLxQY2s48tze0E7IQ), we would like to  advance an updated proposal which we hope will accomodate a wider variety of Hub scenarios and permit future enhancements.

Rather than mandating a particular merge algorithm / CRDT for Hub data, we propose to allow the merge algorithm to be configurable at the per-object level through the introduction of "commit strategies".


## Background

Our team has spent some time debating the potential approaches to hub replication. In particular, we have discussed the tradeoffs between two main proposals:

- Our [previous proposal for storage and replication](https://hackmd.io/OInEIRLxQY2s48tze0E7IQ), which relies on a straightforward CRDT-inspired protocol that favors simplicity over robust merge capabilities.

- A suggestion to use the protocol designed by the [Automerge library](https://github.com/automerge/automerge), which offers automatic merging of complete JSON structures using an innovative CRDT, at the expense of some additional complexity and storage overhead.

We came to the conclusion that there is no one-size-fits-all merge algorithm that efficiently meets the needs of all the scenarios we envision (see appendix for example scenarios). Furthermore, the generic nature of Hubs mean that future scenarios will undoubtedly materialize whose needs we cannot yet even predict.

For these reasons, we believe it makes sense to implement an extensible storage model, where Hubs can support multiple merge algorithms.

## Proposal

Rather than mandating a single merge algorithm for all Hub objects, we propose to define a standard set of "commit strategies" which can be chosen at a per-object level.

All of the proposals discussed so far involve the same base mechanics:

1. Objects are represented by an append-only set of immutable commits
2. The current object state is derived by running some deterministic algorithm over the set of currently known commits
3. The algorithm requires some sort of commit metadata (timestamp, vector clocks, etc.)

A commit strategy defines a particular imlementation of the concepts above. It defines the deterministic merge algorithm, as well as the structure and metadata of each commit that the algorithm will operate over. This includes details like whether each commit will store the full object state or some sort of delta.

When creating a Hub object, the client must specify the commit strategy to be used, and provide an initial commit in the appropriate format. All further updates to that object must follow the same strategy.

A client may support only a partial set of commit strategies, especially since more may be added in the future. When communicating with a Hub, the client should indicate which strategies it supports, and the Hub should only expose compatible objects to that client.

Each Hub strategy is represented by a string identifer, and the set of available strategies will be community-defined.



### Initial strategies

Initially we suggest two strategies:

- The `basic` strategy uses a simple last-writer-wins algorithm based on the client-provided timestamp of each commit. This is identical to our [previous proposal](https://hackmd.io/OInEIRLxQY2s48tze0E7IQ) when using only the `state` field of each commit.

- The `automerge` strategy uses the protocol designed by the [Automerge library](https://github.com/automerge/automerge) to allow deterministic merging of arbitrary JSON documents.

Further strategies can be defined in the future as additional use cases are identified.

#### Basic strategy

The basic strategy follows the algorithm identified in our [earlier proposal](https://hackmd.io/OInEIRLxQY2s48tze0E7IQ), with the slight change that each commit must contain the full state of the object (the patch-based delta system is replaced with the `automerge` strategy described below).

Essentially, each commit contains the complete current state of the object, and the commit with the highest client-provided timestamp is deemed to be the final state.

This strategy offers:

- A straightforward implementation which can easily be supported across languages
- Easily understandable behavior when commits are infrequent or only from a single actor
- Support for garbage collection of older history

However, the inability to cleanly merge updates together makes it unsuitable for scenarios involving active collaboration or extended offline editing.

##### Sample update commit:

:::warning
The exact commit structure is still under discussion; the examples here are intended as a visual aid and not a definitive specification.
:::

Example of an object update using the `basic ` strategy:

```json
{
    "rev": "fe4fd3240ff1c68a85730b3cf35742c8a241a0a70bd98e8869b4b593",
    "operation": "update",
    "committed_at": 1530309810,
    "parents": ["3a9de008f526d239d89905e2203fa484f6e68dfc096a7c051eb80f15"],
    "object": "3a9de008f526d239d89905e2203fa484f6e68dfc096a7c051eb80f15",
    "strategy": "basic",
    "state": {
        "meta": {
            "@context": "http://schema.org",
            "@type": "MusicPlaylist",
            "name": "Sample playlist",
        },
        "data": {
            "@context": "http://identity.foundation",
            "@type": "EncryptedPayload",
            "encryptedData": "TG9yZW0gaXBzdW0gZG9sb3Igc2l0IGFtZXQ..."
        }
    }
}
```

#### Automerge strategy

With the `automerge` strategy, each commit stores a serialized list of modifications in the format designed by the [Automerge library](https://github.com/automerge/automerge). These commits can be deterministically merged together using the protocol implemented by this same library.

Automerge offers several advantages over our previous proposal, which suggested JSON Path for diff-based merges:

- An identity-based merge (rather than a diff-based one) offers a more intuitive developer experience and reduces the chance of unexpected interactions.
- In the case of an unavoidable conflict, a winning change is chosen deterministically, and losing changes are available in a `_conflicts` property for easy inspection.
- Vector clocks offer a more accurate way to track change causality.

However, these advantages come with some tradeoffs:

- Although the concept is intuitive, the algorithm implementation is relatively complex and requires additional effort to port cross-platform.
- An object's complete history is always persisted, which increases storage overhead. However, the creator believes a garbage collection mechanism is possible.

With the `automerge` strategy, each commit stores a list of changes made during that commit, as well as some additional metadata including a vector clock.

To reconstruct the current object state, the changes from all known commits are run through the Automerge algorithm, which uses the vector clock embedded in each commit to deterministically merge the changes into a final state.

Our implementation will need to handle the presence of both plaintext (metadata) and encrypted (payload) data. There are a couple of options for this:

- We could use two side-by-side standard Automerge change lists, one in plaintext and one encrypted. As long as the same actor ID is used for changes to both lists, the algorithm should resolve to a consistent set of changes.

- Alternatively, we could define a custom serialization format containing two separate change lists (one encrypted, one plaintext) and a shared actor ID, sequence number, and message. The client can then decrypt this format and transform it into the format expected by the Automerge library.

##### Sample update commit:

```json
{
    "rev": "fe4fd3240ff1c68a85730b3cf35742c8a241a0a70bd98e8869b4b593",
    "operation": "update",
    "committed_at": 1530309810,
    "object": "3a9de008f526d239d89905e2203fa484f6e68dfc096a7c051eb80f15",
    "strategy": "automerge",
    "changes": [
        {
            "ops": [
                {
                    "action": "makeList",
                    "obj": "aae86acc-e210-4512-92dd-1e163b1de9af"
                },
                {
                    "action": "ins",
                    "obj": "aae86acc-e210-4512-92dd-1e163b1de9af",
                    "key": "_head",
                    "elem": 1
                },
                {
                    "action": "set",
                    "obj": "aae86acc-e210-4512-92dd-1e163b1de9af",
                    "key": "49322644-49f7-48a2-adb3-9d2cfec142e7:1",
                    "value": "abc"
                },
                {
                    "action": "ins",
                    "obj": "aae86acc-e210-4512-92dd-1e163b1de9af",
                    "key": "49322644-49f7-48a2-adb3-9d2cfec142e7:1",
                    "elem": 2
                },
                {
                    "action": "set",
                    "obj": "aae86acc-e210-4512-92dd-1e163b1de9af",
                    "key": "49322644-49f7-48a2-adb3-9d2cfec142e7:2",
                    "value": "def"
                },
                {
                    "action": "link",
                    "obj": "00000000-0000-0000-0000-000000000000",
                    "key": "someArray",
                    "value": "aae86acc-e210-4512-92dd-1e163b1de9af"
                }
            ],
            "actor": "49322644-49f7-48a2-adb3-9d2cfec142e7",
            "seq": 1,
            "deps": {},
            "message": "Create an array"
        },
        {
            "ops": [
                {
                    "action": "ins",
                    "obj": "aae86acc-e210-4512-92dd-1e163b1de9af",
                    "key": "49322644-49f7-48a2-adb3-9d2cfec142e7:1",
                    "elem": 3
                },
                {
                    "action": "set",
                    "obj": "aae86acc-e210-4512-92dd-1e163b1de9af",
                    "key": "49322644-49f7-48a2-adb3-9d2cfec142e7:3",
                    "value": "123"
                }
            ],
            "actor": "49322644-49f7-48a2-adb3-9d2cfec142e7",
            "seq": 2,
            "deps": {},
            "message": "Splice an item"
        }
    ]
}
```

:::warning
Open question: Allow multiple changes in each commit, or force one commit per change?
:::
:::warning
Open question: Should verify that two Automerge objects edited in parallel will always both resolve to the same actor's changes
:::

### Hub support

Hubs will be able to store objects using unknown strategies, since the Hub's main responsibility is to accept, store, and return commit data as requested. However, Hubs will likely need to implement most strategies in order to make use of certain objects (e.g. permissions) and provide indexing of metadata.

Our intention is to initially focus on supporting the `basic` strategy in the Hub, which will lay the foundation for progress to be made in other areas that rely on storage (e.g. permissions) and accelerate delivery of end-to-end scenarios.

However, we plan to simultaneously add support for the `automerge` strategy in the JavaScript Hub implementation. We believe this is key to ensureing we design a generic and reusable storage interface that does not rely on specific properties of the `basic` strategy.

Usage of the Automerge strategy is expected to be small at first, due to the additional effort of re-implementing the algorithm in each language-specific Hub SDK. However we hope to have Automerge as a fully-supported strategy by the time that Hubs are fully available for production use.

### Client support

To meet our privacy goals, the decryption and thus the merging of commits must take place client side. This means support for each commit strategy must be re-implemented in the client library of every supported language/platform.

When communicating with a Hub, client apps and SDKs should specify the commit strategies which they support. Hub objects written with incompatible strategies should be ignored.

Initially we intend to implement `basic` and `automerge` support in the JavaScript SDK, as a JavaScript implementation of Automerge is already available. Implementations in other languages will focus on supporting the `basic` strategy at first, as porting the Automerge algorithm will require additional effort.


## Appendix

### Scenarios

When comparing potential merge algorithms, we found that we could already envision scenarios where one approach or the other had a clear advantage:

- An IOT device wishing to push frequent updates to an object (e.g. a smart thermostat updating the current temperature) might be fine with last-writer-wins semantics, especially if only one actor will ever write to the object.

  In this case, Automerge's state tracking overhead and lack of garbage collection could result in an unnecessary amount of overhead.

- Simultaneous or close-in-time collaboration (e.g. on the medical record of a patient undergoing multiple tests in one day) requires robust merging of commits to ensure that multiple users can make edits without overwriting each other's work. This scenario is handled very well by Automerge.

Additionally, we can envision some future scenarios where we may want to add a new commit strategy:

- Collaborative document editing, which works best with character-level tracking of insertions and deletions, would benefit from a separate commit strategy that's specifically designed for this use case.



