<img src="assets/rx-ng.png"/>

### Managing RxJS streams in Angular <!--.element: class="fragment" -->

---

<div style="">
  <img src="assets/bjorn.jpg" width="100" style="border-radius:100%; display: inline-flex;">
  <h1 style="font-size: 0.9em;">Bjorn Schijff</h1>
  <small style="display: inline-flex;">Frontend Software Engineer @ BliepBloep</small><br />
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
    data: DataObject
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

----

```ts
```

---

### Goal for today

- Get familiar with multiple ways of managing your streams <!--.element: class="fragment" -->
- Know when to pick which method <!--.element: class="fragment" -->

---

# 