# API Reference

## REQUIRE JS

Modules! Hit them from the router which is really just backbone.router.extend!







## Models & Collections

Both models and collections are relatively untouched... Enhances `Backbone.Model` and `Backbone.Collection` with the concept of whether or not the model is populated and whether or not it should be automatically fetched. Note that when passing a model or collection to `view.setModel` or `view.setCollection` it must be an instance of `Thorax.Model` or `Thorax.Collection` and not `Backbone.Model` or `Backbone.Collection`, respectively.

### isEmpty *model.isEmpty()* *collection.isEmpty()*

Used by the `empty` helper. In a collection the implementations of `isEmpty` and `isPopulated` differ, but in a model `isEmpty` is an alias for `!isPopulated`. 

For collections, Used by the `empty` helper and the `emptyTemplate` and `emptyItem` options of a `CollectionView` to check whether a collection is empty. A collection is only treated as empty if it `isPopulated` and zero length.


### isPopulated *model.isPopulated()* *collection.isPopulated()*

Used by `setModel` to determine whether or not to fetch the model. The default implementation checks to see if any keys that are not `id` and are not default values have been set.

Used by `setCollection` to determine whether or not to fetch the collection.












## Thorax.View

### A NARRATIVE WILL GO HERE CONCERNING VIEWS IN THORAX, SUPER IMPORTANT CONTEXT

`Thorax.View` provides additive functionality over `Backbone.View` but breaks compatibility in one imporant way in that it does not use an `options` object. All properties passed to the constructor become available on the instance:

    var view = new Thorax.View({
      key: "value"
    });
    view.key === "value"

By default all instance properties are available in the template context. So when setting a key on the view it will by default be available in the template.


### template *view.template*

Assign a template to a view. This may be a string or a function which recieves a single `context` argument and returns a string. If the view has a `name` and a template of the same `name` is available the `template` will be auto-assigned.

    new Thorax.View({
      template: Handlebars.compile("{{key}}")
    });

### context *view.context()*

Used by `render` to determine what attributes are available in the view's `template`. The default context function returns `this` + `this.model.attributes` if a `model` is present on the view. The `context` method may be overriden to provide a custom context:

    new Thorax.View({
      template: Handlebars.compile('{{key}}'),
      context: function() {
        return _.defaults(this.model.attributes, {
          key: 'value'
        });
      }
    });

### appendTo *view.appendTo(element)*

Appends the view to a given `element` which may be a CSS selector or DOM element. `ensureRendered` will be called and a `ready` event will be triggered. This is the preferred way to append your outer most view onto a page.

### children *view.children*

A hash of child view's indexed by `cid`. Child views may become attached to the parent with the `view` helper or may be automatically attached `HelperView` instances created by helpers created with `regsterViewHelper` (such as the `collection` and `empty` helpers).

### parent *view.parent*

If a view was embedded inside another with the `view` helper, or a generated `HelperView` (for instance the `collection` or `empty` helpers) it will have a `parent` view attribute. In the case of `HelperView`s, the `parent` will be the view that declared the helper in its template.

### retain *view.retain([owner])*

Prevents a view from being destroyed if it would otherwise be. If a parent is destroyed all it's children will be destroyed, or if it was previously passed to `setView`

Given the code below:

    a.retain();
    Application.setView(a);
    Application.setView(b);
    Application.setView(c);

`b` will be destroyed, and `a` will not be.

When the optional `owner` parameter is passed, the retain reference count will automatically be reduced when the owner view is destroyed.

### release *view.release()*

Release a view that was previously retained. If `release` is called and the view has a reference count of zero it will be destroyed, which will release all children, remove all events, unbind all models and collections, call `remove` and trigger the `destroyed` event.

`release` is usally called automatically if a view was attached to a `LayoutView` with the `setView` method, and another view is then passed to `setView`.

Generally this method is not needed unless you are `retain`ing views.

### setModel *view.setModel(model [,options])*

Setting `model` in the construtor will automatically call `setModel`, so the following are equivelent:

    var view = new Thorax.View({
      model: myModel
    });
    // identical functionality as above
    view.setModel(myModel);

Sets the `model` attribute of a view then attempts to fetch the model if it has not yet been populated. Once set the default `context` implementation will merge the model's `attributes` into the context, so any model attributes will automatically become available in a template. In addition any events declared via `view.on({model: events})` will be bound to the model with `listenTo`.

