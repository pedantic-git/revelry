# Revelry

Revelry is a project builder for
[Reveal.js](http://lab.hakim.se/reveal-js/).

If you make a lot of presentations with Reveal, you will find yourself
unnecessarily duplicating huge chunks of boilerplate. With Revelry
you can concentrate on your presentation content and let Revelry take
care of the rest.

This is an alpha release. See
[the issues tracker](https://github.com/pedantic-git/revelry/issues)
for the list of planned features and known issues.

## Installing

    $ npm install -g revelry

**Note:** if you are cloning this repo and installing locally, you will need to
specify the `--unsafe-perm` option to `npm install`. This is because the
postinstall needs access to the reveal.js dependency.

## Alternative: Using Docker

If you have Docker installed, you can use Revelry without needing to
install anything. For example:

    $ docker run -v $(pwd):/work -u $(id -u) fishpercolator/revelry new myproject

Or create an alias:

    $ alias revelry='docker run -v $(pwd):/work -u $(id -u) fishpercolator/revelry'

## Creating a project

    $ revelry new myproject

    $ revelry new myproject -t "My amazing presentation" -d "A short description of my presentation"

## The anatomy of a Revelry project

    myproject/
	  Revfile.json
	  slides.html
      img/
	  custom/
	    custom.css
		header.html

**Revfile.json** is where all the configuration for your presentation
  goes. In here you can specify the title and description as well as
  Reveal options such as theme and plugins. (Note: Revfile.json is
  really JavaScript code, rather than JSON, so you can include
  functions and other complex types in here. The usual warnings about
  editing executable code apply.)

**slides.html** is a [Handlebars](http://handlebarsjs.com/)-enabled
  HTML file containing your slides. Each slide is given an HTML
  `<section>`. The contents of `Revfile.json` are used as the
  Handlebars context.

**img/** is a directory where you can put images you need for your
  presentation. Its contents will be copied to the built site's `img/`
  dir when your project is built.

**custom/** contains files that can be used to override some of the
  default HTML and CSS provided by Reveal. (See below.)

## Example slides.html

```handlebars
<section>
  <h1>{{title}}</h1>
  <h2>{{description}}</h2>
</section>
<section>
  <h2>My first slide</h2>
  <ul>
    <li class="fragment">Bullet one</li>
	<li class="fragment" style="color:red">Bullet two</li>
  </ul>
</section>
```

## Partials

You can break your presentation into multiple files, which can be a mixture of
Handlebars and Pug. Then you can include them in each other using each
language's native include syntax. For example:

```handlebars
<section>
  <h1>Handlebars example</h1>
</section>
{{! Include intro.html or intro.pug}}
{{> intro}}
```

```pug
section
  h1 Pug example
//- Include another Pug file
include intro.pug
//- Include a Handlebars file
include:handlebars intro.html
```

## Building

Once you have filled out the contents of `Revfile.json` and `slides.html`,
you can build your project.

    $ cd myproject/
    $ revelry build

By default this will create the project in a .gitignored subdirectory
`www` of the project directory, but you can specify any target
directory:

    $ revelry build /var/www/myproject

## Configuration

Most of the configuration of a Revelry project happens in the
`Revfile.json`. Some of the supported keys are:

* **title**: The presentation title.
* **description**: A description of your presentation (used in the
  meta tags).
* **author**: The author's name (used in the meta tags).
* **options**: Overrides for the options passed to the Reveal.js
  constructor. All the Reveal.js options are available here for
  override. (Note: The 'theme' option is special, because it affects
  which CSS file is loaded in your presentation's header.)
* **plugins**: See below.

### Plugins

Including a plugin name in the *plugins* array of `Revfile.json` will
cause the appropriate files to be copied to the built project from
Reveal.js and the necessary JavaScript files loaded as dependencies.

If you want to configure the plugin further, you'll need to add the
config separately to *options*. See the Reveal.js documentation for
more details.

Currently supported plugins are:

* classList
* highlight
* leap
* markdown
* math
* notes
* notes-server *(will cause your presentation to `npm install` so it
  can be used)*
* remotes
* zoom

The *multiplex* plugin is not supported because its configuration is
too complicated to automatically guess.

Revelry currently doesn't support any plugins other than the standard
ones. Watch out for 3rd-party plugin support in a future release.

### Custom CSS / Sass

Add CSS to the `custom.scss` file if you want to override the theme's
default CSS or add classes of your own.

The file is preprocessed with [Sass](http://sass-lang.com/), so you can use
variables, nested selectors, etc. if you wish.

Note that you'll need to put all your overrides inside the `.reveal`
class, like this:

```scss
.reveal {
  h1 {
    color: black;
  }
}
```

### Custom header

If you need to insert any additional info into the HTML header (such
as meta tags or links to CSS or RSS), you can edit the `header.html`
file. The contents of this file will be inserted into the
`<head>` of the presentation.

You can also set any of the `<meta>` tags using the `meta` object in
`Revfile.json`. If you don't have one because your project was created by an
older version, run `revelry upgrade` on your project to bring it up to date.

### Custom footer

Added in 2.0.0. This contains the code to initialize Reveal.js and any custom
JavaScript you want to add to your presentation.

If you have a project that was created before 2.0.0, you'll need to run
`revelry upgrade` to create this file before you can build.

## Pug support

Revelry also supports templates in [Pug](https://pugjs.org/) format,
which is a more concise language than HTML and lends itself quite well
to writing presentations.

If you have a `slides.pug` instead of `slides.html`, it will be
parsed as a Pug file.

You can also create a new Pug project outright, e.g.:

    $ revelry new myproject --pug

The Revfile context will be passed to Pug as locals, so you can use its
values the same way you would in Handlebars.

The earlier example, expressed as Pug, is:

```pug
section
  h1= title
  h2= description
section
  h2 My first slide
  ul
    li.fragment Bullet one
    li.fragment(style='color:red') Bullet two
```

**Note**: The resulting HTML is still processed by Handlebars.

## Author info

Copyright 2014-2015, Fish Percolator.

Released under a BSD-style license.

Fork me on Github: <https://github.com/fishpercolator/revelry>

Issue tracker: <https://github.com/fishpercolator/revelry/issues>
