# DomainDNA Specifications

**DomainDNA: Domain Doings Notation (Abbreviated)**

v0.1 - 2025-11-19

Alexander Mayrhofer, Clemens Moritz; nic.at GmbH

License: 

* This Document itself licensed under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0)
* If derivative works change significant parts of this specification, the resulting work must not be named "DomainDNA", but should mention "based on DomainDNA". This is to prevent "Balkanization" of the concept.

*(Non-legal / less scary version - We would love if you found DomainDNA useful, and even better if you have ideas to improve it! Buuuut please talk to the authors if you think DomainDNA can be changed for the better - and we are more than happy to discuss any ideas / changes for integration into the base specs.)*

## Introduction
Domain Names are typically administrated by a business relationship between a “Registry” and a “Registrar”. The Registry keeps the authoritative information about the state (and, in case of a “thick” registry, also ownership information), while the Registrar typically performs transactions on said domain names, based on requests by their customers (The Registrant). The set of possible transactions as well as their impact on the state of domain are typically defined by the Registry.

The sequence of transactions for a domain can range from just one (namely the initial registration) to several thousand in extreme cases. That sequence is interesting for humans as it represents the (administrative) history of a domain name, but might also contain information suitable as an input to machine learning models.

However, the full data of all transactions on a Domain Name is typically very extensive, unstructured, and very much depends on the policy of the actual Registry Operator. Reducing the essential parts of this data to a compact form makes it more accessible to humans an machine learning models alive. Additionally, if that data element is identical (or at least similar) across different operators, it might ease cross-operator research and data sharing.

Such a compact from represents a "fingerprint" of the most important events in the "life" of a Domain Name - Aaaand... of course, such a format requires both a catchy name as well as an appropriate acronym expansion - and after extensive research we're more than happy to introduce (and specify in this document) the 

*DomainDNA: Domain Doings Notation (Abbreviated)*

![DomainDNA visualization example](assets/example-title.png?raw=true "DomainDNA Visualization")

(_Bah-dum-tss_!)

## The DomainDNA Format

DomainDNA maps the essential information from a Domain Name's life to a compact, human-readable (ASCII) string. Based on our experience, we do believe that the most essential transaction information of a Domain Name is:

* The chronological order and type of major transactions (we call these "*Doings*") performed on the domain 
* The time interval ("*Interval*") between these *Doings*

The transaction life is therefore a repeating sequence of *Intervals* and *Doings*. Mapping each of the *Doings* and *Intervals* to single ASCII characters creates a compact, variable length sequence in the shape  *IDIDID...* (With *I* symbolizing any *Interval Descriptor* and *D* any *Doings Descriptor*). The following sections describe mappings and the structure of DomainDNA in detail.

### Interval Descriptors

Representing continous *Intervals* in a finite set of ASCII characters is not directly possible - therefore, we split that continous interval into ranges/buckets, and assign one *Interval Descriptor* to each bucket. Additionally, time intervals between transactions on Domain Names have a great variety in length - from a few ms to several years - which, obviously, would command for using a logarithmic scale during bucket definition. However, as we want *DomainDNA* to be human readable, we are proposing to use a modified scale that considers "human friendly units" rather than a strict logarithmic scale. The bucket ranges are aligned to typical intervals seen in the domain name ecosystem.

We created statistics about the typical intervals seen at the `.at` registry, so that each bucket contained a similar population of interval exhibits, and assigned numbers in ascending order to these buckets.

Including a few non-numeric characters for exceptional intervals, the resulting mapping is defined as follows.

