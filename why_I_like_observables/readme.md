# 1. What's this
This is a Knowledge Share on Observables. Why I like em', why they're awesome, and how to get that awesomeness.

This is all filled with my opinions and thoughts, so I might be wrong on some takes. Also, I do not have extensive experience in a variety of languages, so my comparison ability to other non-js things, is limited.

Also I used Markdown cuz ~~I'm lazy and didn't really have enough time for Prezi or Powerpoint and~~ that way I can leave this file afterwards, so it can be read in the future and __attempt__ to encapsulate what I explain in written format.

Nonetheless, here we go with the Awesome Explanation of stuff... by:

ME

![Francisco Santorelli][sexyMe]

so sit tight, cuz I'm secretly dumb😉

# Small Intro
## Reactive Programming
Instead of being declarative, and working around `eventListeners`, `eventHandlers`, etc; we think of everything that _behaves_ as a stream of _things_ to which we react and _side effects_ can occur.
It's another way of thinking about code (_allegedly_, I don't feel it's thaaat different as callback handling!), but it does create code that's more declarative (a happens, then b, then c) on what it's doing, but not as imperative (do a, then do b, then do c). I'll try to give examples of this...

## Rxjs
Cool library with a [cool documentation](https://rxjs.dev/api) that adds a lot of `discoverable operators` so we can handle our streams in the way we want.
A caveat I'll mention: Don't fall into the trap of thinking rxjs is a bunch of operators that we have to memorize. Instead, when we build streams (instead of being imperative), we will get random questions like _how can I split my stream into two, and then share the second one but not the first_ or whatever.. and that's when you search for operators that modify the stream in such a way.

Confusing probably. But hear me out.

# 2. Soooo we gotta use Reactive libraries.
<div align="center"><img src="https://c.tenor.com/f0c-54Kd2TwAAAAC/but-why-ryan-reynolds.gif"></div>


- ~~Frontend developers wanted to feel relevant so they made things more complicated~~
- Easier to reason about in comparison to promises / callback hells (C.H.)
- We can keep connections alive for as long as we need them
- Because our company uses Angular so we have to ( ರ ͜ʖ ರೃ )


## Advantages it gives us...? if any
### Differences with Promises
We get more readable code when dealing with concatenated queries.
As an example, I made up a concatenation of calls, then compared how it's done in promises versus non-promises.

```typescript
function findImpressions() {
  getUser().then(
    user => getFavoriteMediaPlans(user).then(
      mediaPlans => getCampaignsForMediaPlan(mediaPlan[0]).then(
        campaigns => getCreativesInCampaigns(campaigns).then(
          creatives => getTotalImpressionsForCreatives(creatives).then(
            impressions => totalImpressions = impressions;
          ).catch(handleError)
          .finally(sendEventToParentComponent)
        ).catch(handleError)
      ).catch(handleError)
    ).catch(handleError)
  ).catch(handleError)
}
```
or
```typescript
async function findImpressions() {
  try {
    const user = await getUser();
    try {
      const mediaPlans = await getFavoriteMediaPlans(user);
	  try {
		const campaigns = await getCampaignsForMediaPlan(mediaPlan[0]);
		try {
		  const creatives = await getCreativesInCampaigns(campaigns );
		  try {
			const totalImpressions = await getTotalImpressionsForCreatives(creatives);
		  } catch (e) { throw e }
		  finally {
			sendEventToParentComponent();
		  }
		} catch (e) { throw e }
	  } catch (e) { throw e }
	} catch (e) { throw e }
  } catch (e) {
	handleError(e);
  }
}
```

--rxjs
```typescript
function findImpressions() {
  getUser().pipe(
    concatMap(user => getFavoriteMediaPlans(user)),
    concatMap(mediaPlans => getCampaignsForMediaPlan(mediaPlan[0])),
    concatMap(campaigns => getCreativesInCampaigns(campaigns),
    concatMap(creatives => getTotalImpressionsForCreatives(creatives)),
  ).subscribe({
    next: totalImpressions = impressions,
    err: handleError,
    complete: sendEventToParentComponent,
  });
}
```
### Code is more readable.. when speaking complexity
So let's say we want to react to user input, and search for something with whatever it gives us.

```typescript
let results = [];
const input = document.getElementById('myInput');
```
```typescript
// imperative way

input.addEventListener('keyup', async () => {
	results = await fetchFromService(input.value);
});
```
```typescript
// reactive(?) way ( ͡° ͜ʖ ͡°)

fromEvent(input, 'keyup').pipe(
	switchMap(() => fetchFromService(input.value)),
).subscribe({
    next: results => results = results
});
```

Reactive way indeed seems worse in this example.
But, our solution is not ideal , for every input we are sending a request to the server. We want to wait half a second after user stops writing to actually send the request.

```typescript
// imperative way

let myDebounce; // <- added
input.addEventListener('keyup', async() => {
    clearInterval(myDebounce); // <- added
	myDebounce = setInterval(() => { // <- added another
	    clearInterval(myDebounce); // <- yet another
		results = await fetchFromService(input.value);
    }, 500); // <- a lot of things
});
```
```typescript
// reactive!! way ( ͡° ͜ʖ ͡°)

fromEvent(input, 'keyup').pipe(
    debounceTime(500), // <- added this
	switchMap(() => fetchFromService(input.value)),
).subscribe({
    next: results => results = results
});
```

Let's say we now don't want to fetch twice for the same input (so if the user types something, but it hasn't changed, we don't  throw a new fetch).
```typescript
// imperative way

let myDebounce;
let lastSearched = ''; // <- added
input.addEventListener('keyup', async() => {
    clearInterval(myDebounce);
	myDebounce = setInterval(() => {
	    clearInterval(myDebounce);
		if (lastSearched !== input.value) { // <- added
		    lastSearched = input.value; // <- added also
			results = await fetchFromService(lastSearched);
		}
    }, 500);
});
```
```typescript
// reactive!! way ( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)

fromEvent(input, 'keyup').pipe(
    debounceTime(500),
    map(() => input.value), // <- two lines
    distinctUntilChanged(), // <- added this time
	switchMap(() => fetchFromService(input)), // <- modified
).subscribe({
  next: results => results = results
});
```

and what if we want to ignore searching if the user has typed less than three (3) characters, and also ignore results with more than ten (10) elements?
```typescript
// imperative way

let myDebounce;
let lastSearched = '';
input.addEventListener('keyup', async() => {
	if (input.value.length < 3) return; // <- this

    clearInterval(myDebounce);
	myDebounce = setInterval(() => {
	    clearInterval(myDebounce);
		if (lastSearched !== input.value) {
		    lastSearched = input.value;
			const list = await fetchFromService(lastSearched); // <- modified
			if (list.length < 10) { // <- added
				results = list; // <- added T_T
			}
		}
    }, 500);
});
// starts to become unreadable. Also, we have variables outside.
// start considering encapsulating into a class?
```
```typescript
// reactive way ( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)

fromEvent(input, 'keyup').pipe(
    debounceTime(500),
    map(() => input.value),
    filter(input => input.length > 3), // <- this is new
    distinctUntilChanged(),
	switchMap(() => fetchFromService(input)),
	filter(list => list.length < 10), // <- and this too
).subscribe({
  next: results => results = results
});
```

AND WHAT IF NOW WE WANT TO STOP AFTER THREE RESULTS ARE FETCHED
_OMG FRAN IS SO ANNOYING_

```typescript
// imperative way =(

let myDebounce;
let lastSearched = '';
let searchesCount = 0;
let inputEvent = input.addEventListener('keyup', async() => { // <- modified
	if (input.value.length < 3) return;

    clearInterval(myDebounce);
	myDebounce = setInterval(() => {
	    clearInterval(myDebounce);
		if (lastSearched !== input.value) {
		    lastSearched = input.value;
			const list = await fetchFromService(lastSearched);
			if (list.length < 10) {
				results = list;
				searchesCount += 1;

				if (searchesCount >= 3) { // <- added
					removeEventListener(inputEvent); // <- added
				} // <- added
			}
		}
    }, 500);
});
// yep.. it looks very spaghetti now
```
```typescript
// reactive way ( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)( ͡° ͜ʖ ͡°)

fromEvent(input, 'keyup').pipe(
    debounceTime(500),
    map(() => input.value),
    filter(input => input.length > 3),
    distinctUntilChanged(),
	switchMap(() => fetchFromService(input)),
	filter(list => list.length < 10),
	take(3), // as simple as this
).subscribe({
  next: results => results = results
});
```
You get the gist of it...
With rxjs building complex reactions to any behavior is done in a cleaner better way. Of course, this requires us to think in a _reactive_ way, in the sense that:
these actions we react to, instead of _triggering_ functions that _communicate_ between them and create complex logic,
we attach ourselves to a stream, that creates side effects, and handles the emissions in whatever way it finds fit.

### Streams can stay alive
So if we wanted to show the results of a fetch in two different elements, we would have to save that result in a variable, which is accessed by both parties:

```typescript
let result;
fetch(something).then(res => result = res);
// and then, start showing `result` in both places.
```
but if both elements have to live at the same time. Who's calling it? Now we need a service. Now we need for both elements to await a result before requesting the data, etc.

In rxjs that would be done with:
```typescript
const result = fetch(something).pipe(
	// make this stream have multiple subscribers,
	// and emit the last value on subscription
	shareReplay(), 
);
// elements can just result.subscribe() and both will get the same
// value with only one fetch call.
```

But what if one of the elements wants to do something with the resulting data and the other element something else? Now they have to create their own copy of the result, after waiting, etc.
But in streams __all streams are immutable__, this means that any operation done to an observable, creates a completely new one, that doesn't affect the logic or life of the original unmodified stream.

<p align="center"><img align="center" src="https://media0.giphy.com/media/75ZaxapnyMp2w/giphy.gif?cid=ecf05e47s01ma01hte7mn2svnr9fgi12tt3nxsjc1j02u8z0&rid=giphy.gif&ct=g"></p>

#### plus other stuff I probably forgot.. which brings me to

# Tools that rxjs gives us
![rxjs tools][rxjsTools]

## Hot / Cold Observables (what is multicasting)
<dl>
  <dt>Cold 🥶</dt>
  <dd>Think of https calls. These observables deliver the whole stream to a single observer, and is not initialized until a subscription has been made. If multiple observers subscribe, a stream is created for each one of them</dd>
  <dt>Hot 🥵</dt>
  <dd>Think of event callbacks (like the input example above). These observables emit unrelated to whether someone is listening. They can <i>multicast</i>,  which is a fancy word for saying they can have multiple listeners in parallel<dd>
</dl>

Cold observables [can be transformed into hot observables](https://www.learnrxjs.io/learn-rxjs/operators/multicasting) through some operators given by rxjs, but the new hot observable would _require_ the cold observable to have started.
This means that at least one subscriber is needed on the original cold observable, to kickstart the hot-created one.

Another thing to keep in mind, is that cold observables complete after they do their inner logic, while hot observables tend to continue until the stream has ended. If we exit prematurely, the cold observable would continue and close. But the hot one wouldn't close, while not existing anymore(???), so we gotta be careful not to create memory leaks with hot observables.

## Subjects
These are special Objects that rxjs provides, because they are Observables and Streams at the same time.
There are four types for them, so I will try to briefly go to each and give a use case, as they are kinda niche.
Also, they all are multicasting observables. So be wary to avoid memory leaks!.

#### Subject
```typescript
const base = new Subject<Type>();
```
works as a stream to which we can pass (`.next`) emissions by hand. Any new subscriber will start receiving events after subscription (previous events are lost).

Let's say we want to wait for some image to load to send a message to the user, and after wards, everytime the image is clicked, we send another. We can create a Subject that the message_receiver will attach to, and we can send our first value with `.next` after image loads, and then we merge that with the click emitter to keep sending the messages. Subjects allows us to create streams in an inner way by hand.

#### BehaviorSubject
```typescript
const behave = new BehaviorSubject<Type>(initialValue);
```
This is a superset of `Subject`, with the difference that it emits it's latest value to new subscribers on emission. This is useful in something like a Football score stream. Where we want any new subscriber to get the latest score of the game, even if they arrived 'later' than when the goal happened.

It requires an initial value though, to emit something to new subscribers.

#### AsyncSubject
```typescript
const asyncsub = new AsyncSubject<Type>();
```
It collects inner emissions, and then emits whatever happened last, when the subject completes.
Imagine a click game in which we count the amount of clicks given in a set time. Then we emit the value when the game completes (which would complete the AsyncSubject).

#### ReplaySubject
```typescript
const replay = new ReplaySubject<Type>(4);
```
The ReplaySubject is like the BehaviorSubject, but this one keeps a history of emissions, up to a point (length is the value given to it on creation). In the code above, if this is emitting one number at a time from 0 to 10, and I subscribe after 6 is emitted, I'd receive four (4) emissions on subscription: 3, 4, 5 and 6; as it is replaying up to 4 states in the past for me. Then it behaves as a Subject.

An example would be to display a history tab on some part of the app, to show 'the last 5 movements' on a chess game, for example.


## Some useful Operators (for me)
They are divided by the [learn rxjs web](https://www.learnrxjs.io/learn-rxjs/operators) in categories. I'll share some info about some of them:
#### Combination
- _combineLatests_: After all observables emit at least once, keep sending the last emission of each.
- _forkJoin_: After all observables complete, emit the last value of each in an array.
- _pairwise_: give the previous and new emission of an observable in an array.
- _race_: whichever observable emits first, that's used.
- _startWith_: add an initial value to an observable (just like a BehaviorSubject)

#### Conditional
- _every_: If all values pass some test, then it emits `true` before completion, or if it breaks, it emits false. Used to check if any value is not passing the test to complete the test and raise an alarm.

#### Creation
- _empty_: An observable that completes on subscription.
- _from_: Transform a Promise to an observable (iterables also get transformed).
- _fromEvent_: Make event listeners become observables.
- _interval_: Create an observable that starts emitting at a given interval numbers in order. Beware as this won't end by itself.
- _range_: Emit numbers in a range from - to.
- _of_: Create an observable of any value.
- _throw_: Create an observable that errors on subscription.

#### Error handling
- _catchError_: Intercept an error and return any follow-up observable to continue the stream.
- _retry_: After error, attempt the given amount of times before emitting error and completing.

#### Multicasting
- _share_: Every operator and side-effect of a stream will happen only once per observer, up to the point this operator is in.
- _shareReplay_: Make an observable become hot and emit the last value to new subscribers.

#### Filtering
- _debounceTime_: ignore emissions until given time has passed after last emission, then emit that last value.
- _distinctUntilChanged_: ignore emissions that are equal to immediate past emission.
- _filter_: ignore emissions that do not past the given test.
- _take_: accept the given number of emissions, and then complete.
- _takeUntil_: make the observable complete after the given observable emits. Useful to close hot observables with the `destroy$` pattern given below.
- _throthle_: every given time, all the emissions that happened during that time, are emitted together.
#### Transformation
- _mergeMap_: Make a stream be followed by another stream. Emissions happen as they come.
- _concatMap_: Make a stream follow another, but order of emissions is important.
- _exhaustMap_: Make a stream follow another, but ignore emissions of the first one until the second one emits.
- _switchMap_: Make a stream follow another, but if a new emission of the first comes, cancel the second request and redo it.
- _expand_: Recursively keep subscribing to the emitted second stream.

#### Utility
- _tap_: do some side effect unaffecting the stream, at any point of it.


#### And many more...

# Best practices && Angular
## Avoid Callback Hell (imperative using)

```typescript
// this is akin to callback hell. Don't do it. Use operators of *Map to avoid this.
firstCall().subscribe(value1 => {
  secondCall(value1).subscribe(value2 => {
    thirdCall(value3).subscribe(doSomething);
  });
});
```

## Always close your Subjects 

```typescript
destroy$ = new AsyncSubject();

hotObs = (new Subject()).asObservable().pipe(
	takeUntil(this.destroy$),
);

ngOnDestroy() {
    this.destroy$.next(true);
    this.destroy$.complete();
}
```

## Use the async pipe
Async pipe makes life easier in Angular when thinking on streams. So going back to our example on reading user input.

Our usual solution:

```html
<input class="awesome-input"
	placeholder="search here"
	[(ngModel)]="inputValue"
	(ngModelChange)="onUpdateValue($event)"
/>
<li *ngFor="let item of data">{{ item }}</li>
```

```typescript
data = [];
inputValue = '';

onUpdateValue(value: string) {
	this.service.fetch(value).subscribe(data => this.data = data);
}
```

More complex, but more stream-y solution:

```html
<input class="awesome-input"
	placeholder="search here"
	(ngModel)="inputValue$ | async"
	(ngModelChange)="inputValue$.next($event)"
/>
<li *ngFor="let item of data$ | async">{{ item }}</li>
```

```typescript
inputValue$ = new BehaviorSubject('');
data$ = this.inputValue$.pipe(
	switchMap(value => this.service.fetch(value)),
);

// we made our component more scalable in face of complexity.
// Also removed methods from the class.
```

## Avoid passing streams as inputs
Usually, this is done so the children elements can update their values depending on what the stream is doing. But this is unnecesary with the async pipe

```html
<my-component [my-input]="myStream$ | async"></my-component>
```

this would update the `my-input` value each time the stream emits, but the child component doesn't have to know about any stream, or its lifetime. In this sense, all streams attempt to stay encapsulated to where they are relevant. And communication of streams downwards usually go by using the `async` pipe, and upwards by `EventEmitters`

## Harness the power of ReactiveForms
They already are thought to be handled in a reactive way. They have observables for their classes that notify status changes, validation changes, etc. They are very powerful although a little bit more complex than vanilla _banana in the box_ `[(ngModel)]`, but it's worth the pain.

## And idk what else to tell you. Here are some links so you can have fun if you find this interesting:
- [rxjs documentation](https://rxjs.dev/api)
- [rxjs operator finder](https://rxjs.dev/operator-decision-tree) <- this one seems fun
- [thinking reactively by mike pearson](https://www.youtube.com/watch?v=-4cwkHNguXE)
- [some reading on using OnPush on Angular](https://www.thinktecture.com/en/angular/whats-the-hype-onpush/)

# Hope you enjoyed this :)!

<h2 align="center">Thanks!</h2>

If you have any questions, and stuff, be sure to [drop me a line](mailto:oxspit@gmail.com)!


![mepart2][sexyerMe]

[sexyMe]: https://i.imgur.com/szKfUwh.png
[rxjsTools]: https://rxjs.tools/card.png
[sexyerMe]: https://i.imgur.com/W4NdBkE.png