Accepts any of the following options:

- **fetch** - Boolean, whether to fetch the model when it is set, defaults to true.
- **success** - Callback on fetch success, defaults to noop
- **render** - Render on the view on model:change? Defaults to undefined
  - `true` : Always render on change
  - `false` : Never render on change
  - `undefined` : Rerender if we have already been rendered
- **populate** - Call `populate` with the model's attributes when it is set? Defaults to true.
  - Pass `populate: {children: false}` to prevent child views from having their inputs populated.
  - Pass `populate: {context: true}` to populate using using the view's context rather than directly populating from the model's attributes.
- **errors** - When the model triggers an `error` event, trigger the event on the view? Defaults to true

### setCollection *view.setCollection(collection [,options])*

Setting `collection` in the construtor will automatically call `setCollection`, so the following are equivelent:

    var view = new Thorax.View({
      collection: myCollection
    });
    // identical functionality as above
    view.setCollection(myCollection);

Sets the `collection` attribute of a view then attempts to fetch the collection if it has not yet been populated. In addition any events declared via `view.on({collection: events})` will be bound to the collection with `listenTo`.

Accepts any of the following options:

- **render** - Whether to render the collection if it is populated, or render it after it has been loadedundefined
  - `true` : Always render on change
  - `false` : Never render on change
  - `undefined` : Rerender if we have already been rendered
- **fetch** - Whether or not to try to call `fetch` on the collection if `shouldFetch` returns true
- **success** - Callback on fetch success, defaults to noop
- **errors** - Whether or not to trigger an `error` event on the view when an `error` event is triggered on the collection

Note that while any view may bind a collection only a `CollectionView` will actually render a collection. A regular `Thorax.View` may declare a `collection` helper which in turn will generate and embed a `CollectionView`.

### $.view *$(event.target).view([options])*

Get a reference to the nearest parent view. Pass `helper: false` to options to exclude `HelperView`s from the lookup. Useful when registering DOM event handlers:

    $(event.target).view();

### $.model *$(event.target).model([view])*

Get a reference to the nearest bound model. Can be used with any `$` object but most useful in event handlers.

    $(event.target).model();

A `view` may be optionally passed to limit the lookup to a specific view.

### $.collection *$(event.target).collection([view])*

Get a reference to the nearest bound collection. Can be used with any `$` object but most useful in event handlers.

    $(event.target).collection();

A `view` may be optionally passed to limit the lookup to a specific view.








## Templates

### Templating in a Thorax App: Handlebars
Templating definition. Why it's used, what it does, why handlebars and other options like underscore templates that were not chosen. Role of templating in JS one page apps. Diagram. Remember... templates, application.handlebars... also, model attribute access inside of templates, and inside of the collection helper listed below. Awesome selling point!

### view *{{view name [options]}}*

Embed one view in another. The first argument may be the name of a new view to initialize or a reference to a view that has already been initialized.

    {{view "path/to/view" key="value"}}
    {{view viewInstance}}

