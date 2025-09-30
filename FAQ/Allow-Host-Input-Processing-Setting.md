This setting controls how the Markdown Monster Editor and the Markdown Monster shell (host app) interact. Due to some complications in the WebView2 control used by MM, there are different ways that the WebView control that hosts the editor can interact with the host shell. 

The `AllowHostInputProcessing` flag allows toggling the mode which when `true` processes certain events first in the host shell then in the WebBrowser, and the reverse when using `false`. 

Unfortunately there are side effects to **both of these modes**, meaning that there are some issues that can prop up when using either mode. 

The following describes the issues that are fixed and what doesn't work with the setting enabled. The default is enabled.


## What doesn't work and when

* **Drag Selection past the bottom of the Window**  
When `AllowHostInputProcessing` is `true` (default), when using drag selection and scrolling past the bottom of the window viewport the selection will not automatically keep the window scrolling as expected in other editors. When `false` the window automatically scrolls providing the expected behavior.

* **Accellerator menu and button keys from the editor work**  
When `AllowHostInputProcessing` is `false`, standard Windows accelerator behavior - underlined menus and focused button mnemonic and hot key execution - are not automatically activated *when the editor has focus*. IOW, the accelerator commands are not forwarding properly to the host shell. This diminishes keyboard navigation of many commands somewhat.


> Unfortunately these two behaviors are mutually exclusive for the moment. The former behavior is likely a bug in the WebView2 control and there's [an open issue on the WebView feedback site](https://github.com/MicrosoftEdge/WebView2Feedback/issues/5375) that can be tracked to potentially address this issue in the future. There's also more information on how this issue affects behavior.