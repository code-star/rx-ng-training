<img src="assets/rx-ng.png"/>

### Managing RxJS streams in Angular <!--.element: class="fragment" -->

---

<div style="">
  <img src="assets/bjorn.jpg" width="100" style="border-radius:100%; display: inline-flex;">
  <h1 style="font-size: 0.9em;">Bjorn Schijff</h1>
  <small style="display: inline-flex;">Frontend Software Engineer @ Politie</small><br />
  <img src="assets/codestar.svg" height="30" style="border: 0; background-color: transparent;"><br /><br />
   <small>@Bjeaurn<br /> bjorn.schijff@ordina.nl</small>
</div>

---

## Why?

- Angular as a framework offers plenty of tools to manage it. <!--.element: class="fragment" -->
- RxJS streams have so much potential, and it doesn't always show. <!--.element: class="fragment" -->
- Lot's of ways lead to Rome. <!--.element: class="fragment" -->

----

```ts
export class AppComponent {

    data: DataObject = null

    ngOnInit() {
        this.dataService.getData().subscribe(
            data => this.data = data
        )
    }
}
```

(Basic assignment, not the best)

----

```ts
export class AppComponent {

    data: Observable<DataObject>

    ngOnInit() {
        this.data = this.dataService.getData()
    }
}
```

```html
{{ data | async | json }}
```

(Let Angular take care of it)

<small>(Bonus: TypeScript in strict mode will complain about this one)</small><!--.element: class="fragment" -->

```
Property 'data' has no initializer and is not definitely assigned in the constructor.
```
<!--.element: class="fragment" -->

---

## Goal for today

- Get familiar with multiple ways of managing your streams <!--.element: class="fragment" -->
- Know when to pick which method <!--.element: class="fragment" -->
- Look around and experiment with a few "common" use cases. <!--.element: class="fragment" -->

---

### Unsubscribing
Where does unsubscribe happen?

```ts
export class AppComponent {
    data: Observable<DataObject>
    ngOnInit() {
        this.data = this.dataService.getData()
    }
}
```
```html
{{ data | async | json }}
```

----

Where does unsubscribe happen?

```ts
export class AppComponent {
    data: DataObject = null
    ngOnInit() {
        this.dataService.getData().subscribe(
            data => this.data = data
        )
    }
}
```

It doesn't! We've got a memory leak here. <!--.element: class="fragment" -->

----

### Lifecycle Hooks <!--.element: class="fragment" -->

----

```ts
export class AppComponent {
    data: DataObject = null
    dataSubscription: Subscription

    ngOnInit() {
        this.dataSubscription = this.dataService.getData().subscribe(
            data => this.data = data
        ) // .subscribe() returns a Subscription, which we store.
    }

    ngOnDestroy() {
        this.dataSubscription.unsubscribe() 
        // And unsubscribe from here, when the component is destroyed.
    }
}
```

Is this nice? <!--.element: class="fragment" -->

----

```ts
export class AppComponent {
    data: DataObject = null

    ngOnInit() {
        this.dataService.getData()
            .pipe(
                first() 
                // Adding the first() operator, will make it complete and unsubscribe after the first value is passed.
            )
            .subscribe(
                data => this.data = data
            )
        // So less assignment, no need to keep track of our subscription.
        // This assumes that the opened stream is not a continous stream.
    }
}
```

---

### Letting Angular take care of our subscription

----

```ts
export class AppComponent {
    data: Observable<DataObject> // = of(null) // Optionally!

    ngOnInit() {
        this.data = this.dataService.getData()
            .pipe(
                first() 
                // Adding the first() operator, will make it complete and unsubscribe after the first value is passed.
            )
        // We assign the Observable stream as a whole, and let the HTML/Angular take care of it for us.
    }
}
```

```html
<div>
{{ data | async | json }}
</div>
```

----

### AsyncPipe

<small>

>The async pipe subscribes to an Observable or Promise and returns the latest value it has emitted. 

>When a new value is emitted, the async pipe marks the component to be checked for changes. 

>When the component gets destroyed, the async pipe unsubscribes automatically to avoid potential memory leaks.

</small>

Using the AsyncPipe whenever you can, is a good idea!<!--.element: class="fragment" -->

----

```html
<div *ngIf="data$ | async as data">
 This will give us access to the {{ data }} variable, 
 with all values available in the HTML.

 Given that `data$` is an Observable.
</div>
```

