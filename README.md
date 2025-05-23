# Shaperglot - Test font files for OpenType language support

[![PyPI Version](https://img.shields.io/pypi/v/shaperglot.svg)](https://pypi.org/project/shaperglot)
[![PyPI License](https://img.shields.io/pypi/l/shaperglot.svg)](https://pypi.org/project/shaperglot)
[![Read The Docs](https://readthedocs.org/projects/shaperglot/badge/)](https://https://shaperglot.readthedocs.io/en/latest/)

Try [Shaperglot on the web](https://googlefonts.github.io/shaperglot)!

Shaperglot is a library and a utility for testing a font's language support.
You give it a font, and it tells you what languages are supported and to what
degree.

Most other libraries to check for language support (for example, Rosetta's
wonderful [hyperglot](https://hyperglot.rosettatype.com) library) do this by
looking at the Unicode codepoints that the font supports. Shaperglot takes
a different approach.

## What's wrong with the Unicode codepoint coverage approach?

For many common languages, it's possible to check that the language is
supported just by looking at the Unicode coverage. For example, to support
English, you need the 26 lowercase and uppercase letters of the Latin alphabet.

However, for the majority of scripts around the world, covering the codepoints
needed is not enough to say that a font really _supports_ a particular language.
For correct language support, the font must also _behave_ in a particular way.

Take the case of Arabic as an example. A font might contain glyphs which cover
all the codepoints in the Arabic block (0x600-0x6FF). But the font only _supports_
Arabic if it implements joining rules for the `init`, `medi` and `fina` features.
To say that a font supports Devanagari, it needs to implement conjuncts (which
set of conjuncts need to be included before we can say the font "supports"
Devanagari is debated...) and half forms, as well as contain a `languagesystem`
statement which triggers Indic reordering.

Even within the Latin script, a font only supports a language such as Turkish
if its casing behaving respects the dotless / dotted I distinction; a font
only supports Navajo if its ogonek anchoring is different to the anchoring used in
Polish; and so on.

But there's a further problem with testing language support by codepoint coverage:
it encourages designers to "fill in the blanks" to get to support, rather than
necessarily engage with the textual requirements of particular languages.

## Testing for behaviour, not coverage

Shaperglot therefore determines language support not just on codepoint coverage,
but also by examining how the font behaves when confronted with certain character
sequences.

The trick is to do this in a way which is not prescriptive. We know that there
are many different ways of implementing language support within a font, and that
design and other considerations will factor into precisely how a font is
constructed. Shaperglot presents the font with different strings, and makes sure
that "something interesting happened" - without necessarily specifying what.

In the case of Arabic, we need to know that the `init` feature is present, and that
when we shape some Arabic glyphs, the output with `init` turned on is different
to the output with `init` turned off. We don't care what's different; we only
care that something has happened. _(Yes, this still makes it possible to trick shaperglot into reporting support for a language which is not correctly implemented, but at that point, it's probably less effort to actually implement it...)_

Shaperglot includes the following kinds of test:

- Certain codepoints were mapped to base or mark glyphs.
- A named feature was present.
- A named feature changed the output glyphs.
- A mark glyph was attached to a base glyph or composed into a precomposed glyph (but not left unattached).
- Certain glyphs in the output were different to one another.
- Languagesystems were defined in the font.
- ...

## Using Shaperglot

Shaperglot consists of multiple components:

### Shaperglot Web interface

The easiest way to use Shaperglot as an end-user or font developer is through the
[web interface](https://googlefonts.github.io/shaperglot). This allows you to drag
and drop a font to analyze its language coverage. This is entirely client-side,
and all fonts remain on your computer. Nothing is uploaded.

### Shaperglot command line tools

The next most user-friendly way to use Shaperglot is at the command line. You can
install the latest version with:

    cargo install --git https://github.com/googlefonts/shaperglot

This will provide you with a new tool called `shaperglot`. It has four subcommands:

- `shaperglot check <font> <language> <language>...` checks whether a font supports the given language IDs.
- `shaperglot report <font>` reports all languages supported by the font.
- `shaperglot describe <language>` explains what needs to be done for a font to supportt a given language ID.

```
$ shaperglot describe Nuer
The font MUST support the following Nuer bases and marks: 'a', 'A', 'ä', 'Ä', 'a̱', 'A̱', 'b', 'B', 'c', 'C', 'd', 'D', 'e', 'E', 'ë', 'Ë', 'e̱', 'E̱', 'ɛ', 'Ɛ', 'ɛ̈', 'Ɛ̈', 'ɛ̱', 'Ɛ̱', 'ɛ̱̈', 'Ɛ̱̈', 'f', 'F', 'g', 'G', 'ɣ', 'Ɣ', 'h', 'H', 'i', 'I', 'ï', 'Ï', 'i̱', 'I̱', 'j', 'J', 'k', 'K', 'l', 'L', 'm', 'M', 'n', 'N', 'ŋ', 'Ŋ', 'o', 'O', 'ö', 'Ö', 'o̱', 'O̱', 'ɔ', 'Ɔ', 'ɔ̈', 'Ɔ̈', 'ɔ̱', 'Ɔ̱', 'p', 'P', 'q', 'Q', 'r', 'R', 's', 'S', 't', 'T', 'u', 'U', 'v', 'V', 'w', 'W', 'x', 'X', 'y', 'Y', 'z', 'Z', '◌̈', '◌̱'
The font SHOULD support the following auxiliary orthography codepoints: 'ʈ', 'Ʈ'
Latin letters should form small caps when the smcp feature is enabled
```

### Shaperglot Rust library

See the documentation on https://docs.rs/shaperglot/latest

### Shaperglot Python library

The Python library wraps the Rust library using PyO3. This new PyO3 implementation
_broadly_ follows the same API as the original 0.x Python implementation, but all
imports are at the top level (`from shaperglot import Checker`, etc.) The PyO3
version is available as a pre-release from Pypi.

Python Library Documentation: https://shaperglot.readthedocs.io/en/latest/
