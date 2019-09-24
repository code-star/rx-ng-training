# Ideas to work out for a presentation and some exercises

Showing best practices, but also giving you the tools to make the best decisions for your situation.

### Where to subscribe?
`HTML`, `Component`, `Service`

```html
<div *ngIf="data$ | async as data">
 {{ data | json }}
</div>
```

is a pattern I like very much. Feels very clean, especially when no internal component logic is furtherly required.

###  Where to unsubscribe? (Plus the importance of unsubscribing!)

Well, in the HTML variant; this is done for you (woo!). But in more complex situations, which we'll have to cover. You want to rely on the lifecycle hooks by Angular to determine when a stream needs to be destroyed and unsubscribed. Unless you can do it yourself, by limiting the amount of data expected or completing the observable when possible.

- Optimizing for less subscriptions (merging, concatting, switchMapping etc.)

###  Where to assign?
```ts
export class Component {
    data$: Observable<Data[]> = this.dataService.getData()

    constructor(private dataService: DataService) {}
}
```

vs.

```ts
export class Component {
    data$: Observable<Data[]>

    constructor(private dataService: DataService) {
        this.data$ = this.dataService.getData()
    }
}
```

vs.

```ts
export class Component implements OnInit {
    data$: Observable<Data[]>

    constructor(private dataService: DataService) {}

    ngOnInit() {
        this.data$ = this.dataService.getData()
    }
}
```

When to use which? 

The `ngOnInit()` one is nice when you need to pipe the initial data$ into another one, like having to `switchMap()` (if the service can't do that for you).

This will however make the strict compiler complain about `data$` not being set definitively on the `constructor()` or not having an initial value.

Show ways to mitigate these situations, what works best etc.

Same goes for subscribing. 


- Advanced RxJS in Angular (google search)

Prevent hitting the backend too often
- "Search" example
- Memoization/caching in Angular/RxJS

