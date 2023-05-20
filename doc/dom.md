# Library description

This is frameorc, a JavaScript library that helps building user interfaces
in browsers by manipulating DOM trees.

Brutally simple and efficient, it was created with the goal of endowing
a single programmer and small teams with formidable power.

It provides unique, flexible and composable syntax that is *nothing else
but standard JavaScript*, applied in an intelligent way.

The core is comprised of *13 short functions* in a single file[^0] that
the author never bothered to minify, gzip and measure the resulting size.
By the number of lines, this README is longer than the library code.

It *does not require* any build steps, bundlers, transpilers and such tools
to function. However, if one sees them necessary, frameorc *does not preclude*
their usage in any way.

It *does not impose* any methodology or ideology on the programmer.  Mixes and
matches well with declarative, functional and imperative styles where
appropriate. The same with promises, callbacks or async/await.

*Frameorc is not a framework.* It is a library. It does not demand to be front
and center, and it plays well with other libraries that can manage content
on the same page.

For the best results, it requires to be used by a competent programmer, who, among
other things:
- prefers to understand the technologies one is using and relying upon,
- considers frequent checks with reality not a burden, but the right way,
- knows about files, URLs and methods of transferring data in the Internet,
- has solid understanding of CSS, HTML, DOM and other APIs needed to achieve the goal,
- understands what variables, functions and modules are and can use them to organise
  the code,
- has a certain level of taste allowing one to tell apart things that make the code better
  from the things that make the code worse,
- knows when to stop.


[^0]: Another file is the only dependency, containing 975 lines of Snabbdom.


# In a nutshell

Instead of vague adjectives prone to misinterpretation,
let's have a look at some code:

Compare
```js
c.main.important`Alert`
```
to
```html
<div class="main important">Alert</div>
```

Here is a fragment of a [keypad component example](examples/keypad.html):

```js
body(
 c.H2`Keypad example`,
 c.keypad(
   c.Button(
     'C',
     on.click(() => v(0)),
     css.color`red`.fontWeight`bold`),
   c.Button(
     '-',
     on.click(() => v(-v()))),
   Array.from({ length: 10 }, (_, i) =>
     c.Button(
       i,
       attr.type`button`,
       key('bn', i),
       on.click(() => v(10*v() + i)))),
   c.Input.display(
     hook.insert((el) => el.elm.focus()),
     prop.value(v),
     cls.odd(() => v() % 2),
     cls.even(() => !(v() % 2)),
     on.input(e => v(Number(e.target.value) || 0)))),
);
```

For something more complex see how
[TodoMVC is implemented in frameorc](examples/todomvc/index.js) in just 130 lines.


# How to start

I trust you can obtain the files that comprise this library in any way that
is convenient to you. For example, using `git clone`, or by downloading a zip
archive of a particular branch or tag, or even not downloading anything and 
just hotlinking the content from GitHub.

Let's suppose you have downloaded and put the files into the directory that
is accessible as `/lib` on your server. In that case the minimal working example
will consist of these two files:

**index.html**
```html
<script src="index.js" type="module">
  This is an application. Enable JavaScript to run it.
</script>
```

**index.js**
```js
import { body } from '/lib/dom.js';
body('Hello, world!');
```

After setting up your server[^1], visit the URL on which `index.html` is served, 
and you should see 'Hello, world!' text in the browser.

[^1]: For example, run `python3 -m http.server` in the directory with `index` files;
  Another way: `deno run --allow-net --allow-read https://deno.land/std@0.186.0/http/file_server.ts`

# Components

Some components are to be included into the main frameorc repository.
See their respective documentation pages:

[router](doc/router.md)


# Step-by-step overview

## General approach

The library allows creation of DOM trees with elements, attributes and styles,
and provides highly efficient and expressive syntax to that end. 

> Instead of
inventing yet another templating language, the author concentrated his efforts on
leveraging the power of a programming language that has already been developed
for a long time, reviewed and enhanced, attained a large number of instruments,
such as linters, formatters, debuggers, large variety of other code analysis
and generation tools, including newly emergent AI-based ones, as well as numerous
libraries, textbooks, articles, instruction material of all levels and grades of
quality, and, finally, various accounts of experience of those who tried things,
succeeded or failed. Because of that, all aforementioned things are readily
available to be used with frameorc.

