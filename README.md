# Context Blocks

A minimal representation for independently identifiable context blocks.

**Status:** Draft specification. No stable release has been published.

Each context block consists of exactly two values: an identifier and text.
The format defines explicit block boundaries without assigning meaning
to the text.

```text
@@ctx project-overview
A minimal format for identifiable context blocks.
@@ctx.end project-overview
```

See [SPEC.md](SPEC.md) for the data model and canonical UTF-8 serialization.


## License

[MIT](LICENSE).