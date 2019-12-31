---
layout: post
title:  "data driven testing in Mocha Js"
date:   2019-12-09 10:20:00 +0200
categories: "testing, mocha, javascript"
---
# mocha js and tests driven by input data ("data driven tests")
**one thing I really like and 'dislike' about javaScript is that there is not a single-better option of doing the same thing**
 
what I mean is that the language provides you with enough flexibility to write the same functionality in many different ways,and deciding which is the best or worst way to implement it, will probably be very subjetive. 

**I will explain below a very intuitive approach to use data driven with mochaJS in order to to cover a real life scenario**

## pre-requisites
for this article I will use [nodeJS](https://nodejs.org/en/download/), so make sure that is installed.

## getting started
first, lets create a folder project folder and create our `package.json` file
```bash
$ mkdir mocha-ddt && cd mocha-ddt
$ touch package.json
```

now copy the following snippet in our recently created file. *this is far from a real package.json file but works for the purpose* 
```javascript
{
  "name": "mocha-sample",
  "scripts": {
    "test": "mocha"
  },
  "devDependencies": {
    "chai": "^4.2.0",
    "chai-http": "^4.3.0",
    "mocha": "^6.2.2"
  }
}
```
we just defined some dev dependencies and a script to run our tests. *We are going to use chai for the assertions and http requests*

finally let's create a `test` folder under the root project, so we end up with the following path: `mocha-ddt/test`

## let's keep it simple
firs't thing to understand is how mocha and chai work toguether. create a file called `test.js` under the `test` folder.<br>
then we are going to add a simple test: 
```javascript
var chai = require('chai');  // assertions library
describe('When Math works', function() { // describe is our spec definition
  it('should return the result of a sum operation', function() { // this is our test case
    let sum = 2 + 2; // do I need to explain this?
    chai.expect(sum,'In case this fails this is the error').to.be.equal(4); // very simple validation
  });
});
```
now we can run our test case, so under the root folder execute:
```bash
$ npm run test
```
and we can see the following output:
![terminal output](https://github.com/pjcalvo/pjcalvo.github.io/blob/master/resources/simple-mocha.png?raw=true)


## let's get into a real life test scenario
many times in my career I have being given with a list of links and a test case that says: for all this links, verify which ones work. so we are going to recreate that very simple:

create a new file `mocha-ddt.js` under the `test` folder and add the following:
```javascript
// required libraries + chai-http as a chai plugin
var chai = require('chai'), 
  chaiHttp = require('chai-http'); // for this example we are going to use chai-http to perform http requests
chai.use(chaiHttp); 

// this is the magical data source. we are creating a json list of urls and a expected response code for each
let urls = [
    { 'url' : 'https://www.google.com', 'response_code' : 200},
    { 'url' : 'https://pjcalvo.github.io', 'response_code' : 200},
    { 'url' : 'https://dev.to/error500', 'response_code' : 500},
    { 'url' : 'https://pjcalvo.github.io/notfound/page', 'response_code' : 404}
]

describe('When the links are working as expected', function() {

  urls.forEach(({url, response_code }) => { // foreach is an iterator function that will run thought all the items on urls

    // using ` and $ {} we can simple format the test case name to a meaningful output
    it(`should return '${ response_code }' for request '${ url }'`, function(done) {  // done parameter is required by http-chai to handle the it when the operation is completed

      chai.request(url) // this url comes from urls.foreach element
        .get('/') // just on the 
        .end(function(err, res) {
           // here we will validate that the response call from http-chai will match the url response_code given above
          expect(res,`... instead got ${ res.status}`).to.have.status(response_code);
          done(); // you need this here to tell mocha that the request is completed
        });
    });
  });
});
```
let's add a little explanation before running this sample: **because javaScript is an interpreted language (which means that NodeJS reads everyline at execution time and interpret it) we have a certain flexibility to create test cases during runtime. we could also create describes and add subsets of test cases based on `if` conditions**, unlike Java in which the test cases need to be compiled in order for the test runner to find them. But we are not comparing apples with gallinas here.

so running again the test command `npm run test`, gives us a new very nice output
![terminal output](https://github.com/pjcalvo/pjcalvo.github.io/blob/master/resources/simple-mocha-ddt.png?raw=true)

## duty segregation 
an even cleaner solution would be separate the json data from the test file and add it to a `urls.json` and use a custom wrapper to read, transform and present it to the test files.
``` javascript
// data-wrapper.js
getUrl((condition) => {
  if (condition)
    return JSON.parse(fs.readFileSync('mocha-ddt/test/url.json'));
  else
    return JSON.parse(fs.readFileSync('mocha-ddt/test/thisotherjson.json'));

// test file
let testUrls = datawrapper.getUrl(condition);
```
this is more a suggestion, so we are not fully implementing.


## final thoughts
as I said at the beginning there are many ways to accomplish the same outcome, I like this way of creating data driven tests because it is really easy to explain and with some teawking and condition it can also manage more complex scenarios. I have used this approach with UI testing using WDIO as the UI framework and it also adapts very well. just be mindful that UI tests are normally not stateless tests and to accomplish navegability you need to handle proper before and each scenarios.

Comments?

[mocha](https://mochajs.org/)<br>
[chaiJS](https://www.chaijs.com/)<br>
[chai-http](https://www.chaijs.com/plugins/chai-http/)<br>

cheers :) and remember to be data driven
[https://pjcalvo.github.com](https://pjcalvo.github.com)