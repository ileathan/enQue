# enQue.js
Enable queing asynchronous functions one after the other.

For example if you want to execute 5 asynchronous functions that all operate on the same data, or you just want them done in succession.
Its as simple as

```javascript
const enQue = new require('enQue')
const myQue = enQue([fn1, fn2, fn3, fn4, fn5])
myQue.run()
```

However since a promise is returned and since usually you want to output some data it is better to always attach a `.then()` and `.catch()` so the above would become.

```javascript
myQue.run()
  .then(sucessCallback)
  .catch(errorCallback)
```

```javascript
// USAGE EXAMPLE
const Que = require('enQue');
const que = new Que();

// Used to only go backwards once, otherwise you end up in an infinite cycle.
// IF YOU GO FORWARD IN THE QUE YOU CANNOT GO BACKWARDS TO A PLACE NOT ON THE NEW QUE
// It will throw a error if you attempt to go forward then backwards.
var once = true;

que.add((data, next, done, index) => {
  //return done(); // You can return early if you'd like
  setTimeout(()=>{
    data.msg += ' ONE';
    // You can inject a promise at a specified relative position.
    next({inject:1, promise: new Promise((s,e)=>{data.msg += " ONE AND A HALF"; s(data) }) });
  }, 9000)
});
que.add((data, next) => {
  setTimeout(()=>{
    //throw "errors work too" // You can throw an error and reject the promise
    data.msg += ' TWO';
    next();
  }, 4000)
});
que.add((data, next) => {
  data.msg += ' THREE';
  next()
});

function myFn1(data, next) {
  data.msg += ' FOUR';
  // You can go backwards in the que.
  if(once) { once = false; next(-2); }
  // You can go forwards in the que.
  else next(1);
}

function mFn2(data, next) {
  data.msg += ' FIVE';
  next();
}

// You can add multiple functions at a time.
que.add([myFn1, myFn2]) 

que.run({msg: 'ZERO'})
  .then(res => console.log(res.msg))
  .catch(err => console.log('Woopsie! ' + err))
```

The above code when executed prints

```
ONE ONE AND A HALF TWO THREE FOUR TWO THREE FOUR FIVE
```
