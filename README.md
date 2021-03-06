[jSpaghetti](https://github.com/gresendesa/jSpaghetti) 2.0.0
=================================================

## Changelog

* **2.0.2** error event added.
* **2.0.1** remove onload listener after sequence terminated.
* **2.0.0** `28 december 2020`: `{"wait": "_forTheSignal"}` removed. `_exit` replaced by `{"exit": <condition>}`, `{"gotoif": []}` replaced by `{"jumpif": []}`. Exit instruction is not supported by jumpif anymore.
* **1.0.1** `28 november 2020` SOON: `{"wait": "_forTheSignal"}` will be removed. `_exit` will be replaced by `EXIT`. `_forPageToReload` will be replaced by `page-reload`
* **1.0.0** `23 november 2020` the `next(message)` hook is added to add a asyncrousness to the states transition. If a state returns nothing, then the transition is not performed automatically. It will be necessary to call the refered function to run the transition.

## Synopsis

jSpaghetti is a Javascript API to build automation scripts with sequential and declarative paradigm.

## Motivation

The project came from the necessity of an easy way to write browser javascript bots. The project was generalized to cover other purposes, though.

## Build
As this project is actually a huge single js file, it was broken into parts in order to be more readable. To "compile" the project just process the file main.js with [Filemash](https://github.com/gresendesa/filemash) script using the following command:

```
python3 filemash.py main.js > jSpaghetti.js
```

## Installation

Just include the file on document page, like:
```html
<script type="text/javascript" src="jSpaghetti.js"></script>
```
This code can be also injected on the document page using tools like Greasemonkey or something.

## API Reference
Warning: This manual is just a sketch. Detailed manual coming soon

### Modules
* To assign a module object to a variable. If the module does not exist yet, it is created before assingment.
```js
var moduleObject = $jSpaghetti.module("myModule")
```

### Procedures
Procedures are custom functions to be used as a library for the sequences. Each module has its own procedures. 
* Defining procedures
```js
$jSpaghetti.module("myModule").procedure("myProcedureFoo", function(){
	//Code something in here
})

$jSpaghetti.module("myModule").procedure("myProcedureBar", function(){
	//Code something in here
})

//Define as much as you need
```

### Sequences
A sequence provides a wrapped environment for the execution of the instructions. It manages the execution to provide a shared data between the procedures. The data is not lost when page is reloaded.
* Creating a new sequence
```js
var sequenceObject = $jSpaghetti.module("myModule").sequence("mySequenceBaz")
```

#### Instructions
This is the best part. You can use a declarative way to write the script behavior. Procedures can be arranged to be executed in a defined order:
```js
sequenceObject.intructions = [
	{"step1":["myProcedureFoo", "myProcedureBar"]},
	{"labelX":"myProcedureBax"}
]

//Define as much as you need
```

##### Commands
Internal commands can change the script behavior.
```js 
{"exit": true} //Finish the sequence execution
``` 
```js 
{"exit": "2 == 1"} //It does not finish the sequence execution because the condition returns false
``` 
```js 
{"wait": 1000} //Wait the defined time in ms
``` 
```js 
{"wait": "page-reload"} //Pause the sequence execution and resume after page is reloaded
```
```js 
{"jumpif":["2 == 1", "step1", "labelX"]} // It redirects the program flow to a sequence (In this case to the "labelX")
``` 

#### Running
* To execute a sequence use something like this:
```js
$jSpaghetti.module("myModule").sequence("mySequence").run()
```

#### Stopping
```js
$jSpaghetti.module("myModule").sequence("mySequence").reset()
```
#### Debugging
To see what is happening during the sequence execution, turn on the debug mode. The debugging log will be shown on browser console. Developer mode show as much details as possible.
```js
$jSpaghetti.module("myModule").config.debugMode = true
$jSpaghetti.module("myModule").config.developerMode = true
```

### Shared data
Data can be shared between procedures during the sequence execution. The shared object is received as the first parameter on each procedure. If this object is modified, that changes will be available for the next instruction. The previous procedure return value is always available on the "$" attribute of this object. 

```js
$jSpaghetti.module("myModule").procedure("myProcedureBar", function(sharedData){
	console.log(sharedData.$) //It will show the value returned for the previous procedure
	sharedData.foo = "baz" //This attribute and its value will be available for the next procedures
	return 2000 //This value will be available on the "$" attribute in the next procedure
})
```

### Events
The sequences return the "terminated" event when the sequence is done. All the events:
```js
$jSpaghetti.module("myModule").procedure("myProcedureBar").events.addEventListener("terminated", function(e){
	//Code something in here
})

$jSpaghetti.module("myModule").procedure("myProcedureBar").events.addEventListener("error", function(e){
	console.log(e.detail)
})

$jSpaghetti.module("myModule").procedure("myProcedureBar").events.addEventListener("reset", function(e){
	//Code something in here
})

```

### Examples

```js
var module = $jSpaghetti.module("myModule")

sequence.reset()

module.procedure("A", function(){
    console.log("brown fox jumps ")
})
module.procedure("B", function(){
    console.log("over the lazy dog")
})
module.procedure("C", function(){
    console.log("quick ")
})

var sequence = module.sequence("showPhrase")
sequence.instructions = [
    {0: "C"},
    {"foo": ["A", "B", {"exit": true}]}
]

sequence.run()

//Output: quick brown fox jumps over the lazy dog

```
