---
layout: docs
title: Internationalizing your App
permalink: /docs/use/internationalization/
---

BladeRunnerJS comes with in-built support for internationalization. Below are
the details required to add internationalization support to your BRJS application.

## Basic App Setup

For this example we'll assume you've [created an application](/docs/use/create-app) called `demo`, [created a bladeset](/docs/use/create-bladeset) called `blades` and  [created a blade](/docs/use/create-blade/) called `myblade`. Within that blade, open up the `resources/html/view.html` template and update it to look as follows:

```html
<div class="locale-demo-blade" id="demo.blades.myblade.view-template">
   <h1>Internationalization Demo</h1>
   <div class="hello-world-message" data-bind="text:message"></div>
</div>

```

Now, open up the ViewModel file for the blade, `MybladeViewModel.js`, and update it to match the following:

```js
'use strict';

var ko = require( 'ko' );

function MybladeViewModel() {
  this.message = ko.observable( 'Hello World!' );
}

module.exports = MybladeViewModel;
```

Run the [workbench](/docs/concepts/workbenches) for the blade and it should look like this:

![](/docs/use/img/locale-start.png)

## Understanding how locale URLs are handled in BRJS

Before we continue we need to first understand how the URLs used in an app change depending on the locale currently in use.

When a request is made to `/myapp/` a 'locale forwarding' page is returned to the browser. Using the browsers' `Accept-Language` header; the value of the locale cookie and the locales the app supports this page forwards the browser to a localized app page, for example `/myapp/en/`. This localized app page then contains the neccessary CSS, JS, XML and HTML requests for the app.

In order to change the currently selected locale the browser must re-request the `/myapp/` URL so the locale forwarder can recalculate the locale and send the browser to the correct page.

<div class="alert alert-info">
<p>
This mechanism currently prevents users of your app refreshing the page in order to change their locale. We hope to add a <a href="https://github.com/BladeRunnerJS/brjs/issues/804">mechanism</a> where apps can optionally push users back to the locale forwarder if they are using the incorrect locale.
</p>
</div>

## Set up Supported Locales

Have a look in `app.conf` within the application root directory. It defines the locales that your app will support

```
requirePrefix: <namespace>
locales: en, de, es
```

## Include the Internationalization Bundler

Include the Internationalization Bundler in your app's `default-aspect/index.html` and workbench (`myblade/workbench/index.html`), by adding the `i18n.bundle` line into your `index.html` file.

```html
<head>
    <@base.tag@/>
	<meta charset="UTF-8">

    <title>Workbench</title>

    <@css.bundle theme="standard"@ />

<!-- new code -->
    <@i18n.bundle@/>
<!-- end of new code -->

</head>
```

## Internationalizing via HTML

Let's update the `<h1>` element inside our HTML, which will be automatically internationalized. Internationalization markup in BRJS takes the form of: `@{<token>}`. It's best to namespace your token so that other blades don't overwrite each other's tokens

```html
<div class="locale-demo-blade" id="demo.blades.myblade.view-template">
   <h1>@{demo.blades.myblade.title}</h1>
   <div class="hello-world-message" data-bind="text:message"></div>
</div>
```

Then define the token translation inside the file: `myblade/resources/i18n/en/en.properties`

```
demo.blades.myblade.title=Internationalization Demo
```

Refresh the workbench, and it should look like this.

![](/docs/use/img/locale-html-token.png)

The token replacement works by replacing all i18n tokens in all HTML as it is streamed through the HTML Bundler.

Note: We were able to simply refresh the page here rather than requesting the 'locale forwarder' since we weren't changing the locale we were using.

## Internationalizing via JavaScript

Let's add an internationalized property to the blade's view model at `myblade/src/demo/blades/myblade/MybladeViewModel.js`.

```js
'use strict';

var ko = require( 'ko' );

/*** new code ***/
var i18n = require( 'br/I18n' );
/*** end of new code ***/

function MybladeViewModel() {
  /*** new code ***/
  var text = i18n( 'demo.blades.myblade.helloworldmessage' );
  this.message = ko.observable( text );
  /*** end of new code ***/
}

module.exports = MybladeViewModel;
```

