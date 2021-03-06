# Marionette.Application.

Объект `Backbone.Marionette.Application` является центральной частью вашего приложения. Он организует, инициализирует и координирует различные части вашего приложения. Также он определяет начальную точку запуска вашего приложения из JavaScript блока в HTML или прямо из JavaScript файла.

`Application` предназначен для прямого применения, хотя вы можете его расширить своей собственной функциональностью.

```js
var MyApp = new Backbone.Marionette.Application();
```

## Documentation Index

* [Добавление инициализаторов](#добавление-инициализаторов)
* [События объекта Application](#cобытия-объекта-Application)
* [Запуск приложения](#запуск-приложения)
* [The Application Channel](#the-application-channel)
  * [Event Aggregator](#event-aggregator)
  * [Request Response](#request-response)
  * [Commands](#commands)
  * [Accessing the Application Channel](#accessing-the-application-channel)
* [Regions And The Application Object](#regions-and-the-application-object)
  * [jQuery Selector](#jquery-selector)
  * [Custom Region Class](#custom-region-class)
  * [Custom Region Class And Selector](#custom-region-class-and-selector)
  * [Region Options](#region-options)
  * [Overriding the default RegionManager](#overriding-the-default-regionmanager)
  * [Get Region By Name](#get-region-by-name)
  * [Removing Regions](#removing-regions)
* [Application.getOption](#applicationgetoption)

## Добавление инициализаторов

Ваше приложение нуждается в некоторых нужных вещах, таких как отображение контента в регионах, запуск маршрутизатора и т.д. Для достижения данных целей, убедитесь, что ваш `Application` полностью сконфигурирован, вы можете добавить функции-инициализаторы обратного вызова к своему приложению.

```js
MyApp.addInitializer(function(options){
  // do useful stuff here
  var myView = new MyView({
    model: options.someModel
  });
  MyApp.mainRegion.show(myView);
});

MyApp.addInitializer(function(options){
  new MyAppRouter();
  Backbone.history.start();
});
```

Эти функции обратного вызова буду выполнены, когда стартует ваше приложение,
где в качестве контекста будет выступать объект application. Если переформулировать, то  `this` это объект `MyApp` внутри функции инициализатора.

Аргумент `options` передается из метода `start` (см. ниже).

Функции-инициализаторы будут гарантировано вызваны, независимо от того, когда вы их добавили к объекту app. Если вы их добавили перед стартом app, они будут вызваны после того, как будет вызван метод start объекта app. Если же вы добавили их после старта приложения, они будут вызваны сразу.

## События объекта Application

В течении своего жизненного цикла, объект `Application`вызывает несколько события, используя функцию
[Marionette.triggerMethod](./marionette.functions.md). Эти события можно использовать для дополнительной обработки вашего приложения. Например, вы можете захотеть обработать некоторые данные до того, как сработает функция инициализации. Или вы можете захотеть подождать пока приложение полностью инициализируется, а затем запустить `Backbone.history`.

События, которые сработают:

* **"before:start" / `onBeforeStart`**: запускаеся запуска `Application` и до того, как будут вызваны функции инициализаторы.
* **"start" / `onStart`**: запускается после запуск `Application` и после того, как отработают функции инициализаторы.

```js
MyApp.on("before:start", function(options){
  options.moreData = "Yo dawg, I heard you like options so I put some options in your options!"
});

MyApp.on("start", function(options){
  if (Backbone.history){
    Backbone.history.start();
  }
});
```

Параметр `options` пришел из метода `start` объекта application (см. ниже).

## Запуск приложения

Once you have your application configured, you can kick everything off by
calling: `MyApp.start(options)`.

This function takes a single optional parameter. This parameter will be passed
to each of your initializer functions, as well as the initialize events. This
allows you to provide extra configuration for various parts of your app throughout the
initialization sequence.

```js
var options = {
  something: "some value",
  another: "#some-selector"
};

MyApp.start(options);
```

## The Application Channel

Marionette Applications come with a [messaging system](http://en.wikipedia.org/wiki/Message_passing) to facilitate communications within your app.

The messaging system on the Application is the radio channel from Backbone.Wreqr, which is actually comprised of three distinct systems.

Marionette Applications default to the 'global' channel, but the channel can be configured.

```js
var MyApp = new Marionette.Application({ channelName: 'appChannel' });
```

This section will give a brief overview of the systems; for a more in-depth look you are encouraged to read
the [`Backbone.Wreqr` documentation](https://github.com/marionettejs/backbone.wreqr).

### Event Aggregator

The Event Aggregator is available through the `vent` property. `vent` is convenient for passively sharing information between
pieces of your application as events occur.

```js
var MyApp = new Backbone.Marionette.Application();

// Alert the user on the 'minutePassed' event
MyApp.vent.on("minutePassed", function(someData){
  alert("Received", someData);
});

// This will emit an event with the value of window.someData every minute
window.setInterval(function() {
  MyApp.vent.trigger("minutePassed", window.someData);
}, 1000 * 60);
```

### Request Response

Request Response is a means for any component to request information from another component without being tightly coupled. An instance of Request Response is available on the Application as the `reqres` property.

```js
var MyApp = new Backbone.Marionette.Application();

// Set up a handler to return a todoList based on type
MyApp.reqres.setHandler("todoList", function(type){
  return this.todoLists[type];
});

// Make the request to get the grocery list
var groceryList = MyApp.reqres.request("todoList", "groceries");

// The request method can also be accessed directly from the application object
var groceryList = MyApp.request("todoList", "groceries");
```

### Commands

Commands are used to make any component tell another component to perform an action without a direct reference to it. A Commands instance is available under the `commands` property of the Application.

Note that the callback of a command is not meant to return a value.

```js
var MyApp = new Backbone.Marionette.Application();

MyApp.model = new Backbone.Model();

// Set up the handler to call fetch on the model
MyApp.commands.setHandler("fetchData", function(reset){
  MyApp.model.fetch({reset: reset});
});

// Order that the data be fetched
MyApp.commands.execute("fetchData", true);

// The execute function is also available directly from the application
MyApp.execute("fetchData", true);
```

### Accessing the Application Channel

To access this application channel from other objects within your app you are encouraged to get a handle of the systems
through the Wreqr API instead of the Application instance itself.

```js
// Assuming that we're in some class within your app,
// and that we are using the default 'global' channel
// it is preferable to access the channel like this:
var globalCh = Backbone.Wreqr.radio.channel('global');
globalCh.vent;

// This is discouraged because it assumes the name of your application
window.app.vent;
```

## Regions And The Application Object

Application instances have an API that allow you to manage [Regions](https://github.com/marionettejs/backbone.marionette/blob/master/docs/marionette.region.md).
These Regions are typically the means through which your views become attached to the `document`.

You can create Regions through the `addRegions` method by passing in an object
literal or a function that returns an object literal.

There are three syntax forms for adding a region to an application object.

### jQuery Selector

The first is to specify a jQuery selector as the value of the region
definition. This will create an instance of a Marionette.Region directly,
and assign it to the selector:

```js
MyApp.addRegions({
  someRegion: "#some-div",
  anotherRegion: "#another-div"
});
```

### Custom Region Class

The second is to specify a custom region class, where the region class has
already specified a selector:

```js
var MyCustomRegion = Marionette.Region.extend({
  el: "#foo"
});

MyApp.addRegions(function() {
  return {
    someRegion: MyCustomRegion
  };
});
```

### Custom Region Class And Selector

The third method is to specify a custom region class, and a jQuery selector
for this region instance, using an object literal:

```js
var MyCustomRegion = Marionette.Region.extend({});

MyApp.addRegions({

  someRegion: {
    selector: "#foo",
    regionClass: MyCustomRegion
  },

  anotherRegion: {
    selector: "#bar",
    regionClass: MyCustomRegion
  }

});
```

### Region Options

You can also specify regions per `Application` instance.

```js
new Marionette.Application({
  regions: {
    fooRegion: '#foo-region'
  }
});
```

### Overriding the default `RegionManager`

If you need the `RegionManager`'s class chosen dynamically, specify `getRegionManager`:

```js
Marionette.Application.extend({
  // ...

  getRegionManager: function() {
    // custom logic
    return new MyRegionManager();
  }
```

This can be useful if you want to attach `Application`'s regions to your own instance of `RegionManager`.

### Get Region By Name

A region can be retrieved by name, using the `getRegion` method:

```js
var app = new Marionette.Application();
app.addRegions({ r1: "#region1" });

// r1 === r1Again; true
var r1 = app.getRegion("r1");
var r1Again = app.r1;
```

Accessing a region by named attribute is equivalent to accessing
it from the `getRegion` method.

### Removing Regions

Regions can also be removed with the `removeRegion` method, passing in
the name of the region to remove as a string value:

```js
MyApp.removeRegion('someRegion');
```

Removing a region will properly empty it before removing it from the
application object.

For more information on regions, see [the region documentation](./marionette.region.md) Also, the API that Applications use to
manage regions comes from the RegionManager Class, which is documented [over here](https://github.com/marionettejs/backbone.marionette/blob/master/docs/marionette.regionmanager.md).

### Application.getOption
Retrieve an object's attribute either directly from the object, or from the object's this.options, with this.options taking precedence.

More information [getOption](./marionette.functions.md)
