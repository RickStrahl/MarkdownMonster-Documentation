While writing your content, you can take advantage of AI generated text suggestions by pressing `Ctrl-Space` or using the AI Menu and **Insert Text Suggestions** which pops up a block of suggested text:

![Inline Text Completion via Ghost Text](https://raw.githubusercontent.com/RickStrahl/ImageDrop/refs/heads/master/MarkdownMonster/AiGhostTextInsertionShiftSpace.gif)

Suggestions can be useful as they are provided in context with your document. They tend to work better with more content (ie. more context) or with topics that directly relate to some known concept, product or *thing*. 

It can be a great tool for overcoming writer's block or simply as a idea generating tool as AI generated content often brings up ideas that were not previously considered.

## Keys used for Inline Text Suggestions

* `Ctrl-Space` - load suggestion (takes a second or two)
* `Tab` - Accept the suggestion
* Any other key - Ignore and remove the suggestion


Suggestions call out to an the specified AI server providing your document as input. 

> #### @icon-info-circle Privacy Notice
> This feature sends your document text to the configure AI completion provider you have set up.

## Disabling AutoSuggestions
Although auto-suggestions have to be triggered with `Ctrl-Space` they can pop up on occasion when you hit that key combination inadvertantly. To completely disable this feature even if you have AI providers enabled you can use the following configuration setting: `Editor.NoAiAutoSuggestions` either in the configuration JSON file, or via Configuration Settings UI (**Editor** -> **No Ai Auto Suggestions** ).