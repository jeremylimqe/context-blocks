# Context Blocks

A minimal representation for independently identifiable context blocks.

**Status:** Experimental. The format may change before v1.0.0.

## Why Context Blocks

Context engineering determines what information is available to a model and when. Context Blocks defines how selected text is represented.

Context Blocks provides a minimal, source-independent format for text used across **agentic systems**: independently identifiable units with explicit boundaries and content preserved as supplied.

Judgments about selection and composition remain outside the format. Context Blocks represents the resulting textual units in a predictable form that can be inspected, exchanged, reproduced, and referenced.

## Format

Each context block consists of exactly two values: an identifier and text. The format defines explicit block boundaries without assigning meaning to  the text.

```text
@@ctx project-overview
A minimal format for identifiable context blocks.
@@ctx.end project-overview
```

See [SPEC.md](SPEC.md) for the format definition and [conformance.json](conformance.json) for concrete conformance cases.


## License

[MIT](LICENSE).