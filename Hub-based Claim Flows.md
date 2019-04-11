
### 1. Alice browses available claims of an Issuer

```sequence
participant Alice UA as UA
participant Issuer Hub as IH

UA->IH: 1. Alice queries Collections for CredentialManifests
IH->UA: 2. Returns all CredentialManifests Alice has access to
```

### 2. Alice synchronously obtains a claim

```sequence
participant Alice Hub as AH
participant Alice UA as UA
participant Issuer Hub as IH
participant Issuer EA as EA

UA->IH: 1. Alice sends RequestClaimAction
IH->EA: 2. Issuer processes claim request
EA->IH: 3. Generates IssueClaimAction
IH->UA: 4. Responds with IssueClaimAction
UA->AH: 5. Accepts and stores claim
```

### 3. Alice obtains a claim in an aync flow

```sequence
participant Alice Hub as AH
participant Alice UA as UA
participant Issuer Hub as IH
participant Issuer EA as EA

UA->IH: 1. Alice sends RequestClaimAction
IH->EA: 2. Issuer processes claim request
EA->AH: 3. Delivers IssueClaimAction when finished
AH->UA: 4. Alice is notified of delivery
UA->AH: 5. Alice accepts and stores claim
```

### 3. One of Alice's credential inputs is invalid

```sequence
participant Alice UA as UA
participant Issuer Hub as IH
participant Issuer EA as EA

UA->IH: 1. Alice sends RequestClaimAction
IH->EA: 2. Issuer processes claim request
EA->EA: 3. One of the claim inputs fails
EA->IH: 4. Generates a DenyClaimAction
IH->UA: 5. Responds with DenyClaimAction
UA->UA: 6. Alice retries or accepts rejection
```