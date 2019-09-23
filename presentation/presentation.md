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

Exercise time!

`exercises/1_cleaning_up_streams.md`

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

----

```ts
user$: Observable<User>
news$: Observable<NewsPost[]>
votes$: Observale<VotesPerNewsPost[]>
comments$: Observable<Comments[]>
```
<small>Gets more interesting in this situation, as your HTML will start to get messy.
You may want to setup some logic to combine data before it hits the HTML.</small>

----

So let's assume `votes$` and `comments$` are on a timer, or even live; and update more frequently.

For simplicity sake: 
- `votes$` takes an array of ID's you want the updates for. 
- `comments$` can show you if it has unread messages for an array of ID's.

----

Exercise 2!

`exercises/2_grouping_streams.md`

---

### Optimizing for less subscriptions

- Combining relevant data into a single subscription.
- Naturally splitting data streams into separate components.

----

// TODO: Find an example!

---


