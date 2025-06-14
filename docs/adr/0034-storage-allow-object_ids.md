---
status: "accepted"
---
# Add support for specifying object_ids when calling the storage endpoint

## Context and Problem Statement

Currently an implementation of the TAMS API will need to generate the 'object_id' for a flow segment to use, there is no way the client can control the 'object_id' generated.
Some use cases have come up with TAMS that required a need to "bring your own" `object_id`.
The main use case is around TAMS Store replication.
If a store is going to replicate content from another store without modification it makes sense to re-use the ids in use in the source store.
Sources and Flows already allow for this but `object_id`s do not.

Currently the nearest option to allow this use case involves performing operations to the backend TAMS infrastructure outside of the API as such is not a viable option.  

## Considered Options

- Option 1: Do nothing
- Option 2: Change the validation on the segments endpoint
- Option 3: Add capability to the storage endpoint to allow `object_id`s to be supplied

## Decision Outcome

Chosen option: Option 3, because doing nothing does not solve the problem when replicating objects between stores.
Changing validation will cause complication and confusion.

### Implementation

See the API specification changes in PR [#132](https://github.com/bbc/tams/pull/132).

## Pros and Cons of the Options

### Option 1: Do nothing

Make no changes, replication use cases should use new `object_id`s.
The combination of the Flow Id and Timerange being the same as the source store will have to suffice.

- Good: No changes to the API spec required
- Good: Object Validation remains robust
- Bad: Feels like it is not addressing the need for replication, since different `object_id`s may imply different content.

### Option 2: Change the validation on the segments endpoint

Update the validation on the POST segments endpoint.
Current validation enforces either the supplied `object_id` must be known (issued via a POST to the storage API) or if unknown a get_urls value is required.
One option could be to do this by requiring the client to supply a "special" get_url so that it can be recognised as a special case.

- Good: No breaking changes would be required in the API Spec.
- Bad: Potential to be abused, it would not be able to enforce the use case of replication it might be used in other cases where `object_id`s should be validated
- Bad: The objects endpoint would not be able to determine the `first_referenced_by_flow` field.
- Bad: "Special" get_url feels like an abuse of the field.

### Option 3: Add capability to the storage endpoint to allow `object_id`s to be supplied

Add the ability for the client to supply `object_id`(s) when making a call to the storage endpoint.
If supplied the `object_id` should be checked to determine if it already exists and if so, return a bad request, else it should behave in exactly the same as it currently does except use the provided `object_id`(s).
The supplying of this new `object_ids` will mean the existing `limit` field would be redundant.
Therefore either, `limit` OR `object_ids` would be supported.
The `object_ids` would be a list of string to allow multiple requests to be made similar to how a `limit` value of greater than 1 behaves currently.

- Good - Provides the required functionality
- Good - Backwards compatible
- Good - Retains all existing `object_ids` validation
- Bad - API spec change required.

## More Information

Note: [0017-reuse-of-ids.md](https://github.com/bbc/tams/blob/main/docs/appnotes/0017-reuse-of-ids.md) contains some dicussion that may be relevant to this proposal.
