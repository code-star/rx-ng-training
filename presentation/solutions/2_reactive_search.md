# Exercise 2
1. Make the search less aggressive on the backend

debounceTime, distinctUntilChanged, filter for minimum length.

2. Lessen the pressure on the backend for the data retrieval.

The initial service for loading the data does not cache it and is only called once. The search service does not cache either and seems to be unrelated to the original service call. If there's no search query, it should default to that. Can be solved in either the service (not bad) or in the stream on the component (little more explicit, but not necessarily easier or better)

3. Optionally build a cache for the slow data.

Caching can be done in many ways. Internal shareReplay() on the service would have my preference here.

https://stackblitz.com/edit/rx-exercise-2-solution?file=src/app/app.component.ts