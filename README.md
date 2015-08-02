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
    out
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
	|Struct  |`{ x=1 y=2 }`    |`{ x:1, y:2 }` _("objects" in ECMAScript)_|
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

*  An _Initializer_ (**init**) is syntactic sugar for a _function that returns a structure with the arguments as fields_
```javascript
foo(a b): { a=a b=b }

bar(a b)@

foo = \a b@ # no need to write code like this
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

*  _Updates_ uses the symbol `<-`. Note it is best to avoid side-effects, or at least limit them in the main function.
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

* There are no classes; no inheritance.

* There are not keywords.

* Both null and undefined should never be referred, unless working with an ECMAScript library.

* There is no new. To call ECMAScript library functions use:
```javascript
new(class args=[]) {
	obj = { __proto__ = class.prototype }
	class.apply(obj args)
	obj
}
```

* The output of compiling the file _zekai_ is the same as zekai.js.

* Code should be written to be statically type inferred.

* The implementation of the compiler translates Zekai code to ECMAScript.

    (Zekai works on any ECMAScript environment)

* A raw way to run Zekai code on a browser environment without using the editor:
```html
<html>
<head>
<script src="zekai.js"></script>
<script>
eval(zekai.compile('main(): alert(0)')).main()
</script>
</head>
<body>
</body>
</html>
```

* To run Zekai code in a Node environment:
```javascript
// ECMAScript
var read = function(path, onload) { return require('file').readFile(path, onload); };
read('zekai.js', function(err, data) {
    eval(data);
    read(arguments[2], function(err, data) {
        eval(zekai.compile(data));
    });
});
```

* Another option is to save the compiled output, and load it as regular ECMAScript.

* Pure code is always portable.

* To write portable impure code, a layer is needed. Use a library.

* It is not possible to know beforehand what IO API the software needs.

* The IO system will always be present, preferably encapsulated in main.

* To get the compiled ECMAScript run:
```javascript
main():
    console.log(zekai.compile(code))
```

* To access files use zekai\` code \`


## License

Zekai is MIT-licensed.