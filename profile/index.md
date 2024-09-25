Based on : [https://github.com/workflowhub-eu/about/tree/master/Workflow-RO-Crate](https://github.com/UoA-eResearch/ro-crate-py/pull/1)

# GPG Crate (DRAFT)

<!--  https://signposting.org/FAIR/  markup --->

<!-- <link href="https://github.com/ResearchObject/ro-crate/blob/profiles/docs/1.2-DRAFT/profiles.md" rel="type"  />
<link href="http://purl.org/dc/terms/Standard" rel="type"  />
<link href="https://schema.org/CreativeWork" rel="type"  />

<link href="https://spdx.org/licenses/BSD-3-Clause" rel="license"  />

<link href="ro-crate-metadata.json" rel="describedby" type='application/ld+json; profile="https://w3id.org/ro/crate"' />
<link href="ro-crate-metadata.jsonld" rel="describedby" type='application/ld+json; profile="https://w3id.org/ro/crate"'  />
<link href="ro-crate-preview.html" rel="describedby" type='text/html'  /> -->

<!-- repeat of hasPart in RO-Crate -->
<!-- <link href="index.html" rel="item"  />
<link href="licenses/" rel="item" />
<link href="languages/" rel="item" />
<link href="example/" rel="item" />
<link href="https://w3id.org/ro/crate/1.1/context" rel="item" />
<link href="https://pypi.org/project/rocrate/" rel="item" />
<link href="https://github.com/inab/WfExS-backend/" rel="item" />
<link href="https://www.nationalarchives.gov.uk/PRONOM/x-fmt/263" rel="item" />
<link href="https://workflowhub.eu/" rel="item" />
<link href="https://bioschemas.org/ComputationalWorkflow" rel="item" />
<link href="https://bioschemas.org/FormalParameter" rel="item" />
<link href="http://schema.org/HowTo" rel="item" />
<link href="http://schema.org/ImageObject" rel="item" />
<link href="https://github.com/KockataEPich/CheckMyCrate/blob/master/CheckMyCrate/profile_library/ro_crate_1.1_basic.json" rel="item" /> -->

<!-- ![GPG RO-Crate]({{ '/assets/img/ro-crate-workflow.svg' | relative_url }})  -->

* Permalink: `TODO`
* Version: [0.0.1](https://github.com/UoA-eResearch/ro-crate-py/pull/1)

<!-- As Encrypted crates differ for how they are constructed in memory as opposed to how they are written on disk a profile crate is provided for each.
* [Profile Crate - In Memory `ro-crate-metadata.json`](TODO)
  - [Profile Crate preview](TODO)
* [Profile Crate - On Disk `ro-crate-metadata.json`](TODO)
  - [Profile Crate preview](TODO)
* [Example RO-Crate - In Memory`ro-crate-metadata.json`](TODO)
  - [Example RO-Crate profile preview](TODO)
* [Example RO-Crate - On Disk `ro-crate-metadata.json`](TODO)
  - [Example RO-Crate profile preview](TODO) -->



_GPG RO-Crate_ is an extension of [_RO-Crate_](https://researchobject.github.io/ro-crate/) for storing and transmitting sensitive metadata. It depends on [GnuPGP(GPG)](https://gnupg.org/) for encryption and key management.

The encryption and decryption processes described in this profile are implemented as part of https://github.com/UoA-eResearch/ro-crate-py/tree/encrypted-metadata.

## Concepts

This section uses terminology from the [RO-Crate 1.1 specification](https://w3id.org/ro/crate/1.1) and [PGP standard](https://www.ietf.org/rfc/rfc4880.txt).


## Conforms To

The Metadata File Descriptor `conformsTo` MUST be a list that contains at least this profile and the RO-Crate profile `"https://w3id.org/ro/crate/1.1"`.

Any *EncryptedContextEntity*, *EncryptedGraphMessage* or *Recipient* MAY list this profile via `conformsTo`.

## Context

The Crate JSON-LD MUST be valid according to RO-Crate 1.1 and SHOULD use the RO-Crate 1.1 `@context` https://w3id.org/ro/crate/1.1/context

Additional Terms are provided as

## Encrypted Context Entities

Data is encrypted on an Entity by Entity basis, as *EncryptedContextEntities*.

An *EncryptedContextEntity* MUST be a context entity.

An *EncryptedContextEntity* MUST NOT be the root entity.

If a data entity has associated sensitive metadata these SHOULD be created as separate *EncryptedContextEntities* linked to the data entity using fields such as `about`, `subjectOf,` or `additionalProperty` etc.

Any *EncryptedContextEntity* MAY OPTIONALLY contain `EncryptedContextEntity` in its type. *(as almost any type of entity MAY be an encryptedcontextentity with their original typing to be retained unchanged, for this reason EncryptedContextEntities SHOULD be defined programmatically in the library writing or reading the crate rather by their @type)*.

Any *EncryptedContextEntity* MUST have a least one entity as a `recipient` and that *recipient* MUST have at least one valid gpg public key fingerprint listed via the `pubkeyFingerprints` property.

Values specified in an *EncryptedContextEntity*'s `recipients` property SHOULD refer to other context entities within the graph via the standard `"recipients":[{"@id":"<id>"}]` format. *(they should not be raw strings or references to external files)*.

Encrypted context entities MUST only exist in a decrypted state only while in memory.

Once the crate is written to disk as `ro_crate_metadata.json` *EncryptedContextEntities* are aggregated based on common sets of `recipient` `pubkeyFingerprints` and written as the `encrypted_graph` property of an `EncryptedGraphMessage`.

When data is decrypted from an *EncryptedGraphMessage* it MUST be decrypted into an *EncryptedContextEntity* it MAY be manually redesignated as a *Context Entity* later if the data is no longer to be encrypted.

## Encrypted Graph Messages

When encrypted data is written to disk it is written as an *EncryptedGraphMessage* which packages a PGP encrypted string with metadata regarding its encryption.

An *EncryptedGraphMessage* MUST be of the type `["SendAction", "EncryptedGraphMessage"]`.

### Writing Encrypted Context Entities as part of Encrypted Graph Messages

If an encrypted string is present it must be stored as an `"encryptedGraph"`. This MUST be a PGP Encrypted Message as described in 11.3 of the [PGP standard](https://www.ietf.org/rfc/rfc4880.txt).

Encrypted messages stored via `"encryptedGraph"` MUST be an aggregation of *EncryptedGraphMessage*s that share a complete set of public keys across each of their recipients. They are aggregated by serialising the a list containing the JSON representation of each *EncryptedGraphMessage*.

The raw JSON representation of each *EncryptedGraphMessage* encrypted as part of an `"encryptedGraph"` MUST be a JSON string that can be read as a valid RO-Crate Entity.

Encrypted messages stored via the `"encryptedGraph"` property MUST be asymmetrically encrypted using a cipher method available via [GPG](https://en.wikipedia.org/wiki/GNU_Privacy_Guard#Algorithms) or [libcrypto](https://wiki.openssl.org/index.php/Libcrypto_API) before being written to disk.

Encrypted messages stored via `"encryptedGraph"` MAY be signed using the *private key* of the entity that wrote and encrypted the RO-Crate.

### Decrypting Encrypted Graph Messages

An *EncryptedGraphMessage* once decrypted MUST be removed from the graph to prevent data duplication or overwriting with old data. (the encrypted context entities it contained can later be re-encrypted)

An *EncryptedGraphMessage* that cannot be decrypted MAY be removed from the graph.

When an *EncryptedGraphMessage* is read from a *crate* stored on disk an attempt SHOULD be made to decrypt the message stored in `"encryptedGraph"` using any private keys available to the system reading the *crate*.

If the `"encryptedGraph"` of an *EncryptedGraphMessage* can be decrypted back into plaintext JSON it MUST be de-aggregated reconstructed into the original *EncryptedContextEntity*'s it consists of and these *EncryptedContextEntity* MUST be re-inserted back into the `@graph` with their appropriate `@id` values and other metadata intact.

An *EncryptedGraphMessage* once decrypted MUST be removed from the graph to prevent data duplication or overwriting with old data. (the encrypted context entities it contained can later be re-encrypted).

An *EncryptedGraphMessage* that cannot be decrypted MAY be removed from the graph.

### Encrypted Graph Message metadata

An *EncryptedGraphMessage* SHOULD record its status via `"actionStatus"`. E.g. `'actionStatus":"PotentialActionStatus"` for a message that is yet to be decrypted or sent.

An *EncryptedGraphMessage* SHOULD record the message format of the message encrypted as `"encryptedGraph"` property via the `deliveryMethod` property.

An the `deliveryMethod` property of an *EncryptedGraphMessage* SHOULD point to a URI of a standard or documentation that provides context to identify and the message stored in `'encryptedGraph"` format, this SHOULD be sufficient to determine a decryption method for the message. For example "https://doi.org/10.17487/RFC4880" for PGP encrypted messages.

Any *EncryptedGraphMessage* MUST list all `"recipients"` matching the complete set of `"recipients"` of any *EncryptedContextEntities* that were aggregated and encrypted as part of the *EncryptedGraphMessage*.

Values specified in an *EncryptedGraphMessage*s `"recipients"` property SHOULD refer to other context entities within the graph via the `"recipients":[{"@id":"<id>"}]` format. *(they should not be raw strings or references to external files)*

`"recipients"` of an  *EncryptedGraphMessage* SHOULD refer to the complete set of private key holders that are able to decrypt the message stored in `"encryptedGraph"`. They MAY list contact information to identify these individuals and  MAY identify their public keys via `"pubkeyFingerprints".

## Recipients

A *Recipient* SHOULD be of the type `"ContactPoint"` `"Person"` and/or `"Audience"`.

*Recipients* of *EncryptedGraphMessage*s and *EncryptedContextEntities* MUST be referred to via the `"recipients"` property.

*Recipients*  and MAY refer back to a*EncryptedGraphMessage*s and *EncryptedContextEntities* using the `"recipientOf"` property.

*Recipients* of *EncryptedGraphMessage*s and *EncryptedContextEntities* MUST store at least one public key fingerprint via the `pubkeyFingerprints` property.

The fingerprints stored via *Recipients*' `pubkeyFingerprints` MUST refer to public keys accessible to the system writing the *crate* either locally or via a keyserver.

*Recipients* MAY list a keyserver from which the public keys matching their `pubkeyFingerprints` can be retrieved via `keyserver`.

## Summary: Reading and Writing a GPG-Crate 

### Writing an RO-Crate with encrypted Entities

![Writing an RO-Crate containing encrypted elements to disk](EncryptedRO-Crate.drawio(1).svg)

### Reading an RO-Crate with encrypted Entities

![Reading an RO-Crate that contains encrypted elements](ReadingEncryptedRO-Crate.drawio(1).svg)

### Signatures
(TBD)



### ro-crate-metadata.jsonld Example
A minimal example of _GPG Crate_ metadata, containing example sensitive banking and health information.

#### As written to disk (after encryption)
```json
{
{
	"@context": "https://w3id.org/ro/crate/1.1/context",
	"@graph": [
    	{
        	"@id": "./",
        	"@type": "Dataset",
        	"datePublished": "2024-09-17T23:46:39+00:00"
    	},
    	{
        	"@id": "ro-crate-metadata.json",
        	"@type": "CreativeWork",
        	"about": {
            	"@id": "./"
        	},
        	"conformsTo": [
          	{"@id": "https://w3id.org/ro/crate/1.1"},
          	{"@id": "THIS URL"}
        	]
    	},
    	{
        	"@id": "https://orcid.org/0000-0002-1825-0097",
        	"@type": "Person",
        	"email": "JCarberry@psychoceramics.brown.com",
        	"name": "Josiah Carberry",
        	"keyserver": "https://keyserver.ubuntu.com",
        	"pubkeyFingerprints": "985E471827FEF4D193C2CDBF65322C25ED00AB00"
    	},
    	{
        	"@id": "#Encrypted_Message985E471827FEF4D193C2CDBF65322C25ED00AB00",
        	"@type": [
            	"SendAction",
            	"EncryptedGraphMessage"
        	],
        	"actionStatus": "PotentialActionStatus",
        	"deliveryMethod": "https://doi.org/10.17487/RFC4880",
        	"encryptedGraph": "-----BEGIN PGP MESSAGE-----\n\nhF4Dy9uOJGGmSI4SAQdApZDEDcRWXvcYyndH3YQJDaY3bFtoii/jAaYTw+g2sEsw\neY9h46EeGo3KIyD61wu4sTJLUFCihFoLzb3jzJuNmGzTAIIpZjwQSnZTmIOBf9+V\n1MBFAQkCENnCJLcOhqgchWNkv2HcyrK+//QVmRyICdK3DPaIPIJgzgI6fjGB3Ck+\nmn2HYbr29p1PiKv/ijeJ8jEa4CD0DXchjZzyQ8m5EDlZv9pscIwmYjppz0exyZKH\n/BYaSdGuI1xYcov3tdJNb87XspFq7e7Hg6E1K4x7EWoxM33CJHFP0MeyMGIjx8qg\nnXQUCEEVafxn8jxHrj0wU5bu22EOxCoZwFCgQmKakYUzs9BUzcKB5zeEE5xx0wlL\noOs3+qKuXdPWYoVIpLc6q7jhIeTZfcDrXXRWQBC2r8dhLjbTFHln1YkqCqv+fcCw\nUN7RhtYYK84q5PyS1iE5crquXrZaz1gd\n=rKbY\n-----END PGP MESSAGE-----\n",
        	"recipients": [
            	{
                	"@id": "https://orcid.org/0000-0002-1825-0097"
            	}
        	]
    	}
	]
}
```

#### In memory (before encryption/after decryption)

```json
{
	"@context": "https://w3id.org/ro/crate/1.1/context",
	"@graph": [
    	{
        	"@id": "./",
        	"@type": "Dataset",
        	"datePublished": "2024-09-18T01:19:14+00:00"
    	},
    	{
        	"@id": "ro-crate-metadata.json",
        	"@type": "CreativeWork",
        	"about": {
            	"@id": "./"
        	},
        	"conformsTo": [
          	{"@id": "https://w3id.org/ro/crate/1.1"},
          	{"@id": "THIS URL"}
        	]
    	},
    	{
        	"@id": "https://orcid.org/0000-0002-1825-0097",
        	"@type": "Person",
        	"email": "JCarberry@psychoceramics.brown.com",
        	"name": "Josiah Carberry",
        	"keyserver": "https://keyserver.ubuntu.com",
        	"pubkeyFingerprints": "985E471827FEF4D193C2CDBF65322C25ED00AB00"
    	},
    	{
        	"@id": "#ExampleSensitiveDataBank",
        	"@type": "BankAccount",
        	"accountOverdraftLimit": "$50000000",
        	"name": "Carberry Research Grant Account",
        	"recipients": "https://orcid.org/0000-0002-1825-0097"
    	},
    	{
        	"@id": "#ExampleSensitiveDataMedical",
        	"@type": "MedicalCondition",
        	"name": "Memory Bus Factor Syndrome",
        	"naturalProgression": "full psychoceramic breakdown",
        	"recipients": "https://orcid.org/0000-0002-1825-0097"
    	}
	]
}
```

#### Encrypted message
The contents of
```
"-----BEGIN PGP MESSAGE-----\n\nhF4Dy9uOJGGmSI4SAQdApZDEDcRWXvcYyndH3YQJDaY3bFtoii/jAaYTw+g2sEsw\neY9h46EeGo3KIyD61wu4sTJLUFCihFoLzb3jzJuNmGzTAIIpZjwQSnZTmIOBf9+V\n1MBFAQkCENnCJLcOhqgchWNkv2HcyrK+//QVmRyICdK3DPaIPIJgzgI6fjGB3Ck+\nmn2HYbr29p1PiKv/ijeJ8jEa4CD0DXchjZzyQ8m5EDlZv9pscIwmYjppz0exyZKH\n/BYaSdGuI1xYcov3tdJNb87XspFq7e7Hg6E1K4x7EWoxM33CJHFP0MeyMGIjx8qg\nnXQUCEEVafxn8jxHrj0wU5bu22EOxCoZwFCgQmKakYUzs9BUzcKB5zeEE5xx0wlL\noOs3+qKuXdPWYoVIpLc6q7jhIeTZfcDrXXRWQBC2r8dhLjbTFHln1YkqCqv+fcCw\nUN7RhtYYK84q5PyS1iE5crquXrZaz1gd\n=rKbY\n-----END PGP MESSAGE-----\n"
```
Decrypts to exactly the string:
```
"{
	"@id": "#ExampleSensitiveDataBank",
	"@type": "BankAccount",
	"accountOverdraftLimit": "$500000",
	"name": "Carberry Research Grant Account",
	"recipients": "https://orcid.org/0000-0002-1825-0097"
},
{
	"@id": "#ExampleSensitiveDataMedical",
	"@type": "MedicalCondition",
	"name": "Memory Bus Factor Syndrome",
	"naturalProgression": "full psychoceramic breakdown",
	"recipients": "https://orcid.org/0000-0002-1825-0097"
}"
```
