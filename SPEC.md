# Context Blocks Specification

**Status:** Draft

This draft defines the core data model and framing structure. It is not yet a complete byte-level serialization or conformance specification.

The words **MUST** and **MUST NOT** express mandatory requirements of this draft.

## 1. Purpose

Context Blocks is a minimal representation for dividing context into stable, independently identifiable blocks.

## 2. Context block

A context block consists of exactly two semantic values: `id` and `text`.

### Identifier

`id` is the stable identifier supplied by the producer for the block.

An identifier:

- MUST be non-empty.
- MUST be valid UTF-8.
- MUST occupy exactly one line.
- MUST NOT contain control characters.
- MUST NOT begin or end with whitespace.

### Text

`text` is UTF-8 text and may be empty.

Context Blocks assigns no intrinsic meaning to its contents. Identifier character restrictions do not apply to text.

## 3. Preservation

Serialization and decoding MUST preserve both supplied values byte-for-byte in UTF-8. They MUST NOT trim, normalize, summarize, or otherwise transform either value. This includes Unicode normalization and newline normalization.

Invalid UTF-8 MUST be rejected rather than replaced or repaired.

## 4. Framing

A block is framed by an opening delimiter and a matching closing delimiter.

The delimiter line contents are:

```text
opening = "@@ctx"     + SP + id
closing = "@@ctx.end" + SP + id
```

Here, `SP` is exactly one ASCII space (U+0020), and `id` is the identifier's UTF-8 representation. These expressions exclude line terminators. The framing tokens are case-sensitive.

Delimiters MUST occur on their own lines. All content after the framing token and its separating space belongs to the identifier. There are no additional delimiter fields.

The identifier in the closing delimiter MUST match the identifier in the opening delimiter byte-for-byte.

The block has this conceptual structure, not an exact byte layout:

```text
@@ctx <id>
<text>
@@ctx.end <id>
```

Context Blocks does not recognize nested blocks. Within an open block, a decoder MUST end the block at the first line whose contents match that block's closing delimiter byte-for-byte.

Every other line MUST be treated as text, including lines resembling opening delimiters or closing delimiters with a different identifier. Such lines MUST NOT be interpreted as nested blocks or rejected as delimiter mismatches.

## 5. Representability

Text MUST NOT contain a line exactly equal to the closing delimiter of its containing block. This restriction includes a final line without a line terminator.

Context Blocks defines no escaping mechanism.

A producer MUST reject an `id` or `text` value that cannot be represented without violating these requirements. A serializer MUST NOT change either supplied value to avoid a delimiter collision or otherwise make the block representable.
