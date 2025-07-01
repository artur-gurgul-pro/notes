---
layout: default
title: Javascript - notes
categories: programming
path: js/notes
---

Useful tool:  Why did you render
https://react.dev/learn/react-developer-tools

### Currying - JavaScript


```js
function currying(fn, ...args) {
    return (..._arg) => {
        return fn(...args, ..._arg);
    }
}
```

```js
function sum(a,b,c) {
    return a + b + c
}
let add = currying(sum,10);
```

#### Advanced implementation

```js
function curry(func) {
    return function curried(...args) {
        if (args.length >= func.length) {
            return func.apply(this, args)
        } else {
            return function(...args2) { 
                return curried.apply(this, args.concat(args2))
            }
        }
    }
}
```

Usage

```js
function sum(a, b, c) {
  return a + b + c;
}

let curriedSum = curry(sum);

// Callable normally
curriedSum(1, 2, 3) 

// Currying of 1st argument
curriedSum(1)(2,3) 

// Currying all arguments
curriedSum(1)(2)(3)
```


### Closures

```javascript
var NextClosure = function() {
    var a = 0;
    return function next() {
        a++
        return a
    }
}
```

```javascript
var next = NextClosure()

console.log(next())
console.log(next())
console.log(next())
```


### Manipulating SVG

```html
<svg id="display" width="50" height="50" style="background-color: aqua;"></svg>
```


```javascript
let element = document.createElementNS("http://www.w3.org/2000/svg", 'rect')
element.setAttribute("x", 25)
element.setAttribute("y", 25)
element.setAttribute("width", 25)
element.setAttribute("height",25)
element.setAttribute("fill", "black")
let display = document.getElementById("display")
display.appendChild(element)
```

### Operations on arrays

```js
let test = [1,2,3,4,5,6,7]
console.log(test.splice(-5, 5)) // [ 3, 4, 5, 6, 7 ]
console.log(test) // [ 1, 2 ]
```

- `test.splice(-5, 5)` counting from the end
-  `test.splice(5)` from 5th to the end of the array

```js
// Take just first element
console.log(process.argv.shift()
```


## Call function in place


```js
(function waitForAsync() {
	if (end) {
		console.log("Synchronous operation can proceed now")
	} else {
		setImmediate(waitForAsync)
	}
})()
```

## Spawn process

```js
const { spawn } = require('child_process')
const child = spawn(command, params, {cwd: process.cwd()});

//for text chunks
child.stdout.setEncoding('utf8')

child.stdout.on('data', (chunk) => {
	//process.stdout.write(chunk)
})

// since these are streams, you can pipe them elsewhere
// child.stderr.pipe(dest);

child.on('close', (code) => {
	console.log(`ENDED: ${command} ${params.join(" ")} $ => ${code}`);
});
```


## Handling working directory

```js
function dir(wd, commands) {
	let saveWD = process.cwd()
	process.chdir(wd)
	commands()
	process.chdir(saveWD)
}
```

## Execute script loaded from string


```js
const util = require('util')
const vm = require('vm')

const sandbox = {
	animal: 'cat',
	count: 2
}

const script = new vm.Script('count += 1; name = "kitty"')
const context = new vm.createContext(sandbox)

for (let i = 0; i < 10; ++i) {
	script.runInContext(context)
}

console.log(util.inspect(sandbox))
```

or 

```js
primes = [ 2, 3, 5, 7, 11 ]
subset = eval('primes.filter(each => each > 5)')
console.log(subset)
```

```js
(async () => {
	let file = await readSource()
})()
```


mjs => node module, allows to use await/async at the top level


```json
{
  "name": "om",
  "version": "1.0.0",
  "main": "dist/index.js",
  "type": "module",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "ts-node src/index.ts",
    "build": "tsc && cp src/om dist/om",
    "run": "node dist/index.js",
    "watch": "nodemon --watch 'src/**' --ext 'ts,json' --ignore 'src/**/*.spec.ts' --exec 'tsc || true'"
  },
  "author": "",
  "license": "ISC",
  "description": "",
  "dependencies": {
  "commander": "^12.0.0",
  "pg": "^8.11.5"
  },
  "devDependencies": {
    "nodemon": "^3.1.0",
    "ts-node": "^10.9.2",
    "typescript": "^5.3.3"
  }
}
```


## Make executable

```js
#!/usr/bin/env node
require(".")
```

### Get all function argments

```js
function func(1,2 3) {
	console.log(arguments)
}
```

## String literals and parentheses-less calls


```javascript
const showstring = (str, ...values) => {
  console.log(str);
  console.log(values)
}
const name = 'John'
showstring`Hello World ${name} ${name} ${1 + 2} and more`;
```

```javascript
// str
[
  "Hello World ",
  " ",
  " ",
  " and more"
]

// values
[
  "John",
  "John",
  3
]
```

## Generators

```javascript
function* zip(...arrays){
  const maxLength = arrays.reduce((max, curIterable) => curIterable.length > max ? curIterable.length: max, 0);
  for (let i = 0; i < maxLength; i++) {
    yield arrays.map(array => array[i]);
  }
}
```


## `this` and inheritance

```js
function sum(b) {
  return this.a + b;
}

sum.call({a: 4}, 5) // 9
const newSum = sum.bind({a: 4})
newSum(3) // 7
```


```js

function User() {
	this.id = 5

	this.test1 = () => {
		console.log(this.id)
	}

	this.test2 = function() {
		console.log(this.id)
	}
}

User.prototype = {
	
    show: function () {
	    console.log(`user id = ${this.id}`)
    }
}

let user = new User()

user.show() // user id = 5
user.test1() // 5
user.test2() // 5

user.__proto__ = { 
	show: function() {
		console.log(`user: ${this.name}`)
	}
}

user.name = "Artur"
user.show() // user: Artur

user.__proto__ = {
	show: () => { 
		console.log(`user: ${this.name}`)
	}
}

user.show() // user:

```

