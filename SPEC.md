# Context Blocks Specification

**Version:** v0.1.0
**Status:** Experimental

This specification defines the core data model, representability requirements, exact UTF-8 byte framing, and serialization conformance contract of an individual block.

The words **MUST** and **MUST NOT** express mandatory requirements of this specification.

## 1. Purpose

Context Blocks defines a minimal representation for independently identifiable textual content. A context block is the unit of representation defined by this specification.

This specification defines an individual block, not a document or artifact container. How blocks are selected, ordered, or composed is outside its scope.

## 2. Context block

A context block consists of exactly two semantic values: `id` and `text`.

### Identifier

`id` is the identifier supplied by the producer for the block. Context Blocks does not interpret its contents as a path, namespace, or other source-specific syntax.

An identifier:

- MUST be non-empty.
- MUST be valid UTF-8.
- MUST NOT contain control characters.
- MUST NOT contain U+2028 or U+2029.
- MUST NOT begin or end with a whitespace character.

For identifier validation:

- A **control character** is any Unicode code point in the ranges U+0000–U+001F or U+007F–U+009F.
- A **whitespace character** is any Unicode code point in U+0009–U+000D, U+0020, U+0085, U+00A0, U+1680, U+2000–U+200A, U+2028, U+2029, U+202F, U+205F, or U+3000.

All ranges are inclusive. These definitions apply to Unicode code points after UTF-8 decoding, not to UTF-8 byte values.

Context Blocks imposes no additional identifier syntax.

Identifier equality MUST be determined solely by byte-for-byte comparison of the supplied UTF-8 byte sequences.

A producer, serializer, or decoder MUST reject a block whose `id` violates these validity requirements.

### Text

`text` is UTF-8 text and may be empty.

Context Blocks assigns no intrinsic meaning to `text`. The identifier restrictions on control characters, U+2028 and U+2029, and leading or trailing whitespace characters do not apply to `text`.

## 3. Preservation

Serialization and decoding MUST preserve both supplied values byte-for-byte in UTF-8. They MUST NOT trim, normalize, summarize, or otherwise transform either value. This includes Unicode normalization and newline normalization.

Invalid UTF-8 MUST be rejected rather than replaced or repaired.

## 4. Framing

A block is framed by an opening delimiter and a matching closing delimiter.

### 4.1. Byte notation and delimiter lines

In the expressions below, quoted literals denote their ASCII byte sequences, `+` denotes byte concatenation, and:

- `SP` is the single byte `0x20`.
- `LF` is the single byte `0x0A`.
- `I` is the supplied identifier's UTF-8 byte sequence.
- `T` is the supplied text's UTF-8 byte sequence.

The delimiter line contents are:

```text
opening = "@@ctx"     + SP + I
closing = "@@ctx.end" + SP + I
```

These expressions exclude line terminators. The framing tokens are case-sensitive.

For framing and representability checks, lines are separated only by `LF`. A line's contents exclude the terminating `LF`, but include any preceding carriage return (`CR`, byte `0x0D`). Neither `CR` nor Unicode line-separator characters are framing separators.

Delimiters MUST occur on their own lines. All content after the framing token and its separating space belongs to the identifier. There are no additional delimiter fields.

The identifier in the closing delimiter MUST match the identifier in the opening delimiter byte-for-byte.

### 4.2. Exact byte layout

A serializer MUST encode a representable block as exactly:

```text
encoded_block = opening + LF + T + LF + closing + LF
```

The three explicit `LF` occurrences, from left to right, are:

- **Opening LF:** terminates the opening delimiter line.
- **Separator LF:** separates `T` from the closing delimiter line.
- **Closing LF:** terminates the closing delimiter line.

Each is a single byte `0x0A`. All three are mandatory framing bytes at separate byte positions; none belongs to `T`.

The separator LF MUST be emitted even when `T` is empty or already ends in `LF`.

A serializer MUST NOT replace a framing `LF` with `CRLF` or add a byte order mark before the opening delimiter. These restrictions do not remove or modify any bytes supplied in `T`.

### 4.3. Recovering the text

Context Blocks does not recognize nested blocks. Starting immediately after the opening LF, a decoder MUST stop at the first line whose contents match that block's closing delimiter byte-for-byte. It MUST NOT skip an exact match to seek a later closing delimiter.

