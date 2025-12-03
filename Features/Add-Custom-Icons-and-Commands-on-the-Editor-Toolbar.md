The Editor toolbar at the top of the Window allows embedding of common Markdown markup on the selected text. The toolbar is kept relatively simple and so it doesn't wrap all operations that are supported. 

It's possible to extend the toolbar with custom operations:

* Adding additional built-in commands
* Adding custom HTML markup commands


You can do this by using the `Configuration.Editor.AdditionalToolbarIcons` which is a collection of an icon name plus a an action name. For example:

```json
"Editor": {
    "AdditionalToolbarIcons": {
      // internal MM Editor Commands
      "Pencil": "mark",
      "ArrowDown": "small",
      // html tag wrapping
      "Coffee": "html|cite",
      // template for text selection
      "Bolt": "html|<b><i>{0}</i></b>"
    },
```

![](/images/AddCustomIcons.png)

The item in each map is a [FontAwesome 6 Icon](https://fontawesome.com/icons) to be used as an icon, the second is the name of an existing command. The last (Coffee) example, uses `html|cite` as the value which results in custom tag created around the text as `<cite>text</cite>`.

### Built-in Commands 
Built in commands handle the visible toolbar operations, plus a few additional ones that are not actually shown on the toolbar. The following 3 commands are not already on the toolbar:

* `small` - adds text in `<small>txt</small>`
* `mark` - adds textin `<mark>txt</mark>`
* `numberedlist` - formats lines of text as a numbered list
* `pagebreak` - inserts an HTML pagebreak into the document

All available commands can be found in the code shown at the end of this topic:

[MarkdownDocumentEditor::MarkupMarkdown()](#available-html-operation-commands)

> You can also create custom addins that extend the default markup 'commands' by implmenting [MarkdownMonsterAddin.OnEditorCommand](VFPS://Topic/MarkdownMonsterAddin.OnEditorCommand) and creating your own commands.

### Custom HTML Tag Wrapping
You can also create a custom HTML commands that wrap the selected text into an HTML element tag. For example:

```json
"AdditionalToolbarIcons": {
   "Pencil": "html|cite"
}
``` 

```html
<cite>selected text</cite>
```

This gives you access to formatting just about any HTML markup.

### Template Content
Additionally you can also create custom HTML output using a simple 'wrapping' template where you use `{0}` to signify the selection.  For example:

```json
"AdditionalToolbarIcons": {
    "Bolt": "html|<b><i>{0}</i></b>"
}
```

Which renders:

```html
<b><i>selected Text</i></b>
```

### Custom Operations
If you need to add toolbar buttons with more control you can also use either the [Commander Addin](https://github.com/RickStrahl/Commander-MarkdownMonster-Addin) or a [full Addin](VFPS://Topic/_4NB0SE717).


```cs
// Font Awesome Icon
public void AddEditToolbarIcon(string iconName, string markdownActionCommand, 
                               ToolBar toolbar = null, ICommand command = null )
                               
// Image Source Icon (sized to 16 high)
public void AddEditToolbarIcon(ImageSource icon, string markdownActionCommand, 
                               ToolBar toolbar = null, ICommand command = null)
```        
                         
For example using the Commander Addin you could use the following:

```cs
Model.Window.AddEditToolbarIcon("Microphone", "html|speak");
```

which can be triggered off a hotkey (or by running the script).

Using a full addin gives you the added capability of loading icon on application startup without requiring an explicit hotkey.

> Note: You can also pass completely custom commands to the `AddEditToolbarIcon()` method using an addin so you can completely take over custom button processing.

### Available Html Operation Commands
The built-in Html markup commands and actions are described by the conditionals in the following code block. Note that many of the commands are already on the toolbar, but a few obscure ones are on lower level menus or invoked implicitly.


```csharp
public async Task<MarkupMarkdownResult> MarkupMarkdown(string action, string input, string style = null)
{
    var result = new MarkupMarkdownResult();

    if (string.IsNullOrEmpty(action))
    {
        result.Html = input;
        return result;
    }

    action = action.ToLower();
    input = input ?? string.Empty;

    string html = input;
    int cursorMovement = 0;

    if (action == "softbreak")
    {
        var lines = input.GetLines();
        html = string.Empty;
        foreach (var line in lines)
        {
            html += line + mmApp.Configuration.Markdown.SoftReturnSymbol + mmApp.GetNewLine();
        }
        //if (lines.Length > 1)
        //    html = html.TrimEnd(new char[] { '\n', '\r'});
    }


    if (action == "bold")
    {
        html = WrapValue(input, "**", "**", stripExtraSpaces: true, multiline: true, allowEmpty: true);
        if (string.IsNullOrEmpty(input))
            cursorMovement = -2;
    }
    else if (action == "italic")
    {
        var italic = mmApp.Configuration.Markdown.ItalicSymbol;
        html = WrapValue(input, italic, italic, stripExtraSpaces: true, multiline: true, allowEmpty: true);
        if (string.IsNullOrEmpty(input))
            cursorMovement = -1;
    }
    else if (action == "inlinecode")
    {
        if (input.Contains('`'))
        {
            // nested ` reqiures double escaping
            if (input.StartsWith("`"))
                input = ' ' + input;
            if (input.EndsWith("`"))
                input = input + ' ';

            html = "``" + input + "``";
        }
        else
        {
            html = WrapValue(input, "`", "`", stripExtraSpaces: true, allowEmpty: true);
            if (string.IsNullOrEmpty(input))
                cursorMovement = -1;
        }
    }
    else if (action == "small")
    {
        // :-( no markdown spec for this - use HTML
        html = WrapValue(input, "<small>", "</small>", stripExtraSpaces: true, multiline: true, allowEmpty: true);
        if (string.IsNullOrEmpty(input))
            cursorMovement = -8;
    }
    else if (action == "underline")
    {
        // :-( no markdown spec for this - use HTML
        html = WrapValue(input, "<u>", "</u>", stripExtraSpaces: true, multiline: true, allowEmpty: true);
        if (string.IsNullOrEmpty(input))
            cursorMovement = -4;
    }
    else if (action == "strikethrough")
    {
        html = WrapValue(input, "~~", "~~", stripExtraSpaces: true, multiline: true, allowEmpty: true);
        if (string.IsNullOrEmpty(input))
            cursorMovement = -2;
    }
    else if (action == "mark")
    {
        html = WrapValue(input, "<mark>", "</mark>", stripExtraSpaces: true, multiline: true, allowEmpty: true);
        if (string.IsNullOrEmpty(input))
            cursorMovement = -7;
    }
    else if (action == "pagebreak")
        html = "\n<div style='page-break-after: always'></div>\n";
    else if (action == "nonbreakingspace")
    {
        html = "&nbsp;";
    }
    else if (action == "nonbreakinghyphen")
    {
        html = "&#8209;";
    }
    else if (action == "h1")
    {
        html = await PrefixSelectedLine("# ", "#", " ", "\t");
        cursorMovement = -1;  // linefeed
    }
    else if (action == "h2")
    {
        html = await PrefixSelectedLine("## ", "#", " ", "\t");
        cursorMovement = -1;  // linefeed
    }
    else if (action == "h3")
    {
        html = await PrefixSelectedLine("### ", "#", " ", "\t");
        cursorMovement = -1;  // linefeed
    }
    else if (action == "h4")
    {
        html = await PrefixSelectedLine("#### ", "#", " ", "\t");
        cursorMovement = -1;  // linefeed
    }
    else if (action == "h5")
    {
        html = await PrefixSelectedLine("##### ", "#", " ", "\t");
        cursorMovement = -1;  // linefeed
    }
    else if (action == "h6")
    {
        html = await PrefixSelectedLine("###### ", "#", " ", "\t");
        cursorMovement = -1;  // linefeed
    }
    else if (action == "quote")
    {
        input = input.Trim();

        // if there's no selection - select the current line
        if (Editor != null && string.IsNullOrEmpty(input))
        {
            var lineno = await Editor.GetLineNumber();
            await Editor.SetSelectionRange(lineno, 0, lineno, 999999);
            input = await Editor.GetSelection();
            input = input?.Trim();
        }


        var sb = new StringBuilder();
        var lines = StringUtils.GetLines(input);
        foreach (var line in lines)
        {
            var ln = line.Trim();
            if (!ln.StartsWith(">"))
                sb.AppendLine("> " + ln);
            else
                sb.AppendLine(ln);
        }

        html = sb.ToString();

        if (lines.Length < 2)
            html = html.TrimEnd(); // strip line feed

    }
    else if (action == "list" || action == "checklist" || action == "numberlist")
    {

        input = input.Trim();
        bool isEmptySelection = string.IsNullOrEmpty(input);

        var listPrefix = "* ";
        if (action == "checklist")
            listPrefix = "* [ ] ";
        else if (action == "numberlist")
            listPrefix = "{0}. ";

        // if there's no selection - select the current line
        if (Editor != null && isEmptySelection && listPrefix != "{0}. ")
        {
            html = await PrefixSelectedLine(listPrefix, "*", " ");
        }
        else
        {
            var sb = new StringBuilder();
            var lines = input.GetLines();

            var ct = 0;
            foreach (var line in lines)
            {
                ct++;
                var ln = line.Trim();

                var prefix = string.Format(listPrefix, ct);

                if (!ln.StartsWith(prefix))
                    sb.AppendLine(prefix + ln);
                else
                    sb.AppendLine(ln);
            }

            html = sb.ToString();
        }

        if (isEmptySelection)
        {
            cursorMovement = -1;
            //    html = html.TrimEnd() + " "; // strip off LF
        }
    }
    else if (action == "table")
    {
        if (Editor != null)
        {
            var line = await Editor.GetCurrentLine();
            line = line.Trim();


            if (line.StartsWith("|") ||
                line.StartsWith("+-") ||
                line.StartsWith("+="))
            {
                // Edit existing table
                var pos = await Editor.GetCursorPosition();
                var tableMarkdown = await TableParser.SelectPipeAndGridTableMarkdown();
                await Editor.EditorSelectionOperation("table", tableMarkdown, pos);
            }
            else
            {
                // This one display non-modal so you can access
                // the document and move the cursor around
                var form = new TableEditor();
                form.Owner = Window;
                form.Show();     // html remains unchanged = nothing updated
            }
        }

        // if running modal
        // var form = new TableEditor();
        // DialogResult = form.ShowDialog();
        //if (!form.Cancelled)
        //    html = form.TableHtml;
    }
    else if (action == "emoji")
    {
        result.DontSetEditorFocus = true;
        if (_emojiWindow is { IsClosed: false })
        {
            _emojiWindow.Activate();
            _emojiWindow.MoveWindowToToolButtonAndFocus();
        }
        else
        {
            _emojiWindow = new EmojiWindow(Window) { LeaveOpenAfterSelection = mmApp.Configuration.ToolWindows.LeaveEmojiWindowOpen };
            _emojiWindow.Show();
        }
        
        //var dresult = await Window.Dispatcher.InvokeAsync(() => form.ShowDialog());
        //if (dresult.HasValue && dresult.Value)
        //html = form.EmojiString + " ";
    }
    else if (action == "youtube")
    {
        var form = new PasteYouTubeWindow(Window);
        form.AutoEmbed = true;  // form sets editor text directly
        form.Show();
    }
    else if (action == "href")
    {
        var form = new PasteHref()
        {
            Owner = Window,
            LinkText = input,
        };
        if (Editor != null)
            form.MarkdownFile = Editor.MarkdownDocument.Filename;

        // check for links in input or on clipboard
        string link = input;
        if (string.IsNullOrEmpty(link))
            link = ClipboardHelper.GetText();

        if (!(input.StartsWith("http:") ||
              input.StartsWith("https:") ||
              input.StartsWith("mailto:") ||
              input.StartsWith("ftp:")))
            link = string.Empty;
        form.Link = link;

        bool? res = await Window.Dispatcher.InvokeAsync(() => form.ShowDialog());
        if (res != null && res.Value)
        {
            if (form.IsExternal)
                html = $"<a href=\"{form.Link}\" target=\"_blank\">{form.LinkText}</a>";
            else if (form.IsLinkReference)
                // this doesn't set Html it directly updates the document
                await AddLinkReference(form.Link, form.LinkText);
            else
                html = $"[{form.LinkText}]({form.Link.Replace(" ", "%20")})";
        }
    }
    // code based href
    else if (action == "href2")
    {
        var ct = ClipboardHelper.GetText();
        if (ct.StartsWith("http"))
        {
            html = $"[{input}]({ct?.Replace(" ", "%20")})";
            //cursorMovement = (ct.Length + 1) * -1;
            //result.SelectionLength = ct.Length;
        }
        else
        {
            html = $"[{input}]()";
            cursorMovement = -1;
        }
    }
    else if (Editor != null && action == "image")
    {
        var form = new PasteImageWindow(Window)
        {
            ImageText = input,
            MarkdownFile = Editor.MarkdownDocument.Filename
        };


        // check for links in input or on clipboard
        string link = input;
        if (string.IsNullOrEmpty(link))
            link = ClipboardHelper.GetText();

        if (!(input.StartsWith("http:") || input.StartsWith("https:") || input.StartsWith("mailto:") ||
              input.StartsWith("ftp:")))
            link = string.Empty;

        if (input.Contains(".png") || input.Contains(".jpg") || input.Contains(".gif"))
            link = input;

        form.Image = link;

        bool? res = await Window.Dispatcher.InvokeAsync(() => form.ShowDialog());

        if (res != null && res.Value && form.Image != null)
        {
            var image = form.Image;
            if (!image.StartsWith("data:image/"))
            {
                if (string.IsNullOrEmpty(form.ImageText))
                    form.ImageText = StringUtils.BreakIntoWords(System.IO.Path.GetFileNameWithoutExtension(image));

                html = $"![{form.ImageText}]({image.Replace(" ", "%20")})";
            }
            else
            {
                var id = "image_ref_" + DataUtils.GenerateUniqueId();

                object pos = await Editor.EditorHandler.JsInterop.GetCursorPosition();
                object scroll = await Editor.EditorHandler.JsInterop.GetScrollTop();

                // the ID tag
                html = $"{mmApp.NewLine}{mmApp.NewLine}[{id}]: {image}{mmApp.NewLine}";

                // set selction position to bottom of document
                await Editor.GotoBottom();
                await Editor.SetSelection(html);

                WindowUtilities.DoEvents();

                // reset the selection point
                await Editor.EditorHandler.JsInterop.SetCursorPosition(pos);

                if (scroll != null)
                    await Editor.EditorHandler.JsInterop.SetScrollTop(scroll);

                WindowUtilities.DoEvents();

                html = $"{mmApp.NewLine}![{form.ImageText}][{id}]";
            }
        }

        form?.Close();
    }
    else if (action == "image2")
    {
        html = "![" + input + "]()";
        cursorMovement = -1;
    }
    else if (action == "code")
    {
        var form = new PasteCode { Owner = Window };
        if (!string.IsNullOrEmpty(input))
            form.Code = input;
        else
        {
            // use clipboard text if we think it contains code
            string clipText = ClipboardHelper.GetText();
            if (!string.IsNullOrEmpty(clipText) &&
                clipText.Contains("{") ||
                clipText.Contains("/>") ||
                clipText.Contains("="))
                form.Code = clipText;
        }

        form.CodeLanguage = mmApp.Configuration.ToolWindows.LastCodeSyntaxUsed;

        var code = form.Code?.Trim() ?? string.Empty;
        if (code.StartsWith("```") && code.EndsWith("```"))
            form.CodeLanguage = "markdown";

        bool? res = await Window.Dispatcher.InvokeAsync(() => form.ShowDialog());

        if (res.HasValue && res.Value)
        {
            var delims = "```";
            code = form.Code?.Trim() ?? string.Empty;
            if (code.StartsWith("```") && code.EndsWith("```"))
                delims = "~~~";

            var lang = form.CodeLanguage;
            if (lang == "none")
                lang = string.Empty;

            html = delims + lang + mmApp.NewLine +
                   code + mmApp.NewLine +
                   $"{delims}{mmApp.NewLine}";

            if (string.IsNullOrEmpty(code))
                cursorMovement = -5;
        }

        if (Editor != null)
            Editor?.SetEditorFocus().FireAndForget();
    }
    else if (action == "subscript")
    {
        html = WrapValue(input, "<sub>", "</sub>", stripExtraSpaces: true);
        if (string.IsNullOrEmpty(input))
            cursorMovement = -6;
    }
    else if (action == "superscript")
    {
        html = WrapValue(input, "<sup>", "</sup>", stripExtraSpaces: true);
        if (string.IsNullOrEmpty(input))
            cursorMovement = -6;
    }
    else if (action == "bolditalic")
    {
        html = WrapValue(input, "<b>" + mmApp.Configuration.Markdown.ItalicSymbol, mmApp.Configuration.Markdown.ItalicSymbol + "</b>", stripExtraSpaces: true);
        if (string.IsNullOrEmpty(input))
            cursorMovement = -8;
    }
    else if (action == "smallitalic")
    {
        html = WrapValue(input, "<small>" + mmApp.Configuration.Markdown.ItalicSymbol, mmApp.Configuration.Markdown.ItalicSymbol + "</small>", stripExtraSpaces: true);
        if (string.IsNullOrEmpty(input))
            cursorMovement = -9;
    }
    else if (action == "mermaid")
    {
        if (string.IsNullOrEmpty(input))
        {
            input = ClipboardHelper.GetText();
        }

        html = WrapValue(input, "```mermaid\n", "\n```", stripExtraSpaces: true);
    }
    else if (action == "plantuml")
    {
        if (string.IsNullOrEmpty(input))
        {
            input = ClipboardHelper.GetText()?.Trim();
            if (!string.IsNullOrEmpty(input) && !input.Contains("@startuml"))
                input = string.Empty;
        }

        if (!string.IsNullOrEmpty(input))
        {
            if (input.Trim().StartsWith("```plantuml"))
                html = input;
            else
                html = WrapValue(input, "```plantuml\n", "\n```", stripExtraSpaces: true);
        }
        else
        {
            html = WrapValue(" ", "```plantuml\n@startuml\n", "\n@enduml\n```");
            cursorMovement = -12;
        }
    }
    // casing
    else if (action == "uppercase")
    {
        html = input?.ToUpper();
    }
    else if (action == "lowercase")
    {
        html = input?.ToLower();
    }
    else if (action == "propercase")
    {
        html = StringUtils.ProperCase(input);
    }
    else if (action == "camelcase")
    {
        html = StringUtils.ToCamelCase(input);
    }

    else if (action == "htmlencode")
    {
        html = WebUtility.HtmlEncode(input);
    }
    else if (action == "urlencode")
    {
        html = WebUtility.UrlEncode(input);
    }
    else if (action == "normalizeindentation")
    {
        html = StringUtils.NormalizeIndentation(input).TrimEnd();
    }
    // Custom HTML commands like
    // html|mark   html|small   html|custom
    // creates wrapped element with start/end tag wrapped around selected text
    // https://markdownmonster.west-wind.com/docs/_5im10bjpw.htm#template-content
    else if (action.StartsWith("html|"))
    {
        // format is `html|markup|optionalTitle`
        var tokens = action.Split('|');
        action = tokens[1];

        if (action.Contains("{0}"))
            // html|<b><i>{0}</i></b>   // {0} is text selection
            html = action.Replace("{0}", input);
        else
            // html|cite  (html keyword)
            html = WrapValue(input, $"<{action}>", $"</{action}>", stripExtraSpaces: true);

        cursorMovement = (action.Length + 3) * -1;
    }
    else
    {
        // allow addins to handle custom actions
        string addinAction = await AddinManager.Current.RaiseOnEditorCommand(action, input);
        if (!string.IsNullOrEmpty(addinAction))
            html = addinAction;
    }

    result.CursorMovement = cursorMovement;
    result.Html = html;

    return result;
}
```