This is my personal favorite way to manage streams, if you can use it.

----

```ts
export class AppComponent {
    data$: Observable<DataObject> = this.dataService.get()

    constructor(private dataService: DataService) {}
}
```

Works best when you have very little logic in your component.

----

To summarize:

- Using the AsyncPipe let's Angular manage your subscriptions.
- It'll update your values as they change.
- Works best when the surrounding logic is minimal.

---

### Reactively managing your lifecycle
What if you can't let Angular manage your Observable?<!--.element: class="fragment" -->

----

You can tie in with the ngOnDestroy() lifecycle:

```ts
data$: Observable<User>
private destroy$: Subject<void> = new Subject<void>() // Aaah a Subject! Make it private!

ngOnInit() {
    this.data$ = this.dataService.getStream().pipe(
        takeUntil(this.destroy$)
        // takeUntil will take values, until the given Observable triggers.
    )
}

ngOnDestroy() {
    this.destroy$.next()
    this.destroy$.complete()
    this.destroy.unsubscribe()
    // Neat little trick that'll forcibly destroy any subscribers not completed yet.
}
```

----

This is "overkill" for a couple of reasons:

- <span class="fragment">Assumes the `getStream()` is infinite (or hot)</span>
- No reason this example cannot be managed by the HTML.<!--.element: class="fragment" -->

<small>So stay vigilant!</small><!--.element: class="fragment" -->

---

Exercise time!

`exercises/1_cleaning_up_streams.md`

---

### Easening pressure on the backend
Or hitting the backend less often.

----

Why do we do this?

- Frontend Performance<!--.element: class="fragment" -->
- Better UX<!--.element: class="fragment" -->
- Less stress on backend<!--.element: class="fragment" -->

----

```ts
@Injectable()
export class DataService {
    searchData(searchQuery: string): Observable<Data> {
        const url = 'my-data-url/:query'.replace(':query', searchQuery)
        return this.http.get(url)
    }
}
```

Hits the backend everytime, when a part of the application subscribes.<!--.element: class="fragment" -->

---

```ts
@Component({...})
export class SearchComponent {
    ngOnInit() {
        this.results$ = reactiveSearch.valueChanges.pipe(
            switchMap(query => this.dataService.searchData(query))
        )
    }
}
```

Hits the backend on every value change.<!--.element: class="fragment" -->

----

## We can do better!

---

## Debouncing, buffering, filtering & unique values only

----

```ts
this.results$ = rxSearch.valueChanges.pipe(
    debounceTime(400),
    switchMap(query => this.dataService.search(query))
)
```

Buffers for 400ms, and passes the latest values after the debounce finishes.<!--.element: class="fragment" -->

----

```ts
const MINIMUM_LENGTH = 2
this.results$ = rxSearch.valueChanges.pipe(
    filter(query => query.length >= MINIMUM_LENGTH)
)
```

Makes sure no queries with too little characters hits the backend.<!--.element: class="fragment" -->

----

```ts
this.results$ = rxSearch.valueChanges.pipe(
    distinctUntilChanges()
)
```

Only sends the value through the Reactive pipeline when the value has changed.<!--.element: class="fragment" -->

---

## Caching & Memoization

----

```ts
@Injectable()
export class DataService {
    getData(): Observable<Data[]> {
        const url = 'my-data-url/'
        return this.http.get(url)
    }
}
```

Hits the backend everytime, when a part of the application subscribes.<!--.element: class="fragment" -->

----

```ts
private cache: Data[] = []
getData(): Observable<Data[]> {
    if(this.cache) {
        return of(this.cache)
    } else {
        return this.fetchData()
    }
}
```

```ts
private fetchData(): Observable<Data[]> {
    const url = 'my-data-url/'
    return this.http.get(url).pipe(
        tap(data => this.cache = data)
    )
}
```
Very simple caching. But not very Reactive...<!--.element: class="fragment" -->

----

```ts
private cache: Observable<Data[]>

getData() {
    if(!this.cache) {
        this.cache = this.fetchData().pipe(
            shareReplay(1) // Buffersize of 1
        )
    }
    return this.cache
}
```

Better! But the same issue persists. <!--.element: class="fragment" -->

Cache doesn't invalidate.<!--.element: class="fragment" -->

----

Cache should be cleaned up regularly. Many ways you can achieve this! 
But important to keep it simple.

