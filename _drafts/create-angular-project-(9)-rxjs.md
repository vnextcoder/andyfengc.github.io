---
layout: post
title: Create Angular v2+ project (9) - rxjs
author: Andy Feng
---

# Introduction #
[Asynchronous programming](https://en.wikipedia.org/wiki/Asynchrony_(computer_programming)) is an important technique to create web applications. It allows units of work to run separately from the primary application thread and makes main application responsive.

[RxJS](http://reactivex.io/rxjs) is a library for reactive programming using Observables. It help us create asynchronous or callback-based applications quicker and easier. [Angular 2+](https://angular.io) uses [RxJS](https://angular.io/guide/rx-library) to implement asynchronous operations.

# Observables in rxjs #
[RxJS](http://reactivex.io/rxjs) is a library for reactive programming using Observables. It help us create asynchronous or callback-based applications quicker and easier. [Angular 2+](https://angular.io) uses [RxJS](https://angular.io/guide/rx-library) to implement asynchronous operations.

`Observable` provides support for passing messages between publishers and subscribers in our application. It helps us solve event handling, asynchronous programming issues.

> Observable defines a subscriber function to publish events/values to consumers(observers) subscribe to it.
> 
> An Observable instance begins publishing values only when someone subscribes to it.
> 
> We subscribe by calling the subscribe() method of the instance, passing an observer object to receive the notifications.

`Observer` is a handler for receiving observable notifications implements the Observer interface. It is an object that defines callback methods to handle the three types of notifications that an observable can send:

- `next()`	Required. A handler for each delivered value. Called after execution starts. It defines the actual handler logic.
- `error()`	Optional. A handler for an error notification. An error halts execution of the observable instance.
- `complete()`	Optional. A handler for the execution-complete notification. Delayed values can continue to be delivered to the next handler after execution is complete.

`Observer` is event handler and receives event data published by an observable as a stream.

![](/images/posts/20180411-rxjs-2.png)

## Steps to use observable/observer ##

1. create an `Observable` instance

		// Use the Observable constructor to create an observable instance
		const sequence = new Observable();	

	or

		// define a observable instance. It emits values in a sequence to subscribers(consumers)
		const sequence = Observable.of(...items);
		
	or

		//Converts its argument to an Observable instance. This method is commonly used to convert an array to an observable.
		const sequence = Observable.from(iterable)

		
1. we define a subscriber function inside this instance. this function accepts observer object and put into a list

		const sequence = new Observable((observer) => {
			  // synchronously deliver 1, 2, and 3, then complete
			  observer.next(1);
			  observer.next(2);
			  observer.next(3);
			  observer.complete();
		}

	equivalent to

		const sequence = Observable.of(1, 2, 3);

1. a consumer(observer) calls the subscribe() method of the observable instance. Then pass event handlers

		const sequenceSubscription1 = sequence.subscribe({
			// event handlers
			next() { ... }
			error() { ... }
		});

		const sequenceSubscription2 = sequence.subscribe({
			// event handlers
			next() { ... }
		});

	Subscriber function received an `observer` object, the observer object defines next(), or error()/complete() methods.

1. The observable publish events as a stream and pass values to observers' next() method

## Broadcasting/multicasting ##
Typically, a typical observable creates a new, independent execution for each subscribed observer. When an observer subscribes, the observable wires up a separate event handler and delivers values to that observer.

If we want each subscription of observer(consumer) get the same value, we need multicasting technique. Multicasting is the practice of broadcasting to a list of multiple subscribers in a single execution with the same event data.

We make some changes in above steps. When we subscribe observer to the observable, we add observers to an array(list):

		function multicastSequenceSubscriber() {
		  const seq = [1, 2, 3];
		  // Keep track of each observer (one for every active subscription)
		  const observers = [];
		
		  // Return the subscriber function (runs when subscribe()
		  // function is invoked)
		  return (observer) => {
		    observers.push(observer);
		    return {
		      unsubscribe() {
		        // Remove from the observers array so it's no longer notified
		        observers.splice(observers.indexOf(observer), 1);
		      }
		    };
		  };
		}

		// Create a new Observable that will deliver the above sequence
		const multicastSequence = new Observable(multicastSequenceSubscriber);

# RxJS #
[Reactive programming](https://en.wikipedia.org/wiki/Reactive_programming) is an asynchronous programming paradigm concerned with data streams and the propagation of change. It assumes all events published as a stream and listenable, e.g. keystrokes, an HTTP response, or an interval timer.

`RxJS (Reactive Extensions for JavaScript)` is a library for reactive programming using observables that makes it easier to compose asynchronous or callback-based code. `RxJS` provides an implementation of the Observable type. The library also provides utility functions for creating and working with observables. These utility functions can be used for:

- Converting existing code for async operations into observables
- Iterating through the values in a stream
- Mapping values to different types
- Filtering streams
- Composing multiple streams

## Create observables ##
`RxJS` offers a number of functions that can be used to create new observables. These functions help us create observables from events, timers, promises, and so on.

- Create an observable from a promise

		import { fromPromise } from 'rxjs/observable/fromPromise';
		
		// Create an Observable from a promise
		const data = fromPromise(fetch('/api/endpoint'));
		// Subscribe to begin listening for async result
		data.subscribe({
		 next(response) { console.log(response); },
		 error(err) { console.error('Error: ' + err); },
		 complete() { console.log('Completed'); }
		});

- Create an observable from a counter

		import { interval } from 'rxjs/observable/interval';
		
		// Create an Observable that will publish a value on an interval
		const secondsCounter = interval(1000);
		// Subscribe to the observable
		secondsCounter.subscribe(n =>
		  console.log(`It's been ${n} seconds since subscribing!`));

- Create an observable from an event

		import { fromEvent } from 'rxjs/observable/fromEvent';
		
		const el = document.getElementById('my-element');
		
		// Create an Observable that will publish mouse movements
		const mouseMoves = fromEvent(el, 'mousemove');
		
		// Subscribe to start listening for mouse-move events
		const subscription = mouseMoves.subscribe((evt: MouseEvent) => {
		  // define the event handling logic - Log coords of mouse movements
		  console.log(`Coords: ${evt.clientX} X ${evt.clientY}`);
		
		  // When the mouse is over the upper-left of the screen,
		  // unsubscribe to stop listening for mouse movements
		  if (evt.clientX < 40 && evt.clientY < 40) {
		    subscription.unsubscribe();
		  }
		});

- Create an observable that creates an AJAX request

		import { ajax } from 'rxjs/observable/dom/ajax';
		
		// Create an Observable that will create an AJAX request
		const apiData = ajax('/api/data');
		// Subscribe to create the request
		apiData.subscribe(res => console.log(res.status, res.response));

## Operators ##
`Operators` are functions that build on the observables foundation to enable sophisticated manipulation of collections. e.g. map(), filter(), concat(), and flatMap(). An operator observes the source observable’s emitted values, transforms them, and returns a new observable of those transformed values.

	import { map } from 'rxjs/operators';
	
	Observable.of(1, 2, 3).map((val: number) => val * val).subscribe(x => console.log(x))

RxJS provides many operators (over 150 of them). 

e.g. `catchError` operator that lets us handle known errors from the events of observable. It helps us catch this error and supply a default value, then our stream continues to process values rather than erroring out.

	import { ajax } from 'rxjs/observable/dom/ajax';
	import { map, catchError } from 'rxjs/operators';
	// Return "response" from the API. If an error happens,
	// return an empty array.
	const apiData = ajax('/api/data')
		.pipe(
		  map(res => {
		    if (!res.response) {
		      throw new Error('Value expected!');
		    }
		    return res.response;
		  }),
	  	catchError(err => Observable.of([]))
	)
	.subscribe({
	  next(x) { console.log('data: ', x); },
	  error(err) { console.log('errors already caught... will not run'); }
	});

## Pipe ##
pipes to link operators together and allows us to combine multiple operator functions into a single function.

	import { pipe } from 'rxjs/util/pipe';
	import { filter, map } from 'rxjs/operators';
	
	Observable.of(1, 2, 3, 4, 5)
	.pipe(
	  filter(n => n % 2),
	  map(n => n * n)
	)
	.subscribe(x => console.log(x));

# Subject in rxjs #
Subject is a practice of publisher-subscriber model in RxJS. It allows us to define our own observable and observer.

- Observable is an object allows us to emit/publish an event. It has all the Observable operators, and we can subscribe to him.
- Observer is an object allows us to subscribe an observable.
- Subject is both an Observable and Observer allows us to both publish and subscribe.

Steps to use Subject:

1. create a Subject instance of component: 
	
		const subject = new Subject<data_type/event_type>(); 

	datatype can be boolean, string, number...

1. Subject is observable and it means he has all the operators (map, filter, etc. ) and we can subscribe to him.

		subject.subscribe(val => console.log(`First observer ${val}`));

	or

		subject.map(value => `Observer one ${value}`).subscribe(value => {
		  console.log(value);
		});

	here, we subsribe to the subject object. When the subject object changes, the console logs. 

2. Subject is observer and it listens to observable with next(), error(), and the complete() methods. Here is the Subject object methods:

	![](/images/posts/20180411-rxjs-1.png)

		subject.next(event.target.value);

	When we call the next() method of Subject object, it publish the value of event and every subscriber will get this value. 

	We can also trigger error() and complete() of Subject object.

In typical senario, we have the source `Observable` and many `observers`, and multiple observers share the same Observable execution.

# Subject demo #
We will have a textbox. when we enter something inside the textbox, it bounds x seconds and display the result.

1. template: `ng2.component.html`

 		<input type="text" placeholder="Enter message" (keyup)="keyup($event)" [(ngModel)]="message">

1. component: `ng2.component.ts`

		import { Component, OnInit, OnDestroy } from '@angular/core';
		import { Subject } from 'rxjs';
		import { NgModel } from '@angular/forms';
	
		@Component({
		  selector: 'app-ng2',
		  templateUrl: './ng2.component.html',
		  styleUrls: ['./ng2.component.scss']
		})
		export class Ng2Component implements OnInit, OnDestroy {
		
		  constructor() { }
		  
	  	  public message : string;
		  // define a Subject object as observer and it publish string as event objects
		  public messageSubject = new Subject<string>();
		
		  ngOnInit() {
		    // we have some observers subscribe to this subject object
		    this.messageSubject.debounceTime(1000).subscribe(value => {
		      console.log('I waited 1 seconds ', value);
		    })
		    this.messageSubject.debounceTime(2000).subscribe(value => {
		      console.log('I waited 2 seconds ', value);
		    })
		    this.messageSubject.debounceTime(3000).subscribe(value => this.welcome(value))
		  }
		
		  ngOnDestroy() {
		    this.messageSubject.unsubscribe();
		  }
		
		  keyup($event) : void{
		     // it is publishing this value to all the subscribers that have already subscribed to this message
		     this.messageSubject.next(this.message);
		  }
		
		  welcome(value : string) : void{
		    console.log('welcome ', value);
		  }
		}

1. run the demo. enter something.

	![](/images/posts/20180411-rxjs-3.png)

# Observables in Angular #

Angular makes use of observables as an interface to handle a variety of common asynchronous operations. e.g.

- The EventEmitter class extends `Observable`, specifically `Subject`
- The HTTP module uses observables to handle AJAX requests and responses.
- The Router and Forms modules use observables to listen for and respond to user-input events.

//todo

## Event emitter ##
## HTTP ##
## Async pipe ##
## Router ##
## Reactive forms ##

# References #
[reactivex Subject](http://reactivex.io/documentation/subject.html)

[Angular observables](https://angular.io/guide/observables)

[rxjs operations](https://github.com/btroncone/learn-rxjs/blob/master/operators/complete.md)