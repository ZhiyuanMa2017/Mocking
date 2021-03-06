# Unit Testing With 'Nock' Mocking


In this workshop, you will learn about mocking, specifically mocking responses to http requests to RESTful apis without needing to make a real call.

## Setup and Background

Clone this repository. Change into directory and run `npm install`.

Run `npm test`. You should see something like:

(WINDOWS USERS:   edit `package.json` and change line 7 to `".\node_modules\.bin\mocha --reporter spec test\*.js"`.  For the curious...[why Windoes uses backslash](http://www.os2museum.com/wp/why-does-windows-really-use-backslash-as-path-separator/))

![image](https://cloud.githubusercontent.com/assets/742934/18939521/dd0610e8-85ce-11e6-879a-30a52f7c0907.png)

### Mocking

A primary goal of mocking is to switch between real sources of data and mock data without impacting the implementation of the code. 

There are several libraries that support mocking. One common feature in mocking libraries is the ability to easily build mock objects that will have a fixed behavior for a specific test case. For example, using mockito, you could test the `mostRetweeted` function by building a mock object of tweet statuses.

(This is just a example, not something you need to run for this workshop)
```java
List<Status> statuses = new ArrayList<Status>()
testStatus = Mockito.mock(Status.class);
when(testStatus.getId()).thenReturn(900000L);
when(testStatus.getRetweetCount()).thenReturn(33);
statuses.add(testStatus);

Status bestTweet = best.mostRetweeted(statuses);
assertEquals(bestTweet.getId(), 900000L );
assertEquals(bestTweet.getRetweetCount(), 33 );
```

Another concern is the ability to support mocking of service calls. That is the primary focus on this workshop, which will use 
[nock](https://github.com/node-nock/nock#use) to help intercept requests to a url and instead return mock data.

For example, this will return a simple mock object if someone makes a request to `https://api.github.com/users/testuser/repos`.

```javascript
var mockService = nock("https://api.github.com")
   .get("/users/testuser/repos")
   .reply(200, JSON.stringify([{"name:","repo"}]) );
```

### Test Suites

Unit testing often involves running a set of test cases that exist in a test suite. A test case typically focuses on one specific behavior of a unit (class/file) and usually focuses on a single function.

[Mocha](https://mochajs.org/) is a tool for managing test cases for javascript programs. Mocha will run all tests in the "test" directory.

In this example, you can see a test suite with two test cases and a `before` hook, which will be called before any test case is run. You can also call `after` hooks, which can be used to clean up after a set of test cases are run.

```javascript
describe('earth', function(){
  before(function(){
    console.log('see.. this function is run ONCE only')
  })
  it('birds should fly', function(){ /** assertions */ })
  it('horse should gallop', function(){ /** assertions */ })
})
```

At the heart of a test case is an assertion. Look at this simple test case for checking the behavior of the `indexOf` function.

```javascript
it('should return -1 when the value is not present', function() {
    assert.equal(-1, [1,2,3].indexOf(4));
});
```

Now, there are alternative ways to write assertions, some people prefer a more "fluid" style of writing assertions, which is what [chai](http://chaijs.com/) provides. In the workshop, we use the "expect" style of assertions.

![image](https://cloud.githubusercontent.com/assets/742934/18939984/b0d08004-85d2-11e6-85f3-ea690f2c5145.png)

## Unit Testing a Github Issue Analytics Program

Suppose you had a program that performed some basic analytics on a code repo, such as finding who is assigned to fix bugs the most often, how many closed issues there are, and analyzing the amount of words in a body versus title.
Performing analytics can be tricky because getting calculations right can take some time and there may be many weird edge cases to catch. Further, if you did test with real live data, you might exhaust your api access in running lots of tests.

TDD (test-driven development), where you write test cases as you develop code is a great fit for this type of program. Mocking also helps us test our code with minimal dependency on having a live github service.

### Let's get our tests to pass!

1. In testMain.js, uncomment the `/repos/testuser/Hello-World/issues` interceptor. This will allow us to get mock data if we try accessing that url in our tests.

2. *should find 4 closed issues* seems to be returning 6 instead of 4. Can we fix this?

3. *should handle empty body for issue #2* doesn't seem to have any interceptors that trigger: `RequestError: Nock: No match for request...`. Can we add a new mock intercept?

4. Oh no, *should handle empty body for issue #2*, is running, but now it is returning 1 instead of "NA". Why?

5. Write a new function, "maxStars" that returns the name of repo with the most stars. You should be able to use existing `github.getRepos()` function and the `stars` data inside `mock.js` as mock data. But, this will require digging through the program in order to understanding how everything works.

6. Test your new function!