```ts
// Just an example. Effective, but not very reactive.
private cache: { cache: Observable<Data[]>, 
                lastRequest: number, 
                lastCached: number } = {
    cache: of([]),
    lastRequest: 0,
    lastCached: 0
}
```
<!--.element: class="fragment" -->

---

## Exercise 2

Clean up the search so it does not ask the backend as much. Keep it reactive!
`exercises/2_reactive_search.md`

---

## Managing Reactive Forms

----

```html
<button (click)="submitFn()">Save</button>
```

```ts
submitFn() {
    console.log(this.formGroup.value)
}
```

----

### Is that the best we can do?

Unfortunately, so far it's close.<!--.element: class="fragment" -->

<div class="fragment">FormGroup has no `onSubmit` stream.</div>

----

```html
<button #submitBtn type="submit">Save</button>
```

```ts
@ViewChild('submitBtn') submitBtn: ElementRef

// import { fromEvent } from 'rxjs'

fromEvent(this.submitBtn.nativeElement, 'click')
        .subscribe(console.log)
```

Quite nice! But requires an additional ViewChild to be setup.<!--.element: class="fragment" -->

----

FormGroup does offer a `(ngSubmit)="submitFn()"`

```html
<form [formGroup]="formGroup" (ngSubmit)="submitFn()">
    ...
</form>
```

Works similar to the (click)="submitFn()" solution, not really reactive unfortunately...<!--.element: class="fragment" -->

----

In summary:

- You can get a stream from click events.<!--.element: class="fragment" -->
- You will have to manage the subscription yourself though!<!--.element: class="fragment" -->
- Other options are hard to properly make reactive.<!--.element: class="fragment" -->

So choose wisely!<!--.element: class="fragment" -->

---

#### SwitchMap / MergeMap / ConcatMap / ExhaustMap

Note: Talked about this shortly in RxJS 101.

----

### SwitchMap

- Switches to new Observable, maps result.<!--.element: class="fragment" -->
- Cancels previous (in-flight) calls.<!--.element: class="fragment" -->

----

### MergeMap
- Merges in a new Observable, maps result<!--.element: class="fragment" -->
- Keeps running all inner Observables.<!--.element: class="fragment" -->
- Does not care about the order of completing inner Observables.<!--.element: class="fragment" -->
- Used for "write calls", when you do not want to cancel an in-flight call.<!--.element: class="fragment" -->

----

### ConcatMap
- Same as MergeMap, except it does wait for the order of Observables to complete.<!--.element: class="fragment" -->

----

### ExhaustMap
- Waits for the inner Observable to complete, before accepting new inputs.<!--.element: class="fragment" -->
- Basically waits for an in-flight call to finish or error before accepting new inputs.<!--.element: class="fragment" -->

----

In short; when you want to prevent a form from submitting multiple times, `ExhaustMap` is your friend. You will need to use a stream as an input; it will not work as expected if the base (outer) Observable completes.

---

## Exercise 3

We have a reactiveForm that we want to submit.
Make sure it's all hooked up properly and sends off the results to our "backend", without hitting the backend too often.

`exercises/3_send_to_backend.md`

---

## Honorable mention!

```sh
ng add @typebytes/ngx-template-streams
```

```html
<button type="submit" (*click)="clicks$">Submit</button>
```
```ts
@ObservableEvent()
clicks$: Observable<any>;

clicks$.subscribe(console.log)
```

Whilst we wait for new reactive implementations using Ivy<!--.element: class="fragment" -->

---

### Summary

- Be thoughtful of managing your subscriptions.<!--.element: class="fragment" -->
- Let Angular take care of it where you can!<!--.element: class="fragment" -->
- RxJS can help you manage the load on backends<!--.element: class="fragment" -->
- Different operators can help you manage your streams.<!--.element: class="fragment" -->
- Many options in ReactiveForms.<!--.element: class="fragment" -->

---

Thank you! 👏

----

<div style="">
  <img src="assets/bjorn.jpg" width="100" style="border-radius:100%; display: inline-flex;">
  <h1 style="font-size: 0.9em;">Bjorn Schijff</h1>
  <img src="assets/codestar.svg" height="30" style="border: 0; background-color: transparent;"><br /><br />
   <small>@Bjeaurn<br /> bjorn.schijff@ordina.nl</small>
</div>