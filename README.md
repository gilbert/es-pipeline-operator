# ES2016 Proposal: The Pipeline Operator

This proposal introduces a new operator `|>` similar to
  [F#](https://en.wikibooks.org/wiki/F_Sharp_Programming/Higher_Order_Functions#The_.7C.3E_Operator),
  [OCaml](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Pervasives.html#VAL%28|%3E%29),
  [Elixir](https://www.safaribooksonline.com/library/view/programming-elixir/9781680500530/f_0057.html),
  [Elm](https://edmz.org/design/2015/07/29/elm-lang-notes.html),
  [Julia](http://docs.julialang.org/en/release-0.4/stdlib/base/?highlight=|%3E#Base.|%3E),
  and [LiveScript](http://livescript.net/#piping),
  as well as UNIX pipes. It's a backwards-compatible way of streamlining chained function calls in a readable, functional manner, and provides a practical alternative to extending built-in prototypes.

## Sidenote

Hello, functional-programming friends! If you like this proposal, you will certainly like the [proposal for easier partial application](https://github.com/mindeavor/es-papp). Take a look and star if you like it!

## Introduction

The pipeline operator is essentially a useful syntactic sugar on a function call with a single argument. In other words, `sqrt(64)` is equivalent to `64 |> sqrt`.

This allows for greater readability when chaining several functions together. For example, given the following functions:

```js
function doubleSay (str) {
  return str + ", " + str;
}
function capitalize (str) {
  return str[0].toUpperCase() + str.substring(1);
}
function exclaim (str) {
  return str + '!';
}
```

...the following invocations are equivalent:

```js
let result = exclaim(capitalize(doubleSay("hello")));
result //=> "Hello, hello!"

let result = "hello"
  |> doubleSay
  |> capitalize
  |> exclaim;

result //=> "Hello, hello!"
```

### Functions with Multiple Arguments

The pipeline operator does not need any special rules for functions with multiple arguments; JavaScript already has ways to handle such cases.

For example, given the following functions:

```js
function double (x) { return x + x; }
function add (x, y) { return x + y; }

function boundScore (min, max, score) {
  return Math.max(min, Math.min(max, score));
}
```

...you can use an arrow function to handle multi-argument functions (such as `add`):

```js
let person = { score: 25 };

let newScore = person.score
  |> double
  |> _ => add(7, _)
  |> _ => boundScore(0, 100, _);

newScore //=> 57

// As opposed to: let newScore = boundScore( 0, 100, add(7, double(person.score)) )
```

*Note: The use of underscore `_` is not required; it's just an arrow function, so you can use any parameter name you like.*

As you can see, because the pipe operator always pipes a single result value, it plays very nicely with the single-argument arrow function syntax. Also, because the pipe operator's semantics are pure and simple, it could be possible for JavaScript engines to optimize away the arrow function.

### Usage with Function.prototype.papp

If the [papp proposal](https://github.com/mindeavor/es-papp) gets accepted, the pipeline op would be even easier to use. Rewriting the previous example:

```js
let person = { score: 25 };

let newScore = person.score
  |> double
  |> add.papp(7)
  |> boundScore.papp(0, 100);

newScore //=> 57
```

Clean! Also, `papp` is [easy to polyfill](https://github.com/mindeavor/es-papp#polyfill), so you don't have to wait to make use of its benefits.

## Motivating Examples

### Object Decorators

Mixins via `Object.assign` are great, but sometimes you need something more advanced. A **decorator function** is a function that receives an existing object, adds to it (mutative or not), and then returns the result.

Decorator functions are useful when you want to share behavior across multiple kinds of objects. For example, given the following decorators:

```js
function greets (person) {
  person.greet = () => `${person.name} says hi!`;
  return person;
}
function ages (age) {
  return function (person) {
    person.age = age;
    person.birthday = function () { person.age += 1; };
    return person;
  }
}
function programs (favLang) {
  return function (person) {
    person.favLang = favLang;
    person.program = () => `${person.name} starts to write ${person.favLang}!`;
    return person;
  }
}
```

...you can create multiple "classes" that share one or more behaviors:

```js
function Person (name, age) {
  return { name: name } |> greets |> ages(age);
}
function Programmer (name, age) {
  return { name: name }
    |> greets
    |> ages(age)
    |> programs('javascript');
}
```

### Validation

Validation is a great use case for pipelining functions. For example, given the following validators:

```js
function bounded (prop, min, max) {
  return function (obj) {
    if ( obj[prop] < min || obj[prop] > max ) throw Error('out of bounds');
    return obj;
  };
}
function format (prop, regex) {
  return function (obj) {
    if ( ! regex.test(obj[prop]) ) throw Error('invalid format');
    return obj;
  };
}
```

...we can use the pipeline operator to validate objects quite pleasantly:

```js
function createPerson (attrs) {
  attrs
    |> bounded('age', 1, 100)
    |> format('name', /^[a-z]$/i)
    |> Person.insertIntoDatabase;
}
```

### Usage with Prototypes

Although the pipe operator operates well with functions that don't use `this`, it can still integrate nicely into current workflows:

```js
import Lazy from 'lazy.js'

getAllPlayers()
  .filter( p => p.score > 100 )
  .sort()
|> _ => Lazy(_)
  .map( p => p.name )
  .take(5)
|> _ => renderLeaderboard('#my-div', _);
```

### Real-world Use Cases

Check out the [Example Use Cases](https://github.com/mindeavor/es-pipeline-operator/wiki/Example-Use-Cases) wiki page to see more possibilities.