The thirteen functions comprising the library are: `body`, `c`, `attr`, `css`,
`on`, `Val`, `prop`, `cls`, `key`, `hook`, `Ref`, `attach`, `operator`.

## Static capabilities

This functionality is provided by the `body` setter function, and three
combinators: `c`, `attr` and `css`. By the term "combinator" within the scope of
this introductory text a higher-order function is understood such that can
exhibit abilities, examples of which are provided further.

### `body` function, setting the content of elements

1. Setting the content of an element is just a function call. There is a
special function `body` that sets the content of the current document's 
`<body>` element.

```js
body('Hello')
```

2. To set the content, you can call functions with any number of arguments. 
In this example, `body` is used, but the same stands for `c`, `Val` and `Ref`
explained further in the text.

```js
body('Hello ', 'world')
```

3. Nested arrays in any order and of any depth will be flattened and set
as element's children:

```js
body(
  'Hello ',
  [
    'brave ',
    ['and ', 'new ']
  ],
  'world'
)
```

4. Variables and functions can be used, obviously, as we are just 
writing code in JavaScript:

```js
let a = ['and ', 'new '];
let b = (v) => [v, a];
let c = 'world';
body('Hello ', b('brave '), c);
```

5. Not just function calls, but functions per se, as well as functions
that return functions which may return functions and so on can be used *as is*.
`f()` and `g()` would obviously work in the next example, as these calls produce
simple values. What we demonstrate is that `f` and `g`, note the lack of parentheses
designating a call, work as-is too, in combination with nested references and
arrays.

```js
let f = () => 'Hello';
let g = () => [f, ', ', 'world'];
body(g);
```

6. `null`, `undefined` and `false` are skipped, not generating any child content
in the element. All other data types are converted to strings and added as such.

```js
body(
  false && 'this will not be displayed',
  null,
  undefined,
  [],
  [[], [[]]],
  'this will be displayed: ',
  0,
  '', // an empty text node
  true)
```

### `c` combinator

7. To create an element, use the `c` combinator. By default, it creates a `<div>`

```js
body(
  c('Hello'),
  c('World'))
```

`c` and `body` can take exactly the same arguments: strings, arrays, functions,
other `c`s and operators discussed further in the document.


8. To create an element with a required **tag**, use its name starting with an
**uppercase latin letter**:

```js
body(
  c.Span('Hello '),
  c.B('World'))
```

9. Setting the **classes** of an element is done with a name starting with any
character other than an uppercase letter:

```js
body(
  c.important('text'))
```

10. Tags and classes perform the conversion from *CamelCase* to *kebab-case*

```js
body(
  c.MyElement('my-element'),
  c.multiWordClass('multi-word-class'))
```

The first underscore is replaced with minus. Any uppercase latin letter is
changed into lowercase. If this uppercase letter is not the beginning
character, it will be prepended by minus. This results in the following
substitutions:

```
"BackgroundColor" => "background-color",
"_backgroundColor" => "-background-color", and
"_BackgroundColor" => "--background-color".
```

This choice of case translation rules conform to the way JavaScript identifiers
correspond to property names in CSSOM. The translation is automatically applied
by frameorc to tag names, class names and CSS property names (see further the
`css` combinator section).

The translation is not done automatically for attributes (presented in the
`attr` section) and properties (`prop` combinator), as the former may have
both camel-case and dashed names, depending on the tastes of the people who
dictated the standards at the time of their introduction[^2]; the latter are
all JavaScript entities and, therefore, conform to its camelCase syntax
preference.

[^2]: For instance, look at the
  [list of SVG attributes](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute)
  and see for yourself what a variety of identifier styles can be found there.


11. Multiple classes and tags can be set to an element. Classes are accumulated.
Of *tags, only the last one wins,* others are discarded.

```js
body(
  c.MyElement.withClassOne.andClassTwo.andClassThree('yes, we can do that'))
```

12. It is obvious that in JavaScript code, tags and classes can be assigned
dynamically from variables.

```js
let tagName = 'MyElement';
let class1 = 'withClassOne';
let class3 = 'andClassThree';
body(c[tagName][class1].andCassTwo[class3]('yes'));
```

