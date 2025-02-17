# Stempler - Components and Props

Stempler provides an ability to create developer-driven template components as virtual tags.

## Simple Component

In many cases, your templates will not only reuse the parent layout, but also template partials, for example:

```html
<extends:layout.base title="Homepage"/>

<block:content>
    This is the homepage.

    <div class="article">
        <div class="title">Article title</div>
        <div class="preview">article preview</div>
    </div>

    <div class="article">
        <div class="title">Article title 2</div>
        <div class="preview">article preview 2</div>
    </div>

    <div class="article">
        <div class="title">Article title 3</div>
        <div class="preview">article preview 3</div>
    </div>
</block:content>
```

We can move the article div into a separate template `app/views/partial/article.dark.php`:

```html
<div class="article">
    <div class="title">Article title</div>
    <div class="preview">article preview</div>
</div>
```

To use this partial on your page, first import it using the `<use:element path=""/>` control tag:

```html
<extends:layout.base title="Homepage"/>
<use:element path="partial/article"/>

<block:content>
    This is the homepage.

    <article/>
    <article/>
    <article/>
</block:content>
```   

> **Note**
> Read more about mass-importing partials below.

### Props

It's is not very useful to create partials without the ability to configure their content. Use the `block:name`
or `${name|default}` syntax (similar to the one described [here](/stempler/inheritance.md)) to define replaceable parts:

In our partial `app/views/partial/article.dark.php`:

```html
<div class="article">
    <div class="title">${title}</div>
    <div class="preview">
        <block:preview>
            default preview
        </block:preview>
    </div>
</div>
```

You can pass values similar way as in the `extend` control tag:

```html
<extends:layout.base title="Homepage"/>
<use:element path="partial/article"/>

<block:content>
    This is the homepage.

    <article>
        <block:title>Article 1 title</block:title>
        <block:preview>
            This is article 1 preview.
        </block:preview>
    </article>

    <article title="Article 2">
        <block:preview>
            <block:parent/>
            This is article 1 preview.
        </block:preview>
    </article>

    <article title="Article 3" preview="This is article 3 preview."/>
</block:content>
```

> **Note**
> You can include the original block content using the `block:parent` tag. Component expansion is also allowed.

The resulted HTML:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Homepage</title>
    <link rel="stylesheet" href="/styles/welcome.css"/>
</head>
<body class="default">
This is homepage.
<div class="article">
    <div class="title">Article 1 title</div>
    <div class="preview">
        This is article 1 preview.
    </div>
</div>

<div class="article">
    <div class="title">Article 2</div>
    <div class="preview">
        Default content.
        This is article 1 preview.
    </div>
</div>

<div class="article">
    <div class="title">Article 3</div>
    <div class="preview">
        This is article 3 preview.
    </div>
</div>
</body>
</html>
```

> **Note**
> Components do not cause any performance penalty, use as many components as you need.

## Import Components

Stempler provides several options for importing components into your template.

### Import Element

To import a single component, use `<use:element path=""/>` before component invocation.

```html
<extends:layout.base title="Homepage"/>
<use:element path="partial/article"/>

<block:content>
    <article title="Article" preview="This is article preview."/>
</block:content>
```

The component will be available using the filename, in this case it's `article`. To define a custom import alias, use the tag attribute `as`:

```html

<extends:layout.base title="Homepage"/>
<use:element path="partial/article" as="custom-article"/>

<block:content>
    <custom-article title="Article" preview="This is article preview."/>
</block:content>
```

### Import Directory

To import all the partials from a given directory, use `<use:dir dir="" ns=""/>`. You must specify a namespace prefix to 
avoid collisions with other components and default HTML tags:

```html
<extends:layout.base title="Homepage"/>
<use:dir dir="partial" ns="partials"/>

<block:content>
    <partials:article title="Article" preview="This is article preview."/>
</block:content>
```

### Inline Import

To define a component specific to a given template without creating a physical view file,
use the `<use:inline name=""></use:inline>` control tag. In `app/views/home.dark.php`:

```html
<extends:layout.base title="Homepage"/>

<use:inline name="article">
    <div class="article">
        <div class="title">${title}</div>
        <div class="preview">${preview}</div>
    </div>
</use:inline>

<block:content>
    <article title="Article" preview="This is article preview."/>
</block:content>
```

### Bundle Import

Import multiple directories, components and/or inline components using bundled import via `<use:bundle path="">`.

Create a view file `app/views/my-bundle.dark.php` to define your bundle:

```html
<use:element path="partial/article" as="article"/>

<use:inline name="article-alt">
    <div class="article">
        <div class="title">${title}</div>
        <div class="preview">${preview}</div>
    </div>
</use:inline>
```

You can use any of the defined components in your `app/views/home.dark.php` template:

```html
<extends:layout.base title="Homepage"/>
<use:bundle path="my-bundle"/>

<block:content>
    <article title="Article" preview="This is article preview."/>
    <article-alt title="Article" preview="This is article preview."/>
</block:content>
```

To isolate an imported bundle via the prefix, use the `ns` attribute of the `use:bundle` tag:

```html
<extends:layout.base title="Homepage"/>
<use:bundle path="my-bundle" ns="my"/>

<block:content>
    <my:article title="Article" preview="This is article preview."/>
    <my:article-alt title="Article" preview="This is article preview."/>
