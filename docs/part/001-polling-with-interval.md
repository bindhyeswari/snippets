# Polling with interval

Idea is to perform polling with an interval. Features

 - Make an API call every `interval` seconds.
 - If an existing call is in progress, do not proceed with another call.  
 
```js
const axios = require('axios');

const SAMPLE_URL = 'https://raw.githubusercontent.com/bindhyeswari/interview-prep/master/fixtures/sliced-wine-reviews.json?token=ABAZB7K4FZMNYMO3JIAFZIK6AKC54';

/**
 * @function fetchWineReviews
 * @desc An asynchronous function that retrieves the url data and returns 
 *  a promise
 * @param {String} url
 * @return {Promise} promise
 * */
const fetchWineReviews = async url => {
  const {data} = await axios(url);
  return data;
};

/**
 * @function poller
 * @desc A function that takes in a function that returns a promise, an 
 *  interval and a callback and invokes the function at max, once during 
 *  the interval, by checking if the previous call has completed and 
 *  invoking the appropriate callback.
 * @param {Function} fetchFn A function, that once invoked, returns a promise
 * @param {Number} interval Interval in milliseconds
 * @param {Function} callback A callback with the signature callback(err, results)
 * @return {Function} stopPoller A function that shuts down the poller
 * */
function poller(fetchFn, interval, callback) {
  let inProgress = true;
  // Fire up the initial call

  // Note that I am not using a .finally as it is not supported on older versions of NodeJS
  // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/finally

  fetchFn().then(results => {
    inProgress = false;
    callback(null, results);
  }, err => {
    inProgress = false;
    callback(err, null);
  }).catch(ex => {
    inProgress = false;
    callback(ex, null);
  });

  // Fire up the subsequent calls
  const intervalId = setInterval(() => {
    if (!inProgress) {
      console.log('The previous call is completed, hence making the API call.');
      inProgress = true;
      fetchFn().then(results => {
        inProgress = false;
        callback(null, results);
      }, err => {
        inProgress = false;
        callback(err, null);
      }).catch(ex => {
        inProgress = false;
        callback(ex, null);
      });
    } else {
      console.log('The previous call is still in progress, hence skipping.');
    }
  }, interval);

  return clearInterval.bind(null, intervalId);
}

// Example
(async function () {
  const fetchFn =  fetchWineReviews.bind(null, SAMPLE_URL);
  console.log(fetchFn);
  let counter = 0;
  const stopPoller = poller(fetchFn, 100, (err, reviews) => {
    console.log(++counter);
    if (err) console.log('Encountered an arror while making the API call');
    else console.log('Got the data: ', reviews[0].title);
  });

  // Shut down the poller after 5 seconds
  setTimeout(() => {
    stopPoller();
  }, 5000)
}());
```  
