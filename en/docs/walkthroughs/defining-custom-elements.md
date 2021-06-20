# Defining Custom Elements

In the previous example, we all use the default built-in `block` node element, such as `paragraph` using `<div>`, in fact, we can define any type of custom `block` node,
Such as block references, code blocks, list items, etc.

Next we show how to customize code block nodes in `slate-ng`:

1. First Create Code Component
    ```
    import {Component, ElementRef, Inject} from '@angular/core';
    import {BaseElementComponent, Key, KEY_TOKEN, NsDepsService, NsEditorService} from 'slate-ng';
    
    @Component({
        selector: 'pre[custom-code]',
        template: `
           <code>
            <ng-container *ngFor="let portal of portals; trackBy: trackBy">
                <ng-template [cdkPortalOutlet]="portal"></ng-template>
            </ng-container>
           </code>
        `
    })
    export class CodeComponent extends BaseElementComponent {
        static type = 'code'; // 注意，这非常重要，必须存在，不然会造成这个Component无法被应用
        
        constructor(
            @Inject(KEY_TOKEN) readonly key: Key,
            public deps: NsDepsService,
            public editorService: NsEditorService,
            public elementRef: ElementRef,
        ) {
            super(key, deps, editorService, elementRef);
        }
    }
    ```
    
   %accordion%**Analyze:**：Here is an explanation of the above code：%accordion%
      - **Code Block 1**：
           ```
           <ng-container *ngFor="let portal of portals; trackBy: trackBy">
              <ng-template [cdkPortalOutlet]="portal"></ng-template>
           </ng-container>
           ```
           This is to render `childPortals`, that is, child elements. Of course, you can directly use the `ns-portal-child` component provided in `slate-ng` instead:
           ```
           <code>
             <ns-portal-entry [portals]="portals"></ns-portal-entry>
           </code>
           ```
        However, the use of `ns-portal-entry` is not recommended. The main reason is that using this component to render sub-elements will add a layer of dom element wrapping inside.
        Although this does not affect the operation of the editor, it will deepen a layer of internal element nodes, so it is not recommended to use it in complex node components.
      - **Code Block 2**：
        ```
        export class CodeComponent extends BaseElementComponent
        ```
        This inherits `BaseElementComponent`, which helps us to do a certain amount of `attr` increase and related content initialization, 
        such as the previous use of `portals`, this is also handled by `BaseElement` to export, and for this reason, we should also pay attention We need the corresponding dependencies of `super`, you can check xxx for details[BaseElementComponent](https://github.com/chongqiangchen/slate-ng/blob/015c77ca710ff52dbabcb71374be87c2e394cc13/projects/slate-ng-view/src/components/element/base-element.ts#L25)
      - **Code Block 3**:
        ```
        constructor(
           @Inject(KEY_TOKEN) readonly key: Key,
           public deps: NsDepsService,
           public editorService: NsEditorService,
           public elementRef: ElementRef,
        ){//...}
        ```
        Here is the node information that we can obtain. First, each node element will have its own `key` as an identifier, which we obtain through `@Inject(KEY_TOKEN)`.
        It can be combined with `NsDepsService` to get all the dependencies required by the current component. For specific dependencies, please refer to[token](https://github.com/chongqiangchen/slate-ng/blob/015c77ca710ff52dbabcb71374be87c2e394cc13/projects/slate-ng-view/src/components/element/token.ts#L4)（注意：token还没合理的划分，所以并非全部的token在element中都可被获取，API文档将会给出具体，待完成）
      - **Code Block 4**
        `static type = 'code';`
        **Note!** This is very important and is mainly used to mark which node type the Component belongs to.
   %/accordion%
   
2. Import related dependencies into Module
   ```
   @NgModule({
      declarations: [
         AppComponent,
         CodeComponent // <-- add
      ],
      imports: [
         BrowserModule,
         AppRoutingModule,
         SlateNgModule,
         PortalModule // <-- add
      ],
      bootstrap: [AppComponent]
   })
   export class AppModule {
   }
   ```
   Note that because we will use `Angular CDK Portal`, we must inject `PortalModule`, and secondly, because we will use `<ng-template>` template syntax, `CodeComponent` also needs to be registered in `module`
3. The last step is to use `RegistryNsElement` in `app.component.ts` to register to `slate-ng` and add trigger conditions
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
         private registryNsElement: RegistryNsElement <-- add
      ) {
      }
      
      ngOnInit() {
         this.registryNsElement.add([CodeComponent]); <-- add
      }
      
      onKeyDown = (event: KeyboardEvent) => { <-- update
         if (event.key === '`' && event.ctrlKey) {
            event.preventDefault();
            // Otherwise, set the type of the currently selected blocks to "code"
            Transforms.setNodes(
               this.editor,
               { type: 'code' } as any,
               { match: n => Editor.isBlock(this.editor, n) }
            );
         }
      }
    }

   ```

Ok, let's open the page again, select part of the content, press Ctrl + \` to see the related styles~
