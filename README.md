# Scriptdown

Scripdown, a manuscript authoring tool.

**This document is under draft status**

## Introduction

The name is inspired by the famous Markdown,
and the tool's job is similar to it, to convert plain text source files to formatted
file formats, e.g. HTML. The difference is that Markdown is tool written for web
writers to get rid of HTML boilerplates, but Scriptdown's goal is to help mostly
mainly tech or academic authors to write manuscript in structured plain text files.
The source file could have various contents other than text, such as tables, charts,
program codes, image etc. The source file could also contain structuring and
formatting  information so that Scriptdown renderer could automatically format the
output. In this sense, Scriptdown is an alternative to TeX system, but not actually.
TeX (also LaTeX) is focused on press-level layout and formatting, where the easiness
for writer is somehow compromised. In contrast, Scriptdown is a tool used to help
authors to write manuscript, so that writer friendliness is considered first than
layout beautifulness. And Scriptdown will not explicitly support complex and unusual
custom layouts. It focus on providing the authors a writing tool to express their
mental work instead.

Scriptdown is supposed to be a formal formatting language and also a toolset and
environment to support authoring in Scriptdown and to compile (render) Scriptdown
source codes to HTML, PDF etc. Scriptdown is supposed to be extendable, and modular
so that authors from different fields could easily add functionality to support
authoring in their problem domain. This project is supposed to provide the basic
architecture for the Scriptdown toolset environment, and a set of core modules.
Other functionalities and extensions can be later provided by different projects.

The Scriptdown runtime will be written in JavaScript and Java. In this sense,
Scriptdown is going to be a hybrid system, which core will be in JavaScript, with
supportive tools and plugins being in either JavaScript or Java. The reason
choosing these two language is their popularity and platform-independency. By
writing the core with JavaScript, Scriptdown should easily run in any modern
browser. But JavaScript has its limitations and it would be hard to compose complex
and solid system with it currently. Java is an old and popular language, which
provides good modularity, security, and has a very large community. The runtime of
Java, the JVM is also the runtime for several popular languages other than Java
itself. JRuby, Jython, Groovy, Scalar are good examples, and actually JavaScript
also has a few implementations on JVM. JVM also provides a standard and portable
way to interoperate with platform native codes through JNI, which is missing in
JavaScript. By choosing a hybrid solution with JavaScript and Java, distribution
and maintenance of Scriptdown is expected to be easier.

The syntax of Scriptdown language will be based on Markdown, as it provides an
intuitive syntax for structured document. But due to the fact that Scriptdown is
not supposed to be tied to HTML, HTML tags and entities are not valid, or
effective in Scriptdown. Scriptdown's current main target is technical and
academical authors, so the core modules should provide the following language
features:

- structural description (e.g. define chapters, sections etc)
- inline math expression
- inline data sheet
- inline table
- inline plot chart
- inline relationship chart
- external resource reference (images, audio etc..)
- attachment reference
- cross-reference
- bibliographical reference
- revision control integration

In the early stage, Scriptdown is supposed to be a mix of the following projects
(languages):

