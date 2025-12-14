Inside your Markdown Monster Addin you have full access to he active Markdown Document as well as the entire UI of the application. 

The Addin interface exposes a number of useful methods that abstract some common operations such as accessing the selection points in the document to read and write content, retrieve and set the entire Markdown document, Create new documents, close documents and so on.

Your top level interface in Markdown Monster is via the `Model` property which is an instance of Markdown Monster's [AppModel class](https://markdownmonster.west-wind.com/docs/Class-Reference/Class-Reference/MarkdownMonster/AppModel-Class.html) that gives access to things like the active editor, document, window, the configuration, the AI engine, open documents and much more.

### Manipulating the selected Editor Text
One of the most common things you do in an Addin is manipulate the text of the document. You can use the `.GetSelection()` and `.SetSelection()` methods of the Addin object you are subclassing to extract the selected text, modify it and paste it back out.

For example the following Addin `OnExecute()` implementation wraps the selected text into an underline HTML block:

```csharp
public override async Task OnExecute(object sender)
{
    var editor = Model.ActiveEditor;
    string sel = await GetSelection();

    if (string.IsNullOrEmpty(sel))
    {
        ShowStatusError("No text to underline. Please select some text to underline first.");
        return;
    }

    sel = "<u>" + sel + "</u>";
    await SetSelection(sel);

    // show status message that goes away after 4 secs
    ShowStatus("Text underlined...", 4000);

    // Set focus back to editor
    await SetEditorFocus();

    // update the preview if active
    RefreshPreview();
}
```


The code captures the selection, then wraps the HTML `<ul>` tag around the selected text and then simply sets the selection of the document which effectively updates the current selection with the new value.

It's that easy to manipulate the document.

Notice also that this code calls `SetEditorFocus()` which refocuses the editor after the edit operation  - not so important in this example because we're not bring up custom UI, but if you do you'll want to make sure the editor is refocused. It also calls `UpdatePreview()` which forces the Preview window to refresh if it is open. Finally `ShowStatus()` is used to update the Status bar to let the user know that something happened. The method takes a second timeout parameter which resets the status bar to its default message after the timeout.

> This example uses high level Addin methods that expose common operations you might perform. They map to underlying methods in the Editor and the editor's interop interface which provides many more functions than what's exposed directly on the Addin interface. To access these many more lower level look at [Model.ActiveEditor](https://markdownmonster.west-wind.com/docs/Class-Reference/Class-Reference/MarkdownMonster/MarkdownDocumentEditor-Class.html) and [Model.ActiveEditor.JsEditor](https://markdownmonster.west-wind.com/docs/Class-Reference/Class-Reference/MarkdownMonster-Windows/EditorWebViewJavaScriptInterop-Class.html).

There are many other things you can do:

### Getting entire Document
You can get and set the entire document with `GetMarkdown()` and `SetMarkdown()`. Use this when you manipulate the entire document and you need to replace it.

### Opening and Closing Tabs
If you need to open a new document or close an existing tab, you can do so using the Addin's `OpenTab()` and `CloseTab()` methods. `OpenTab()` takes an optional file name or `untitled` to open a new blank tab.

### Accessing the Markdown Document
You can access the Markdown Document using `Model.ActiveDocument` which holds information about the document. You can get the `Filename`,`CurrentText`,`IsDirty` status, the `HtmlRenderFileName` used for previews and you can call methods like `RenderHtml()` and `RenderHtmlToFile()` to create HTML from your Markdown.

### Accessing the Markdown Editor 
You can also gain access to the `Model.ActiveEditor` which gives access to the underlying Editor control in the UI. Here you get access to the raw editor WebView instance and the underlying JavaScript Ace Editor instance that the editor is based on. There are is ton of low level  functionality exposed in this object that gives you fine grained control over the editor. Even more low level features are available on `Model.ActiveEditor.JsEditor`.

There are methods like `MarkupMarkdown()` that let you trigger operations like `bold`, `italic`, `image`, `code` etc that match the toolbar operations. There's `SpecialKey()` which lets you trigger most of the hotkeys that Markdown Monster uses like Ctrl-n, Ctrl-s, Ctrl-O etc. You can also switch the Editor's syntax highlighting to a different language (it's `markdown` by default) but you can switch to `csharp` for example and then display a `.cs` file. You can call the `SetDirty()` method to force the document to be marked as dirty (or not dirty). 

### Accessing the Markdown Editor JavaScript Editor
The Editor also has an `JsEditor` property that references the JavaScript Editor wrapper instance called `textEditor` in `editor.js`. This object is a wrapper JavaScript object around ACE editor that provides high level common operations directly to Markdown Monster's C# interface. This essentially allows you to directly interface with ACE editor from an Addin.

```csharp
var editor = Model.ActiveEditor;
var ace = Model.ActiveEditor.JsEditor;

// access raw ACE editor methods (don't use these types)
await ace.Invoke("editor.setTheme", "github");
await ace.Invoke("editor.getSession().setMode", "ace/mode/" + lang);

// or if available use the MM wrappers instead
await editor.SetEditorSyntax("javascript");
editor.SetSpellChecking(true);  // disable
```

You can call functions defined in the `Editor\editor.js` file on the main `textEditor` instance with `Invoke()`. For more info on what's available take a look at the `editor.js` source code or the ACE editor documentation.

### Accessing the Main Window
You can also gain access to the Main UI window of Markdown Monster by using `Model.Window`. This is the top level window and it gives you access to a large list of Methods to manipulate the UI including all sub controls, tabs, sidebars etc. This is an advanced feature and if you plan to do this you're on your own for drilling into the object model, but you do have access the full UI through this interface.

[Window Documentation](https://markdownmonster.west-wind.com/docs/Class-Reference/Class-Reference/MarkdownMonster/MainWindow-Class.html)