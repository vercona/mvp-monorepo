[Fastify quick link](https://github.com/fastify/fastify)

At the moment, the main priority of the server is <u>the actual service</u>.

Long term we may consider ssr and hosting solutions:
- https://start.solidjs.com/getting-started/what-is-solidstart
- https://github.com/fastify/fastify-dx/tree/main/packages/fastify-solid
- https://github.com/OrJDev/create-jd-app

but thats not a priority atm.

as a note, im not opposed to using solids reactive core on the backend, I've found such things useful in the past.


file handling?
- https://github.com/OrJDev/solid-uploadthing
- https://github.com/pingdotgg/uploadthing/pull/41
- https://docs.uploadthing.com/backend-adapters/fastify
- https://docs.uploadthing.com/getting-started/solid


One of the architectural possibilities is that we use a versioned database to handle submissions and merging:
  - Example TerminusDB: here a [branch is created](https://terminusdb.com/docs/branch-a-project/) per submission and apply those diffs to the main branch when the vote is over.
    - [terminus overview](https://terminusdb.com/products/terminusdb/)
    - [js client](https://github.com/terminusdb/terminusdb-client-js)
  - another such database is Dolt: which also provides [versioning at the data layer](https://docs.dolthub.com/introduction/use-cases/vc-your-app)
    - [js example](https://github.com/dolthub/dolt-knexjs-example/blob/main/index.js)

Another alternative is to use SpacetimeDB as the versioning database such that a module provides a custom/tailored diff algorithm.
  - this is in fact quite promising as you can store raw diffs for a minimal memory footprint while also preserving their contextual nature that is inherent to the given data type. Consider versioning an AST vs Lines changed, its more human readable, contextual, and less prone to adjacent merge conflicts.
  - that being said, STDB wasn't really *intended* to be a [CRDT type system](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type), so it might be a bit of an uphill battle to implement it

you can still take a similar approach in the other two databases (in the server rather than module). but it seems a waste to roll your own versioning system on top of a versioned db rather than just making due with it.

tbd...
