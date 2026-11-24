---
name: frontend-primeng
description: Angular 17+ / PrimeNG patterns - p-table lazy loading, p-popover, p-message, portal component styling, i18n keys, template escaping. Use when editing Angular templates, PrimeNG components or translation files.
---

# Frontend Patterns (Angular / PrimeNG)

## Disclaimers and notices
- Use `<p-message severity="info" icon="pi pi-info-circle">` for disclaimers; match the project's existing style first.
- For hover-revealed details use `p-popover` (not `pTooltip`). `pTooltip` is fine only for short single-line hints.

## Translation keys
- Keep i18n values plain text: no inline `style=""`, no HTML in JSON strings.
- Split content into semantic keys and compose structure in the HTML template.

## Angular 17+ templates
- `@` must be escaped as `&#64;` in `.html` templates. In JSON translation values it stays `@`.

## PrimeNG p-message
Content is a flex row by default. For multi-line content:
```scss
::ng-deep .my-class .p-message-text {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
```

## Portal components (Popover, Dialog, Toast)
They render under `<body>`, so component `::ng-deep` does not reach them. Set `styleClass` and put the override in `public/scss/_shared.scss`, targeting the class directly:
```scss
.my-popover.p-popover {
  border-radius: 16px;
  overflow: hidden; // required for border-radius to clip content
}
```

## Preferred p-popover pattern
Show on hover, keep open so users can interact, close on outside click:
```html
<div (mouseenter)="pop.show($event)" (click)="pop.toggle($event)">
  <!-- trigger content -->
</div>
<p-popover #pop styleClass="my-popover">
  <!-- popover content -->
</p-popover>
```
Do NOT use `(mouseleave)="pop.hide()"`. Default `dismissable` handles outside click.

## p-table lazy loading: no duplicate API calls
Golden rule: `[lazy]="true"` + `(onLazyLoad)` is the single source of truth for loading.

Never alongside onLazyLoad:
- `(onPage)="pageChange($event)"` (double call)
- `[first]="first"` (`Table.ngOnChanges` emits `onLazyLoad` when `first` changes and lazy)
- `this.dt.first = 0` via `@ViewChild` (triggers `ngOnChanges`, second `onLazyLoad`)

Correct template:
```html
<p-table [lazy]="true" (onLazyLoad)="loadData($event)"
  [rows]="rows" [totalRecords]="totalRecords" [paginator]="true"
  [rowsPerPageOptions]="[5, 10, 20]">
```
No `[first]`, no `(onPage)`, no `#dt` ViewChild.

Search: Subject + debounce, not per keystroke:
```typescript
private readonly searchSubject = new Subject<string>();

ngOnInit(): void {
  this.searchSubject.pipe(debounceTime(300)).subscribe(value => {
    this.filter.searchWord = value;
    this.loadData({ first: 0, rows: this.rows });
  });
}

onSearch(event: KeyboardEvent): void {
  this.searchSubject.next((event.target as HTMLInputElement).value);
}
```
Add `switchMap` after `debounceTime` for race-sensitive APIs.

Signal-based refresh without firing on init:
```typescript
toObservable(this.refreshToken).pipe(skip(1)).subscribe(() => {
  this.loadData({ first: 0, rows: this.rows });
});
```

effect(): never call `loadData()` unguarded (fires on init and every change). Use `toObservable(this.mySignal).pipe(skip(1)).subscribe(() => this.loadData())`.

Responsibility map:
| Trigger | Handler |
|---|---|
| Initial load, page/rows change | `onLazyLoad` only |
| Search input | `Subject` + `debounceTime(300)` |
| Filter reset / apply | direct `loadData()` call |
| Post-mutation refresh | signal + `toObservable().pipe(skip(1))` |
