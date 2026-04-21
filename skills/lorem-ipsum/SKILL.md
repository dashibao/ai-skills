---
name: lorem-ipsum
description: Generate filler text for long context testing. Specify token count as argument (e.g., /lorem-ipsum 50000). Outputs approximately the requested number of tokens.
---

# Lorem Ipsum: Filler Text Generator

Generate filler text for testing long context scenarios.

## Usage

```
/lorem-ipsum [token_count]
```

Default: 10,000 tokens. Maximum: 500,000 tokens.

## Examples

```
/lorem-ipsum            # Generate ~10,000 tokens
/lorem-ipsum 50000      # Generate ~50,000 tokens
/lorem-ipsum 200000     # Generate ~200,000 tokens
```

## Output Format

The output consists of common English words arranged in sentences and paragraphs. It's designed to:
- Be approximately the requested token count
- Test context window limits
- Benchmark token handling

## Technical Details

- Uses verified 1-token English words
- Sentences are 10-20 words
- Paragraph breaks occur roughly every 5-8 sentences
- Maximum cap is 500,000 tokens for safety

## Common Use Cases

- Testing context window limits
- Benchmarking model performance with long inputs
- Testing token counting accuracy
- Verifying context compression behavior