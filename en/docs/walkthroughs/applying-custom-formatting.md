# Applying Custom Formatting

In the previous guide, we learned how to create a custom `block` type to render different text blocks. But `Slate` supports more than just `blocks` for customization.

In this guide, we will show you how to add custom formatting options, such as bold, then continue to apply extensions to the editor:

1. First, let’s think about it. If we need to change the style of text rendering, then we need to customize the `leaf` component, so we first create a new `leaf` component (note: the creation method is the same as the previous `codeComponent block` node component creation method, However, the acquired dependencies will be different, you can refer to the API documentation for details)
   
    ```
    @Component({
       selector: 'span[custom-leaf]',
       template: `
         <ng-template [cdkPortalOutlet]="leafChild"></ng-template>
       `
    })
    export class LeafComponent extends BaseLeafComponent {
       static type = 'leaf'; // If you need to replace `leaf`, this `type` is immutable
       
       @HostBinding('style.font-weight') fontWeight = this.leaf.bold ? 'bold' : 'normal';
       
       constructor(
          @Inject(LEAF_CHILD_PORTAL_TOKEN) readonly leafChild: ComponentPortal<any>,
          @Inject(LEAF_TOKEN) readonly leaf: any
       ) {
         super();
       }
    }
    ```
    Note: If you want to replace Leaf, you must use `static type ='leaf'`

2. Import related dependencies into the module
   ```
   @NgModule({
     declarations: [
       AppComponent,
       CodeComponent,
       LeafComponent <-- add
     ],
     imports: [
       BrowserModule,
       AppRoutingModule,
       SlateNgModule,
       PortalModule
     ],
     bootstrap: [AppComponent]
   })
   export class AppModule {
   }
   ```
3. Register `leafComponent` to `slate-ng` and add trigger logic:
   ```
   @Component({
     selector: 'app-root',
     templateUrl: './app.component.html',
     styleUrls: ['./app.component.less'],
     providers: [NsEditorService, NsDepsService, RegistryNsElement]
   })
   export class AppComponent implements OnInit {
     title = 'angular-slate-demo';
     editor = withHistory(withAngular(createEditor()));
     value = [
       {
         type: 'paragraph',
         children: [{text: ''}]
       }
     ];
   
     constructor(
       private registryNsElement: RegistryNsElement
     ) {
     }
   
     ngOnInit() {
       this.registryNsElement.add([CodeComponent, LeafComponent]); <-- add
     }
   
     onKeyDown = (event: KeyboardEvent) => {
       const editor = this.editor;
       if (!event.ctrlKey) {
         return;
       }
   
       switch (event.key) {  <-- update
         case '`': {
           event.preventDefault();
           const [match] = Editor.nodes(editor, {
             match: (n: any) => n.type === 'code',
           });
           Transforms.setNodes(
             editor,
             { type: match ? null : 'code' } as any,
             { match: n => Editor.isBlock(editor, n) }
           );
           break;
         }
   
         case 'b': {
           event.preventDefault();
           Transforms.setNodes(
             editor,
             { bold: true } as any,
             { match: n => Text.isText(n), split: true }
           );
           break;
         }
       }
     }
   }
   ```

Alright, now we can select an area on the page and use `Ctrl + b` to view the bolding effect
