Markdown natively does not have a lot of support for custom styling, so you don't have a lot of control beyond base document CSS styling of how documents render. For documentation and writing solutions in general that's a good thing - you don't want to be burdened with markup and design issues while writing.

However there are a few things that frequently need to be styled:

* Images
* Media Elements
* Tables

## Markdown and Element Styling
Markdown supports two ways to provide styling:

* [Generic Attributes Extension Syntax](#generic-attributes)
* [Raw Html Markup](#raw-html-markup) 

### Generic Attributes
Generic Attributes is a Common Markdown extension that allows for custom attributes to be provided to Markdown block elements. Essentially it allows you to add custom attributes to the rendered element. 

It looks like this:

```markdown
## Some Block Element{.class #id attr style="style:val" }
```

* [Generic Attributes Syntax](https://talk.commonmark.org/t/consistent-attribute-syntax/272)

The syntax uses a combination of raw attribute syntax and CSS styling (. for CSS Classes, # for Id and raw full text for custom attributes).

Because the syntax feels a little awkward and is non-standard you want to make minimal use of this functionality, and always ensure that your final Markdown Rendering platform (ie. GitHub, Azure Dev Ops, generated site or your own Web Site) supports this format. Not all do, specifically GitHub does not. Markdown Monster's Previewer and [Documentation Monster's](https://documentationmonster.com) Web site generation support it.


#### Examples 
Here are a couple of common examples.

**Images (very common)**

```markdown
![Markdown Monster Banner](MarkdownMonsterSplash.png){style="max-width: 600px; width: 100%"}

Markdown Monster is an easy to use (more text)...
```

**Highlight Color**

```markdown
### Breaking and Recommended Changes{.text-danger}     
* **Recommend full uninstall and reinstall for Full Installations**  
```

### Raw Html Markup
Markdown is a superset of Html so technically raw Html markup inside of a Markdown document is fully supported in Markdown. 

By default Markdown Monster will render **any inline Html** inside of Markdown and accept it as valid (with some limitations for security sensitive tags blocked by default via **Allow Script Tags in Markdown**).

This means you can:

* Embed individual elements like `img`, `media` etc.
* Embed entire Html Blocks
* Embed inline elements into paragraphs
* Use styling and attributes 

Some examples:





> ### @icon-warning Not all Platforms support extended Features
> Generic Attributes is a Markdown Parser specific feature extension and not all Markdown Parsers support it. Custom Html markup may also not be supported at all or supported only for certain elements.
>
> For example, GitHub's Markdown Rendering does not support Generic Attributes or Html embedding except for a few elements like `<img>` and then only supports several deprecated attributes (like `width` and `height`).  
>
> In short, make sure your target Markdown Platform that hosts your final rendered Markdown supports the features you're using in your markup. 
>
> **Just because Markdown Monster can render output does not guarantee that your target platform can.**



###