- Markdown
- LaTeX (for math expressions): [MathJax](http://www.mathjax.org/)
- DOT (relationship graphs): [Viz.js](https://github.com/mdaines/viz.js/)
- Gnuplot (plotted charts): [gnuplot-js](https://github.com/chhu/gnuplot-JS)
- Emacs Org-mode like Table

## The Scriptdown Language

The Scriptdown language is very simple but extensible. There's core grammar, and
new lexical and syntactic rules can be added by plugins.

### Core Grammar

Compositions:

- Text
- Invocations
    + Macro
    + External Renderer
    + Cross Reference
- Compiler Instructions

#### Text

In Scriptdown, texts are taken as they are, except that as in XML or HTML,
consequent white spaces are taken as one. In the core grammar set, an empty new
line doesn't have any syntactic meaning, but there's a grammatical extension
provided by the core modules which convert free text blocks into paragraphs taking
empty new lines as the separator.

#### Invocations

Invocations are the way Scriptdown shows its power. Invocations will render the relevant
texts into something different than its source. An example of invocation is:

```
[some red text](font.color:red)
```

The above code represents an invocation of a macro, `font.color`, with `red` as
argument, and `some red text` as its payload. So the result in HTML will be:

```
<span style="color:red">some red text</span>
```

which means the text will be shown in red.

Invocations have the following two forms, both are effectively equivalent:

- the *tag* form:

    <pre><code>&lt;<em>invocation</em> <em>args</em>>
        <em>payload..</em>
&lt;/<em>invocation</em>></code></pre>

- the *bracket* form:

    <code>\[<em>payload..</em>\](<em>invocation</em>:<em>args</em>)</code>

One difference is that the *tag* form is preferred to be applied on a long
payload, which may contain line breaks, and the *bracket* form is preferred to
be applied on a short payload, which may not contain any line break. The other
one is that you can chain invocations with the *bracket* form easily. For example
`[Hey Scriptdown](font.color:red;font.size:2em)` is equivalent to

```
<font.size 2em>
  <font.color red>
    Hey Scriptdown
  </font.color>
</font.size>
```

and will be rendered to 

`<span style="font-size:2em"><span style="color:red"></span></span>`

in HTML.

There're three types of invocations:

- Macro (e.g. `macro`)
- External Renderer (e.g. `!exrend`)
- Cross Reference (e.g. `@crossref`)

#### Macros

A valid macro name starts with a character. Macros is the most basic type of
invocations. It transform the **parsed** payload
into another form, by calling a predefined *macro function* associated with the
macro name. By saying **parsed**, it means that if the payload itself contains
other invocations, those invocations will be performed first, and only then be
passed into the associated macro function as input.

So for example, we have a macro called `font.color`, which is associated with
the following JavaScript function (the code is simplified for easier
understanding):

```
var fontColor=function (payload,args) {
  return '<span style="color:' + args + '">' + payload + '</span>';
}
```

writing `[Hey Scriptdown](font.color:red)` will be replaced by the return value
of `fontColor('Hey Scriptdown','red')`.

#### External Renderer

A valid external renderer name starts with a exclamation mark (`!`). An external
renderer invocation is very similar to a macro invocation, except the payload is
passed into the *renderer function* without being transformed by the Scriptdown
compiler. In the core modules, there're two important syntax sugars defined for
external renderer invocations:

##### The URL sugar

<code>!\[<em>url</em>\](<em>another_invocation</em>)</code>

will be translated to:

<code>\[<em>url</em>\](!url;<em>another_invocation</em>)</code>

which will fetch *url* and feed the contents as payload to *another_invocation*,
which is useful when for example writing:

`a cat's picture: ![https://upload.wikimedia.org/wikipedia/commons/2/22/Turkish_Van_Cat.jpg](!img)`

will insert the image from <https://upload.wikimedia.org/wikipedia/commons/2/22/Turkish_Van_Cat.jpg> behind of "a cat's picture:".

##### The inlined external source sugar

<code>``<em>exrend</em>;<em>encoding</em> <em>payload</em>``</code>

will be translated to:

<pre><code>&lt;!<em>exrend</em>>
  &lt;!<em>decoder</em>>
    <em>payload</em>
  &lt;/!<em>decoder</em>>
&lt;/!<em>exrend</em>>
</code></pre>

where the *decoder* is grabbed from a known encoding registry. For example, if
you specify `base64` as the *encoding*, `!codec.base64.decode` will be used as
the *decoder*. Also, the *encoding* part is optional, and multiple encodings
could be chained together so that the raw contents will be decoded for multiple
times (it's a weird feature, but users may need it). Here're two examples:

- Example 1 -- inlining a DOT graph:

    ```
``dot
digraph Greetings {
        Hello->New->World;
        Hello->Scriptdown;
}
``
```

- Example 2 -- inlining an image:

    ```
``img:png;base64
iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAACdUlEQVQ4jY2Ta2iNARjHf+97
ttNh5rC5xFyzXMsXtSQSUT5QPoxcUjS3j0T2ASVJTdkH5bJcasIkpLZl5CxsEofNzpiwyWm2
s43mPXZs5/K+798H5lbGr54vz9P/99RTjyEpDPj5Bz1xm5fvP5OXm/VrO5oGZAMZ/xJ0RROU
3HlLS0eMVfPGk2YaAGkmYA8UfBn+QNGlWkYPTWfb0incCLZR+bS9f2ybA4UdV8T6UpTeamDf
mQB5U4axfM5YKuoiRD7FAfiroKb+DfPWH2LkUC/nCldSFWzh6t0m1i2YSKYvnernnT8FSdum
9EYN2/aeZueh89TUvWbS2BF09/SyYf855s7MIX/hLM7fDuExxNyp2TSEo6QcF1zJ2nP0ssha
JjIWi8GL5Jucr8p79XoQahGzC/Tg2RsFX7VrzpYShSPdevbO0taSJ+qw+iyzN56k9Pp9kPv9
njbxzo8Un61gdu44fBk+gk1hJozy4/EYtH7oIWuIF49p0JtwMD2mSZY/A1I2YHwr22XMqOGk
bIeU4zBkkJek7eAC3nQPtiMkYRhg+rxpHN61hnHTJ0NmJvj9zF+xgKLda7l48xGu47IkbwYN
LR0gyM0ZTsSKAwb+wekgyZKkyMeoyqvrFHj4QomUrWvVdWLaBhUeuyZJWn3girYXl0uSjlc1
6+DVF3JdWT8Ef1JZG9KOojJ96UvoSFmtZm08robmiNq6+1RwMqg7oU5JAwj6KSl/ogmri3U5
0ChJ2lvWqMILIfUm7f8TNL3rUtXjZsWTjk4H3mrTicdqDP+IWEiKDSToJ9DYqc2ngnre+tu+
mKH/fOcvCRsrZpOT7fu1Hf0K5C7pVCUOwRoAAAAASUVORK5CYII=
``
```

#### Cross Reference

A valid cross reference invocation name starts with an at mark (`@`). A
cross reference invocation just register the payload, as an item, in the
cross reference registry, which could be used by some plugins, e.g. when
generating a Table of Contents, cross references of all chapter, section
names are useful. Here's an example:

```
[Section 1](structure.section:@sec1,@sec1)
```

#### Compiler Instruction

Texts and invocations will only affect how the output looks like, the logical
contents in other words. If you need to change the behavior of the compiler,
e.g. you want to use a plugin that's not in the core modules, or define a new
syntax sugar, you have to use compiler instructions.

Compiler instruction(CI) is a special type of invocation, and a compiler instruction
will not considered as an invocation at the runtime. Compiler instructions' name
starts with two exclamation mark (`!!`), taking no argument, and perform the
modification suggested by its name to the compiler. CI's can take payload, and
can appear anywhere in the source file, but the compiler will execute all
CI's before processing other texts or invocations wherever the CI's are. Plugins
can also contribute CI's.

##### Compiler Instruction -- `!!use`:

<code>\[*package*.*plugin_name*\](!!use)</code>

Invoke `!!use` compiler instruction to import and use a plugin. The
*plugin_name* can be an asterisk, which serves as a wildcards that will import
and use all plugins in the specific package. *package* cannot contain wildcard
though. Example:

`[scriptx.dot.*](!!use)`

Note that all plugins in `script.core` package and its sub-packages is imported
and used by default.

# Core modules

The core modules define plugins under the parent package `script.core`. Here's
a list of sub-packages and plugins planed to be added to the core modules:

- script.core
  + InlinedExternalSourceSugar

- script.core.structure
  + Chapter{Macro,Sugar}
  + Section{Macro,Sugar}
  + LeveledSection{Macro,Sugar}

- script.core.style
  + FontSizeMacro
  + FontColorMacro
  + FontFamilyMacro
  + FontStyle{Macro,Sugar}

- script.core.list
  + OrderedList{Macro,Sugar}
  + UnorderedList{Macro,Sugar}

- script.core.quote
  + Blockquote{Macro,Sugar}

- script.core.table
- script.core.math
  + LaTeXMath{Macro,Sugar}

- script.core.codec
  + Base64Renderer
  + Base64EncodingDefinition

- script.core.url
  + Url{Rendered,Sugar}
  + HttpUrlRendered

- script.core.image
  + ImageRenderer
  + PngImageRenderer
  + JpgImageRenderer

# Standard extension modules

The standard extension modules define plugins under the parent package
`scriptx`. Here's a list of sub-packages planed to be added to the standard
extension modules:

- scriptx.code
- scriptx.dot
- scriptx.toc
- scriptx.datasheet
- scriptx.plot
- scriptx.bib

## License

Scriptdown is supposed to be released under the Apache License, Version 2.0
