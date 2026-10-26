# Agent instructions

Read [the specification](docs/specification.md) in full before implementation. It is the canonical owner of scope, language semantics, architecture, and acceptance criteria.

- Build an independent TypeScript library. Do not import code, contracts, configuration, credentials, or data from a host product.
- Implement the query prototype only. Workflow execution, `watch`, AI interpretation, SQL execution, and a separate visual query builder are outside the first release.
- Keep parser, semantic analysis, language service, editor, and host adapters separate. Core packages must run without React, a DOM, a database, or network access.
- Preserve stable identities through edits, serialization, copy/paste, reopening, and renamed display labels. Never silently resolve ambiguous labels or replace missing references by another item with the same name.
- Hosts supply catalogs, reference providers, execution, authorization, and storage. Do not hardcode HTTP, IndexedDB, tenant scopes, or collection names.
- Test the semantic and interaction acceptance criteria in the specification, including stale asynchronous responses and failure cases. Do not substitute snapshots for identity or query-equivalence tests.
- Treat the repository as public. Use synthetic fixtures only; never copy proprietary application source or production data.
- Keep durable design changes in the canonical specification and keep the root README short. State any proposed semantic change explicitly before implementing it.
- You may not be alone in the repository. Preserve unrelated edits and inspect the working tree before making changes.

The owner has authorized implementation planning through this repository's specification; the initial repository setup itself does not implement the prototype. Begin future implementation work with the host-independent core and an end-to-end vertical slice, then cover the remaining acceptance cases. Report what works, what was tested, and any remaining gaps.
