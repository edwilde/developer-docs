---
title: Javascript Development
summary: Advanced documentation about writing and customizing javascript within Silverstripe CMS.
iconBrand: js
---

# Javascript Development

The following document is an advanced guide on building rich javascript interactions within the Silverstripe CMS and
a list of our best practices for contributing and modifying the core javascript framework.

## ES6 and build tools
The remainder of this tutorial is written in [ECMAScript 6](http://es6-features.org/#Constants), or _ES6_
for short. This is the new spec for Javascript (currently ES5) that is as of this writing
only partially implemented in modern browsers. Because it doesn't yet enjoy vast native support,
it has to be [transpiled](https://www.stevefenton.co.uk/2012/11/compiling-vs-transpiling/) in order to work
in a browser. This transpiling can be done using a variety of toolchains, but the basic
 principle is that a browser-ready, ES5 version of your code is generated in your dev
 environment as part of your workflow.
   
   As stated above, there are many ways to solve the problem of transpiling. The toolchain
   we use in core Silverstripe CMS modules includes:
   * [Babel](http://babeljs.io) (ES6 transpiler)
   * [Webpack](http://webpack.js.org) (Module bundler)

[notice]
The following documentation regarding jQuery, jQueryUI and Entwine does not apply to React components or sections powered by React.
If you're developing new functionality in React powered sections please refer to
[ReactJS, Redux, and GraphQL](./reactjs_redux_and_graphql).
[/notice]

## jQuery, jQuery UI and jQuery.entwine: Our libraries of choice

We predominantly use [jQuery](http://jquery.com) as our abstraction library for DOM related programming, within the
Silverstripe CMS and certain framework aspects.

For richer interactions such as drag'n'drop, and more complicated interface elements like tabs or accordions,
Silverstripe CMS uses [jQuery UI](http://ui.jquery.com) on top of jQuery.

For any custom code developed with jQuery, you have four choices to structure it: Custom jQuery Code, a jQuery Plugin, a
jQuery UI Widget, or a `jQuery.entwine` behaviour. We'll detail below where each solution is appropriate.

## Custom jQuery Code

jQuery allows you to write complex behavior in a couple of lines of JavaScript. Smaller features which aren't likely to
be reused can be custom code without further encapsulation. For example, a button rollover effect doesn't require a full
plugin. See "[How jQuery Works](http://docs.jquery.com/How_jQuery_Works)" for a good introduction.

You should write all your custom jQuery code in a closure.


```js
(function($) {
    $(document).ready(function(){
        // your code here.
    })
})(jQuery);
```

## jQuery Plugins

A jQuery Plugin is essentially a method call which can act on a collection of DOM elements. It is contained within the
`jQuery.fn` namespace, and attaches itself automatically to all jQuery collections. The basics for are outlined in the
official [jQuery Plugin Authoring](http://docs.jquery.com/Plugins/Authoring) documentation.

There a certain [documented patterns](http://www.learningjquery.com/2007/10/a-plugin-development-pattern) for plugin
development, most importantly:

*  Claim only a single name in the jQuery namespace
*  Accept an options argument to control plugin behavior
*  Provide public access to default plugin settings
*  Provide public access to secondary functions (as applicable)
*  Keep private functions private
*  Support the [Metadata Plugin](http://docs.jquery.com/Plugins/Metadata/metadata)

Example: A plugin to highlight a collection of elements with a configurable foreground and background colour
(abbreviated example from [learningjquery.com](http://www.learningjquery.com/2007/10/a-plugin-development-pattern)).


```js
// create closure
(function($) {
  // plugin definition
  $.fn.highlight = function(options) {
    // build main options before element iteration
    var opts = $.extend({}, $.fn.highlight.defaults, options);
    // iterate and reformat each matched element
    return this.each(function() {
      $this = $(this);
      // build element specific options
      var o = $.meta ? $.extend({}, opts, $this.data()) : opts;
      // update element styles
      $this.css({
        backgroundColor: o.background,
        color: o.foreground
      });
    });
  };
  // plugin defaults
  $.fn.highlight.defaults = {
    foreground: "red",
    background: "yellow"
  };
// end of closure
})(jQuery);
```

Usage:


```js
(function($) {
  // Highlight all buttons with default colours
  jQuery(':button').highlight();

  // Highlight all buttons with green background
  jQuery(':button').highlight({background: "green"});

  // Set all further highlight() calls to have a green background
  $.fn.highlight.defaults.background = "green";
})(jQuery);
```

## jQuery UI Widgets

UI Widgets are jQuery Plugins with a bit more structure, targeted towards interactive elements. They require jQuery and
the core libraries in jQuery UI, so are generally more heavyweight if jQuery UI isn't already used elsewhere.

Main advantages over simpler jQuery plugins are:

*  Exposing public methods on DOM elements (incl. pseudo-private methods)
*  Exposing configuration and getters/setters on DOM elements
*  Constructor/Destructor hooks
*  Focus management and mouse interaction

See the [official developer guide](http://jqueryui.com/docs/Developer_Guide) and other
[tutorials](http://bililite.com/blog/understanding-jquery-ui-widgets-a-tutorial/) to get started.

Example: Highlighter

```js
(function($) {
  $.widget("ui.myHighlight", {
    getBlink: function () {
      return this._getData('blink');
    },
    setBlink: function (blink) {
      this._setData('blink', blink);
      if(blink) this.element.wrapInner('<blink></blink>');
      else this.element.html(this.element.children().html());
    },
    _init: function() {
      // grab the default value and use it
      this.element.css('background',this.options.background);
      this.element.css('color',this.options.foreground);
      this.setBlink(this.options.blink);
    }
  });
  // For demonstration purposes, this is also possible with jQuery.css()
  $.ui.myHighlight.getter = "getBlink";
  $.ui.myHighlight.defaults = {
    foreground: "red",
    background: "yellow",
    blink: false
  };
})(jQuery);
```

Usage:


```js
(function($) {
  // call with default options
  $(':button').myHighlight();

  // call with custom options
  $(':button').myHighlight({background: "green"});

  // set defaults for all future instances
  $.ui.myHighlight.defaults.background = "green";

  // Adjust property after initialization
  $(':button').myHighlight('setBlink', true);

  // Get property
  $(':button').myHighlight('getBlink');
})(jQuery);
```

### jQuery.Entwine

jQuery.entwine is a third-party plugin, from its documentation:
"A basic desire for jQuery programming is some sort of OO or other organisational method for code. For your
consideration, we provide a library for entwineUI style programming. In entwineUI you attach behavioral code to DOM
objects. entwine extends this concept beyond what is provided by other libraries to provide a very easy to use system
with class like, ploymorphic, namespaced properties."

Use jQuery.entwine when your code is likely to be customised by others, for example for most work in the CMS interface.
It is also suited for more complex applications beyond a single-purpose plugin.

Example: Highlighter


```js
(function($) {
  $(':button').entwine({
    Foreground: 'red',
    Background: 'yellow',
    highlight: function() {
      this.css('background', this.getBackground());
      this.css('color', this.getForeground());
    }
  });
})(jQuery);
```

Usage:


```js
(function($) {
  // call with default options
  $(':button').entwine().highlight();

  // set options for existing and new instances
  $(':button').entwine().setBackground('green');

  // get property
  $(':button').entwine().getBackground();
})(jQuery);
```

This is a deliberately simple example, the strength of jQuery.entwine over simple jQuery plugins lies in its public
properties, namespacing, as well as its inheritance based on CSS selectors. Please see the [project
documentation](http://github.com/hafriedlander/jquery.entwine/tree/master) for more complete examples.

When working in the CMS, the CMS includes the jQuery.entwine inspector. Press Ctrl+` to bring down the inspector.
You can then click on any element in the CMS to see which entwine methods are bound to any particular element.

## Architecture and Best Practices

### Keep things simple

Resist the temptation to build "cathedrals" of complex interrelated components.  In general, you can get a lot done in
jQuery with a few lines of code.  Your jQuery code will normally end up as a series of event handlers applied with `jQuery.on()` or jQuery.entwine, rather than a complex object graph.

### Don't claim global properties

Global properties are evil. They are accessible by other scripts, might be overwritten or misused. A popular case is the `$` shortcut in different libraries: in PrototypeJS it stands for `document.getElementByID()`, in jQuery for `jQuery()`.


```js
// you can't rely on '$' being defined outside of the closure
(function($) {
  var myPrivateVar; // only available inside the closure
  // inside here you can use the 'jQuery' object as '$'
})(jQuery);
```

You can run `[jQuery.noConflict()](http://docs.jquery.com/Core/jQuery.noConflict)` to avoid namespace clashes.
NoConflict mode is enabled by default in the Silverstripe CMS javascript.

### Initialize at document.ready

You have to ensure that DOM elements you want to act on are loaded before using them. jQuery provides a wrapper around
the `window.onload` and `document.ready` events.


```js
// DOM elements might not be available here
$(document).ready(function() {
  // The DOM is fully loaded here
});
```

See [jQuery FAQ: Launching Code on Document
Ready](http://docs.jquery.com/How_jQuery_Works#Launching_Code_on_Document_Ready).

### Bind events "live"

jQuery supports automatically reapplying event handlers when new DOM elements get inserted, mostly through Ajax calls.
This "binding" saves you from reapplying this step manually.

Caution: Only applies to certain events, see the [jQuery.on() documentation](http://api.jquery.com/on/#direct-and-delegated-events).

Example: Add a 'loading' classname to all pressed buttons

```js
// manual binding, only applies to existing elements
$('input[[type=submit]]').on('click', function() {
  $(this).addClass('loading');
});

// binding, applies to any inserted elements as well
$('.cms-container').on('click', 'input[[type=submit]]', function() {
  $(this).addClass('loading');
});
```

### Assume Element Collections

jQuery is based around collections of DOM elements, the library functions typically handle multiple elements (where it
makes sense). Encapsulate your code by nesting your jQuery commands inside a `jQuery().each()` call.


```js
$('div.MyGridField').each(function() {
  // This is the over code for the tr elements inside a GridField.
  $(this).find('tr').hover(
    // ...
  );
});
```

### Use plain HTML and jQuery.data() to store data

The DOM can make javascript configuration and state-keeping a lot easier, without having to resort to javascript
properties and complex object graphs.

Example: Simple form change tracking to prevent submission of unchanged data

Through CSS properties


```js
$('form :input').bind('change', function(e) {
  $(this.form).addClass('isChanged');
});
$('form').bind('submit', function(e) {
  if($(this).hasClass('isChanged')) return false;
});
```

Through jQuery.data()

```js
$('form :input').bind('change', function(e) {
  $(this.form).data('isChanged', true);
});
$('form').bind('submit', function(e) {
  alert($(this).data('isChanged'));
  if($(this).data('isChanged')) return false;
});
```

See [interactive example on jsbin.com](http://jsbin.com/opuva)

You can also use the [jQuery.metadata Plugin](http://docs.jquery.com/Plugins/Metadata/metadata) to serialize data into
properties of DOM elements. This is useful if you want to encode element-specific data in markup, for example when
rendering a form element through the Silverstripe CMS templating engine.

Example: Restricted numeric value field

```ss
<input type="text" class="restricted-text {min:4,max:10}" />
```

```js
$('.restricted-text').bind('change', function(e) {
  if(
    e.target.value < $(this).metadata().min
    || e.target.value > $(this).metadata().max
  ) {
    alert('Invalid value');
    return false;
  }
});
```

See [interactive example on jsbin.com](http://jsbin.com/axafa)

### Return HTML/JSON and HTTPResponse class for AJAX responses

Ajax responses will sometimes need to update existing DOM elements, for example refresh a set of search results.
Returning plain HTML is generally a good default behaviour, as it allows you to keep template rendering in one place (in
Silverstripe CMS PHP code), and is easy to deal with in JavaScript.

If you need to process or inspect returned data, consider extracting it from the loaded HTML instead (through id/class
attributes, or the jQuery.metadata plugin). For returning status messages, please use the HTTP status-codes.

Only return evaluated JavaScript snippets if unavoidable. Most of the time you can just pass data around, and let the
clientside react to changes appropriately without telling it directly through JavaScript in AJAX responses. Don't use
the [Form](api:SilverStripe\Forms\Form) Silverstripe CMS class, which is built solely around
this inflexible concept.

Example: Autocomplete input field loading page matches through AJAX

Template:


```ss
<ul>
<% loop $Results %>
  <li id="Result-$ID">$Title</li>
<% end_loop %>
</ul>
```

PHP:


```php
use SilverStripe\Control\HTTPResponse;
use SilverStripe\View\ViewableData;

class MyController 
{
  public function autocomplete($request) 
  {
    $results = Page::get()->filter("Title", $request->getVar('title'));
    if(!$results) return new HTTPResponse("Not found", 404);

    // Use HTTPResponse to pass custom status messages
    $this->getResponse()
      ->setStatusCode(200)
      ->addHeader('X-Status', "Found " . $results->Count() . " elements");

    // render all results with a custom template
    $vd = new ViewableData();
    return $vd->customise([
      "Results" => $results
    ])->renderWith('AutoComplete');
  }
}
```

HTML


```ss
<form action"#">
  <div class="autocomplete {url:'MyController/autocomplete'}">
    <input type="text" name="title" />
    <div class="results" style="display: none;">
  </div>
  <input type="submit" value="action_autocomplete" />
</form>
```

JavaScript:


```js
$('.autocomplete input').on('change', function() {
  var resultsEl = $(this).siblings('.results');
  resultsEl.load(
    // get form action, using the jQuery.metadata plugin
    $(this).parent().metadata().url,
    // submit all form values
    $(this.form).serialize(),
    // callback after data is loaded
    function(data, status) {
      resultsEl.show();
      // get all record IDs from the new HTML
      var ids = jQuery('.results').find('li').map(function() {
        return $(this).attr('id').replace(/Record\-/,'');
      });
    }
  );
});
```

Although they are the minority of cases, there are times when a simple HTML fragment isn't enough.  For example, if you
have server side code that needs to trigger the update of a couple of elements in the CMS left-hand tree, it would be
inefficient to send back the HTML of entire tree. Silverstripe CMS can serialize to and from JSON (see the [Convert](api:SilverStripe\Core\Convert) class), and jQuery deals very well with it through
[jQuery.getJSON()](http://docs.jquery.com/Ajax/jQuery.getJSON#urldatacallback), as long as the HTTP content-type is
properly set.

### Use events and observation to link components together

The philosophy behind this javascript guide is **component driven development**: your javascript should be structured as
a set of components that communicate. Event handlers are a great way of getting components to community, as long as
two-way communication isn't required.  Set up a number of custom event names that your component will trigger.  List
them in the component documentation comment.

jQuery can bind to DOM events and trigger them through custom code. It can also
[trigger custom events](http://docs.jquery.com/Events/trigger), and supports [namespaced
events](http://docs.jquery.com/Namespaced_Events).

Example: Trigger custom 'validationfailed' event on form submission for each empty element


```js
$('form').bind('submit', function(e) {
  // $(this) refers to form
  $(this).find(':input').each(function() {
    // $(this) in here refers to input field
    if(!$(this).val()) $(this).trigger('validationfailed');
  });
  return false;
});

// listen to custom event on each <input> field
$('form :input').bind('validationfailed',function(e) {
  // $(this) refers to input field
  alert($(this).attr('name'));
});
```

See [interactive example on jsbin.com](http://jsbin.com/ipeca).

Don't use event handlers in the following situations:

*  If two-way communication is required, for example, calling an method in another component, which returns data that
you then use.  Event handlers can't have return values.
*  If specific execution order is required.  Event handlers are executed in parallel, which makes it difficult to know
the exact order in which code in different threads will execute.  If the execution order is likely to cause problems, it
is better to use a code structure that is executed sequentially. An example might be two events modifying the same piece
of the DOM.

### Use callbacks to allow customizations

Callbacks are similar to events in that other components can ask your component to execute a piece of code.  The
advantage is that they lack the two problems listed in bullets just above. The disadvantage of callbacks is that you
need to define an custom API for configuring the callbacks; whereas, event observation is a jQuery provided API that
leaves components very loosely coupled.

### Use jQuery.entwine to define APIs as necessary

By default, most of your JavaScript methods will be hidden in closures like a jQuery plugin, and are not accessible from
the outside. As a best practice, each jQuery plugin should only expose one method to initialize and configure it. If you
need more public methods, consider using either a jQuery UI Widget, or define your behaviour as jQuery.entwine rules
(see above).

### Write Documentation

Documentation in JavaScript usually resembles the JavaDoc standard, although there is no agreed standard. Due to the
flexibility of the language it can be hard to generate automated documentation, particularly with the predominant usage
of closure constructs in jQuery and jQuery.entwine.

To generate documentation for Silverstripe CMS code, use [JSDoc toolkit](http://code.google.com/p/jsdoc-toolkit/) (see
[reference of supported tags](http://code.google.com/p/jsdoc-toolkit/wiki/TagReference)). For more class-oriented
JavaScript, take a look at the [jsdoc cookbook](http://code.google.com/p/jsdoc-toolkit/wiki/CookBook). The `@lends`
and `@borrows` properties are particularly useful for documenting jQuery-style code.

JSDoc-toolkit is a command line utility, see [usage](http://code.google.com/p/jsdoc-toolkit/wiki/CommandlineOptions).

Example: jQuery.entwine

```js
/**

 * Available Custom Events:
 * <ul>
 * <li>ajaxsubmit</li>
 * <li>validate</li>
 * <li>reloadeditform</li>
 * </ul>
 *
 * @class Main LeftAndMain interface with some control panel and an edit form.
 * @name ss.LeftAndMain
 */
$('.LeftAndMain').entwine('ss', function($){
  return/** @lends ss.LeftAndMain */ {
    /**

     * Reference to some property
     * @type Number
     */
    MyProperty: 123,

    /**

     * Renders the provided data into an unordered list.
     *
     * @param {Object} data
     * @param {String} status
     * @return {String} HTML unordered list
     */
    publicMethod: function(data, status) {
      return '<ul>'
        + /...
        + '</ul>';
    },

    /**

     * Won't show in documentation, but still worth documenting.
     *
     * @return {String} Something else.
     */
    _privateMethod: function() {
      // ...
    }
  };
]]);
```

### Unit Testing

It is important to verify that your code actually does what it says, and the best way to ensure this are **automated
tests**. For jQuery, we use two different tools with different uses: **unit testing** with
[QUnit](http://docs.jquery.com/QUnit) (also used by the jQuery team for the core libraries), and **behaviour driven
testing** with [JSpec](http://visionmedia.github.com/jspec/). There are overlaps between the two solutions, if in doubt
start with JSpec, as it provides a much more powerful testing framework.

Example: QUnit test (from [jquery.com](http://docs.jquery.com/QUnit#Using_QUnit)):

```js
test("a basic test example", function() {
  ok( true, "this test is fine" );
  var value = "hello";
  equals( "hello", value, "We expect value to be hello" );
});
```

Example: JSpec Shopping cart test (from [visionmedia.github.com](http://visionmedia.github.com/jspec/))

```
describe 'ShoppingCart'
  before_each
    cart = new ShoppingCart
  end
  describe 'addProduct'
    it 'should add a product'
      cart.addProduct('cookie')
      cart.addProduct('icecream')
      cart.should.have 2, 'products'
    end
  end
end
```

## Related

* [Unobtrusive Javascript](http://www.onlinetools.org/articles/unobtrusivejavascript/chapter1.html)
* [Quirksmode: In-depth Javascript Resources](http://www.quirksmode.org/resources.html)