</block:content>
```

## Props

The ability to pass values into components makes it possible to create complex elements that are condensed into simple tags.
You are allowed to pass PHP values and echoes to your components.

Modify your controller to invoke the template like this:

```php
return $this->views->render('home', ['value' => 'Hello&world!']);
```

Create `app/views/partial/input.dark.php`:

```html
<div class="input">
    <label>${label}</label>
    <input type="text" value="${value}"/>
</div>
```

You can invoke this component in your template with a user supplied value:

```html
<extends:layout.base title="Homepage"/>
<use:element path="partial/input" as="my:input"/>

<block:content>
    <my:input label="Some Value" value="{{ $value }}"/>
</block:content>
```

The generated PHP:

```php
...
<body class="default">
    <div class="input">
        <label>Some Value</label>
        <input type="text" value="<?php echo htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'utf-8'); ?>"/>
    </div>
</body>
...
```

### PHP in Components

Not only can you inject values into plain HTML, but you can also inject source code into a PHP component. It can be achieved using an AST modification of the underlying template via the macro function `inject("name", default)`.

> **Note**
> The injection will automatically extract the variable or statement from the passed `{{ echo }}`, `<?php $variable ?>` or `<?=$variable?>` attributes.

To demonstrate it, modify `app/views/partial/input.dark.php`:

```html
<div class="input">
    <label>${label}</label>
    <input type="text" value="{{ strtoupper(inject('value')) }}"/>
</div>
```

Now the generated code will look like this:

```html
<body class="default">
<div class="input">
    <label>Some Value</label>
    <input type="text"
           value="<?php echo htmlspecialchars(strtoupper($value), ENT_QUOTES | ENT_SUBSTITUTE, 'utf-8'); ?>"/>
</div>
</body>
```

You can pass PHP values in combination with string prefixes, in `app/views/home.dark.php`:

```html
<extends:layout.base title="Homepage"/>
<use:element path="partial/input" as="my:input"/>

<block:content>
    <my:input label="Some Value" value="hello {{ $value }} world"/>
</block:content>
```

The compiled template:

```html
<body class="default">
<div class="input">
    <label>Some Value</label>
    <input type="text"
           value="<?php echo htmlspecialchars(strtoupper('hello '.$value.' world'), ENT_QUOTES | ENT_SUBSTITUTE, 'utf-8'); ?>"/>
</div>
</body>
```

### Complex Props

You can inject your props not only in echo statements, but also in any PHP code of your component. Let's create the `select` component `app/views/partial/select.dark.php`:

```html
<select name="${name}">
    @foreach(inject('values', []) as $key => $label)
    <option value="{{ $key }}">{{ $label }}</option>
    @endforeach
</select>
```

Modify your controller to pass an array:

```php
return $this->views->render('home', [
    'values' => [1 => 'first', 2 => 'second']
]);
```

You can use this component in your template:

```html
<extends:layout.base title="Homepage"/>
<use:element path="partial/select" as="my:select"/>

<block:content>
    <my:select name="My Select" values="{{ $values }}"/>
</block:content>
```

The generated template:

```html
<body class="default">
<select name="My Select">
    <?php foreach($values as $key => $label): ?>
    <option value="<?php echo htmlspecialchars($key, ENT_QUOTES | ENT_SUBSTITUTE, 'utf-8'); ?>">
        <?php echo htmlspecialchars($label, ENT_QUOTES | ENT_SUBSTITUTE, 'utf-8'); ?>
    </option>
    <?php endforeach; ?>
</select>
</body>
```

You are allowed to inject PHP blocks into default PHP tags. `app/views/partial/select.dark.php` can be changed like this:

```html
<select name="${name}">
    <?php
  $selectValues = array_map('strtoupper', inject('values', []));
  ?>
    @foreach($selectValues as $key => $label)
    <option value="{{ $key }}">{{ $label }}</option>
    @endforeach
</select>
```

The generated template:

```html

<body class="default">
<select name="My Select">
    <?php
    $selectValues = array_map('strtoupper', $values);
    ?>
    <?php foreach($selectValues as $key => $label): ?>
    <option value="<?php echo htmlspecialchars($key, ENT_QUOTES | ENT_SUBSTITUTE, 'utf-8'); ?>"><?php echo htmlspecialchars($label, ENT_QUOTES | ENT_SUBSTITUTE, 'utf-8'); ?></option>
    <?php endforeach; ?>
</select>
</body>
```

> **Note**
> Attention, make sure to escape your values properly!

### Dynamic Attributes

In some cases, you might want to bypass some attributes into elements directly. For example, to a allow user-driven
`style` attribute for select, we have to do the following:

```html
<select name="${name}" style="${style}">
    @foreach(inject('values', []) as $key => $label)
    <option value="{{ $key }}">{{ $label }}</option>
    @endforeach
</select>
```

Use `attr:aggregate` to scale this approach:

```html
<select name="${name}" attr:aggregate>
    @foreach(inject('values', []) as $key => $label)
    <option value="{{ $key }}">{{ $label }}</option>
    @endforeach
</select>
```

Now we can pass arbitrary attributes to our component from `app/views/home.dark.php`:

```html
<extends:layout.base title="Homepage"/>
<use:element path="partial/select" as="my:select"/>

<block:content>
    <my:select name="My Select" values="{{ $values }}" style="color: red" class="custom-select"/>
</block:content>
```

The resulted HTML:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Homepage</title>
    <link rel="stylesheet" href="/styles/welcome.css"/>
</head>
<body class="default">
<select name="My Select" style="color: red" class="custom-select">
    <option value="1">first</option>
    <option value="2">second</option>
</select>
</body>
</html>
```
