## Plan: VS Code DocMonster Template Syntax

Create a standalone VS Code language extension for DocMonster HTML templates that preserves HTML highlighting, adds explicit support for {{ expression }}, {{! raw expression }}, and {{% code }} delimiters, and treats the content inside those delimiters as embedded C#. The extension should auto-associate .dmhtml files and the current workspace should also add files.associations so the existing theme .html files use the custom language.

**Steps**
1. Define the extension shape and language contract.
   - Create a standalone VS Code extension project rather than modifying the theme folder directly.
   - Choose a stable language id and display name oriented around DocMonster templates, not generic Handlebars, because the delimiters and C# rules are custom.
   - Register .dmhtml as a native extension contribution.
   - Plan to pair the extension with workspace files.associations for the existing theme .html files, since folder-specific .html remapping is a workspace concern, not an extension contribution.
2. Build the base grammar as HTML-first with custom delimiter rules.
   - Contribute a TextMate grammar whose root scope derives from HTML semantics so regular tags, attributes, strings, comments, script blocks, and style blocks keep working.
   - Add higher-priority rules for the three DocMonster forms before the base HTML include:
     - {{ expression }}   
     - {{! raw expression }}
     - {{: explicit Html encoded expression }}
     - {{% code }}   (single line)
     - {{% multi-line code blocks
      }}
     - Handle nested delimiters and edge cases such as adjacent   expressions in the same attribute, C# code brackets from C# structured statements and the Handlebars end delimiter.
     Example:  
      ```html
      {{%
         var name = "Rick";
         if (string.IsNullOrEmpty(name)) { }}
               <div>{{name}}</div>
      {{% } }}     
      ```           
   - Treat each delimiter pair as a self-contained embedded region, because current templates use {{% ... }} as inline server-code statements that open or close C# control flow around intervening HTML instead of a single multi-line wrapper around HTML.
3. Embed C# tokenization inside each delimiter form.
   - Wrap expression bodies in meta.embedded.inline.csharp and code-block bodies in meta.embedded.block.csharp scopes.
   - Map those scopes to the csharp language using embeddedLanguages so bracket matching, comment commands, and editor behaviors treat the inner content as C#.
   - Include the built-in C# grammar inside those regions so string literals, keywords, operators, interpolated strings, method calls, null-conditional operators, and generics tokenize like C#.
   - Give the opening and closing delimiters their own punctuation scopes so themes can distinguish delimiters from C# code.
4. Add language configuration for mixed-template ergonomics.
   - Provide bracket and auto-closing settings that do not fight HTML editing.
   - Keep the language configuration minimal; rely on the grammar and embeddedLanguages for most behavior.
   - Avoid custom comment rules for the whole file unless they are proven necessary, because the document is primarily HTML and global comment toggling can become surprising in mixed syntaxes.
5. Add workspace association for the current theme folder.
   - Add a workspace .vscode settings entry mapping the Dharkan theme’s .html template files to the new language id.
   - Keep the association scoped to this workspace so normal HTML files elsewhere are unaffected.
6. Validate against real template shapes from the theme.
   - Verify inline expressions in text nodes and attributes.
   - Verify raw expressions using {{! ... }}.
   - Verify explicit HTML-encoded expressions using {{: ... }}.
   - Verify C# statements in {{% ... }} blocks, including control-flow openers like if (...) { and closers like }.
   - Verify nested quotes and escaped content inside C# strings, such as Topic.Updated.ToString("MMM dd, yyyy") and interpolated strings in mailBody creation.
7. Document scope and limitations.
   - Document that this phase provides syntax coloring and embedded-language editor behavior.
   - Explicitly exclude full C# IntelliSense, Roslyn diagnostics, rename, and semantic symbol awareness inside mixed HTML template regions unless a later phase adds a custom language server or virtual-document projection design.

**Relevant files**
- c:\Users\rstrahl\Documents\Documentation Monster\MarkdownMonster\_docmonster\themes\Dharkan\_layout.html — primary reference for all three delimiter forms, attribute embeddings, inline text output, and mixed HTML/C# control-flow structure.
- c:\Users\rstrahl\Documents\Documentation Monster\MarkdownMonster\_docmonster\themes\Dharkan\topic.html — additional reference for raw output with {{! ... }}, conditional blocks, and common expression shapes.
- c:\Users\rstrahl\Documents\Documentation Monster\MarkdownMonster\_docmonster\themes\Dharkan\classheader.html — useful reference for repeated raw-output expressions and typical helper-call patterns.

**Verification**
1. Open representative files and switch them to the new language mode; confirm HTML tokens still color correctly outside delimiters.
2. Use Developer: Inspect Editor Tokens and Scopes on delimiter braces, C# keywords, strings, helper calls, and property chains to confirm the expected TextMate scopes and embedded language mapping.
3. Test editing commands inside a delimiter region: bracket matching, auto-closing pairs, and line-comment behavior should behave as C# inside the embedded scopes.
4. Confirm current theme .html files auto-open in the custom language through workspace files.associations, while .dmhtml files auto-open through the extension’s language contribution.
5. Smoke-test a few edge cases from real templates: adjacent expressions in the same attribute, null-conditional calls, string interpolation, and closing control-flow code such as {{% } }}.

**Decisions**
- Include support for {{! expression }} in addition to the two originally listed forms because it exists in current templates.
- Deliver as a standalone VS Code extension.
- Support both current .html templates in this workspace and a new .dmhtml extension for future files.
- Use workspace files.associations for current .html files.
- Target syntax coloring and embedded-language editing support, not full semantic C# tooling.

**Further Considerations**
1. Recommended language naming: use a DocMonster-specific name such as DocMonster Template or DocMonster HTML rather than Html Handlebars, because the syntax differs from standard Handlebars and that reduces user confusion in VS Code’s language picker.
2. If implementation proves the full custom root grammar too brittle against HTML edge cases, the fallback design is an HTML-targeted injection grammar plus a dedicated custom language id; keep that as a contingency, not the first choice.
3. Map VS Code .dmhtml and .wwscript and .wwshtml extensions and secondary to .html association to the same language id so they share the same grammar and configuration, but allow .dmhtml to be the primary extension for the language contribution to avoid confusion with generic HTML files in other contexts.
4. Ensure Html Intellisense works
5. Ensure C# Syntax coloring works and if possible IntelliSense
6. Ensure that bracket parsing for C# code blocks works for multi-line blocks with nested `{`  and `}` inside of the `{{% ... }}` delimiters. This is critical for control flow statements like if/else and loops and has in previous iterations. (see InitialLanguageParsing-Summary.md for details on past issues and approaches)


## Directions

- Run to completion without asking for permissions. Ok to write files in extension folder and workspace .vscode folder.
- Ask for clarification if any part of the plan is unclear or if you need more information to execute a step.
- Test the extensions on `_layout.dmhtml`