Except for the separator LF removed below, every byte after the opening LF and before the matching closing delimiter MUST be treated as text. This includes lines resembling opening delimiters or closing delimiters with a different identifier. Such lines MUST NOT be interpreted as nested blocks or rejected as delimiter mismatches.

The matching closing delimiter MUST be terminated by the closing LF. The byte sequence after the opening LF and before the first byte of the matching closing delimiter MUST contain at least one byte and MUST end in `LF`.

A decoder MUST remove exactly the final byte—the separator LF—from that byte sequence to recover `T`. It MUST preserve all other bytes, including any trailing `LF` or `CR` supplied as text. The opening LF MUST NOT also serve as the separator LF.

A block missing a required framing `LF` or a complete matching closing delimiter MUST be rejected.

### 4.4. Byte examples

The quoted values below use escaped notation: `\n` denotes byte `0x0A` and `\r` denotes byte `0x0D`. The surrounding quotes are not serialized. Every example uses `id = "a"`.

| Supplied `text` | Encoded block bytes |
| --- | --- |
| `""` | `"@@ctx a\n\n@@ctx.end a\n"` |
| `"hello"` | `"@@ctx a\nhello\n@@ctx.end a\n"` |
| `"hello\n"` | `"@@ctx a\nhello\n\n@@ctx.end a\n"` |
| `"hello\n\n"` | `"@@ctx a\nhello\n\n\n@@ctx.end a\n"` |
| `"\n"` | `"@@ctx a\n\n\n@@ctx.end a\n"` |
| `"hello\r\n"` | `"@@ctx a\nhello\r\n\n@@ctx.end a\n"` |
| `"hello\r"` | `"@@ctx a\nhello\r\n@@ctx.end a\n"` |

In the last example, the adjacent `CR` and `LF` have different ownership: `CR` belongs to `text`, while `LF` is the separator LF.

`"@@ctx a\n@@ctx.end a\n"` is not a valid encoding of empty text: it contains an opening LF and a closing LF, but no separator LF.

## 5. Representability

For a block to be representable, all of the following requirements MUST be satisfied:

- its `id` satisfies §2;
- its `text` is valid UTF-8; and
- in `T + LF`, where `LF` is the separator LF defined in §4.2, no line has contents equal byte-for-byte to that block's `closing` delimiter. Lines are defined as in §4.1.

Context Blocks defines no escaping mechanism.

A producer or serializer MUST reject an unrepresentable block. It MUST NOT modify either supplied value to avoid a delimiter collision or otherwise make the block representable.

## 6. Serialization conformance

The accompanying [conformance.json](conformance.json) supplies concrete serialization cases for this version of the specification. Each case supplies the values for one block and its expected byte output or rejection.

The file is a JSON array of independent cases. This array organizes test cases only; it does not define a Context Blocks collection or composition format.

### 6.1. Case format

Each case contains the following fields:

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | String | The supplied block identifier. |
| `text` | String | The supplied block text. |
| `expected` | String or `null` | The exact expected serialized output, or rejection. |
| `description` | Optional string | An informative explanation of the case. |

The fixture file is UTF-8 JSON. JSON escaping is used only to represent test values in the fixture and is interpreted once when the JSON is parsed; it is not part of Context Blocks serialization.

After parsing, the UTF-8 encodings of `id` and `text` are the values supplied to Context Blocks. For example, `"\n"` supplies one LF byte, while `"\\n"` supplies a backslash followed by `n`.

### 6.2. Expected results

When `expected` is a string, its UTF-8 encoding is the complete expected byte output for the supplied block. Serialization MUST produce exactly those bytes. No byte order mark, newline, or other bytes are implicitly added to the expected output.

Actual and expected output MUST be compared byte-for-byte, without trimming, normalization, or other transformations before comparison.

When `expected` is `null`, serialization MUST reject the supplied block as required by §5. Here, `null` denotes the expected rejection outcome, not an API return value, empty output, or serialized bytes. This contract does not prescribe an exception type or diagnostic message. 

### 6.3. Authority and coverage

The conformance cases are authoritative for the outcomes they define. Any disagreement between a case and the specification is a defect to resolve.

This JSON-string corpus does not supply malformed UTF-8 byte sequences. Their rejection remains required by §3.