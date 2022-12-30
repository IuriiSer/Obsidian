[Clean Up](https://dev.to/pallymore/clean-up-async-requests-in-useeffect-hooks-90h) Async Requests in `useEffect` Hooks

## A Simple Solution

Knowing this, you might think: oh well, that's great, we can set a flag that is set to false when the component unmounts so we can skip all the state updates.

And you are right, that's indeed a very simple solution to this problem:  

```
  useEffect(() => {
    let isCancelled = false;
    const fetchData = async () => {
      dispatch(requestStarted());

      try {
        // fetch logic omitted...
        const data = await response.json();

        if (!isCancelled) {
          dispatch(requestSuccessful({ data }));
        }
      } catch (e) {
        if (!isCancelled) {
          dispatch(requestFailed({ error: e.message }));
        }
      }
    };

    fetchData();

    return () => {
      isCancelled = true;
    };
  }, [url]);
```

In this code - whenever a new effect runs (or the component unmounts), the previous' effect's `isCancelled` is set to `true` - and we only update the state when it is `false`. This makes sure that your `requestSuccessful` and `requestFailed` actions are only dispatched on the latest request.

Mission accomplished!...?

## But You Really Should Do This

There is a better way though. The code above is fine, however, if your `fetch` request is really slow, even if you don't need the results anymore, it is still going on in the background, waiting for a response. Your user might be clicking around and leaving a bunch of stale requests behind - did you know? There is a limit of how many concurrent requests you can have going on at the same time - usually 6 to 8 depending on which browser your users are using. (This applies to HTTP 1.1 only though, things are changing thanks to HTTP/2 and multiplexing, but that's a different topic.) Your stale requests will be blocking newer requests to be executed by the browser, making your app even slower.

Thankfully, there is a new feature in the DOM API called `AbortController` which allows you to cancel `fetch` requests! It is well supported by most browsers (No IE11 though) and we should definitely take advantage of it.

The `AbortController` is very easy to work with. You can create a new one like this:  

```
const myAbortController = new AbortController();
```

and you will find two fields on the instance: `myAbortController.signal` and `myAbortController.abort()`. `signal` is to be provided to the `fetch` call you want to cancel, and when `abort` is called that `fetch` request will be cancelled.  

```
fetch(url, { signal: myAbortController.signal });

// call the line below to cancel the fetch request above.
myAbortController.abort(); 
```

If the request has already completed, `abort()` won't do anything.

Awesome, now we can apply this to our hook:  

```
  useEffect(() => {
    const abortController = new AbortController();

    const fetchData = async () => {
      dispatch(requestStarted());

      try {
        fetch(url, { signal: abortController.signal });

        // code omitted for brevity

        dispatch(requestSuccessful({ data }));
      } catch (e) {
        dispatch(requestFailed({ error: e.message }));
      }
    };

    fetchData();

    return () => {
      abortController.abort();
    };
  }, [url]);
```

Now our `fetch` request will be promptly cancelled for each new effect, or right before the component unmounts.

### [](https://dev.to/pallymore/clean-up-async-requests-in-useeffect-hooks-90h#handling-cancelled-requests)Handling Cancelled Requests

Just one little thing though - when a request is cancelled it actually throws an error, so our `catch` block will be executed. We probably don't want to dispatch a `requestFailed` action in this case. Fortunately we can tell if a request has been aborted by checking the `signal` on the `AbortController` instance.

Let's do that in our `catch` block:  

```
try {
 // ...
} catch (e) {
  // only call dispatch when we know the fetch was not aborted
  if (!abortController.signal.aborted) {
    dispatch(requestFailed({ error: e.message }));
  }
}
```

## [](https://dev.to/pallymore/clean-up-async-requests-in-useeffect-hooks-90h#wrapping-it-up)Wrapping It Up

Now our hook can properly cleans up after itself! If your hook does something async, in most cases they should be cleaned up properly to avoid any unwanted side-effects.

If you are using `fetch`, then `abort` your requests in the clean up function. Some third party libraries also provide a way to cancel requests (like the `CancelToken` from `axios`).

If you want to support older browsers, or your effect doesn't use `fetch`, but is using some other async operations (like `Promise`), before cancelable `Promise`s becomes a reality, use the `isCancelled` flag method instead.

### [](https://dev.to/pallymore/clean-up-async-requests-in-useeffect-hooks-90h#resources)Resources

[https://developer.mozilla.org/en-US/docs/Web/API/AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)

[https://reactjs.org/docs/hooks-effect.html](https://reactjs.org/docs/hooks-effect.html)