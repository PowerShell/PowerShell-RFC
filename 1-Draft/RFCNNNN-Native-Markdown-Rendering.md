---
RFC: 
Author: Aditya Patwardhan
Status: Draft
SupercededBy: 
Version: 0.1
Area: Formatting and Output
Comments Due: 03/26/2018
Plan to implement: Yes
---

# Native Markdown Rendering

Render markdown content on console to improve readability. 

## Motivation

Markdown is a common authoring format used by the community.
There is no easy way in PowerShell to visualize a markdown document on console.

## Specification

This RFC proposes to use VT100 escape sequences to render markdown content.
`ConvertFrom-Markdown` cmdlet as part of `Microsoft.PowerShell.MarkdownRender` PowerShell module would consume a string or a file path and output a VT100 encoded string.
Optionally, the output can be formatted as HTML.
For converting strings to VT100 coded strings, we will be writing an extension to [markdig](https://github.com/lunet-io/markdig).
The extension will insert VT100 escape sequences as appropriate.

```PowerShell
ConvertFrom-Markdown [-Path] <string[]> -AsHTML -AsPlainText  [<CommonParameters>]

ConvertFrom-Markdown -LiteralPath <string[]> -AsHTML -AsPlainText  [<CommonParameters>]

ConvertFrom-Markdown -InputObject <psobject> -AsHTML -AsPlainText  [<CommonParameters>]
```

### Parameters

- **Path** : Accepts an array of file paths with markdown content.
- **LiteralPath** : Accepts an array of literal paths with markdown content.
- **InputObject** : Accepts an InputObject of `System.IO.FileSystemInfo`, `string` type.
- **AsHTML** : When selected, the output will be HTML.
- **AsPlainText** : When selected, the output will be plain text.

The default output will be a string encoded with VT100 escape sequences.

## Supported Markdown Elements

We will be supporting a limited set of markdown elements in the initial version.

### Block Elements

The block elements include paragraphs and line breaks, headers, block quotes, code blocks and horizontal rules.

#### Paragraphs and line breaks

These will be rendered as plain text.

#### Headers

VT100 escape sequences for headers are as follows:

| Element |  Markdown | Rendered | Escape Sequences |
|---------|-------------|----------|----------|
| ATX Header 1 | ![](../assets/MarkdownRendering/Header1-MD.png) | ![](../assets/MarkdownRendering/Header1.png) | ESC[7mHeader 1ESC[0m |
| ATX Header 2 | ![](../assets/MarkdownRendering/Header2-MD.png) | ![](../assets/MarkdownRendering/Header2.png) | ESC[4;93mHeader 1ESC[0m |
| ATX Header 3 | ![](../assets/MarkdownRendering/Header3-MD.png) | ![](../assets/MarkdownRendering/Header3.png) | ESC[4;94mHeader 1ESC[0m |
| ATX Header 4 | ![](../assets/MarkdownRendering/Header4-MD.png) | ![](../assets/MarkdownRendering/Header4.png) | ESC[4;95mHeader 1ESC[0m |
| ATX Header 5 | ![](../assets/MarkdownRendering/Header5-MD.png) | ![](../assets/MarkdownRendering/Header5.png) | ESC[4;96mHeader 1ESC[0m |
| ATX Header 6 | ![](../assets/MarkdownRendering/Header6-MD.png) | ![](../assets/MarkdownRendering/Header6.png) | ESC[4;97mHeader 1ESC[0m |
| Setext Header 1 | ![](../assets/MarkdownRendering/SetextHeader1-MD.png) | ![](../assets/MarkdownRendering/SetextHeader1.png) | ESC[4;92mHeader 1ESC[0m |
| Setext Header 2 | ![](../assets/MarkdownRendering/SetextHeader2-MD.png) | ![](../assets/MarkdownRendering/SetextHeader2.png) | ESC[7mHeader 1ESC[0m |

#### Code Blocks

Code blocks will be rendered with a lighter background color and a dark foreground text.
Background color will be applied for the console width.
No syntax highlighting will be done.
`ESC[500@` is used to have sufficiently wide background color.

| Markdown | Rendered | Escape Sequences |
|-------------|----------|----------|
| ![](../assets/MarkdownRendering/CodeBlock-MD.png) | ![](../assets/MarkdownRendering/CodeBlock.png) | ESC[48;2;155;155;155;38;2;30;30;30mTextESC[500@ESC[0m |

### Span Elements

Span elements will have varying degree of support depending on element type.

#### Links

Links in paragraphs will keep the link label and append the link URL surrounded by parenthesis.
The link will be colored to differentiate from plain text.

| Markdown | Rendered | Escape Sequences |
|----------|----------|-------------|
| ![](../assets/MarkdownRendering/Link-MD.png) | ![](../assets/MarkdownRendering/Link.png) | ESC[4;34m( )ESC[0m |

If the links are part of a container like a pipe table, block quote or list; then just the link label will be shown.

#### Emphasis

Emphasis will be rendered using escape sequence `ESC[1mBold TextESC[0m` and `ESC[36mItalics TextESC[0m` for bold text and italics text respectively.

| Element | Markdown | Rendered | Escape Sequences |
|---------|-------------|----------|----------|
| Bold | ![](../assets/MarkdownRendering/Bold-MD.png) | ![](../assets/MarkdownRendering/Bold.png) | ESC[1mBold TextESC[0m |
| Italics | ![](../assets/MarkdownRendering/Italics-MD.png) | ![](../assets/MarkdownRendering/Italics.png)| ESC[36mItalics TextESC[0m |

#### Code

Inline code elements will be rendered similar to code blocks, with the exception on background color to not extend beyond the markdown element.

|  Markdown | Rendered | Escape Sequences |
|-------------|----------|----------|
| ![](../assets/MarkdownRendering/Code-MD.png) | ![](../assets/MarkdownRendering/Code.png) | ESC[48;2;155;155;155;38;2;30;30;30mTextESC[0m`|

#### Images

The alt-text of the image will be surrounded by `[` and `]` and colorized to differentiate from plain text.
Escape code for images alt-text

| Markdown | Rendered | Escape Sequences |
|-------------|----------|----------|
| ![](../assets/MarkdownRendering/Image-MD.png) | ![](../assets/MarkdownRendering/Image.png) | ESC[33m[alt-text]ESC[0m |

## Future Work

`Paragraphs and Line Breaks`, `Block Quotes`, `Lists`, `Horizontal Rules`, `Pipe tables` and `HTML code` will be considered for future versions and rendered as plain text in this version.

## Alternate Proposals and Considerations

Introduce a new sigil to define a markdown string.
This would be later rendered using formating files.
The limitations of this approach are:

1. Can be used only in script.
2. Adds a new dependency of `markdig` to `System.Management.Automation`.

## Open Questions

- Investigate line wrapping behavior.