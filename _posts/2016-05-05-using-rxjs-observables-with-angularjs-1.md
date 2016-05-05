---
title: Using RxJS Observables with AngularJS 1
description: A guide to reactive programming with RxJS Observables in AngularJS 1 applications.
categories:
  - Programming
tags:
  - Angular
  - RxJS
  - JavaScript
image:
    src: 2016/05/rxjs-n-angularjs-small.jpg
    alt: Using RxJS with AngularJS
    align: none
---

Reactive programming with RxJS has become quite popular lately in the frontend world, partly because it is included in Angular 2 and many people have already started learning it even though a stable version is not out yet. But if you are currently working with Angular 1, you don't have to wait until you start using Angular 2 in your production apps to start using RxJS, since RxJS itself is stable and can be used with any framework. 

Whether you already know RxJS and the reactive programming concepts, or just want to learn more about them before you start using Angular 2 for real, this post will show how you can integrate RxJS with Angular 1 and get into reactive programming right now. 

<!--more-->

Reactive programming in itself is a complex topic, but there are many [great tutorials](http://reactivex.io/tutorials.html) about it online so I will not cover it here in much detail. Some level of understanding about reactive programming concepts and Rx is assumed to be able follow along this post. 

## Rx Bindings for AngularJS

There is an official library for integrating RxJS with Angular 1 called [rx.angular.js](https://github.com/Reactive-Extensions/rx.angular.js/). This library works as a bridge between RxJS observables and AngularJS, making it easy to work with Observables in an Angular 1 application.

The main features of the library are:

* Trigger digest cycle on a scope when an observable emits a value
* Create observables from scope watch expressions
* Create observable functions on a scope
* Convert scope events to observables

Here is a small example where we create an autocomplete input field that can be used to search GitHub usernames:

{% gist cvuorinen/34494452537860b242d263cda82f482c %}

To create an observable that emits the input elements values, we create an observable function called `update` to the scope usig the `$createObservableFunction` scope method that _rx.angular.js_ added. We then call that function in the template from the input element's `ng-change`. You could also use `Rx.Observable.fromEvent(element, 'change')` if you have a reference to the input element, but in this case I wanted to showcase the features _rx.angular.js_ offers and how they can be used with normal Angular directives etc.

After we have the input values as an observable, we need to create the autocomplete logic and make the network requests. So, here is a break down of what the code does:

1. We wait until user has stopped typing with `debounce`
2. Then trim any trailing whitespace
3. Filter out empty values
4. Only continue if value changed from previous (no new request if only added whitespace that was then trimmed)
5. Search GitHub using [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) and convert the response to an Observable
6. Since we mapped the keyword to the search function that returned an Observable, we now have an Observable of Observables. So we use `switch` to flatten it back to a single Observable that will emit the search results. This will also discard the previous inner Observable if it was still running, making sure we get the results in correct order even if the network requests don't complete in the same order that they were sent.
7. We then use `digest` method (added by _rx.angular.js_) to assign the emitted values to a property on the scope. This is needed since we need to trigger a digest cycle because we used Fetch instead of $http service.
8. Finally we have an empty subscription. It's empty since we already assigned the emitted value to the scope, but it's needed since Observables are lazy so nothing will happen if there are no subscriptions.

Here is a link to a working JS Bin: [jsbin.com/fekimuz](http://jsbin.com/fekimuz/edit?js,output)

## Async pipe

If you are familiar with Angular 2 development, you might know about the AsyncPipe (pipes are for Angular 2 what filters are for Angular 1). The Async pipe allows binding asynchronous values in the view, like so:

{% raw %}
```liquid
{{ observable | async }}
```
{% endraw %}

It works by creating a subscription to the observable that updates the view value when the Observable emits a new value. It also works with Promises (by adding a "then"  callback that sets the view value to the resolved value).

There isn't really anything magical about this implementation and I decided to make an Async filter so that I could use the same approach in Angular 1 applications, so I wrote [angular1-async-filter](https://github.com/cvuorinen/angular1-async-filter).   

One of the key differences between Angular 1 and Angular 2 is the way the change detection works. In Angular 2 the pipes get a reference to the component change detector, that they can use to tell the change detection system that the component needs checking. In Angular 1 filters don't have a way to get a handle of the current scope so that they would be able to trigger a digest cycle.

It was quite easy to make the _Async filter_ work with $q promises, as they will automatically trigger a digest cycle, but RxJS Observables don't have that kind of integration out of the box. You can of course use the _rx.angular.js_ bindings mentioned above, and call `.digest()` on the Observable in your controller/service to make it work, but I started wondering if there was any way to pass the scope into the filter as a parameter. And turns out you can just use `this` as an argument in an expression to pass in the scope. 

So I just added a check if a second argument was passed in and then use it to trigger a digest cycle in the listener when a new value is emitted. To use the _Async filter_ without _rx.angular.js_, you need to pass in the scope like so:

{% raw %}
```liquid
{{ observable | async:this }}
```
{% endraw %}

You can install the Async filter to your own Angular 1 project from npm like this:

```
npm install angular1-async-filter --save
```

### Async filter example

As an example, here is the same autocomplete directive but this time using the _Async filter_ instead of _rx.angular.js_:

{% gist cvuorinen/98447e065922e8e7e6860ffe1a94a3c6 %}

Most of it is the same as before, here are the key differences:

* We create the Observable using the `fromEvent` method and pass in a reference to the input element. Since we are not using any Angular features here, there are no digests cycles happening on every key stroke.
* Since the Observable will now emit the native DOM event, we need to dig out the actual input field string value (from the target.value property on the event).
* After thet it's exactly the same as in the first example, except that there is no subscription and we store a reference to the final observable in a property on the scope (`scope.suggestions`).
* Finally, in the view we use the Async filter on `suggestions`, which will subscribe to it and update the view when a new value is emitted. We also need to pass the current scope to it using `async:this` so that it can trigger a digest cycle, otherwise we would not see anything appear in the `ng-repeat`.

Working JS Bin: [jsbin.com/kelutu](http://jsbin.com/kelutu/edit?js,output)

## Conclusion

As you can see, there is no reason you can't use RxJS with your existing Angular 1 applications. You just need to understand how the Angular digest cycle works so that you know when you need to trigger it. _rx.angular.js_ offers many ways you can integrate Observables to your Angular code base. My _Async filter_ is a more simple approach, but it should cover the basic situation where you need to update the view at the end of an Observable pipeline. You could also go the most simple route and not add any additional dependencies at all, and just `.do(() => $scope.$applyAsync())` to trigger a digest cycle when an Observable emits a value. You can always start simple and add more libraries if you need more features.

Actually, the _Async filter_ is not even limited to just working with RxJS observables, you could quite easily use it with other reactive libraries like [Bacon.js](https://baconjs.github.io/) or [Kefir.js](https://rpominov.github.io/kefir/). You would just need to change the `.subscribe` method to use `.onValue` instead (pull requests are welcome to make it more interoperable).

<br><br>
Photo credits: [Highway 101 by kromped](https://www.flickr.com/photos/30602491@N00/15755743274/) licensed under [CC BY-NC 2.0](https://creativecommons.org/licenses/by-nc/2.0/) & modified by me.