If a block is specified it will be assigned as the `template` to the view instance:

    {{#view viewInstance}}
      viewInstance will have this block
      set as its template property
    {{/view}}

### collection *{{collection [collection] [options...]}}*

Creates and embeds a `CollectionView` instance, updating when items are added, removed or changed in the collection. If a block is passed it will be used as the `item-template`, which will be called with a context of the `model.attributes` for each model in the collection.

    {{#collection tag="ul"}}
      <li>{{modelAttr}}</li>
    {{/collection}}

Options may contain `tag`, `class`, `id` and the following attributes which will map to the generated `CollectionView` instance:

- `item-template` &rarr; `itemTemplate`
- `item-view` &rarr; `itemView`
- `empty-template` &rarr; `emptyTemplate`
- `empty-view` &rarr; `emptyView`
- `loading-template` &rarr; `loadingTemplate`
- `loading-view` &rarr; `loadingView`
- `item-context` &rarr; `itemContext`
- `item-filter` &rarr; `itemFilter`

Any of the options can be specified as variables in addition to strings:

    {{collection item-view=itemViewClass}}

By default the collection helper will look for `this.collection`, but if your view contains multiple collections a collection argument may be passed:

    {{collection myCollection}}

When rendering `this.collection` many properties will be forwarded from the view that is declaring the collection helper to the generated `CollectionView` instance:

- `itemTemplate`
- `itemView`
- `itemContext`
- `itemFilter`
- `emptyTemplate`
- `emptyView`
- `loadingTemplate`
- `loadingView`
- `loadingPlacement`

As a result the following two views are equivelenet:

    // render with collection helper, collection
    // properties are forwarded
    var view = new Thorax.View({
      collection: new Thorax.Collection(),
      itemView: MyItemClass,
      itemContext: function(model, i) {
        return model.attributes;
      },
      template: Handlebars.compile('{{collection}}')
    });

    // directly create collection view, no property
    // forwarding will occur
    var view = new Thorax.View({
      collectionView: new Thorax.CollectionView({
        collection: new Thorax.Collection(),
        itemView: MyItemClass
        itemContext: function(model, i) {
          return model.attributes;
        }
      }),
      template: Handlebars.compile('{{view collectionView}}')
    });

### button *{{#button methodName [htmlAttributes...]}}*

Creates a `button` tag that will call the specified methodName on the view when clicked. Arbitrary HTML attributes can also be specified.

    {{#button "methodName" class="btn"}}Click Me{{/button}}

The tag name may also be specified:

    {{#button "methodName" tag="a" class="btn"}}A Link{{/button}}

A `trigger` attribute will trigger an event on the declaring view:

    {{#button trigger="eventName"}}Button{{/button}}

A button can have both a `trigger` attribute and a method to call:

    {{#button "methodName" trigger="eventName"}}Button{{/button}}

The method may also be specified as a `method` attribute:

    {{#button method="methodName"}}Button{{/button}}

### link *{{#link url [htmlAttributes...]}}*

Creates an `a` tag that will call `Backbone.history.navigate()` with the given url when clicked. Passes the `url` parameter to the `url` helper with the current context. Do not use this method for creating external links. Like the `url` helper, multiple arguments may be passed as well as an `expand-tokens` option.

    {{#link "articles/{{id}}" expand-tokens=true class="article-link"}}Link Text{{/link}}

To call a method from an `a` tag use the `button` helper:

    {{#button "methodName" tag="a"}}My Link{{/button}}

Like the `button` helper, a `trigger` attribute may be specified that will trigger an event on the delcaring view in addition to navigating to the specified url:

    {{#link "articles" id trigger="customEvent"}}Link Text{{/link}}

The href attribute is required but may also be specified as an attribute:

    {{#link href="articles/{{id}}" expand-tokens=true}}Link Test{{/link}}

### empty *{{#empty [modelOrCollection]}}*

A conditional helper much like `if` that calls `isEmpty` on the specified object. In addition it will bind events to re-render the view should the object's state change from empty to not empty, or visa versa.

    {{#empty collection}}
      So empty!
    {{else}}
      {{#collection}}{{/collection}}
    {{/empty}}

To embed a row within a `collection` helper if it the collection is empty, specify an `empty-view` or `empty-template`. Or use the `else` block of the `collection` helper:

    {{#collection tag="ul"}}
      <li>Some very fine data</li>
    {{else}}
      <li>So very empty</li>
    {{/collection}}

### template *{{template name [options]}}*

Embed a template inside of another, as a string. An associated view (if any) will not be initialized. By default the template will be called with the current context but extra options may be passed which will be added to the context.

    {{template "path/to/template" key="value"}}

If a block is used, the template will have a variable named `@yield` available that will contain the contents of the block.

    {{#template "child"}}
      content in the block will be available in a variable
      named "@yield" inside the template "child"
    {{/template}}

This is useful when a child template will be called from multiple different parents.










## Thorax.CollectionView

NARRATIVE HERE... Most of the time, `Thorax.Collectionview` will be called for you when you do {{#collection theNameOfACollection}} but sometimes you want manual control... IN OUR NOTES we talked about the alex wishlist example?? and mentioned that this would be done in the view instance like var view new application.views collection:

here begins this is the old api...

A class that renders an `itemTemplate` or `itemView` for each item in a `collection` passed to it in its constructor, or via `setCollection`. The view will automatically update when items are added, removed or changed.

The `collection` helper will automatically create and embed a `CollectionView` instance for you. If programatic access to the view's methods are needed (for instance calling `appendItem` or specifying an `itemFilter`) it's best to create a `CollectionView` directly and embed it with the `view` helper as you would any other view.

### itemTemplate *view.itemTemplate*

A template name or template function to use when rendering each model. If using the `collection` helper the passed block will become the `itemTemplate`. Defaults to `view.name + '-item'`

### itemView *view.itemView*

A view class to be initialized for each item. Can be used in conjunction with `itemTemplate`.

### itemContext *view.itemContext(model, index)*

A function in the declaring view to specify the context for an `itemTemplate`, recieves model and index as arguments. `itemContext` will not be used if an `itemView` is specified as the `itemView`'s own `context` method will instead be used.

A collection helper may specify a specific function to use as the `itemContext` if there are multiple collections in a view:

    {{#collection todos item-context="todosItemContext"}}

### itemFilter *view.itemFilter(model, index)*

A method, which if present will filter what items are rendered in a collection. Recieves `model` and `index` and must return boolean. The filter will be applied when models' fire a change event, or models are added and removed from the collection. To force a collection to re-filter, trigger a `filter` event on the collection.

Items are hidden and shown with `$.hide` and `$.show` rather than being removed or appended. In performance critical views with large collections consider filtering the collection before it is passed to the view or on the server.

A collection helper may specify a specific function to use as the `itemFilter` if there are multiple collections in a view:

    {{#collection todos item-filter="todosItemFilter"}}

### emptyTemplate *view.emptyTemplate*

A template name or template function to display when the collection is empty. If used in a `collection` helper the inverse block will become the `emptyTemplate`. Defaults to `view.name + '-empty'`

### emptyView *view.emptyView*

A view class to create an instance of when the collection is empty. Can be used in conjunction with `emptyTemplate`. Empty view exists because empty templates can be complex - for instance, an ampty car t may have a collection of suggested products in it. 

### appendItem *view.appendItem(modelOrView [,index] [,options])*

Append a model (which will used to generate a new `itemView` or render an `itemTemplate`) or a view at a given index in the `CollectionView`. If passing a view as the first argument `index` may be a model which will be used to look up the index.

By default this will trigger a `rendered:item` event, `silent: true` may be passed in the options hash to prevent this. To also prevent the appeneded item from being filtered if an `itemFilter` is present pass `filter: false` in the options hash.

### removeItem *view.removeItem(model)*

Remove an item from the view.

### updateItem *view.updateItem(model)*

Equivelent to calling `removeItem` then `appendItem`. Note that this is mainly meant to cover edge cases, by default changing a model will update the needed item (whether using `itemTemplate` or `itemView`).










# PAGE LAYOUT IN THORAX

In any given single page application, you may have different persistent content areas that will update and change independently - think of a sidebar and a main area (the persistent header and footer are typically taken care of in the application.handlebars file). There are a number of important things to understand about layouts. The first is that there's a default. By default, in application.handlebars, there is a single {{layout-element}} that will be nuked each time you call setView from your router, and the view's associated template will be put in its place. 

If you want manual control, you ... INSERT NARRATIVE HERE.


## Thorax.LayoutView

A view to contain a single other view which will change over time, (multi-pane single page applications for instance), triggering a series of events . By default this class has no template. If one is specified use the `layout` helper to determine where `setView` will place a view. A `Thorax.LayoutView` is a subclass of `Thorax.View` and may be treated as a view in every regard (i.e. embed multiple `LayoutView` instances in a parent view with the `view` helper).

### setView *view.setView(view [,options])*

Set the current view on the `LayoutView`, triggering `activated`, `ready` and `deactivated` events on the current and previous view during the lifecycle. `ensureRendered` is called on views passed to `setView`. By default `destroy` is called on the previous view when the new view is set.

### getView *view.getView()*

Get the current view that was previously set with `setView`.















# APPENDIX APPENDIX APPENDIX APPENDIX have some visual reference that this is different, and organize it as such. not such big fonts and weights, much more reference-y





## Catalog of Built-in Thorax Events

### rendered *rendered ()*

Triggered on a view when the `rendered` method is called.

### child *child (instance)*

Triggered on a view every time a child view is appened into the view with the `view` helper.

### ready *ready (options)*

Triggered when a view is append to the DOM with `appendTo` or when a view is appeneded to a `LayoutView` via `setView`. Setting focus and other behaviors that depend on the view being present in the DOM should be handled in this event.

This event propagates to all children, including children that will be bound after the view is created. `options` will contain a `target` view, which is the view that triggered the event.

### activated *activated (options)*

Triggered on a view immediately after it was passed to a `LayoutView`'s `setView` method. Like `ready` this event propagates to children and the `options` hash will contain a `target` view.

### deactivated *deactivated (options)*

Triggered on a view when it was previously passed to the `setView` method on a `LayoutView`, and then another view is passed to `setView`. Triggered when the current view's `el` is still attached to the parent. Like `ready` this event propagates to children and the `options` hash will contain a `target` view.

### destroyed *destroyed ()*

Triggered on a view when the `release` method is called and the reference count is zero. Useful for implementing custom view cleanup behaviors. `release` will be also be called if it was previously passed to the `setView` method on a `LayoutView`, and then another view is passed to `setView`.

### change:view:start *change:view:start (newView [,oldView] ,options)*

Trigged on a `Thorax.LayoutView` immediately after `setView` is called.

### change:view:end *change:view:end (newView [,oldView] ,options)*

Trigged on a `Thorax.LayoutView` after `setView` is called, the old view has been destroyed (if present) and the new view has been attached to the DOM and had its `ready` event triggered.

### helper *helper (name [,args...] ,helperView)*

Triggered on a view when a view helper (such as `collection`, `empty`, etc) create a new `HelperView` instance.

### helper:name *helper:name ([,args...] ,helperView)*

Triggered on a view when a given view helper creates a new `HelperView` instance.

    {{#collection cats}}{{/collection}}

    view.on('helper:collection', function(collection, collectionView) {

    });

### serialize *serialize (attributes)*

Triggered on a view when `serialize` is called, before `validateInput` is called with the serialized attributes.

### validate *validate (attributes, errors)*

Triggered on a view when `serialize` is called, passed an an attributes hash and errors array after `validateInput` is called. Use in combination with the `invalid` event to display and clear errors from your views.

    Thorax.View.on({
      validate: function(attributes, errors) {
        //clear previous errors if present
      },
      invalid: function(errors) {
        errors.forEach(function(error) {
          //lookup input by error.name
          //display error from error.message
        });
      }
    });

### invalid *invalid (errors)*

Triggered on a view when `serialize` is called, if validateInput returned an array with any errors.

### populate *populate (attributes)*

Triggered on a view when `populate` is called. Passed a hash containing the attributes that the view will be populated with.

### rendered:collection *rendred:collection (collectionView, collection)*

Triggered on a `CollectionView` or a the view calling the `collection` helper every time `render` is called on the `CollectionView`.

### rendered:item *rendered:item (collectionView, collection, model, itemElement, index)*

Triggered on a `CollectionView` or a the view calling the `collection` helper every time an item is rendered in the `CollectionView`.

### rendered:empty *rendered:empty (collectionView, collection)*

Triggered on a `CollectionView` or a the view calling the `collection` helper every time the `emptyView` or `emptyTemplate` is rendered in the `CollectionView`.








## Event Enhancements

Thorax adds inheritable class events for all Thorax classes and significant enhancements to the Thorax.View event handling.

### Inheritable Events *ViewClass.on(eventName, callback)*

All Thorax classes have an `on` method to observe events on all instances of the class. Subclasses inherit their parents' event handlers. Accepts any arguments that can be passed to `viewInstance.on` or declared in the `events` hash.

    Thorax.View.on({
      'click a': function(event) {

      }
    });

### Model Events

When a model is bound to a view with `setModel` (automatically called by passing a `model` option in the constructor) any events on the model can be observed by the view in this way. For instance to observe any model `change` event when it is bound to any view:

    Thorax.View.on({
      model: {
        change: function() {
          // "this" will refer to the view
        }
      }
    });

### Collection Events

When a collection is bound to a view with `setCollection` (automatically called by passing a `collection` option in the constructor) any events on the collection can be observed by the view in this way. For instance to observe any collection `reset` event when it is bound to any view:

    Thorax.View.on({
      collection: {
        reset: function() {
          // "this" will refer to the view
        }
      }
    });

### View Events *view.events.viewEventName*

The `events` hash has been enhanced to allow view events to be registered along side DOM events:

    Thorax.View.extend({
      events: {
        'click a': function(event) {},
        rendered: function() {}
      }
    });

### DOM Events *view.on(eventNameAndSelector, callback [,context])*

The `on` method will now accept event strings in the same format as the events hash, for instance `click a`. Events separated by a space will still be treated as registering multiple events so long as the event name does not start with a DOM event name (`click`, `change`, `mousedown` etc).

DOM events observed in this way will only operate on the view itself. If the view embeds other views with the `view` helper that would match the event name and selector, they will be ignored. For instance declaring:

    view.on('click a', function(event) {})

Will only listen for clicks on `a` elements within the view. If the view has children that has `a` elements, this handler will not observe clicks on them.

DOM events may be prefixed with the special keyword `nested` which will apply the event to all elements in child views:

    view.on('nested click a', function() {})

Thorax will add an attribute to the event named `originalContext` that will be the `Element` object that would have been set as `this` had the handler been registered with jQuery / Zepto:

    $('a').on('click', function() {});
    view.on('click a', function(event) {
      // event.originalContext === what "this" would be in the
      // first handler
    });

### _addEvent *view._addEvent(eventParams)*

This method is never called directly, but can be specified to override the behavior of the `events` hash or any event arguments passed to `on`. For each event declared in either manner `_addEvent` will be called with a hash containing:

- type "view" || "DOM"
- name (DOM events will begin with ".delegateEvents")
- originalName
- selector (DOM events only)
- handler

All of the behavior described in this section is implemented via this method, so if overriding make sure to call `Thorax.View.prototype._addEvent` in your child view.








## Thorax.Util

### tag *Thorax.Util.tag(name, htmlAttributes [,content] [,context])*

Generate an HTML string. All built in HTML generation uses this method. If `context` is passed any Handlebars references inside of the htmlAttributes values will rendered with the context.

    Thorax.Util.tag("div", {
      id: "div-{{number}}"
    }, "content of the div", {
      number: 3
    });


## HTML Attributes

Thorax and its view helpers generate a number of custom HTML attributes that may be useful in debugging or generating CSS selectors to be used as arguments to `$` or to create CSS. The `*-cid` attributes are generally used only internally. See `$.model`, `$.collection` and `$.view` to get a reference to objects directly from the DOM. The `*-name` attributes will only be present if the given objects have a `name` property.</p>

<table class="table table-bordered table-striped">
  <thead>
    <tr>
      <th>Attribute Name</th>
      <th>Attached To</th>
    </tr>
  </thead>
  <tbody>
    <tr><td><code>data-view-cid</code></td><td>Every view instances' <code>el</code></td></tr>
    <tr><td><code>data-view-name</code></td><td>Same as above, only present on named views</td></tr>
    <tr><td><code>data-collection-cid</code></td><td>Element generated by the `collection helper`</td></tr>
    <tr><td><code>data-collection-name</code></td><td>Same as above, only present when the bound collection is named</td></tr>
    <tr><td><code>data-collection-empty</code></td><td>Set to "true" or "false" depending on whether the bound collection <code>isEmpty</code></td></tr>
    <tr><td><code>data-collection-element</code></td><td>Set by the <code>collection-element</code>, determines where a collection in a <code>CollectionView</code> will be rendered.</td></tr>
    <tr><td><code>data-model-cid</code></td><td>A view's <code>el</code> if a model was bound to the view or each item element inside of elements generated by the collection helper</td></tr>
    <tr><td><code>data-model-name</code></td><td>Same as above, only present if the model is named</td></tr>
    <tr><td><code>data-layout-cid</code></td><td>The element generated by the <code>layout</code> helper or <code>el</code> inside of a <code>LayoutView</code> or <code>ViewController</code> instance</td></tr>
    <tr><td><code>data-view-helper</code></td><td>Elements generated by various helpers including <code>collection</code> and <code>empty</code> from the collection plugin</td></tr>
    <tr><td><code>data-call-method</code></td><td>Elements generated by the <code>link</code> and <code>button</code> helpers</td></tr>
    <tr><td><code>data-trigger-event</code></td><td>Elements generated by the <code>link</code> and <code>button</code> helpers</td></tr>
  </tbody>
</table>

When creating CSS selectors it's recommended to use the generated attributes (especially `data-view-name`) rather than assigning custom IDs or class names for the sole purpose of styling.

    [data-view-name="my-view-name"] {
      border: 1px solid #ddd;
    }

## Error Handling

### onException *Thorax.onException(name, error)*

Bound DOM event handlers in Thorax are wrapped with a try / catch block, calling this function if an error is caught. This hook is provided primarily to allow for easier debugging in Android environments where it is difficult to determine the source of the error. The default error handler is simply:

    Thorax.onException = function(name, error) {
      throw error;
    };

Override this function with your own logging / debugging handler. `name` will be the event name where the error was thrown.
