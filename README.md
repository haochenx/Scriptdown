# Scriptdown

Scripdown, a manuscript authoring tool. The name is inspired by the famous Markdown,
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

## License

Scriptdown is supposed to be released under the Apache License, Version 2.0
