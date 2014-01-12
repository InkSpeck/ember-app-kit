---
layout: default
title: "Using Modules & the Resolver"
permalink: using-modules.html
---

Rather than use AMD (Require.js) or CommonJS (Browserify) modules, apps built 
using the Ember App Kit use ES6 modules through the 
[ES6 module transpiler](https://github.com/square/es6-module-transpiler). This 
means that you can build your apps using syntax from future JavaScript versions, 
but output AMD modules that can be used by existing JavaScript libraries today.

If you've built Ember.js apps before, you're probably used to stuffing 
everything into a global namespace, following naming conventions so the app can
automatically resolve its dependencies: `App.FooRoute` would know 
to render `App.FooView` by default. Using the custom resolver, Ember App Kit 
applications have similar abilities, but using ES6 modules instead of a global 
namespace.

For example, this route definition in `app/routes/index.js`:

{% highlight js %}
var IndexRoute = Ember.Route.extend({
  model: function() {
    return ['red', 'yellow', 'blue'];
  }
});

export default IndexRoute;
{% endhighlight %}

Would result in a module called `routes/index`. Using the resolver, when Ember 
looks up the index route, it will find this module and use the object that it 
exports.

You can also export directly, i.e., without having to declare a variable:

{% highlight js %}
export default Ember.Route.extend({
  model: function() {
    return ['red', 'yellow', 'blue'];
  }
});
{% endhighlight %}

Of course, while automatic resolving is awesome, you can always manually 
require dependencies with the following syntax:

{% highlight js %}
import MyModel from "appkit/models/foo-model";
{% endhighlight %}

Which will load the `default` export (aliased as `MyModel`) from 
`app/models/foo-model.js`.

Note that the name of the variable used in the exported module doesn't have any 
influence on the resolver. It's the filename that is used to resolve modules. 
Similarly, you can give any name to the variable into which you import a module
when doing so manually; see how the module `foo-model` is assigned to variable 
`MyModel` in the example above.

Only the `default` export is supported, at the moment.


### Resolving from template helpers

Ember has several template helpers that are used to easily resolve and render 
views and their contexts within a template. The resolver works with these 
helpers, too:

`{% raw %}{{partial "foo"}}{% endraw %}` will render the template within `templates/foo.hbs`

`{% raw %}{{view "foo"}}{% endraw %}` will render the view within `views/foo.js`

`{% raw %}{{render "foo" "bar"}}{% endraw %}` will render the view within `views/foo.js` using the 
controller within `controllers/bar.js`


###	Resolving Handlebars helpers

Handlebars helpers will only be found automatically by the resolver if their 
name contains a dash (`reverse-word`, `translate-text`, etc.) This is the 
result of a choice that was made in Ember, to help both disambiguate properties 
from helpers, and to mitigate the performance hit of helper resolution for all 
bindings. The other option is to define only the function used by the helper 
and then load it explicitly, like so:

In `app/helpers/example.js`:

{% highlight js %}
export default function(value, options) {
  return value.toUpperCase();
};
{% endhighlight %}
	
In `app.js`:

{% highlight js %}
import exampleHelper from 'appkit/helpers/example';
Ember.Handlebars.registerBoundHelper('example', exampleHelper);
{% endhighlight %}

In `some-template.hbs`:

{% highlight html %}
{% raw %}
{{example "foo"}}
{% endraw %}
{% endhighlight %}

In this example, because the helper is loaded explicitely, it's the first 
argument to `registerBoundHelper` which makes the Handlebars renderer find it. 
The file name (`example.js`) and the name of the variable it's been imported 
into (`exampleHelper`) could have been anything.