13. A construct that defines an element is not an element itself. It is
a function that will produce the element later. Therefore, it can be
referenced from a variable, and used to produce multiple different
instances of an element with the same properties and content.

```js
let elm = c.Li.numbered('Item');
body(c.Ul(elm, elm, elm));
```

14. An element construct can have classes added even after it has been
"called" (a call is just a syntax to add some children):

```js
body(
  c.Span.important('Really?').additionalClass1.andClass2)
```
  
```js
let elm = c.Li.numbered('Item');
body(c.Ul(elm, elm.selected, elm));
```

15. An element can have its tag changed at a later time:

```js
body(
  c.Span.important('This is a div.important').Div)
```

```js
let elm = c.Li.numbered('Item');
body(
  c.Ul(
    elm,
    elm.Dt.current, // <li> becomes <dt>
    elm));
```

16. An element can have content added after it has been defined, by using a
function call syntax. Later in this text it will be demonstrated that not only
content, but combinators that change the properties of the element can be added
in the same way.

```js
let elm = c.Li.numbered('Item');
body(
  c.Ul(
    elm,
    // The following will contain the text:
    // "Item with more content"
    elm(' with more content'),
    elm));
```

17. All the mentioned operations above can be chained and repeated. This is
useful when we have functions that generate the content that we want to amend
or override later

```js
body(
  c.Span.cls1('Hello')
   .Div.red.bold(', brave')
   (' and new ')
   .darkBorder.italicFont('world'))
```

has the same effect as

```js
body(
  c.Div.cls1.red.bold.barkBorder.italicFont(
    'Hello', ', brave', ' and new ', 'world'))
```

18. If the element does not have any complex content, it may be convenient to
save typing extra brackets and use the template string syntax

```js
body(c`Hello`)
```

19. The template string syntax works to the full extent and combines with other
parts of the library, such as value containers, described further in this
introduction. If necessary, you can do things like this:

```js
let world = c.Span` World`;
body(c`Hello, ${ world }!`);
```

The other equivalents are:

```js
body(
  c(
    'Hello, ',
    c.Span`World`,
    '!'))
```

```js
body(c`Hello, `(c.Span`World`, '!'))
```

These mechanisms are primarily meant for complex cases, such as authoring
and styling several layers of components. In simple cases, it is recommended
to keep it simple. The author hopes that more powerful techniques, when used
responsibly, will allow competent programmers to write succinct, elegant and
efficient code.


20. SVG elements can be created as any other elements. The library handles
namespace-related intricacies automatically.

```js
body(c.Svg(c.Rect()))
```

### `attr` combinator

21. To assign attributes to an element, use the `attr` combinator as the child
of the element. This combinator can be repeated, and can be used in any position
between the other children of the element. The same is true about furtherly
discussed other combinators that affect various properties of elements.

```js
body(
  c.Svg(
    c.Rect(
      attr.x(0),
      attr.y(0),
      attr.height(10),
      attr.width(10),
    )))
```

22. `attr` allows to chain the attributes

```js
body(
  c.Svg(
    c.Rect(
      attr.x(0).y(0).height(10).width(10),
    )))
```

23. `attr` can combine the attributes to which the same value is assigned

```js
body(
  c.Svg(
    c.Rect(
      attr.x.y(0).height.width(10),
    )))
```

24. All attribute values are converted to strings. If there are several
arguments in parentheses, they are converted to strings and concatenated.

```js
attr.fill('#', '00', 12, '34')
```

25. Arrays and functions can be used as intuitively expected:

```js
const toHex = v => ('00' + v.toString(16)).slice(-2);
let r = 0, g = 12, b = 34;
let rect = c.Rect(attr.fill('#', [r, g, b].map(toHex)));
body(
  c.Svg(
    rect(attr.x.y(10).height.width(20)),
    rect(attr.x(10).y(40).height.width(20)),
  ));
```

26. Template strings work as expected:

```js
attr.fill`#001234`.stroke`#000000`
```

### `css` combinator

27. Inline styles can be assigned in the same way as attributes. 
The combinator is called `css` for brevity, but it affects the `style` property
of a DOM element. 

Inline styles have the highest priority, **overriding anything assigned by the
CSS content** from linked stylesheet files and `<style></style>` tags. Depending
on your views, that may be seen as an advantage, because the style of the element
will clearly be as defined right in the place.

```js
let size = 10;
body(
  c.P(
    css.fontSize.lineHeight(size, 'px')
       .color`#333`.border`solid black 1px`,
    'Styles can be applied inline'));
```

28. As we have a programming language at our disposal, we should remember that we
can use variables and functions, and that it is in the programmer's power to
apply any kind of aesthetics or lack thereof to the code he is authoring.

```js
let negative = css.backgroundColor`#222`.color`#ccc`;
body(
  c(negative, 'Hello. Missed me?'));
```

29. This follows from the previous examples, but I'd like to highlight one
important point: it is entirely your choice whether to use inline styles or not.
The same stands for any other approach or methodology. This library is not
forcing anyone to commit to any certain way in particular. My other advice is not
to make any zealous commitments, and to be open to approaches that work in each
individual case, based on reason and good understanding of reality.

```js
body(
  c.Style`
    @import url("/lib/fonts/inter.css");

    body {
      font: 12px/1.35 Inter, sans-serif;
      margin: 0;
    }
  `,
  c.H1`Hello and welcome`,
  c.P`This is a frameorc tutorial`,
)
```

If the code above develops into something less pleasant to manage, I expect that
the user of this library knows how to factor it using variables, functions and
other mechanisms one learns while mastering the art of programming. The same
goes without saying about the knowledge of CSS, HTML, DOM APIs and related topics.


## Dynamic capabilities

The mechanisms discussed so far are mostly similar to static HTML, but with
the benefit of more flexible and powerful syntax. On top of that foundation,
the library provides building blocks that allow you to create DOM subtrees
with dynamic attributes and structure. These building blocks are simple and
composable, and they work well regardless of whether the programmer chooses
a declarative, functional, imperative, or combination approach.

### `on` combinator

30. The event handlers are assigned with `on` combinator. It behaves in the same
way as `attr` or `css`, except that its arguments are not converted to strings.
The accepted arguments of `on` combinator are functions. Strings and template 
strings are not supported, as they do not make much sense as event handlers[^3].

[^3]: Actually, strings work as event handlers in attributes (`attr` combinator),
where defined accordingly in HTML standard. But we are discussing `on` combinator
here, which is equivalent to addEventListener method, operating on functions.

```js
body(
  c.Button(
    on.click.tap((evt) => alert('Ouch')) // two events, same handler, and...
      .dblclick.dbltap( // for both events use these two handers:
        (evt) => alert('Yeow!'), // handler 1
        (evt) => alert('Ow!')),  // handler 2
    'Click me'))
```

31. Same as every combinator in this library, `on` can appear before, between 
or after any other children of its parent element, in any place, and can be 
repeated any number of times, with different or the same events. The following
code illustrates that, and has the same effect as the code in the previous example:

```js
let mkHandler = s => (evt) => alert(s),
    ouch = mkHandler('Ouch'),
    yeow = mkHandler('Yeow!'),
    ow = mkHandler('Ow!');
body(
  c.Button(
    on.click(ouch),
    on.tap(ouch),
    on.dblclick(yeow),
    on.dblclick(ow),
    on.dbltap(yeow),
    on.dbltap(ow),
    'Click me'));
```

### `body.refresh` method

32. `body.refresh` updates the contents of the body. If a function is used
(NB. *used*, not called) *anywhere* in the code that participated in the
formation of the contents, it will be called again.

```js
body(
  'Current time is: ',
  // This is a function, and the resulting text
  // will update on every refresh call:
  () => new Date().toISOString(),

  '. The time when we started was ',
  // The following is not a function, just the result of a call.
  // It will not be updating.
  new Date().toISOString(),
);

setInterval(body.refresh, 1000);
```

33. "*Anywhere*" includes nested element content, arrays and combinators such as
attributes, inline styles and so on.

```js
let currentTime = () => new Date().toISOString();
let clock = (caption) => c.clock(
  c.caption(caption),
  c.time(currentTime), // NB. a function, not a call
);

