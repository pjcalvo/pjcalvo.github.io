---
layout: post
title:  "data driven testing with WDIO"
date:   2019-12-09 10:20:00 +0200
categories: "testing"
---
# webdriverIO and tests driven by input data ("data driven tests")
**one thing I really like and dislike about javaScript is that there is not a single-better option of doing the same thing**
 
what I mean is that the language provides you with enough flexibility to write the same functionality in many different ways,and deciding which is the best or worst way to implement it, will probably be very subjetive. 

**so when writing UI tests I find it very handy to use the following implementation to write data driven tests**

```javascript
// include pages
const Home = require('../../home.po');
...

// define data source

tests = [
  	{
      issue:"MOD-001",
      user:{
        username:"pepito@",
        password:"notequiere",
        status:"valid"
      },
      result:true
    },
    {
      issue:"MOD-002",
      user:{
        username:"pepito@",
        password:"sitequiere",
        status:"invalid",
      },
      result:true
    }
];

//now we write the spec

tests.forEach(({ issue, user, result }) => {

  	describe(`[{ issue }] When the user is { user.status } then the tests result should be { result }`, () => {
  		before(() => {
    		Home.open();
  	  });
      
      //its and steps go here
      it(`Attempt to login with user: { user.username }/{ user.password }`, () => {
        Home.loginWithUser(user);
      })
      
      ...
      
      it ('This is the verification', () => {
        // verification changes here
        expect(Home.isUserSignedIn()).to.equal(result);
      });
      
      after(() => {
        //cleackCookies, signout or somehting
      });
      
}); // end of foreach loop
```

**I really like this approach because it is really easy to read and undestand.** 

[webdriverIO](https://webdriver.io/)
[mocha](https://mochajs.org/)
[chaiJS](https://www.chaijs.com/)

cheers