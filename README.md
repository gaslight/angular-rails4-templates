# Angular Rails4 Templates

Adds your HTML templates into Angular's `$templateCache` using Rails asset pipeline. This gem is the updated version
of https://github.com/pitr/angular-rails-templates that supports Sprockets3 and Rails4.

It removes the need for AJAX calls to retrieve the templates (or for you to manually set them into the DOM).

## Usage

### 1. Add the Gem

In Gemfile

```ruby
gem 'angular-rails4-templates'
```

### 2. Include Templates in Rails Asset Pipeline

Then, in your `application.js` file, require `angular-rails4-templates` and your templates:

```javascript
//= require angularjs
// ...
//= require angular-rails4-templates
//
// Templates in app/assets/javascript/templates
//= require_tree ./templates
// OR
// Templates in app/assets/templates (but see step 5)
//= require_tree ../templates
```

Make sure to `require angular-rails4-templates` **before** you require your templates.

You'll need to use specific extensions to run the right preprocessor and load the resulting
compiled template into the angular template cache.

```
foo.nghtml
foo.html.nghaml
foo.html.ngslim
```
Caution: *`.ngslim` is no longer supported!*

Angular Rails Templates will try to load support for the following preprocessors if their gems are present:

| Extension | Required gem                                             |
|---------- |----------------------------------------------------------|
| .nghaml   | haml                                                     |
| .ngslim   | slim                                                     |

See [Advanced](#advanced-configuration) if you would like to use other markup languages.

### 3. Add a Dependency in your Angular Application Module

Your Angular module needs to depend on the `templates` module. (configurable, see [Advanced Configuration](#configuration-option-module_name))

```javascript
angular.module('myApplication', ['templates']);
```

### 4. Use your Templates

No matter what the source file extension is, your template's url will be  `#{base_name}.html`

For example:
```ruby
main.nghtml => main.html
widget.nghaml => widget.html
```

The templates can then be accessed via `templateUrl` as expected:

```javascript
// Template: app/assets/templates/yourTemplate.html.nghaml
{
  templateUrl: 'yourTemplate.html'
}
```

Or anything else that uses `$templateCache` or `$templateRequest`

```html
<div ng-include='"yourTemplate.html"'></div>
```

### 5. Avoid name collisions

If you have `app/assets/javascript/user.js` and `app/assets/templates/user.html`, the former one will actually hide the latter. This is due to how Rails asset pipeline sees asset files, both are served under `/assets/user.js`. Please use namespacing to combat this issue.

## Advanced Configuration

Angular Rails Templates has some configuration options that can be set inside `config/application.rb`

Here are their default values:
```ruby
# config/application.rb
config.angular_templates.module_name    = 'templates'
config.angular_templates.ignore_prefix  = %w(templates/)
config.angular_templates.inside_paths   = [Rails.root.join('app', 'assets')]
config.angular_templates.markups        = %w(erb str haml slim md)
config.angular_templates.htmlcompressor = false
```

### Configuration Option: `module_name`

This configures the module name that your templates will be placed into.
It is used to generate javascript like:

```javascipt
angular.module("<%= module_name %>")...
```

Although it is not recommended, you can set `module_name` to the name of your main application module and remove `require angular-rails4-templates` from your javascript manifest to have your templates directly injected into your app.

### Configuration Option: `ignore_prefix`

*If you place your templates in `app/assets/templates` this option is mostly useless.*

`ignore_prefix` will be stripped from the beginning of the `templateUrl` it reports to angularjs.

Since the default ignore_prefix is [`templates/`], any templates placed under `app/assets/javascripts/templates` will automatically have short names. If your templates are not in this location, you will need to use the full path to the template.

You can set `config.angular_templates.ignore_prefix` to change the default ignore prefix. Default is [`templates/`].


``` javascript
// Templates in: app/assets/javascripts/templates (default)
// ignore_prefix: templates/ (default)
{
  templateUrl: 'yourTemplate.html'
}
// This won't work:
{
  templateUrl: 'templates/yourTemplate.html'
}
```

``` javascript
// Templates in: app/assets/javascripts/my_app/templates (custom)
// ignore_prefix: templates/ (default)
{
  templateUrl: 'my_app/templates/yourTemplate.html'
}

// ignore_prefix: my_app/templates/ (custom)
{
  templateUrl: 'yourTemplate.html'
}
```


### Configuration Option: `inside_paths`

Templates only from paths matched by `inside_paths` will be used. By default anything under app/assets can be templates. This option is useful if you are using this gem inside an engine. Also useful if you DON'T want some files to be processed by this gem (see issue #88)


### Configuration Option: `markups`

Any markup that Tilt supports can be used, but you may need to add a gem to your Gemfile. See [Tilt](https://github.com/rtomayko/tilt) for a list of the supported markups and required libraries.

```ruby
# Gemfile
gem "asciidoctor"
gem "radius"
gem "creole"
gem "tilt-handlebars"

# config/application.rb
config.angular_templates.markups.push 'asciidoc', 'radius', 'wiki', 'hbs'
```
If you would like to use a non-standard extension or you would like to use a custom template, you just need to tell Tilt about it.

```ruby
# config/initializers/angular_rails_templates.rb
Tilt.register Tilt::HamlTemplate, 'nghaml'

# config/application.rb
config.angular_templates.markups.push 'nghaml'
```
Note: You would still need to use `foo`**`.html`**`.nghaml`


### Configuration Option: `htmlcompressor`

The [htmlcompressor gem](https://github.com/paolochiodi/htmlcompressor) is in alpha, not activly maintained, and has several known bugs. Beware if you are using windows. The `simple_boolean_attributes` option is known to mangle angular templates. It depends on a three-year-old version of yui-compressor. However, it does a good job of compressing html!

If you would like to use htmlcompressor add it to your Gemfile and Enable the configuration option.

```ruby
# Gemfile
gem 'htmlcompressor'
```

```ruby
# config/application.rb
config.angular_templates.htmlcompressor = true
```

You can also pass an options hash to `htmlcompressor` that will be directly passed to ```HtmlCompressor::Compressor.new```. See the [ruby project](https://github.com/paolochiodi/htmlcompressor#usage) or the [java project](https://code.google.com/p/htmlcompressor/#Compressing_HTML_and_XML_files_from_a_command_line) for descriptions of the options.

```ruby
# config/application.rb
config.angular_templates.htmlcompressor = {
  :remove_quotes => true
}
```


## License

MIT License. Copyright 2014 Pitr

## Authors & contributors

* Damien Mathieu <42@dmathieu.com>
* pitr <pitr.vern@gmail.com>
* Jeremy Ebler <jebler@gmail.com>
* Chris Nelson <chris@teamgaslight.com>
