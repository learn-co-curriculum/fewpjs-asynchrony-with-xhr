# Asynchrony with XHR

## Learning Goals

- Identify `XMLHttpRequest` as a means for asynchronous data fetch in JavaScript
- Create an instance of the XHR class
- Configure the destination for an XHR instance's data fetch
- Configure a success handler on an XHR instance
- Configure a error handler on an XHR instance
- Request remote data using a configured XHR instance

## Introduction

Asynchrony feels unusual when programming because we're so used to programming
in languages that are synchronous: they run like they read, top down, left to
right, maybe a loop here or there, down into a function, back out from the
function, etc...

But in our day-to-day lives, we _love_ asynchrony.

- Pop some microwave popcorn in the microwave, then wander to put a DVD in the
  DVD-player: **asynchrony**
- Send one project team member out for coffee while the other team-members keep
  working: **asynchrony**
- Send a text message and then browse social media until you get an alert saying
  that you have a response: **asynchrony**

Asynchrony is really just a programmer word for the idea of "multi-tasking."
While the code might look a little bit strange, it should feel like a modest
stretch from your fundamentals in event-based JavaScript programming.

For the moment, we're going to pretend like `fetch()` doesn't exist. While a
great help, `fetch()` hides, or _abstracts away_ some of "what's really going on."
We'll use more primitive tools so that we can ***really*** understand asynchrony
in JavaScript.

Your code assistant in making asynchronous calls to remote servers for data is
(the long-named) `XMLHttpRequest`, or XHR object.

> _ASIDE_: It can retrieve other things besides XML. These days, it mostly
> retrieves JSON.

**Note:** If you would like to follow along using the code snippets in this lesson, create a basic `index.html` file, open the file in Chrome, then open the [console][]. From here, you can paste in the JavaScript code provided.

## Instantiate an XHR Instance

To create a new helper, we're going to create a new _instance_ of the XHR
_object_. We do this by invoking the `new` keyword on the name of the class.
Recall, the _class_ acts like a blueprint for creating multiple _instances_ of
that class. The classic example is that there is a single blueprint for a `Dog`
but there are a wondrous variety of instances of that "blueprint."

```js
let my_data_fetcher = new XMLHttpRequest();
```

This will create an XHR instance called `my_data_fetcher`.

## Direct an XHR Instance's Request

In order to retrieve data, our little buddy, `my_data_fetcher`, needs to know
where to search for information. We communicate that through the `open`
method on the XHR instance.

```js
// my_data_fetcher.open(HTTP_VERB, URL);
my_data_fetcher.open('GET', 'http://api.reddit.com');
```

The first argument to `open()` is the HTTP verb that we should use to connect
to the server. HTTP, the language of the web, recognizes multiple types of
connections: `GET` means "I want to retrieve information;" POST means "I want
to send you information." There are many HTTP verbs, but `GET` is the default and
is the verb that you use when you're "browsing the web."

The second argument is the URL you want the XHR instance to go visit. Here we
want to talk to the [reddit.com][] API to find out what's going on on the front
page.

It's very important to note **no work is being done yet.** The name `open()` is
slightly misleading. It does not mean "open a connection". It's more like,
"When we invoke `send()`, our last step (as you'll see later), this is where you
should go."

Before we trigger the real work by invoking `my_data_fetcher.send()`, we need
to configure what `my_data_fetcher` should do when it gets information. To do
this, we'll set up a "handler" for the case that our connection works
flawlessly ("success") **as well as** for the case that things don't go _quite_
right ("error"). XHR accomplishes this by having you set two event listeners
&mdash; just like you know from doing DOM programming.

## Configuring the Success Behavior for "load"

We use the exact same event-listening declaration to listen for success as you
might intuit from your JavaScript DOM programming experience. Recall:

```js
document.getElementById('razzle').addEventListener('click', () => {
  alert("You've been razzled!");
});
```

Similarly, we use `addEventListener` on our instance of the XHR class. For
example:

```js
my_data_fetcher.addEventListener('load', e => {
  console.log(`load finished event: ${e}`); // The "load" event is 'e'
  console.log(`Whole lotta JSON: ${my_data_fetcher.response}`); //JSON obj
});
```

