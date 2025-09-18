Markdown Monster includes a number of **RenderExtensions** that are add-on features to Markdown HTML generation. They provide often requested extensions to Markdown that can be selectively be used in your Markdown markup.

The following built-in RenderExtensions are available:

{{ Helpers.ChildTopicsList() }}

## More about Render Extensions
Render extensions - as the name suggests - are extensions that modify the standard Markdown rendering process used by the built-in and selected Markdown parser. They are meant to provide extended functionality that the base parser lacks (like Mermaid and PlantUml support for example) or can provide completely custom functionality that matches specific custom Web site behavior (for example, rendering generated content like ads into a document so it closer matches what actual rendered output looks like).
These extensions can significantly enhance your Markdown workflow by allowing you to integrate complex diagrams, custom scripts, or other interactive features seamlessly into your documents.

A few things to consider:

> #### @icon-warning RenderExtensions are Custom!
>  Although Markdown Monster will render the extension in the previewer and any directly generated Html and Pdf output, not all of these extensions are supported by all Markdown hosting provider services. None of these extensions are 'standard' Markdown, so even though the Markdown Monster previewer renders the output, if you're hosting your Markdown on a platform like GitHub, WordPress or some Jekyll service, make sure the extension format is supported by your host. 

> #### @icon-lightbulb Create your own Render Extensions in C#
> You [can create custom RenderExtensions using C#](dm-topic://_5I30SBA89) as part of a Markdown Monster Addin to extend Markdown preview and output rendering with custom behavior.