Refreshing the application will result in a message that indicates the requested property could not be found:

![](/docs/use/img/locale-property-not-found.png)

To fix this add the translation to the i18n resource file.

```
demo.blades.myblade.title=Internationalization Demo
/*** new code ***/
demo.blades.myblade.helloworldmessage=Internationalized Hello World!
/*** end of new code ***/
```

After a refresh of the blade workbench, your token should have been replaced, and it should look like this:

![](/docs/use/img/locale-js-token.png)

## Add a New Language

Create a file `myblade/resources/i18n/de/de.properties` file, containing:

```
demo.blades.myblade.title=Internationalisierung Demonstration
demo.blades.myblade.helloworldmessage=Internationalized Hallo Welt!
```

Loading the app with a German locale set in the browser, will display like this:

![](/docs/use/img/locale-german.png)

## Local Date Format

*By default BRJS supports English, German and Chinese locale date formats*. It is possible to add additional customized date formats too.

For this example we'll demonstrate English and German date formats.

Localized dates can be retrieved using the `date` function from the `i18N` module. Update `MybladeViewModel` as follows:

```js
'use strict';

var ko = require( 'ko' );

/*** new code ***/
var i18n = require( 'br/I18n' );
/*** end of new code ***/

function MybladeViewModel() {
  /*** new code ***/
  var epochDate = new Date( 1970, 0, 1 );
  var formattedDate = i18n.date( epochDate );
  /*** end of new code ***/

  var text = 'The epoch date was ' + formattedDate;
  this.message = ko.observable( text );
}

module.exports = MybladeViewModel;
```

The English output and formatted date looks as follows:

![](/docs/use/img/locale-en-date.png)

And the German:

![](/docs/use/img/locale-de-date.png)

If we want to see dates with times down to the second we need to change the date format. In order to change the date format you need to update the locale settings for the aspect, and those settings are inherited by the blade. This is because date formats should almost always be the same throughout the application rather than differing from blade to blade, so the date formats are defined up at the aspect level.

Update the English properties file, found in `default-aspect/resources/i18n/en/en.properties` to define a `br.i18n.date.format`:

```
br.i18n.date.format=DD, MMMM YYYY hh:mm:ss
```

And do the same for the German properties file, found in `default-aspect/resources/i18n/de/de.properties`:

```
br.i18n.date.format=DD, MMMM YYYY hh:mm:ss
```

With these changes in place the English date format will be updated:

![](/docs/use/img/locale-en-date-long.png)

As will the German:

![](/docs/use/img/locale-de-date-long.png)

*Note: you'll probably be thinking that the German version has English text hard-coded. We'll fix this later using Parameterized Translations*.

## Local Number Format

Number formatting too is subject to locale. If we were to display a large numerical value within our blade it's nicer to format this with the appropriate thousands separator character and decimal point character.

Note that formatting of numbers falls under the remit of internationalization because the characters differ by locale. In English we use the comma as a thousands separator and the full stop as a decimal point, but in French the space is used as a thousands separator and comma is used for the decimal point. German uses the full stop as a thousands separator.

To format a number to the user's locale, we can use the `number` function. To demonstrate this we can display the [number of milliseconds since the epoch](http://en.wikipedia.org/wiki/Unix_time) in addition to the epoch timestamp.

```js
'use strict';

var ko = require( 'ko' );
var i18n = require( 'br/I18n' );

function MybladeViewModel() {
	var epochDate = new Date( 1970, 0, 1 );
	var formattedDate = i18n.date( epochDate );

	/*** new code ***/
	var millisSinceEpoch = i18n.number( Date.now() );
	/*** end of new code ***/

	var text = 'The epoch date was ' + formattedDate;
	/*** new code ***/
	text += ' This is ' + millisSinceEpoch + 'ms ago.';
	/*** end of new code ***/
	this.message = ko.observable( text );
}

module.exports = MybladeViewModel;

```