| Interval Descriptor | Interval Length | Reasoning |
|-----------|-----------|-----------|
| /  | - | no previous Doing (beginning of domain life) | 
| &  | <200ms | Instantaneous |
| 0 | < 550 ms | Fast succession | 
| 1 | < 3s |  |
| 2 | < 2 min | | 
| 3 | < 2 hours | | 
| 4 | < 36 hours | | 
| 5 | < 7 days |  Covers typical create grace periods | 
| 6 | < 65 days |  Covers most redemption / quarantaine /  renew grace periods | 
| 7 | < 438 days | 1 year + 2 months - covers eg. auto-renew grace period | 
| 8 | < 3 years | | 
| 9 | < 7 years | | 
| + | >= 7 years | | 

Note that the three non-numerical characters used for specific interval cases. We believe that the choice of these characters allows us to extend the number of Interval buckets, and, to our experience, improves human readability (eg. "+" for "over 7 years" - see Examples below).

### Doings Descriptors

The range of Transactions performed on a domain name is vast, and greatly differs between registry operators, based on their local policy. It is therefore impossible to consider all of these transactions when mapping to ASCII characters, and we have considered the most relevant transaction types in the mapping: 

* Transactions which start or end the life of a Registration
* Transactions which interrupt the publication of delegation information in the DNS
* Major administrative and technical changes to the domain registration data

The Doings Descriptor mapping is as follows:

| Doings Descriptor | Doing | Example command/status | 
|----|---|---|
| R  | Registration | domain:create | 
| T | Transfer | domain:transfer | 
| N | Nameserver Change | domain:update |
| D | Delete | domain:delete, expiration | 
| L | Lock | serverHold, clientHold set |
| U | Restore (Unlock) | domain:restore, (server|client)Hold unset |
| P | Purge | Final removal of registration, eg. after quarantaine (Domain falls back into registration pool) | 
| S | DNSSEC Change | domain:update | 
| C | Registrant Change | domain:udate | 
| > | Now | - | 

Most notably, we have not included any renewal transactions in this mapping, but the outcome of not renewing, or active cancellation of a registration are mapped to the *D* Doing. 

The Doings Descriptor contains one special purpose character, namely *>*. This Doing does not refer to a transaction, but to 'Now', and allows to represent the *Interval* between the last *Doing* and the current point in time. 

### String Construction - Full DomainDNA

Constructing a Full DomainDNA from the full transaction information of a domain name is performed as follows:

