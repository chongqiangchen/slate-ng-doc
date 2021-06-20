# 自定义节点

在之前的例子中，我们都是使用默认内置的block节点元素，如`paragraph`使用`<div>`，事实上我们是可以定义任何类型的自定义 block 节点，
比如块引用，代码块，列表项等。

接下来我们展示slate-ng中是如何进行自定义代码块节点的
1. 首先创建代码块Component
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
    
   %accordion%**解析**：在这里对上面的代码做一定解释：%accordion%
      - **代码块1**：
           ```
           <ng-container *ngFor="let portal of portals; trackBy: trackBy">
              <ng-template [cdkPortalOutlet]="portal"></ng-template>
           </ng-container>
           ```
        这是是将childPortals，也就是子元素进行渲染，当然你可以直接使用slate-ng中提供的ns-portal-child组件进行替代：
           ```
           <code>
             <ns-portal-entry [portals]="portals"></ns-portal-entry>
           </code>
           ```
        至于为什么例子中没用使用ns-portal-entry，主要原因是若是使用该组件进行渲染子元素，会给内部多增加一层dom元素包裹，
        这虽然不影响编辑器运行，但是会加深一层内部的元素节点，**故不推荐在复杂节点组件中进行使用**。
      - **代码块2**：
        ```
        export class CodeComponent extends BaseElementComponent
        ```
        这里继承了BaseElementComponent，里面帮我们做了一定的attr增加和相关内容的初始化，如前面使用的`portals`，这个也是BaseElement处理好导出的,
        正因如此，我们也要注意我们需要super相应依赖，具体可以查看[BaseElementComponent](https://github.com/chongqiangchen/slate-ng/blob/015c77ca710ff52dbabcb71374be87c2e394cc13/projects/slate-ng-view/src/components/element/base-element.ts#L25)
      - **代码块3**:
        ```
        constructor(
           @Inject(KEY_TOKEN) readonly key: Key,
           public deps: NsDepsService,
           public editorService: NsEditorService,
           public elementRef: ElementRef,
        ){//...}
        ```
        这里是我们可获知的节点信息，首先每个节点元素都会有一个自己的key作为标识，我们通过`@Inject(KEY_TOKEN)`获取。
        其结合`NsDepsService`可以获取到当前组件所需要的所有依赖，具体存在的依赖可参考[token](https://github.com/chongqiangchen/slate-ng/blob/015c77ca710ff52dbabcb71374be87c2e394cc13/projects/slate-ng-view/src/components/element/token.ts#L4)（注意：token还没合理的划分，所以并非全部的token在element中都可被获取，API文档将会给出具体，待完成）
      - **代码块4**
        `static type = 'code';`
        注意！这个非常重要，主要用于标记该Component属于什么节点类型。
   %/accordion%
   
2. 导入相关的依赖到Module中
   ```
   @NgModule({
      declarations: [
         AppComponent,
         CodeComponent // <-- 新增
      ],
      imports: [
         BrowserModule,
         AppRoutingModule,
         SlateNgModule,
         PortalModule // <-- 新增
      ],
      bootstrap: [AppComponent]
   })
   export class AppModule {
   }
   ```
   注意，因为我们会使用到Angular CDK Portal,必须注入`PortalModule`，其次由于会使用到<ng-template>模板语法，故CodeComponent也需要在module中注册
3. 最后一步，在`app.component.ts`中使用`RegistryNsElement`注册到slate-ng中，并增加触发条件
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
         private registryNsElement: RegistryNsElement <-- 新增
      ) {
      }
      
      ngOnInit() {
         this.registryNsElement.add([CodeComponent]); <-- 新增
      }
      
      onKeyDown = (event: KeyboardEvent) => { <-- 更新
         if (event.key === '`' && event.ctrlKey) {
            event.preventDefault();
            // 否则，把当前选择的 blocks 的类型设为 "code"。
            Transforms.setNodes(
               this.editor,
               { type: 'code' } as any,
               { match: n => Editor.isBlock(this.editor, n) }
            );
         }
      }
    }

   ```

好了，我们再次打开页面，选中一部分内容，按下Ctrl + \`即可看到相关的样式~
