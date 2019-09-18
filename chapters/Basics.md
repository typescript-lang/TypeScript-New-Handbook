# TypeScript Basics

__toc__

JavaScript 中的每个值都有一些行为，你可以通过运行不同的操作来观察这些行为。
这听起来很抽象，这里作为一个简单的例子，我们可以在名为 `foo` 的变量上做一些不同操作来尝试下。

```js
// 访问 'foo' 上的 'toLowerCase' 属性，然后调用它
foo.toLowerCase();

// 调用 'foo'
foo();
```

如果我们将上面的代码分解，第一行代码访问了一个名为 `toLowerCase` 的属性，然后调用它，第二行代码尝试直接调用 `foo`。

但假设我们不知道 `foo` 的值——这很普遍——我们无法准确的说出我们试图运行的任何代码的结果，每项操作的结果完全取决于我们最开始获取的是什么值，是 `foo` 是否可调用？或 `foo` 是否有 `toLowerCase` ？如果有，那 `toLowerCase` 是否可调用？如果所有这些值都可以调用，它们会返回什么？在我们写 JavaScript 时我们的头脑中总是在考虑这些问题，我们希望能掌握跟多的细节。

假设`foo`是按以下方式定义的。

```js
let foo = "Hello World!";
```

你可能猜到，如果我们尝试运行 `foo.toLowerCase()` ，我们将得到相同的但都是小写字母的字符串。

那第二行代码呢？如果你熟悉 JavaScript ，你会知道这会失败并返回下面的异常：

```txt
TypeError: foo is not a function
```

如果我们能避免这样的错误，那就太棒了。当我们运行代码时，JavaScript 运行时是通过计算值的 *类型* 选择执行结果 ——它具有哪些行为和功能。上面的 `TypeError` 包含了——没有什么可以调用字符串 `"Hello World"`。

对于某些值，例如基本类型 `string` 和 `number` ，我们能可以使用 `typeof` 运算符在运行时识别它们的类型。但是对于像函数等这些类型，我们没有相应的运行时机制识别它们的类型。比如，我们看下面这个函数：


```js
function fn(x) {
    return x.flip();
}
```

我们可以通过阅读代码知道—— *observe* ，只有给函数传入一个可以调用的 `flip` 属性的对象时上面代码才能执行，但是 JavaScript 并没有以我们可以在代码运行时检查的方式显示此信息。 在原生 JavaScript 中，只能通过输入不同的值然后调用 `fn` 来确定会发生什么。这种行为让我们很难在代码运行前预测代码的运行结果，这意味着我们在编写代码时很难知道代码在运行时的行为。

从这个角度看， *类型* 是描述哪些值传递给 `fn` 是合法的而哪些是不合法的概念。JavaScript 的确提供了 *动态* 类型——运行代码才能确定会发生什么。

另外一种方法是使用 *静态* 类型系统在代码运行 *之前* 预测哪些代码是合法的。

## 静态类型检查

回想下我们前面调用 `string` 得到的 `TypeError` 。
*大多数人* 不喜欢代码运行时出现任何错误——这些都被当做 bug！当我们编写新代码时，我们也会尽可能避免引入新的 bug。

如果我们只是新增了一点代码，保存文件，刷新应用，并且立刻看到错误，我们可能快速处理掉问题；但事实不总是如此。我们对该功能可能没有足够的测试，所以我们可能永远也不会触发隐藏的问题！或者我们很幸运发现了这些问题，最后我们可能做了大量重构并添加了很多不得不深入研究的代码。

理想情况下，我们可以使用工具在代码运行 *之前* 帮我们发现这些 bug。这正是像 TypeScript 这类静态类型检查工具所做的事。*静态类型系统* 描述了我们的程序运行时值的形状和行为。像 TypeScript 这类类型检查器会告诉我们什么时候可能会出现问题。

```ts
let foo = "hello!";

foo();
```

把上面的例子运行在 TypeScript 中，我们会在代码执行之前得到错误提示。

