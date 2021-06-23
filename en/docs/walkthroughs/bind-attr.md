# Bind Attr

In fact, the creation of the various custom components in the front is in fact automatically bound to the `host` with the help of `BaseElement`, `BasePlaceholder`, and `BaseLeaf`.
Therefore, users do not need to deal with it, but not all scenarios need to be bound to `host`, so we need to use `directive`: `element-attrs`, `leaf-attrs`, `placeholder-attrs` to bind to Designated location,

Next we show the use of `element-attrs.directive`:

1. Suppose we need a `Table` component, I expect to bind `element attr` to `tbody` instead of `table`/`host`, we can do this
    ```
    // Use nsElementAttrs on tbody
    <tbody nsElementAttrs>
      <ng-container *ngFor="let portal of portals; trackBy: trackBy">
        <ng-template [cdkPortalOutlet]="portal"></ng-template>
      </ng-container>
    </tbody>
    
    // Override `useHostAttrs` function
    useHostAttrs() {
       return false;
    }
    ```
   In this way, the element attrs can be placed in the specified position