// A call, but it returns a structure in which, deeply embedded, there is a "live" function
body(clock('Current time'));
setInterval(body.refresh, 1000);
```

34. `body.refresh()` is ok to call frequently, as it only schedules an update
at the next cycle. It returns a promise. If you need to continue after the real
DOM content has been updated, `await` for the call, or use `.then()` to queue
a function which will be called after that single update operation really
completes.

When setting the content by calling `body(...)`, this call also returns the
promise, as `body.refresh()` does. All calls to those functions will return
the same `Promise` object until it gets resolved upon the application of changes
to the real DOM.


### `Val` accessor

35. There is a special value container, called `Val`. When a value is being put 
into it, it will queue a refresh. Don't be afraid to put values into `Val`s
or to call `body.refresh()` too often. It will not cause, but only schedule
an update, which will happen once, at the next cycle of the event loop.

```js
let name = Val('World'), count = Val(0);
body(
  c(
    c.Span`Hello, ${ name }!`,
    c.Label(
      'Name: ',
      c.Input(on.input((evt) => name(evt.target.value))))),
  c('Number of clicks: ', count),
  c(c.Button(on.click(() => count(count() + 1)), 'Click me')));
```

36. Vals can contain not only primitive values, but anything: arrays, functions,
and even combinators. That makes them a perfect building block for components.

For example, the code below defines and uses the component `Status`, which changes
its contents after being embedded into the DOM tree. Changes happen asynchronously,
as additional information is being loaded. Note also a technique allowing the component
to restart its process in case of an error.

```js
function Status() {
  let content = Val();
  async function worker() {
    content(c.status.loading('Loading...'));
    try {
      let res = await fetch('/status');
      let data = await res.json();
      content(c.status.ready('Status: ', data.status));
    } catch(e) {
      content(c.status.error(
        c.error(e),
        c.Button('Retry', on.click(worker));
      ));
    }
  }
  worker();
  return content;
}

body(Status());
```

37. You also can pass Vals around to create logical links between different
components operating on the same data.

```js
let count = Val(0), newCount = Val('0');

body(
  c.Button('-', on.click(() => count(count()-1))),
  c.Span('Count: ', count),
  c.Button('+', on.click(() => count(count()+1))),
  c.Input(attr.value(count), on.input(e => newCount(e.target.value))),
  c.Button('set', on.click(() => count(Number(newCount())))));
```

The attentive reader will notice that the value in the input field is initially
updating as `+` and `-` buttons are being clicked. However, after something
is entered into that input field, the updates will stop, even after the entered
value coincides again with the counter value. This may be a feature or a bug,
depending on the effect you desire to obtain. If you need a dynamic behaviour,
replace `attr` with `prop` in the example above and see the next section for a
more detailed description of `prop` combinator.


### `prop` combinator

38. Sometimes, `attr` is not dynamic enough. The most common case is for input 
elements, where `value` *attribute* only specifies the initial value of the field,
and the current value is set and retrieved from the `value` *property*.

To manage properties, frameorc has `prop` combinator. It behaves like `attr`
combinator, but it does not automatically convert its arguments to strings. 
The arguments are assigned to element properties as they were passed to this
combinator. If several arguments are supplied, they are assigned as an array.

`prop` also supports template strings.

If the same property is being set several times, the last assignment overrides
all previous.

```js
let v = Val('');
body(
  c.H1`These inputs are synchronised`,
  c.Label(
    'Input 1',
    c.Input(prop.value(v), on.input(e => v(e.target.value)))),
  c.Label(
    'Input 2',
    c.Input(prop.value(v), on.input(e => v(e.target.value)))),
  c.Button(on.click(() => v('')), 'Clear'));
```

### `cls` combinator

39. Classes can be set by `cls` combinator:

```js
body(c(cls.important, 'Important!'))
```

```js
body(c(cls.important.red.bold, 'Important!'))
```

```js
body(c(cls.important().red.bold(), 'Important!'))
```

40. `cls` combinator can take arguments. If any of the arguments is `true`, or 
is the value that JavaScript coerces to boolean `true`, the preceding *chain of
classes* will be added to the DOM element.

```js
body(
  c(
    // <div class="important red bold">
    // NB. both .green and .old are rejected due to condition following them
    cls.important(2 === 2).green.old(3 < 2).red.bold(2 !== 3),
    'Important!'))