## 非异常（Non-exception）错误

目前为止我们一直在讨论运行时发生的比较明确的错误，JavaScript 在运行时抛出这些错误并且给出错误信息，这是因为 [ECMAScript 规范](https://tc39.github.io/ecma262/) 明确说明了语言在运行到意外情况时应该如果表现。

例如，规范明确指出如果试图调用不可调用的东西时应该抛出错误。这听起来像是一个“明显的行为”，你可能会想象访问一个对象上不存在的属性时也会抛出一个错误，但是，JavaScript 给了我们不一样的行为，它返回了值 `undefined`：

```js
let foo = {
    name: "Daniel",
    age: 26,
};

foo.location; // 返回 undefined
```

根本上，静态类型系统一定会反馈出在其系统中标记为错误的代码，即使是不会立即报出错误的“有效”的 JavaScript 代码。在 TypeScript 中，以下代码会产生一个 `location` 未定义的错误：

```ts
let foo = {
    name: "Daniel",
    age: 26,
};

foo.location; // 返回 undefined
```

虽然有时这意味着要在表达内容上进行权衡，但目的是捕捉我们程序中的合法 bug。并且 TypeScript 会捕获*很多*合法的 bug，例如：拼写错误，

```ts
let someString = "Hello World!";

// 你能很快的发现这些拼写错误吗？
someString.toLocaleLowercase();
someString.toLocalLowerCase();

// 我们的本意可能是这样的……
someString.toLocaleLowerCase();
```

函数未调用，

```ts
function flipCoin() {
    return Math.random < 0.5;
}
```

或者基本逻辑错误。

```ts
const value = Math.random() < 0.5 ? "a" : "b";
if (value !== "a") {
    // ...
}
else if (value === "b") {
  // 不会执行到这里
}
```

## Types for Tooling

<!-- TODO: this section's title sucks -->

在我们的代码出错时 TypeScript 可以捕获这些错误，这非常好，但是 TypeScript *还能* 提前阻止我们犯这些错误。

The type-checker has information to check things like whether we're accessing the right properties on variables and other properties.
Once it has that information, it can also start *suggesting* which properties you might want to use.

That means TypeScript can be leveraged for editing code too, and the core type-checker can provide error messages and code completion as you type in the editor.
That's part of what people often refer to when they talk about tooling in TypeScript.

<!-- TODO: insert GIF of completions here -->

TypeScript takes tooling seriously, and that goes beyond completions and errors as you type.
An editor that supports TypeScript can deliver "quick fixes" to automatically fix errors, refactorings to easily re-organize code, and useful navigation features for jumping to definitions of a variable, or finding all references to a given variable.
All of this is built on top of the type-checker and fully cross-platform, so it's likely that [your favorite editor has TypeScript support available](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support).

<!-- TODO: validate that link -->

## `tsc`, the TypeScript compiler

We've been talking about type-checking, but we haven't yet used our type-*checker*.
Let's get acquainted with our new friend `tsc`, the TypeScript compiler.
First we'll need to grab it via npm.

```sh
npm install -g typescript
```

> This installs the TypeScript Compiler `tsc` globally.
> You can use `npx` or similar tools if you'd prefer to run `tsc` from a local `node_modules` package instead.

Now let's move to an empty folder and try writing our first TypeScript program: `hello.ts`:

```ts
// Greets the world.
console.log("Hello world!");
```

Notice there are no frills here; this "hello world" program looks identical to what you'd write for a "hello world" program in JavaScript.
And now let's type-check it by running the command `tsc` which was installed for us by the `typescript` package.

```sh
tsc hello.ts
```

Tada!

Wait, "tada" *what* exactly?
We ran `tsc` and nothing happened!
Well, there were no type errors, so we didn't get any output in our console since there was nothing to report.

But check again - we got some *file* output instead.
If we look in our current directory, we'll see a `hello.js` file next to `hello.ts`.
That's the output from our `hello.ts` file after `tsc` *compiles* or *transforms* it into a JavaScript file.
And if we check the contents, we'll see what TypeScript spits out after it processes a `.ts` file:

```js
// Greets the world.
console.log("Hello world!");
```

In this case, there was very little for TypeScript to transform, so it looks identical to what we wrote.
The compiler tries to emit clean readable code that looks like something a person would write.
While that's not always so easy, TypeScript indents consistently, is mindful of when our code spans across different lines of code, and tries to keep comments around.

What about if we *did* introduce a type-checking error?
Let's rewrite `hello.ts`:

```ts
// @noImplicitAny: false
// This is an industrial-grade general-purpose greeter function:
function greet(person, date) {
    console.log(`Hello ${person}, today is ${date}!`);
}

greet("Brendan");
```

If we run `tsc hello.ts` again, notice that we get an error on the command line!

```txt
Expected 2 arguments, but got 1.
```

TypeScript is telling us we forgot to pass an argument to the `greet` function, and rightfully so.
So far we've only written standard JavaScript, and yet type-checking was still able to find problems with our code.
Thanks TypeScript!

### Emitting with Errors

One thing you might not have noticed from the last example was that our `hello.js` file changed again.
If we open that file up then we'll see that the contents still basically look the same as our input file.
That might be a bit surprising given the fact that `tsc` reported an error about our code, but this based on one of TypeScript's core values: much of the time, *you* will know better than TypeScript.

To reiterate from earlier, type-checking code limits the sorts of programs you can run, and so there's a tradeoff on what sorts of things a type-checker finds acceptable.
Most of the time that's okay, but there are scenarios where those checks get in the way.
For example, imagine yourself migrating JavaScript code over to TypeScript and introducing type-checking errors.
Eventually you'll get around to cleaning things up for the type-checker, but that original JavaScript code was already working!
Why should converting it over to TypeScript stop you from running it?

So TypeScript doesn't get in your way.
Of course, over time, you may want to be a bit more defensive against mistakes, and make TypeScript act a bit more strictly.
In that case, you can use the `--noEmitOnError` compiler option.
Try changing your `hello.ts` file and running `tsc` with that flag:

```sh
tsc --noEmitOnError hello.ts
```

You'll notice that `hello.js` never gets updated.

## Explicit Types

Up until now, we haven't told TypeScript what `person` or `date` are.
Let's change up our code a little bit so that we tell TypeScript that `person` is a `string`, and that `date` should be a `Date` object.
We'll also use the `toDateString()` method on `date`.

```ts
function greet(person: string, date: Date) {
    console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
```

What we did was add *type annotations* on `person` and `date` to describe what types of values `greet` can be called with.
You can read that signature as "`greet` takes a `person` of type `string`, and a `date` of type `Date`".

With this, TypeScript can tell us about other cases where we might have been called incorrectly.
For example...

```ts
function greet(person: string, date: Date) {
    console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

greet("Maddison", Date());
```

Huh?
TypeScript reported an error on our second argument, but why?

Perhaps surprisingly, calling `Date()` in JavaScript returns a `string`.
On the other hand, constructing a `Date` with `new Date()` actually gives us what we were expecting.

Anyway, we can quickly fix up the error:

```ts
function greet(person: string, date: Date) {
    console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

greet("Maddison", new Date());
                  ^^^^^^^^^^
```

Keep in mind, we don't always have to write explicit type annotations.
In many cases, TypeScript can even just *infer* (or "figure out") the types for us even if we omit them.

```ts
let foo = "hello there!";
    ^?
```

Even though we didn't tell TypeScript that `foo` had the type `string` it was able to figure that out.
That's a feature, and it's best not to add annotations when the type system would end up inferring the same type anyway.

## Erased Types

Let's take a look at what happens when we compile with `tsc`:

```js
function greet(person, date) {
    console.log("Hello " + person + ", today is " + date.toDateString() + "!");
}
greet("Maddison", new Date());
```

Notice two things here:

1. Our `person` and `date` parameters no longer have type annotations.
2. Our "template string" - that string that used backticks (the `` ` `` character) - was converted to plain strings with concatenations (`+`).

More on that second point later, but let's now focus on that first point.
Type annotations aren't part of JavaScript (or ECMAScript to be pedantic), so there really aren't any browsers or other runtimes that can just run TypeScript unmodified.
That's why TypeScript needs a compiler in the first place - it needs some way to strip out or transform any TypeScript-specific code so that you can run it.
Most TypeScript-specific code gets erased away, and likewise, here our type annotations were completely erased.

> **Remember**: Type annotations never change the runtime behavior of your program.

## Downleveling

One other difference from the above was that our template string was rewritten from

```js
`Hello ${person}, today is ${date.toDateString()}!`
```

to

```js
"Hello " + person + ", today is " + date.toDateString() + "!"
```

Why did this happen?

Template strings are a feature from a version of ECMAScript called ECMAScript 2015 (a.k.a. ECMAScript 6, ES2015, ES6, etc. - don't ask).
TypeScript has the ability to rewrite code from newer versions of ECMAScript to older ones such as ECMAScript 3 or ECMAScript 5 (a.k.a. ES3 and ES5).
This process of moving from a newer or "higher" version of ECMAScript to an older or "lower" one is sometimes called *downleveling*.

By default TypeScript targets ES3, an extremely old version of ECMAScript.
We could have chosen something a little bit more recent by using the `--target` flag.
Running with `--target es2015` changes TypeScript to target ECMAScript 2015, meaning code should be able to run wherever ECMAScript 2015 is supported.
So running `tsc --target es2015 input.ts` gives us the following output:

```js
function greet(person, date) {
    console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
greet("Maddison", new Date());
```

> While the default target is ES3, the great majority of current browsers support ES5.
> Today, most developers can safely specify ES5 or even ES2016 as a target unless compatibility with certain ancient browers is important.

## Strictness

Users come to TypeScript looking for different things in a type-checker.
Some people are looking for a more loose opt-in experience which can help validate only some parts of our program and give us decent tooling.
This is the default experience with TypeScript, where types are optional, inference takes the most lenient types, and there's no checking for potentially `null`/`undefined` values.
Much like how `tsc` emits in the face of errors, these defaults are put in place to stay out of your way.
If you're migrating existing JavaScript, that might be desirable.

In contrast, a lot of users prefer to have TypeScript validate as much as it can off the bat, and that's why the language provides strictness settings as well.
These strictness settings turn static type-checking from a switch (either your code is checked or not) into something closer to a dial.
The farther you turn this dial up, the more TypeScript will check for you.
This can require a little extra work, but generally speaking it pays for itself in the long run, and enables more thorough checks and more accurate tooling.
If possible, a new codebase should always turn these strictness checks on.

TypeScript has several type-checking strictness flags that can be turned on or off, and all of our examples will be written with all of them enabled unless otherwise stated.
The `--strict` flag toggles them all on simultaneously, but we can opt out of them individually.
The two biggest ones you should know about are `noImplicitAny` and `strictNullChecks`.

### `noImplicitAny`

Recall that in some places, TypeScript doesn't try to infer any types for us and instead falls back to the most lenient type: `any`.
This isn't the worst thing that can happen - after all, falling back to `any` is just the JavaScript experience anyway.

However, using `any` often defeats the purpose of using TypeScript in the first place.
The more typed your program is, the more validation and tooling you'll get, meaning you'll run into fewer bugs as you code.
Turning on the `noImplicitAny` flag will issue an error on any variables whose type is implicitly inferred as `any`.

### `strictNullChecks`

By default, values like `null` and `undefined` are assignable to any other type.
This can make writing some code easier, but forgetting to handle `null` and `undefined` is the cause of countless bugs in the world - not even just JavaScript!

The `strictNullChecks` flag makes handling `null` and `undefined` more explicit, and *spares* us from worrying about whether we *forgot* to handle `null` and `undefined`.
