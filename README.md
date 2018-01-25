# Fn Function Developer Kit for Node.js

This Function Developer Kit makes it easy to deploy Node.js functions to Fn.
It currently supports default (cold) and hot functions using the JSON format.

## Implementing a Node Function

Writing a Node.js function is simply a matter of writing a handler function
that you pass to the FDK to invoke each time your function is called. 


Start by creating a node function with `fn init` and installing the FDK: 


```bash
$ mkdir myfn 
$ cd myfn 
$ fn init --runtime node 
$ npm install --save-dev fn-fdk
```

Here's a simple hello world function - save this to `func.js`.


```javascript
var fdk=require('fn-fdk');

fdk.handle(function(input){
  var name = 'World';
  if (input) {
    name = input;
  }
  return 'Hello ' + name + ' from Node!';
})
```

The anonymous handler function takes the string input that is sent to the function
and returns a response string.  Using the FDK you don't have to worry about reading
input from standard input and writing to standard output to return your response.
The FDK let's you focus on your function logic and not the mechanics.

For a default function the function's `func.yaml` file might look like this:

```yaml
name: hello
version: 0.0.1
runtime: node
cmd: func.js
```

Since the `format` is not specified it is *not* a hot function and each invocation will
create a new instance which will be destroyed after it returns it's response.  Once
deployed, you and invoke the function:

```sh
echo -n "Tom" | fn call fdkdemo /hello
```

or

```sh
curl -d "Tom" http://localhost:8080/r/fdkdemo/hello
```

In both cases you'll get the response:

```sh
Hello Tom from Node!
```


# Function Context

Function invocation context details are available through an optional function argument.
To receive a context object, simply add a second argument to your handler function.
in the following example the `call_id` is obtained from the context and included in 
the response message:

```javascript
var fdk=require('fn-fdk');

fdk.handle(function(input, ctx){
  var name = 'World';
  if (input) {
    name = input;
  }
  return 'Hello ' + name + ' from Node call ' + ctx.call_id + '!';
})
```

In the case of `default` format functions the context give you access to all environment variables
including those defined through function or app config as well as those automatically provided
by Fn like `app_name`, `path`, `memory`, etc.


# Hot Functions with the FDK

To make our function a hot function we simple specify the function `format` in the `func.yaml`.
The Node FDK only support the `json` format for hot functions.  The `http` format is 
equivalant in functionality so supporting `json` is enough to enable hot functions:

```yaml
name: hello
version: 0.0.1
runtime: node
cmd: func.js
format: json
```

Redeploy your function to apply the change and that's it!  Your function is now hot--no code
changes!

To illustrate the value performance difference between default and hot functions take a look
at the times taken to invoke our function (on a MacBook).  Default functions take over a 
second per invocation but after the first invocation a hot function takes about 40 
milliseconds of real time. The advantage of the FDK is that you can decide which makes 
sense for your function at deployment time rather than at development time based on your needs.

**Default Function Timings**

```shell
$ time curl -d "Tom" http://localhost:8080/r/fdkdemo/hello
Hello Tom from Node!
real  0m1.245s
user  0m0.005s
sys   0m0.005s
$ time curl -d "Tom" http://localhost:8080/r/fdkdemo/hello
Hello Tom from Node!
real  0m1.161s
user  0m0.004s
sys   0m0.005s
$ time curl -d "Tom" http://localhost:8080/r/fdkdemo/hello
Hello Tom from Node!
real  0m1.266s
user  0m0.005s
sys   0m0.006s
$ time curl -d "Tom" http://localhost:8080/r/fdkdemo/hello
Hello Tom from Node!
real  0m1.181s
user  0m0.005s
sys   0m0.006s
```

**Hot Function Timings**

```shell
$ time curl -d "Tom" http://localhost:8080/r/fdkdemo/hello
Hello Tom from Node!
real  0m0.039s
user  0m0.005s
sys   0m0.006s
$ time curl -d "Tom" http://localhost:8080/r/fdkdemo/hello
Hello Tom from Node!
real  0m0.036s
user  0m0.005s
sys   0m0.006s
$ time curl -d "Tom" http://localhost:8080/r/fdkdemo/hello
Hello Tom from Node!
real  0m0.042s
user  0m0.005s
sys   0m0.006s
```

# Asynchronous function responses 

You return an asynchronous response from a function by returning a Javascript `Promise` from the function body: 

```javascript
var fdk=require('fn-fdk');

fdk.handle(function(input, ctx){
  return new Promise((resolve,reject)=>{
     setTimeout(()=>resolve("Hello"),1000);
  });
})
``` 