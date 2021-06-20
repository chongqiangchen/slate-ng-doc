

实现方案介绍 (修改中)

1. 交互逻辑

   <img src="../images/image-20210619163348259.png" alt="image-20210619163348259" style="zoom:50%;" />

2. 方案介绍

   slate-ng大致结构：

   <img src="../images/image-20210619172253182.png" alt="image-20210619172253182" style="zoom:25%;" />

   从图中，我们可以看到slate-ng大概的结构，在slate-ng中将每个Slate Node当做一个组件, 根据一定的要求进行组装成 Component Portal， 并挂载到对应位置进行渲染显示，而在这里面核心点就在children下的逻辑，这里涉及几个点：

    - 如何获取每个node对应的组件？

    - 如何处理每个node对应的组件和其子项 ？

    - 如何传递上下文到每个动态组件中 ？

    - 如何优化处理方案 ？

   那么我们一点一点过一下：

    1. 如何获取每个node对应的组件 ？

       这里将区别于react处理方式，在react中通过renderElement，renderLeaf，renderPlaceholder分别传递到组件内进行处理，但是在slate-ng中我们无法较好的享有这种方式带来的好处，故我简化为如下传递和存储（在slate-ng实现上会有一些区别如下，可查看[registry-ns-element](https://github.com/chongqiangchen/slate-ng/blob/master/projects/slate-ng-view/src/services/registry-ns-element.service.ts)）:

       ```
       // 初始存储的内容
       const defaultStore = {
               default: DefalutElementComponent,
               text: TextComponent,
               leaf: LeafComponent,
               placeholder: PlaceholderComponent
       }
       
       // 新建一个Node对应的Component
       @Component({...})
       export class ComponentA {
               static type = 'A' // 组件的标识
       }
           
       // 使用service进行注册
       this.registryService.add([ComponentA, ComponentB])
       
       // 最终得到的store
       const store = {
               ...defaultStore,
               A: ComponentA
       }
       
       // 获取时直接从对象根据type获取
       ```

    2. 如何处理每个node对应的组件和其子项 ？

       在这里我参考了react use-children的一些处理方式，创建了children组件，该组件用于递归遍历editor的数据组装出一个完整的portals，代码如下（简化后的代码）：

       ```
       
        resolvePortals(pNode: Ancestor): Array<ComponentPortal<any>> {
           const children = [];
           const editor = this.editorService.editor;
           // ...
           for (let i = 0; i < pNode.children.length; i++) {
             const cNode = pNode.children[i] as Ancestor | Descendant;
             // ...
             let childPortals = [];
             let portal = null;
             if ((cNode as Ancestor).children) {
               childPortals = this.resolvePortals(cNode as Ancestor); // 重点
             }
             // 通过Injector存储childProtals
             const providers = [
               {provide: CHILD_PORTALS_TOKEN, useValue: childPortals } 
               // ...
             ];
             // ...
             portal = new ComponentPortal(this.regsitorService.get(type), injector.create(providers, this.injector));
             children.push(portal);
           }
           return children;
         }
         
        // portals
        [
                ComponentPortalA,
                ComponentPortalB,
                ...
        ]
       ```

       可能会觉得奇怪，那`child`的内容在哪，这里涉及到`componentportal injector`知识，前面注解也有提到，也就是说，每个`ComponentPortal`事实上带着一个它的`childPortals`，你可以将产出的`protals`理解为这样的结构（真实结构肯定不这样，但是你可以大致这么理解我们有这么一个大概的结构）：

       ```
       [
            ComponentPortalA: {
                 injector: {
                    child_token: [
                       ComponentPortalA0,
                       ComponentPortalA1
                    ]            
                 }
            },
            ComponentPortalB // 同样
       ]
       ```

       最终我们丢到html中进行处理渲染：

       ```
       <ng-container *ngFor="let portal of showPortals;">
         <ng-template [cdkPortalOutlet]="portal"></ng-template>
       </ng-container>
       ```

    3. 如何传递上下文到每个动态组件中 ？

       事实上我们在第二点中也有涉及，那就是Component允许传递injector，我们将所需要的内容通过`Injector.create(providers, this.parentInjector)`传递

    4. 如何优化处理方案 ？

       在这里有个问题，因为我们每次都会创建一个新的ComponentPortal，但是得注意，假设我们有一个序列，结构如下：

       ```
       {
            type: 'list',
            children: [
              {
                 type: 'li',
                 ...
              }
            ]
       }
       ```

       如果进行一次序列增加操作，它将会在children中增加一个li，而在前面的处理中每次都会重新创建一个新的ComponentPortal，这意味着一个小变更我们都需要完全的重新渲染，这是一个性能消耗非常大的问题，故我们需要做缓存，我们只需要让变更的node重新创建新的portal，其他继续使用之前的，[代码地址](https://github.com/chongqiangchen/slate-ng/blob/015c77ca710ff52dbabcb71374be87c2e394cc13/projects/slate-ng-view/src/components/children/children.component.ts#L175)

