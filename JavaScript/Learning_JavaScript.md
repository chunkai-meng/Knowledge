# Chapter 2 Javascript in HTML

### JavaScript Element  ```<script>```
- No other content is load or displayed until All the code inside JavaScript element has been evaluated.
- JavaScript code should avoid using ```</script>```
- ```<script xxx />``` only accepted in XHTML

### Tag Placement
- Scripts in the <head> tag will be download, parsed, and interpreted before the page begins rendering(rendering begins when browser receives the opening <body> Tag).
- modern web applications put the script in body element after the page content.



### Deferred Scripts

`<script type=”text/javascript” defer src=”example1.js”></script>`

Scripts are run after all the page has been parsed.

indicate that a script won’t be changing the structure of the page as it executes.
As such, the script can be run safely after the entire page has been parsed.
The HTML5 specﬁcation indicates that scripts will be executed in the order in which they appear, so the ﬁrst deferred script executes before the second deferred script, and both will execute before the DOMContentLoaded event (see Chapter 13 for more information). In reality, though, deferred scripts don’t always execute in order or before the DOMContentLoaded event, so it’s best to include just one when possible.

### Asynchronous Scripts

`<script type=”text/javascript” async src=”example1.js”></script>`

Applies only to external scripts, browser load the page without waiting for downloading the external script files.

1. This indicates that the browser need not wait for the script to be downloaded and executed before continuing to load.
2. If more than one async tags exist, they are not guaranteed to execute in the order in which they are specified.
3. For the above reason, asynchronous scripts should not modify DOM as they are loading.
4. Ansynchronous scripts can not guaranteed the order of the execution.

### Changes in XHTML

- symbol (<) with its HTML entity (&lt;)
- use [CDATA] to indicates free-form text
- for not not XHTML-compliant
```
//<![CDATA[
//]]>
```
is used

### Inline code versus External files
- Easy to maintain
- Share the cached files
- Don't have XHTML problem and no need to use that comment hacks.

### DOCUMENT MODES

### THE <NOSCRIPT> ELEMENT
When the browser doesn't support scripting or its scripting support is turned off.
Content in the `<NOSCRIPT>` element will be rendered.

## Summary
1. All `<script>` elements are interpreted in the order in which they occur on the page.
2. For nondeferred scripts, the browser must complete interpretation of the code inside a `<script>` element before it can continue rendering the rest of the page.
3. You can defer a script’s execution until after the document has rendered by using the defer attribute. Deferred scripts always execute in the order in which they are speciﬁed.
4. You can indicate that a script need not wait for other scripts and also not block the document rendering by using the async attribute. Asynchronous scripts are not guaranteed to execute in the order in which they occur in the page. 5. By using the <noscript> element, you can specify that content is to be shown only if scripting support isn’t available on the browser.