```

41. If there are any functions in arguments, they will be called to obtain the
value of their result. If a function call returns a function, that function
will be called, and so forth, until a non-callable value is obtained. Then
such value is coerced to boolean for the decision to be made.

If another "truthy" value precedes a function, that function will not be called,
as one is enough for the classes to be set.

```js
let tf = () => 2 < 3;
body(
  c(
    cls.important(2 === 2).red.bold(2 === 3, tf),
    'Important!'));
```

42. If all of the arguments are false, the class will be unset,
if set previously.

```js
body(c(
  cls.important(2 === 2).red.bold(2 === 3, 2 < 3),
  cls.red(2 > 3), // .red will be unset
  'Important!'))
```

43. Classes can be set and unset multiple times. The last (un)setting wins.

```js
body(c(
  cls.important(2 === 2).red.bold(2 === 3, 2 < 3),
  cls.red(2 > 3),   // .red will be unset
  cls.red(2 !== 3), // .red is set again, last setting, affects the result
  'Important!'))
```

### `key` combinator

44. The DOM content is being updated by this library via the virtual DOM diffing
algorithm. It means that the most similar elements existing in the DOM tree will
be found for the new state you are intending to apply, and these elements will be
patched to get them into the required state.

Sometimes, it is necessary to mark elements in a certain way, so that neighbouring
elements are not mistaken for each other. In various virtual DOM libraries that
is achieved through assigning those virtual DOM elements a special property `key`.
That technique is most frequently applied in lists and tables.

Frameorc provides a `key` combinator for that purpose. It is the programmer's task to
make sure all sibling elements have different keys.

```js
body(c.Ul(
  elements.map(data =>
    c.Li(key(data.id), data.text))))
```

`key` can receive multiple arguments, including functions and arrays. It
flattens and evaluates them in order to build the resulting string in the same
way as `attr` combinator does with its arguments.


### `hook` combinator

45. The lifecycle of the virtual DOM elements can be handled with hooks.
Frameorc provides a `hook` combinator, so that your handler functions can be
called upon the events of element creation and destruction, for example.

```js
function ScrollHandler({ update }) {
  let self;
  return c(
    css.display`none`,
    hook
      .create(
        (_, e) => self = e.elm,
        () => window.addEventListener('scroll', update))
      .destroy(
        () => window.removeEventListener('scroll', update)));
}
```

Hooks are based on the underlying [Snabbdom](https://github.com/snabbdom/snabbdom) library,
and the following are supported:

| Name        | Triggered when                                     | Arguments to callback   |
| ----------- | -------------------------------------------------- | ----------------------- |
| `init`      | a vnode has been added                             | `vnode`                 |
| `create`    | a DOM element has been created based on a vnode    | `emptyVnode, vnode`     |
| `insert`    | an element has been inserted into the DOM          | `vnode`                 |
| `prepatch`  | an element is about to be patched                  | `oldVnode, vnode`       |
| `update`    | an element is being updated                        | `oldVnode, vnode`       |
| `postpatch` | an element has been patched                        | `oldVnode, vnode`       |
| `destroy`   | an element is directly or indirectly being removed | `vnode`                 |
| `remove`    | an element is directly being removed from the DOM  | `vnode, removeCallback` |

To receive a real DOM element, use the `.elm` property of a `vnode` passed into your function,
as in the above example.


## Advanced topics

### `Ref` accessor

In addition to `Val`, there is also `Ref`, which acts as an accessor
to a certain property of an object. The value of such property can be
obtained and assigned via the `Ref`, and an assignment will trigger
the interface `refresh`. In the special cases described in `attach`
section below, it will be a `refresh` function from the corresponding,
independent VDOM tree setter.

```js
let obj = { a: 1, b: 2 };
let r = Ref(obj, 'a');
console.log(r()); // 1
r(3);
console.log(obj); // { a: 3, b: 2 }
```

### Reactive handlers

There are two special methods of `Val` and `Ref` accessors: `.on(f)` and
`.off(f)`, allowing you to add and remove the functions which will be called
after an assignment has been made via an accessor. Multiple functions can be
added as such handlers by sequentially calling the `.on` method:
`v.on(f1).on(f2).on(f3)`.

In fact, every `Val` and `Ref` since their creation already have one assignment
event handler installed: `refresh`. In default `Val` and `Ref` it is
`body.refresh`, but in special cases described in the following section covering
the `attach` function, it will be a `refresh` method from the corresponding
VDOM tree setter.


### `attach` function

`attach(el)` takes a DOM element and returns a setter. For example,

```js
let status = attach(document.getElementById('status'));
status(c.Span.red('Alert'));
```

The setter function can be used to set the contents of the element.
`body` setter, from the description of which this document starts, 
is nothing else than a result of `attach(document.body)`.

This function is useful in case one decides to have multiple VDOM roots.
Updates of such roots, triggered by calling the setter function, or by calling
its `refresh` property, will be happening independently of each other.

```js
status('Current time is: ', () => Date.toISOString()); 
setInterval(status.refresh, 1000);
```

Also, every setter obtained in this manner has its own `Val`s and `Ref`s:

```js
let n = status.Val(0);
status(
  c.Span('Number of clicks: ', n),
  c.Button(
    on.click(e => n(n()+1)),
    'Click me'));
