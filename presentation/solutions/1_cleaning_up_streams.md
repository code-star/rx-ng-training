# Exercise 1

Fork the following Stackblitz and clean up the streams so they do not produce memory leaks.

~5-15 minutes.

https://stackblitz.com/edit/rx-exercise-1-solution?file=src/app/app.component.ts

---

Before we show our work; we can prove/show what a memory leak does or causes; the `pipe(tap(console.log))` helps with this; a hot reload keeps the subscription running and you can see the differences.

The two Observables produce memory leaks cause they aren't getting shut down. 

The first one only returns a result once, so we can add first() or take(1) to the pipe. 

The second one is an interval (hot). We can limit its number of values being emitted; or we make sure the component takes care of our subscription, so we delegate it to the AsyncPipe. 

An optional, nice, way to mange both observables would be to let the AsyncPipe take care of both, as there is no reason why it couldn't here.