
# Zekai

Zekai is a toy programming language for hobbyists and enthusiasts.

Its simple [source code (150 lines)](https://github.com/lucazd/zekai/blob/master/zekai) and modern syntax can serve as a basis for building new programming languages.

## Quickstart

* [Download zekai.zip](https://github.com/lucazd/zekai/archive/master.zip), extract it and open the file _zekai-dev.html_.

	Or [**Try it online**](http://htmlpreview.github.io/?https://github.com/lucazd/zekai/master/zekai-dev.html).

* Type in the following zekai code:
```javascript
fib(n):
    n == 0? 0
    n == 1? 1
    fib(n-1) + fib(n-2)

main() {
    i = 2 + 2
    alert('your lucky number is:' + fib(i))
}
```
* Press F8 or Shift+Enter to run.


## Example: Conway's game of life

Next code is more complex:
```javascript
main():
	has(g i j): i >= 0 & i < size(g) & j >= 0 & j < size(g[i])? g[i][j] 0
	add(a b): a + b
	grid = to(40).map(\:to(40).map(\:rand(2)))
	io(canvas(200 200) \in out:
		grid <- grid.map(\row i: row.map(\c j {
			n = range(~1 2).map(\a: range(~1 2).map(\b:
				!a & !b? 0 has(grid i+a j+b)
			).reduce(add)).reduce(add)
			v = n == 2? c n == 3? 1 0
			fill(out v? '#fd0' 'black' j*5 i*5 5 5)
			v
        }))
    )

#
# Utility functions
#

size  (v): v.length
rand  (n): Math.floor(Math.random() * n)
range (i n): n >= 0? Array(n-i).join(' ').split(' ').map(\e v: v+i) []
to    (n): range(0 n)

#
# Output functions
#

canvas(w h) {
	c = document.createElement('canvas')
	c.width <- w
	c.height <- h
	c
}

io(out main) {
	animate() {
		main({} out)
		window.requestAnimationFrame(animate)
	}
	animate()
    document.body.appendChild(out)
}

fill(out c x=0 y=0 w=out.width h=out.height a=1 blend='') {
	q = out.getContext('2d')
	q.save()
	q.globalAlpha <- a
	q.globalCompositeOperation <- blend
	q.fillStyle <- c
	q.fillRect(x y w h)
	q.restore()
}
```

## Syntax Overview

Basics:

* Whitespace is not significative

* Identifiers matches the regex `[\\$A-Za-z_][\\$A-Za-z_\\d]*`

* Line comments starts with `#`

	|Literal |Example          |ECMAScript|
	|--------|-----------------|----------|
	|Number  |`7` `7.77` `0xAF`|_(same)_|
	|String  |`'awesome'`      |_(same)_|
	|Array   |`[1 2 3]`        |`[1,2,3]`|
	|Object  |`{ x=1 y=2 }`    |`{ x:1, y:2 }`|
	|Function|`\a b c: 0`      |`function(a,b,c) { return 0; }`|


* A _program_ is a set of definitions

* A _definition_ is an identifier and an expression, written:
```javascript
id = expr
```

Optionally, the definition of a function expression can be written as:
```javascript
id = \a b c: expr
id(a b c): expr   # better
```

* The _body_ of a function can be either a single expression, or a block of expressions.
Using `:` and `{}` respectively:
```javascript
foo(a b): a + b
bar(a b) {
	a()
	b()
}
```

* Commas and semi-colons are not necessary, but can be used to improve readability

* **Let** expressions are a definition and an expression (`id = expr expr`). Example:
```javascript
main():
	a = 2 + 2
	print(a)
```
Let expressions in blocks are also valid:
```javascript
main() {
	a = 2 + 2
	print(a)
}
```

*  A **struct** is syntactic sugar for a _function that returns an object with the arguments as fields_
```javascript
foo(a b): { a=a b=b }

bar(a b)@

foo = \i=0 j=0 @
```

* **Conditional** expressions `cond? true false`. Example:
```javascript
main() {
	print(1? 2 3) # 2
	print(0? 2 3) # 3
}
```

* Negative numbers are written with _operator `~`_, example:
```javascript
abs(a): a < 0? ~a a
```

*  _Updates_ uses the symbol `<-`. (see _Notes_)
```javascript
main() {
	s = ''
	s <- prompt()
}

# ECMAScript
function main() {
	var s = '';
	s = prompt();
};
```

* Finally, operators that behave like ECMAScript:

	|Operator|Symbol|
	|--------|------|
	|Arithmetic|```+ - * / %```|
	|Comparison|```== != >= <= > <```|
	|Not|```!```|
	|Logic|```|| &&```|
	|Call|```()```|
	|Index|```[]```|
	|Field|```.```|

Refer to the source code grammar for details.


## Project files

Zekai project is composed of the following files:

|File name     |Description|
|--------------|-----------|
|zekai         |source code|
|zekai.js      |source code compiled to ECMAScript|
|zekai-dev.html|editor|
|LICENSE       |MIT|
|README.md     |this file|


## Notes

* Zekai was created as a project for personal study. Becoming mature,
it may serve as a basis for creating new programming languages.

* Zekai should compile to WebAssembly, since it is not here yet,
the best option is ECMAScript.

* There is no need to introduce a new -standard- library. Better choose an existing library which fits the needed functionality.

* Structs instead of classes emphasize there is no inheritance.

* There are not keywords.

* Both `null` and `undefined` should never be referred, unless working with an ECMAScript library.

* There is no `new` idiom. To call ECMAScript library functions use:
```javascript
new(class args=[]) {
	obj = { __proto__ = class.prototype }
	class.apply(obj args)
	obj
}
```
* Use a strategy to encapsulate side-effects (for example; forbid them outside of _main_).

* Code should be written to be statically type inferred.

* Static check analysis can be done by formatting the compiled ECMAScript and then using [Flow](http://flowtype.org/).

* The implementation of the compiler translates Zekai code to ECMAScript.

    (Zekai works on any ECMAScript environment)

* A raw way to run Zekai code on a browser environment without using the editor:
```html
<html>
<head>
<script src="zekai.js"></script>
<script>
eval(zekai.program('main(): alert(0)')).main()
</script>
</head>
<body>
</body>
</html>
```

* To run Zekai code in a Node environment:
```javascript
// ECMAScript
var read = function(path, onload) { return require('fs').readFile(path, onload); };
read('zekai.js', function(err, data) {
    eval(data);
    read(arguments[2], function(err, data) {
        eval(zekai.program(data));
    });
});
```

* Another option is to save the compiled output, and load it as regular ECMAScript.

* The output of compiling the file _zekai_ is the content of _zekai.js_.

* To get the compiled ECMAScript run:
```javascript
main():
    console.log(zekai.program(code))
```

* Pure code is portable on virtual machines.

* To write portable impure code, a layer/library is needed.

* It is not possible to know beforehand what kind of IO actions the software needs.

* Source files should start with ```zekai` ``` and end with ``` ` ``` to bypass the browser's
local file access.


## License

Zekai is MIT-licensed.