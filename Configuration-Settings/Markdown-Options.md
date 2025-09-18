#### AutoLinks
Determines whether URLs are automatically turned into links. `true`

#### Abbreviations
Determines whether common abbreviations are expanded. `false`

#### StripYamlFrontMatter
Strips Yaml FrontMatter from any Markdown content. `true`

#### EmojiAndSmiley
Interprets common emoji and smiley characters like `:smile:` or `:-)` and turns it into Emoji. `true`

#### ListExtras
Provides features like Github Check lists and nested lists: `true`

#### Figures 
Allows for figure referencing in a location other than the current location. true

#### GithubTaskLists
Allows for Github checkable task lists by using `[ ]` and `[x]`.

#### SmartyPants
Converts common typographic characters to proper typographic publishing settings. Quotes to curly quotes dashes to em dashes etc. Use for print publishing of literature - leave off for technical content. `false`

#### Diagrams
Support for Mermaid markup. Note the MM preview doesn't support Mermaid, but the renderer produces it and you can preview in the external browser.

#### RenderLinksAsExternal
Renders all links with a target="_blank" to force links to be externally opened in a new tab/window. Useful in some documentation scenarios where you don't want to lose your place in the document. `false`

#### AllowRenderScriptTags
By default script tags are stripped from Markdown content. Set to true if you need to explicitly run scripts as part of your markdown for things like embedded Gists for example. `false`


### UseSoftlineBreakAsHardlineBreak
If set renders softline breaks (or a simple single new-line) as hard line breaks, which is non-standard Markdown which renders soft returns as spaces. However soft line break as hard line break is supported by a few popular platforms and this option allows the MM preview to render similar these platforms for Preview and Html/Pdf output. `false`

+--------------------------------------------------------------------+-----------------------------------------------------------------+----------------------------------------------------------------+
| Platform                                                           | Treats single line break as whitespace (Gruber)¹                | Treats single line break as hard return                        | 
|                                                                    |                                                                 |  (CommonMark)²                                                 |
+====================================================================+=================================================================+================================================================+
| [GitHub](https://github.com/)                                      | No                                                              | Yes (`.md` files, Wikis, Issues, Pull Requests, Comments, ...) |
+--------------------------------------------------------------------+-----------------------------------------------------------------+----------------------------------------------------------------+
| [GitLab](https://gitlab.com/)                                      | Yes (`.md` files, Wikis, Issues, Merge Requests, Comments, ...) | No                                                             |
+--------------------------------------------------------------------+-----------------------------------------------------------------+----------------------------------------------------------------+
| [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/) | No                                                              | Yes (`.md` files, Wikis, Issues, Pull Requests, Comments, ...) |
+--------------------------------------------------------------------+-----------------------------------------------------------------+----------------------------------------------------------------+
| [Bitbucket](https://bitbucket.org/)                                | No                                                              | Yes (`.md` files, Wikis, Issues, Comments, ...)                |
+--------------------------------------------------------------------+-----------------------------------------------------------------+----------------------------------------------------------------+
| [Stack Overflow](https://stackoverflow.com/)                       | Yes (Posts, Comments, ...)                                      | No                                                             |
+--------------------------------------------------------------------+-----------------------------------------------------------------+----------------------------------------------------------------+
| [Jupyter (nbconvert)](https://nbconvert.readthedocs.io/en/latest/) | Yes (Markdown cells)                                            | No                                                             |
+--------------------------------------------------------------------+-----------------------------------------------------------------+----------------------------------------------------------------+
| [Reddit](https://www.reddit.com/)                                  | in previous version (until 2018)                                | in current version (since 2018)                                |
+--------------------------------------------------------------------+-----------------------------------------------------------------+----------------------------------------------------------------+
| [Discord](https://discord.com/)                                    | No                                                              | Yes (Messages, ...)                                            |
+--------------------------------------------------------------------+-----------------------------------------------------------------+----------------------------------------------------------------+
<small>

> #### Notes
>
> 1. Treats a single line break (not preceded by two spaces or a backslash, and not followed by a blank line) as whitespace, according to [John Gruber’s original Markdown specification](https://daringfireball.net/projects/markdown/syntax#p).
>
> 2. Treats a single line break (named 'soft line break' by MarkDig) as `<br>` (named 'hard line break' by MarkDig), according to the [CommonMark specification](https://spec.commonmark.org/0.31.2/#soft-line-breaks).

</small>

#### MarkdownParserName
This is the name of the Markdown Parser used. MM ships with `MarkDig` which is the default parser, but addins can add additional parsers which are selectable from the dropdown on the status bar. `Markdown`

#### GenericAttributes
Allows attaching generic HTML attributes to the active 'element'. Example: `{ class='error-message' id="MyHeader"}` using explicit attributes or using CSS Selector style syntax `{ .error-message #MyHeader }`. The attributes are  appended to the generated HTML element on the same line (ie. header, list item, paragraph etc.). 
[more info](https://github.com/lunet-io/markdig/blob/master/src/Markdig.Tests/Specs/GenericAttributesSpecs.md)

#### CustomContainers
Allows wrapping a Markdown element or element group with a container element. Using a block like `:::note` and `:::` to denote a block will generate a `<div class="note"></div>` around the Markdown block it surrounds.
 
#### ParseDocFx
If set parses several common DocFx 'commands' like file inclusion, link expansion and a few others.
 
#### MarkdigExtensions
Allows you to explicitly specify MarkDig extensions to load. MM loads a default set and sets a number of them via feature flags, but you can explicitly force additional Markdig extensions to load. 

Some extensions available:

`emphasisextras,pipetables,gridtables,footers,footnotes,citations,attributes,hardlinebreak`

[More info](https://github.com/xoofx/markdig/blob/3535701d70066d055d06adaeaa1531b10601c6ac/src/Markdig/MarkdownExtensions.cs#L552) on available extension names.