# 自定义样式

在之前的指南中我们学习了怎么创建一个自定义的 block 类型，去渲染不同的文本块。但是 Slate 支持自定义的不仅仅是 "blocks"。

在这篇指南中，我们会像你展示如何添加自定义格式选项，如粗体，那么继续对编辑器应用扩展：

1. 首先我们思考下，若需要变更文字渲染的样式，那么我们需要自定义leaf组件，所以我们先创建新的leaf组件（注意：创建方式与先前codeComponent block节点组件创建方式一致，但是获取的依赖会有所不同，具体可以参考API文档，待完成）
   
    ```
    @Component({
       selector: 'span[custom-leaf]',
       template: `
         <ng-template [cdkPortalOutlet]="leafChild"></ng-template>
       `
    })
    export class LeafComponent extends BaseLeafComponent {
       static type = 'leaf'; // 若需要替换leaf，此type不可变
       
       @HostBinding('style.font-weight') fontWeight = this.leaf.bold ? 'bold' : 'normal';
       
       constructor(
          @Inject(LEAF_CHILD_PORTAL_TOKEN) readonly leafChild: ComponentPortal<any>,
          @Inject(LEAF_TOKEN) readonly leaf: any
       ) {
         super();
       }
    }
    ```
   注意：若是想替换Leaf，就得必须使用`static type = 'leaf'`

2. 导入相关依赖至module中
   ```
   @NgModule({
     declarations: [
       AppComponent,
       CodeComponent,
       LeafComponent <-- 新增
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
3. 将leafComponent注册到slate-ng，并且增加触发逻辑：
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
       this.registryNsElement.add([CodeComponent, LeafComponent]); <-- 更新
     }
   
     onKeyDown = (event: KeyboardEvent) => {
       const editor = this.editor;
       if (!event.ctrlKey) {
         return;
       }
   
       switch (event.key) {  <-- 更新
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

好啦，现在我们可以到页面中选中一块区域，使用`Ctrl + b`查看加粗效果了
