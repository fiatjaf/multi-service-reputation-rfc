# Multi Service Reputation

A protocol for identity and reputation sharing and interpersonal trust transfer.

## Goal

Solve all problems with ratings inside websites once and for all.

## Protocol flow

This protocol defines 3 roles: `USER`, `SERVICE` and `INDEXER`.

### `SERVICE`

Each service, for example, microlancer.io, expose two HTTP endpoints:

  * `https://microlancer.io/.well-known/msr/users.json`

    This endpoint must return an array of arrays, each in the form
      ```
        [
          <internal_user_id> : String,
          <external_user_ref> : String
        ]
      ```
    For example, if microlancer.io has user identities tied to a Twitter account or sometimes an email address, this would look like
      ```
        [
          ["3274", "mr_sir@twitter.com"],
          ["1234", "someone@gmail.com"],
          ["3274", "mr.sir@gmail.com"],
          ...
        ]
      ```

  * `https://microlancer.io/.well-known/msr/ratings.json`

    This endpoint must return an array of arrays, each in the form
      ```
        [
          <internal_user_id_source> : String,
          <internal_user_id_target> : String,
          <tag> : String,
          <trust> : Int (between 0 and 5, use 3 if in doubt)
        ]
      ```
    `<tag>` must be at most 20 lowercase characters, without spaces, can be empty, and should probably be one of the tags already being used by other services, some of which are  suggested in this document.

    For example, if user 1234 is satisfied with a task done by user 3274, while at the same time very unsatisfied with a task done by 5512 this could look like
      ```
        [
          ["1234", "3274", "", 3],
          ["1234", "5512", "slow-to-deliver", 0],
          ...
        ]
      ```

`SERVICE` shouldn't expose all these details to its internal users. For example, microlancer.io, when requesting a user for a rating for a task, could display just two buttons: "satisfied" / "unsatisfied", or a 5 star rating, and/or a &lt;select&gt; with 3 or 4 prefixed `<tag>`s each corresponding automatically to a trust value.

### `INDEXER`

An `INDEXER` is a server that uses data from the endpoints served by `SERVICE`s to produce a coherent graph of trust and serve that back to services that want to use that information to guide their users internally.

It must discover `SERVICE` endpoints by itself or accept some form of registration (which also involves filtering out clearly bogus `SERVICE`s), fetch data regularly from these endpoints (not more than twice an hour, probably) and index it so it can serve HTTP requests at `/rep/<source_ref>/<target_ref>.json`.

`<source_ref>` and `<target_ref>` are Strings like `1234@microlancer.io` or `mr_sir@twitter.com`.

That should take the following form of an array of arrays, each corresponding to one rating of `<target_ref>` in relation to `<source_ref>`:

  ```
    [
      <tag> : String,
      <trust> : Int,
      <distance> : Int
    ]
  ```

`<distance>` being the measure of the distance between `<source_ref>` and the user who issued the rating. That should be `0` if `<source_ref>` itself issued the rating and could be 1, 2, 3 for each hop the indexer decides to index after the first level, or it could be a measure like `1 * 5 / <trust>` in each hop, that is a decision specific to `INDEXER`.

### `USER`

The user doesn't know anything about what is happening, except for the fact that each `SERVICE` should tell his users that his in-app ratings are public.

When using a service, for example, microlancer.io, the user may -- when browsing over the profile or hovering other user -- see the list of ratings (taken from a given indexer) from him to that user, and use that to decide to accept a task or offer his services to that other user.

The fetching of that information in this case could happen directly from the JavaScript client.
