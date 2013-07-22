spapp
=====

spapp is a template for writing single-page apps. It provides a good amount of modularism. Unfortunately, it's hard to async-load css correctly, so that's the only thing that has to be maintained in `index.html`. Everything else is loaded via the AMD loader:

`index.html`
```xml
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>spapp</title>
    <link rel="stylesheet" href="http://fonts.googleapis.com/css?family=Open+Sans" />
    <link rel="stylesheet" href="css/reset.css" />
    <link rel="stylesheet" href="css/main_view.css" />
    <link rel="stylesheet" href="css/rand_color_view.css" />
    <script data-main="js/main" src="js/require.js"></script>
  </head>
  <body>
    <!-- no need to put anything here -->
  </body>
</html>
```

Structure
---------

* __js__/ - require.js, main.js, libraries, and one js file for each view and each model
* __tmpl__/ - one or more template files for each view
* __sass__/ - one sass file for each view
* __css__/ - reset.css and files generated by sass

main.js
-------

The main file tells the AMD loader where the templates are and appends the `MainView` to the body when the document is ready.

`js/main.js`
```javascript
require.config({

  paths: {
    'tmpl': '../tmpl',
  },

});

require([

  'jquery',
  'main_view',

], function($, MainView) {

  'use strict';

  var mainView = new MainView();
  mainView.render();
  
  // Append the main view to the body when the document is ready.
  $(document).ready(function() {
    $('body').append(mainView.el);
  });

});
```

Views
-----

The views are backbone views, along with corresponding template and sass files. The templates are specified as dependencies ensuring that they are automatically loaded by the AMD loader. Views extend a `BaseView`, which is useful for app-wide utilities. As an example, a `MainView` is provided which contains a `RandColorView` as a subview:

`js/main_view.js`
```javascript
define([

  'jquery',
  'underscore',
  'base_view',
  'rand_color_view',
  'text!tmpl/main.tmpl',

], function($, _, BaseView, RandColorView, tmplText) {

  'use strict';

  var MainView = BaseView.extend({

    tmpl: _.template(tmplText),

    render: function() {
      this.setHTML(this.tmpl());

      // Create a RandColorView and insert it into the .content div.
      var randColorView = new RandColorView();
      randColorView.render();
      this.$('.content').append(randColorView.el);

      return this;
    },

  });

  return MainView;

});
```

Templates
---------

The templates are underscore templates, and the language is customized in the `BaseView` to ensure that the customization is available to all of the views.

`js/base_view.js`
```javascript
_.templateSettings = {
  interpolate: /\{\{(.+?)\}\}/g, // {{  verbatim  }}
  escape:      /\{\%(.+?)\%\}/g, // {%  escaped   %}
  evaluate:    /\[\[(.+?)\]\]/g, // [[  evaluated ]]
  variable:    'data'
};
```

SASS
----

Use the following command to auto-generate the css:

```console
$ sass --watch sass:css
```

Server
------

I use [hipster-serve](https://github.com/ryanlbrown/hipster-serve) during development, but any server that can serve a directory will work.