1) Acquire the required data (sequence of transaction types, and the time intervals between the transactions) in chronological order.
2) Map each transaction interval and transaction type to the respective *Interval Descriptor* and *Doings Descriptor(s)*. Note that a single transaction type could potentially create multiple *Doings*. If that is the case, these *Doings* must be seperated by a '&' interval descriptor (eg. domain:update -> *N*-*&*-*C*-*&*-*S*).
3) Create a strict, repetitive, chronological sequence starting with exactly one *Interval Descriptor*, followed by exactly one *Doings Descriptor*, repeating until the sequence of descriptor data is exhausted. For the first *Doing* (typically an *R*) the preceding *Interval* is undefined, and  the descriptor */* must be used in this case.
4) At the end of the data sequence, calculate the interval between the last transaction time and the current point in time, map this to the respective *Interval Descriptor*, and append both this and the special '>' *Doing Descriptor* to the end of the string.

TODO: "presentation form" of DomainDNA

### Substrings of Full DomainDNA Strings

As described above, a *Full DomainDNA* (or simply *DomainDNA*) contains the whole set of information from the first time the domain name was registered to the current point in time, even if the domain was re-registered. For a variety of reasons, substrings of that *Full DomainDNA* might be useful, for example:

* Focusing on a single registration rather than the full domain name life.
* Lack of data, eg. very old registrations that precede current registry systems (or even organisation)
* Presenting only a subset of information to domain holders, customer service agents, etc.
* Preparing the data for use in machine learning models

We define the following terminology for the various cases of substrings:

TODO Example (same as above?)

* **Registration DomainDNA** or **Delegation DomainDNA** is defined as the string reflecting the transactions of a single registration of a domain (from Registration to Now or Purge). This includes the *Interval Descriptor* preceding the Registration. 
* **DomainDNA Word** is defined as the two-character sequence of one *Interval Descriptor* followed by one *Doings Descriptor* (eg. '6C').
* **DomainDNA Sequence** is defined as the sequence of at least two *DomainDNA Words*, in the order appearing in the *Full DomainDNA* (eg. '6C&N9>').
* **DomainDNA Fragment** is defined as any substring of a valid *Full DomainDNA*, even violating the *Word* structure (uneven length). (eg. 'C&N').

(Note that because of these definitions, a *Sequence* is always a *Fragment*, as is a *Word* and a *Registration*, but not all *Fragments* are *Sequences*. Or so... )

Also note that the choice of disjoint sets of ASCII characters for *Intervals* and *Doings* allows to clearly distinguish them from each other, and hence understand whether a *Fragment* starts with a *Doing* or an *Interval*

### Presentation Format

*DomainDNA* strings are hard to read, especially when their length exceeds a few *Words*. For readability, we define the ':' character as a visual seperator to be defined between (and only between!) *DomainDNA Words*. TWe anticipate this makes looking at longer *DomainDNA*'s, less daunting, as seasoned internet afficinados are well acquainted to that character's role as a seperator (see URIs, IPv6 addresses).

The seperator ':' must not be used at the start or end of a *DomainDNA* string, and its use should be limited strictly to presentation to humans.

A space (' ') character may also be used as seperator, however, this induces the inherent risk that such a string might be perceived as a set of disjunct *DomainDNA Words*. Therefore, the ':' is the preferred seperator.

## Examples

(Welcome to our new readers who skipped the Specification section. You do review Internet Drafts and RFCs, do you? ;) )

This section contains a list of examples with a brief description of each case.

1) `/R7C2N8T9D5P2R4>` is a *Full DomainDNA* of a domain name that was registered twice (see the two `R`s). In the first registration (note the `/R`) Owner and Nameserver were updated in quick succession (see `C2N` *Fragment*) it was transferred (see *Doings Descriptor* `T`) after / before long periods of time (see  *Fragment* `8T9`). Once deleted, it went through some redemption period (see *Fragment* `D5P`). The subsequent second registration was a reasonably quick drop catch (See  *Fragment* `P2R` - less than 2 minutes). The second registration is pretty fresh (See  *Fragment* `R4>' - less than 36 hours ago). 
2) `/R:7C:2N:8T:9D:5P:2R:4>` is the identical example as 1), but contains the `:` seperator between *Words*, purely for readability.
3) `7R4N&S6N+>` must be a *Delegation DomainDNA* as there is an implicit *Interval Descriptor* before the registration (See *Word* `7R`). There was some DNSSEC activity/changes, at the same time nameserver information was changed (See `N&S` *Fragment*). After a while, the nameservers were changed again, and from there on, no transactions were done for over 7 years until now (See *Sequence* `6N+>`).
4) `3R2N&C5L3P` is a *Sequence* (Note the missing `>` now *Doing* at the end, which would make it a *Delegation DomainDNA*). It reveals that this was a dropcatch, was quicky changed after registration, and - for whatever reason - was pretty quickly `L` locked and `P` purged very shortly thereafter.
5) `grep 'P[&12]R' domaindna-list.txt` assumes a list of *DomainDNA* information in a file, and uses `grep` to find all very fast drop catches by pattern.
6) `6N4SN5D` is an invalid *Fragment* as two *Doing Descriptors* can never occur without an *Interval Descriptor* in between (See `SN`)

## TODO 

This section lists open issues of this document

### Specs

- Describe Status assumptions depending on Doings
- Sort order in & Interval groups
- discuss data quality issues in detail 
- Does Terminology make sense?

## For Further Study

- would it make sense to allow a new, special interval descriptor for unknown intervals?  (eg. "yeah this was registered before but we don't have the data" or "we have no idea when this was entered into the zone")... maybe `?` 
