# Customizing your demo with CSS and Javascript

Gradio allows you to customize your demo in several ways. You can customize the layout of your demo, add custom HTML, and add custom theming as well. This tutorial will go beyond that and walk you through how to add custom CSS and JavaScript code to your demo in order to add custom styling, animations, custom UI functionality, analytics, and more.

## Adding custom CSS to your demo

Gradio themes are the easiest way to customize the look and feel of your app. You can choose from a variety of themes, or create your own. To do so, pass the `theme=` kwarg to the `Blocks` constructor. For example:

```python
with gr.Blocks(theme=gr.themes.Glass()):
    ...
```

Gradio comes with a set of prebuilt themes which you can load from `gr.themes.*`. You can extend these themes or create your own themes from scratch - see the [Theming guide](/guides/theming-guide) for more details.

For additional styling ability, you can pass any CSS to your app as a string using the `css=` kwarg. You can also pass a pathlib.Path to a css file or a list of such paths to the `css_paths=` kwarg.

**Warning**: The use of query selectors in custom JS and CSS is _not_ guaranteed to work across Gradio versions that bind to Gradio's own HTML elements as the Gradio HTML DOM may change. We recommend using query selectors sparingly.

The base class for the Gradio app is `gradio-container`, so here's an example that changes the background color of the Gradio app:

```python
with gr.Blocks(css=".gradio-container {background-color: red}") as demo:
    ...
```

If you'd like to reference external files in your css, preface the file path (which can be a relative or absolute path) with `"/gradio_api/file="`, for example:

```python
with gr.Blocks(css=".gradio-container {background: url('/gradio_api/file=clouds.jpg')}") as demo:
    ...
```

Note: By default, most files in the host machine are not accessible to users running the Gradio app. As a result, you should make sure that any referenced files (such as `clouds.jpg` here) are either URLs or [allowed paths, as described here](/main/guides/file-access).


## The `elem_id` and `elem_classes` Arguments

You can `elem_id` to add an HTML element `id` to any component, and `elem_classes` to add a class or list of classes. This will allow you to select elements more easily with CSS. This approach is also more likely to be stable across Gradio versions as built-in class names or ids may change (however, as mentioned in the warning above, we cannot guarantee complete compatibility between Gradio versions if you use custom CSS as the DOM elements may themselves change).

```python
css = """
#warning {background-color: #FFCCCB}
.feedback textarea {font-size: 24px !important}
"""

with gr.Blocks(css=css) as demo:
    box1 = gr.Textbox(value="Good Job", elem_classes="feedback")
    box2 = gr.Textbox(value="Failure", elem_id="warning", elem_classes="feedback")
```

The CSS `#warning` ruleset will only target the second Textbox, while the `.feedback` ruleset will target both. Note that when targeting classes, you might need to put the `!important` selector to override the default Gradio styles.

## Adding custom JavaScript to your demo

There are 3 ways to add javascript code to your Gradio demo:

1. You can add JavaScript code as a string to the `js` parameter of the `Blocks` or `Interface` initializer. This will run the JavaScript code when the demo is first loaded.

Below is an example of adding custom js to show an animated welcome message when the demo first loads.

$code_blocks_js_load
$demo_blocks_js_load


2. When using `Blocks` and event listeners, events have a `js` argument that can take a JavaScript function as a string and treat it just like a Python event listener function. You can pass both a JavaScript function and a Python function (in which case the JavaScript function is run first) or only Javascript (and set the Python `fn` to `None`). Take a look at the code below:
   
$code_blocks_js_methods
$demo_blocks_js_methods

3. Lastly, you can add JavaScript code to the `head` param of the `Blocks` initializer. This will add the code to the head of the HTML document. For example, you can add Google Analytics to your demo like so:


```python
head = f"""
<script async src="https://www.googletagmanager.com/gtag/js?id={google_analytics_tracking_id}"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){{dataLayer.push(arguments);}}
  gtag('js', new Date());
  gtag('config', '{google_analytics_tracking_id}');
</script>
"""

with gr.Blocks(head=head) as demo:
    gr.HTML("<h1>My App</h1>")

demo.launch()
```

The `head` parameter accepts any HTML tags you would normally insert into the `<head>` of a page. For example, you can also include `<meta>` tags to `head` in order to update the social sharing preview for your Gradio app like this:

```py
import gradio as gr

custom_head = """
<!-- HTML Meta Tags -->
<title>Sample App</title>
<meta name="description" content="An open-source web application showcasing various features and capabilities.">

<!-- Facebook Meta Tags -->
<meta property="og:url" content="https://example.com">
<meta property="og:type" content="website">
<meta property="og:title" content="Sample App">
<meta property="og:description" content="An open-source web application showcasing various features and capabilities.">
<meta property="og:image" content="https://cdn.britannica.com/98/152298-050-8E45510A/Cheetah.jpg">

<!-- Twitter Meta Tags -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:creator" content="@example_user">
<meta name="twitter:title" content="Sample App">
<meta name="twitter:description" content="An open-source web application showcasing various features and capabilities.">
<meta name="twitter:image" content="https://cdn.britannica.com/98/152298-050-8E45510A/Cheetah.jpg">
<meta property="twitter:domain" content="example.com">
<meta property="twitter:url" content="https://example.com">  
"""

with gr.Blocks(title="My App", head=custom_head) as demo:
    gr.HTML("<h1>My App</h1>")

demo.launch()
```



Note that injecting custom JS can affect browser behavior and accessibility (e.g. keyboard shortcuts may be lead to unexpected behavior if your Gradio app is embedded in another webpage). You should test your interface across different browsers and be mindful of how scripts may interact with browser defaults. Here's an example where pressing `Shift + s` triggers the `click` event of a specific `Button` component if the browser focus is _not_ on an input component (e.g. `Textbox` component):

```python
import gradio as gr

shortcut_js = """
<script>
function shortcuts(e) {
    var event = document.all ? window.event : e;
    switch (e.target.tagName.toLowerCase()) {
        case "input":
        case "textarea":
        break;
        default:
        if (e.key.toLowerCase() == "s" && e.shiftKey) {
            document.getElementById("my_btn").click();
        }
    }
}
document.addEventListener('keypress', shortcuts, false);
</script>
"""

with gr.Blocks(head=shortcut_js) as demo:
    action_button = gr.Button(value="Name", elem_id="my_btn")
    textbox = gr.Textbox()
    action_button.click(lambda : "button pressed", None, textbox)
    
demo.launch()
```