The callback, the function that's the second argument to `addEventListener()`,
takes a single argument: an instance of a `load` `Event` object . We can
`console.log` it if we want to inspect it more. When the `load` event fires,
the original XHR instance, `my_data_fetcher`, will have the server's response
stored in its `.response` attribute. When we `console.log()` the `.response`
property, we will see a JSON representation of [reddit.com][]'s front page.

## Configuring the Error Behavior for "error"

Sometimes things don't go properly: the network is down, the remote server is
misconfigured and can't answer our request, etc. To solve this, we need to be
able to receive an `Event` that lets us know Bad Things have happened. To do
this, we listen for an "error" `Event`.

```js
my_data_fetcher.addEventListener('error', ev => {
  console.error('bad things: ${ev}');
});
```

To test your "error" callback, simply add some nonsense to the URL (e.g. `http://redditILovePoodles.com`).

## Putting it All Together

Let's put our code together and, at long last, invoke `send()` on
`my_data_fetcher`. We'll also add a few `console.log()` statements so that we
can see how synchronous code will continue executing _while_ our buddy,
`my_data_fetcher` is off talking to a remote computer on the internet.

```js
console.log('I run first');

let my_data_fetcher = new XMLHttpRequest();
my_data_fetcher.open('GET', 'http://api.reddit.com');

my_data_fetcher.addEventListener('load', e => {
  console.log(
    "On a good day, I run third and the 'error' callback doesn't run!"
  );
  console.log(`load finished event: ${e}`); // The "load" event is 'e'
  console.log(`Whole lotta JSON: ${my_data_fetcher.response}`); //JSON obj
});

my_data_fetcher.addEventListener('error', ev => {
  console.log("On a bad day, I run third and the 'load' callback doesn't run!");
  console.error('bad things: ${ev}');
});

my_data_fetcher.send();

console.log('I run second!');
```

A slightly more interesting `load` callback might do something like

```js
/* Write the top 3 posts' authors name in the browser */

e => {
  // Parse the returned JSON into an actual JavaScript Object
  JSON.parse(my_data_fetcher.response)
    .data // Look up its `data` key
    .children // Look up that result's `children` key
    .map(n => n.data.author) // Return each child's data.author property
    .slice(0, 3) // Take the first three names from the result array
    .forEach(name => document.write(name)); // Write those names to the document
};
```

Final product:

```js
console.log('I run first');

let my_data_fetcher = new XMLHttpRequest();
my_data_fetcher.open('GET', 'http://api.reddit.com');

my_data_fetcher.addEventListener('load', e => {
  console.log(
    "On a good day, I run third and the 'error' callback doesn't run!"
  );
  console.log(`load finished event: ${e}`); // The "load" event is 'e'
  console.log(`Whole lotta JSON: ${my_data_fetcher.response}`); //JSON obj
  JSON.parse(my_data_fetcher.response)
    .data // Look up its `data` key
    .children // Look up that result's `children` key
    .map(n => n.data.author) // Return each child's data.author property
    .slice(0, 3) // Take the first three names from the result array
    .forEach(name => document.write(name)); // Write those names to the document
});

my_data_fetcher.addEventListener('error', ev => {
  console.log("On a bad day, I run third and the 'load' callback doesn't run!");
  console.error('bad things: ${ev}');
});

my_data_fetcher.send();

console.log('I run second!');
```

This returns, in the console:

```text
I run first
I run second!
On a good day, I run third and the 'error' callback doesn't run!
load finished event: [object ProgressEvent]
Whole lotta JSON: {"kind": "Listing", "data": {"modhash": "", "dist": 25, "children": [{"kind": "t3", "data":...
```

## Conclusion

This shows you how to use the most basic tool for retrieving data
asynchronously in JavaScript: the `XMLHttpRequest` object. While most
developers use `fetch()` these days, it's not uncommon to be asked about XHR in
job interview questions. Additionally, XHR shows how the ideas behind
event-based programming were carried forward to support remote data retrieval.

We're going to work with XHR so that we gain a sense of some of its pain points. By understanding
these "rough edges," we'll appreciate why JavaScript added the `Promise` class to its 
bag of tricks and how `Promise`s can be stitched together to creat the `fetch()` function
we know and love.

## Resources

- [Using XMLHttpRequest][1]
- [XMLHttpRequest][2]

[1]: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest
[2]: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest
[reddit.com]: http://reddit.com
[console]: https://developers.google.com/web/tools/chrome-devtools/console/get-started
