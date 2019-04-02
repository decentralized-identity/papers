# Recommended DID Profile Descriptors

The following is a list of recommended descriptors to use when conveying what type of entity a DID represents. Members of the Decentralized Identity Foundation compiled this list based on examination of descriptors commonly used to represent entities across industry verticals and existing system that include semantic role definitions.

>NOTE: Use of the dash (`-`) character in the `Vertical` columns below indicates the descriptor is not from an industry-specific schema and can be used across vertical contexts.

## Descriptors by Entity Type

### Humans

| Descriptor(s)  | Schema | Vertical |
| ------ | ------ | :------: |
| [Person](https://schema.org/Person) | schema.org | - |

### Organizations

There are numerous descriptors across various industry-specific schemas that are commonly used to describe different types of organizations. The user/manager responsible for determining the entity association for a given organization should select the descriptor that is most common in the vertical in which their organization is typically classified.

| Descriptor(s)  | Schema | Vertical |
| ------ | ------ | :------: |
| [Organization](https://schema.org/Organization) | schema.org | - |
| [Corporation](https://schema.org/Corporation) | schema.org | - |
| [LocalBusiness](https://schema.org/LocalBusiness) | schema.org | - |
| [Organization](http://hl7.org/fhir/organization) | hl7.org/fhir | Medical |
| [Organization](https://www.gs1.org/voc/Organization) | gs1.org | Supply Chain |

### Products & Assets

Any type of product, excluding intangibles and service-based offerings.

| Descriptor(s)  | Schema | Vertical |
| ------ | ------ | :------: |
| [Product](https://www.gs1.org/voc/Product) | gs1.org | Supply Chain |
| [Product](https://schema.org/Product) | schema.org | - |

### Apps

| Descriptor(s)  | Schema | Vertical |
| ------ | ------ | :------: |
| [SoftwareApplication](https://schema.org/SoftwareApplication) | schema.org | - |
| [MobileApplication](https://schema.org/MobileApplication) | schema.org | - |
| [WebApplication](https://schema.org/WebApplication) | schema.org | - |
| [VideoGame](https://schema.org/VideoGame) | schema.org | - |

### Services

Any type of service offering.

| Descriptor(s)  | Schema | Vertical |
| ------ | ------ | :------: |
| [Service](https://schema.org/Service) | schema.org | - |

### Industry Vertical: Code Developer

| Descriptor(s)  | Schema | Vertical |
| ------ | ------ | :------: |
| [codeRepository](https://schema.org/codeRepository) | schema.org | - |
| [SoftwareSourceCode](https://schema.org/SoftwareSourceCode) | schema.org | - |

## Profile Object Examples

The `descriptors` property can be an array of objects from different schemas that contribute to the description of what the DID-linked entity is. User Agents and consuming entities should assume a 0-index ascending order of primacy.

> NOTE: The `Profile` object wrapper defined by DIF's Identity Hub schema is not intended to define/contains a large number of properties that describe the target entity. Doing so should be left to the descriptor objects provided in the `descriptors` array.

```javascript
{
    "@context": "https://identity.foundation/schemas/hub",
    "@type": "Profile",
    "did": "did:foo:123"
    "name": "Jeff Lebowski",
    "nickname": "The Dude",
    "email": "ilovebowling@email.com",
    "picture": IMG_URL,
    "descriptors": [
        {
            "@context": "http://schema.org",
            "@type": "Person",
            "name": "Jeffrey Lebowski",
            "description": "That's just, like, your opinion, man.",
            "address": {
              "@type": "PostalAddress",
              "streetAddress": "5227 Santa Monica Boulevard",
              "addressLocality": "Los Angeles",
              "addressRegion": "CA"
            }
        },
        {...}
    ]
}

// Person profile, without top-level props

{
    "@context": "https://identity.foundation/schemas/hub",
    "@type": "Profile",
    "did": "did:sov:456"
    "descriptors": [
        {
            "@context": "http://schema.org",
            "@type": "Person",
            "name": "Jeffrey Lebowski",
            "description": "That's just, like, your opinion, man.",
            "address": {
              "@type": "PostalAddress",
              "streetAddress": "5227 Santa Monica Boulevard",
              "addressLocality": "Los Angeles",
              "addressRegion": "CA"
            }
        },
        {...}
    ]
}
```


<!--stackedit_data:
eyJoaXN0b3J5IjpbMzU3OTczOTU5XX0=
-->