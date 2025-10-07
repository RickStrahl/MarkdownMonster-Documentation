The editor has a couple of override files that can be used to override theming and potentially execute custom Javascript code when the editor is started up.

There are two files that are located in the `<commonFolder>\Markdown Monster\EditorThemes`:


* **editor-user-extensions.css**  
Allows you to override editor theming.

* **editor-user-extensions.js**  
This allows potentially modifying editor behavior, by hooking into ACE editor at startup. You can hook into events or manipulate editor settings but it requires inside knowledge of ACE editor or the code in `editor.js` (installed in `Editor\editor.js`) that controls the editor behavior.


## Theme Overriding
ACE editor comes with a large number of editor themes, plus Markdown Monster adds a few custom themes - specifically Visual Studio and VS Code themes.

Themes are a collection of CSS styles, and you can override these styles in the `editor-user-extensions.css` file.  You can find out theme information by using developer tools from the editor (`ctrl-shift-i`) or by looking at the [ACE editor repository and available themes](https://github.com/ajaxorg/ace/tree/master/src/theme) CSS files for styles available for overriding.


For example, if you wanted to change the selection background color for a particular theme you could use the following CSS:

```css
.ace-github .ace_marker-layer .ace_selection {
  background: green,
  color: whitesmoke
}
```

To apply to a different theme, use `.ace-themeName`, where the theme name is what you see in the **Editor Themes** drop down on the right side of the status bar.

The location for the override file by default is in:

```
%appdata%\Markdown Monster\EditorThemes\editor-user-extensions.css
```

This is a standard CSS file that is loaded **after** ACE editor and theme files have loaded so you can override ACE specific styles as well as general purpose styles.

## JavaScript Code Overriding
You can also override editor behavior with JavaScript. This is less common than theme overrides, but the `editor-user-extensions.js` file in the `%appdata%\Markdown Monster\EditorThemes\editor-user-extensions.js` location allows you to hook into the load pipeline of ACE Editor and `editor.js` which is the MM wrapper around ACE Editor that provides the Markdown Monster specific extensions.

In order to do this you'll need to be somewhat familiar with the ACE's Editor and `editor.js` object models. By looking at `editor.js` you can get a good feel of how MM hooks up to ACE editor and what hook in points exist for hooking up events to fire.