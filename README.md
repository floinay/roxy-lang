# Roxy

An independent TypeScript language for expressing queries as readable sentences, with schema-aware completion and stable references behind human-readable names.

```text
Get first 20 articles from Articles
where Articles tags includes [News]
order by Articles publication_date desc.
```

This repository currently contains the specification for a hypothesis-validation prototype. No language runtime or editor is implemented yet. The display example above is proposed Roxy syntax, not an executable query today.

- [Documentation](docs/index.md)
- [Implementation specification and acceptance criteria](docs/specification.md)
- [Agent instructions](AGENTS.md)

The first dialect is queries. The architecture reserves room for conditions, workflows, and other dialects without implementing them in this prototype. Package names are proposed; nothing has been published to npm.
