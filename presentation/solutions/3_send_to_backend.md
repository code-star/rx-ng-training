
#  Exercise 3
 
We have a reactiveForm that we want to submit. Make sure it's all hooked up properly and sends off the results to our "backend".
 
1) Make sure the submit button does not send to backend when it's already sending.

```ts
fromEvent(submitBtn).pipe(
    exhaustMap() // instead of switchMap, to ignore new click values coming in.
)
```

2) Make the backend get hit less by checking if the values are "valid" (empty is not allowed).

Angular validators are nice here, simply adding "required" to the HTML will do the trick for the bonus.

Here you could have a filter() inside the pipe to check for values, or an if statement within a map.

3) Bonus: Make sure it doesn't send until the values are filled. You can do this by adding Angular validators to the FormControls.

Note: When using the `fromEvent()` as a base (for like a `this.submit$`), this is the most reactive indeed; but considering you've turned it into a Hot observable, make sure you shut it down properly on `ngOnDestroy()`.