```

Setting values to such `Val`s and `Ref`s will trigger an update of their
corresponding VDOM tree, and will leave other VDOM trees intact.

Only the children will be added to the parent DOM element. The element itself
and its previous content are left intact. Any number of arguments can be passed
to the setter function, as you would normally do with `body` function, seen in
the opening examples of this document.

In the examples above, calls to `status.refresh()` and `status(...)` return
promises that are resolved after the changes are applied to the real DOM.


### Attaching to shadow roots

Frameorc works well with custom elements. You can use this library to create
your own elements of that type. No special considerations are required to
operate on shadow DOM, other than understanding its API, of course.

```js
let demoElement = document.getElementById('demo');
let demoRoot = demoElement.attachShadow({ mode: 'open' });
let demo = attach(demoRoot); // just like any other element
demo(
  c.H1('Hello'),
  c.a.b(1).c(2).d.e(3)`4`(5).f(6, attr.x(100))(attr.y`200`),
  // cls.red -- will not work on top-level, shadow root does not have classes
  c.Slot(), // works as expected, if you have content in the element
  c(cls.red), // works too, as we apply the class here to a normal element
);
```

The only reminder is that shadow roots are not usual DOM elements. They do not
have classes and attributes, so it is not surprising that if you use *top level*
`cls` or `attr`, you will get an exception. You must also be careful with
`prop`, `on` and other operators used on *top level*, i.e. shadow root itself.


### Operators

Operators, in terms of this document, are functions that change elements'
properties and contents. In frameorc, `c`, `attr`, `css`, `on`, `prop`, `cls`,
`key` and `hook` are predefined operators. Yes, `c` is an operator in the sense
that it changes the parent element by appending the content generated by itself
to the parent's children. It is implemented exactly that way, no exceptions made.

Syntactically, operators can be among other children of the element, and appear
there in any order, freely intermixing with any other arguments.

Operators can be passed to the setter function, such as `body`, or the one
obtained by calling `attach`. In that case, they will affect the container
element in the same way as they normally do with elements formed by `c`
combinator.

Apart from using `attr`, `prop`, `on` and so on, you can define your own
operator function by wrapping it in `operator` call. When VDOM element is
being formed, it will be called with `(parent, ctx)` arguments, where
`parent` is VDOM element and `ctx` is the processing context, allowing
to pass information from elements to their children and back, and from the
function forming the preceding child to the next one.

One example of how to use `opeartor` to control the focus of an element
is given in the TodoMVC example.


# Acknowledgements

Thanks to Anton Baranov, Daniil Terlyakhin, Fedot Kryutchenko and
Elnur Yusifov for reading drafts, testing multiple versions of this
library over the past eight years, collaborating and exchanging ideas
together. I could have never gone as far as making a public release of
frameorc without your support and encouragement.

For its virtual DOM layer, frameorc is using [Snabbdom](https://github.com/snabbdom/snabbdom),
an excellent MIT-licensed library by Simon Friis Vindum et al.

[TodoMVC example](examples/todomvc/index.js) is based on the work of Addy Osmani,
Sindre Sorhus, Pascal Hartig, Stephen Sawchuk and others.