Now the format of the milliseconds since the epoch differs according to the locale used; English:

![](/docs/use/img/locale-en-number.png)

And German:

![](/docs/use/img/locale-de-number.png)

## Parameterized Translations

The i18n library also allows you to make "parameterized translations". This involves defining an i18n property containing tokens, and then passing in the values for those tokens when you call the translator.

Update the `en.properties` file so that there is a `demo.blades.myblade.epochmessage` that uses `timestamp` and `epochmillis` parameters:

```
demo.blades.myblade.epochmessage=The epoch date was [timestamp]. This is [epochmillis]ms ago.
```

Do the same with `de.properties`:

```
demo.blades.myblade.epochmessage=Die Epoche Datum war [timestamp]. Dies ist vor [epochmillis]ms.
```

Within the messages the parameters are identified by square brackets. When you call the translator, you can pass in the values that should be inserted in place of these parameters.

The parameters are passed as a map object of token name and token value. The map is passed into the translator in addition to the ID of the i18n property you want to use. Here's the updated `MybladeViewModel.js` :

```js
'use strict';

var ko = require( 'ko' );

var i18n = require( 'br/I18n' );

function MybladeViewModel() {
	var epochDate = new Date( 1970, 0, 1 );
	var formattedDate = i18n.date( epochDate );

	var millisSinceEpoch = i18n.number( Date.now() );

	/*** new code ***/
	var params = {
		timestamp: formattedDate,
		epochmillis: millisSinceEpoch
	};

	var text = i18n( 'demo.blades.myblade.epochmessage', params)
	/*** end of new code ***/

	this.message = ko.observable( text );
}

module.exports = MybladeViewModel;
```

And here's what the result looks like in English:

![](/docs/use/img/locale-en-parameterized.png)

And German:

![](/docs/use/img/locale-de-parameterized.png)

## Localized CSS

The CSS used in your application can also be locale specific. For instance, we may want to set the background with national colors or increase the dimensions of elements containing text where words and sentences may be longer in different languages.

To do this creates files called `color_en.css` for the English locale and `colors_de.css` for the German in the `standard/themes/standard` directory. Here you can set the locale-specific css. Note that locale-specific theme css overrides general theme css so let's first define the shared styles in `myblade/themes/standard/style.css`:

```css
.locale-demo-blade {
  width: 250px;
  font-family: helvetica;
  padding: 20px;
  text-align: center;
}

.locale-demo-blade h1,
.locale-demo-blade .hello-world-message {
  padding: 5px;
}

.locale-demo-blade .hello-world-message {
  line-height: 22px;
}
```

In `color_en.css` add the following content:

```css
.locale-demo-blade {
  background-color: red;
  color: white;
}

.locale-demo-blade h1 {
  background-color: blue;
  color: white;
}

.locale-demo-blade .hello-world-message {
  background-color: white;
  color: black;
}
```

Loading the application with the English locale will show this:

![](/docs/use/img/locale-en-css.png)

Finally in `color_de.css` add the following content:

```css
.locale-demo-blade {
  background-color: black;
  color: white;
}

.locale-demo-blade h1 {
  background-color: red;
  color: white;
}

.locale-demo-blade .hello-world-message {
  background-color: gold;
  color: black;
}
```

And loading the German locale will show the following:

![](/docs/use/img/locale-de-css.png)

## Useful Tips

### Example Application

The full [KnockoutJS BRJS Example Todo MVC example](https://github.com/BladeRunnerJS/brjs-todomvc-knockout) demonstrates how to use i18n within and application.

### Missing Translations

If you miss out a translation, then the i18n makes it pretty obvious to you.

![](/docs/use/img/i18n-error.png)

## I18n at Different Levels

You can internationalize at different levels of your application, by locating property files under the different levels in their resource folders

* Aspect: `/app/aspect/resources/i18n`
* BladeSet: `/app/bladeset/resources/i18n`
* Blade: `/app/bladeset/blade/resources/i18n`
