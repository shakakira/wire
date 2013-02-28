# jQuery integration

Wire.js, like all cujo.js projects, integrates with other libraries and
frameworks, such as jQuery.

## jQuery $(selector)

Wire.js provides support for DOM querying via several plugins, including
one that uses jQuery's legendary `$(selector)` functionality.  For more 
information about DOM querying with jQuery, see 
[Working with the DOM](dom.md). 

## jQuery .on()

The full power of jQuery's `.on()` method is available in wire specs.
The [wire/jquery/on](connections.md#dom-events) plugin provides "high in the 
DOM" listening of DOM events.

## jQuery UI widgets

Wire.js has built-in support for [jQuery UI](http://api.jqueryui.com/) widgets
via the wire/jquery/ui plugin.  Other widget libraries that are built on top of 
jQuery UI, such as [wijmo](http://wijmo.com/) widgets, are supported, as well.

To create a jQuery UI widget, use the "widget" factory included in the 
wire/jquery/ui plugin.  The factory requires that you specify the widget's 
constructor ("type") and a DOM node.  You may specify options for the widget,
as well.

jQuery UI widgets don't have properties like normal Javascript objects.  
Therefore, wire.js's "properties" facet tries to be smart when you refer to a
property on a widget.  First, it checks to see if there is a function of the
same name on the widget and, if it finds one, assumes it's an *accessor 
function*.  For instance, "data", "val", "height", and "width" may be set or
get via properties.

If there is no accessor function with the given name, the plugin looks in the
widget's "options" collection.  If an option with the same name exists, the 
plugin gets or sets that option.  If no option exists with that name, the
plugin assumes the developer wishes to access an item in the widget's data
store, instead.

jQuery UI widgets enjoy a special feature that allows direct connections
between widgets by linking the widgets via automatically generated getters
and setters.  When the spec refers to a method whose name starts with "set"
or "get", and there is no *actual* method with that name, the plugin assumes
the method is a getter or a setter.

The following code example shows how to create and configure jQuery UI widgets
(wijmo widgets in this case) as well as how to connect them together via
automatically generated getters and setters.  This example uses a mediator
between the two widgets.  The mediator pattern is a recommended "best practice"
to simplify complex widget relationships, but isn't necessary in this very
simple example.

```js
define({

	// this is a wijmo wizard widget
	wizard: {
		widget: {
			// type of widget
			type: 'wijwizard',
			// where to create this widget in the dom
			node: { $ref: 'dom!pages' },
			// wizard widget options
			options: {
				navButtons: 'none'
			}
		}
	},

	// this is a wijmo pager widget
	pager: {
		widget: {
			// type of widget
			type: 'wijpager',
			// where to create it in the dom
			node: { $ref: 'dom!pager' },
			// pager widget options
			options: {
				pageCount: 3,
				pageIndex: 1,
				mode: 'numeric'
			}
		},
		on: {
			// when page index changes, tell mediator.
			// this could also be done with the pageIndexchanged option, but is
			// more compact when using the "on" facet.
			wijpagerpageindexchanged: 'mediator.pageChanged'
		}
	},
	
	// a mediator to coordinate changes in widgets. 
	mediator: {
		prototype: {
			// if the page changes, copy it from the pager to the wizard.
			// we are using automatically-generated setters and getters here.
			pageChanged: { compose: 'pager.getPageIndex | wizard.setActiveIndex' }
		},
		// at startup, set the wizard's page
		ready: 'pageChanged'
	},

	plugins: [
		{ module: 'wire/jquery/ui' },
		{ module: 'wire/jquery/on' },
		{ module: 'wire/jquery/dom' }
	]
	
});
```