# Rendering

Rendering requires specifying a *template* and a *view*. 

Overview: Templates and Views
-----------------------------

A *template* may be either a string containing mustache markup, or a
string referencing a mustache template file. For simplicity, the suffix
of the file may be omitted, and a suffix will be appended; by default,
this is `.mustache`, but you may also specify your own custom suffix.

Templates will typically provide placeholders for variables, using a
double pair of braces (mustaches!) to enclose the variable name:

```html
Hello, {{name}}!
```

While additional [syntax](syntax.md) is available, this is the most
basic concept to understand.

By default, variable values are *escaped*. This is to provide safe markup
and reduce the chances of cross-site-scripting attacks (XSS). If you
know a value will be safe for output and/or do not want it escaped,
surround the variable using three braces:

```html
Hello, {{{name}}}!
```

A *view* is either an associative array or an object. In the case of an
associative array, template variables will reference array keys. In the
case of an object, template variables may reference either a public
property, or a public method. If a public method is referenced, or if
the property value is a valid PHP callback, the return value from
invoking the member will be returned.

The following are equivalent views:

```php
// Associative array view:
$view = array(
    'name'    => 'Matthew',
    'twitter' => 'mwop',
);

// Anonymous class view:
$view = new stdClass;
$view->name = 'Matthew';
$view->twitter = 'mwop';

// Class-based view:
class Matthew
{
    public $name = 'Matthew';
    public function twitter()
    {
        return 'mwop';
    }
}
$view = new Matthew();
```

While the last example is contrived, it does demonstrate that methods and
properties are interchangeable.

Sometimes, you may want to create nested structures:

```php
$view = array(
    'name' => 'Matthew',
    'contact' => array(
        'twitter' => 'mwop',
        'github'  => 'weierophinney',
    ),
);
```

Within your template, you can use "dot notation" to dereference such nested
structures. Basically, a "dot" indicates that the preceding value should refer
to an associative array or object, and that the segment following it should be
retrieved:

```html
The github user for {{name}} is {{contact.github}}.
```

The above will result in:

```html
The github user for Matthew is weierophinney.
```

Now that you know about basic templating, variable substitution, and views,
let's look at how you actually render using `phly-mustache`.

## Rendering String Templates

```php
$test = $mustache->render(
    'Hello {{planet}}',
    array('planet' => 'World')
);
echo $test;
```

which outputs as 

```html
Hello World
```

In the coming examples I will skip the `echo` statement to reduce code size.
I am also omitting all opening `<?php` tags.

## Rendering File Templates

Let the template be `renders-file-templates.mustache` in your
`templates` folder.  From here onwards, we assume you have your
template in `templates` folder.  Comments inside templates are marked
between `{{!` and `}}`. Please note the character `!`.

```html
{{!renders-file-templates.mustache}}
Hello {{planet}}
```

Now you can render it 

```php
$test = $mustache->render('renders-file-templates', array(
    'planet' => 'World',
));
```

Outputs : 

```html
Hello World
```

## Template Suffix

You may have noticed we have not added the suffix when we pass the template
name.  By default the suffix is ".mustache".  However, you can change the suffix
as desired; as an example, you might want to simply use `.html`.

```php
$mustache->setSuffix('html');
$test = $mustache->render('alternate-suffix', array());
```

The above will look for the file `alternate-suffix.html` in your templates
folder.

## Rendering Object Properties

You are not limited to associative arrays for your views; you can also use
objects, and `phly-mustache` will render object properties.

As an example, consider the following PHP code:

```php
$view         = new stdClass;
$view->planet = 'World';

$test = $mustache->render(
    'Hello {{planet}}',
    $view
);
```

The above will use the `$planet` property of the `$view` object to fill the
template.

Now consider the following template:

```html
{{!render-object-properties.mustache}}
{{content}}
```

With the following script:

```php
$view          = new stdClass;
$view->content = 'This is the content';
$test = $mustache->render('render-object-properties', $view);
```

You will see the content filled from the `$content`  property of the view object.

## Rendering methods which return a value

Let's assume you have a class `ViewWithMethod`, and it contains a method
`taxed_value`.

```php
class ViewWithMethod
{
    public $name  = 'Chris';
    public $value = 1000000;
    public $in_ca = true;

    public function taxed_value()
    {
        return $this->value - ($this->value * 0.4);
    }
}
```

If `taxed_value` is referenced in the template, the return value
of that method will be used.

```html
{{!template-with-method-substitution.mustache}}
Hello {{name}}
You have just won ${{taxed_value}}!
```

The following renders the above template with an instance of the previous view:

```php
$chris = new ViewWithMethod();
$test = $mustache->render(
    'template-with-method-substitution',
    $chris
);
```

Giving us the following output:

```html
Hello Chris
You have just won $600000!
```
