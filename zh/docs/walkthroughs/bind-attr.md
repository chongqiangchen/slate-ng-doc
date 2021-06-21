# 绑定相关attr至指定元素

前面各个自定义组件的创建事实上都在`BaseElement`,`BasePlaceholder`,`BaseLeaf`帮助下将需要的`attr`标识自动绑定到了`host`上，
故用户不需要处理，但是并非所有的场景都需要绑定到`host`上，所以我们需要使用到`directive`: `element-attrs`, `leaf-attrs`, `placeholder-attrs`绑定到指定的位置，

接下来我们展示`element-attrs.directive`的使用：

1. 假定我们需要一个Table组件，我期望将element attr绑定在tbody上而不是table/host上，故我们可以这样做
    ```
    // 在tbody上使用nsElementAttrs
    <tbody nsElementAttrs>
      <ng-container *ngFor="let portal of portals; trackBy: trackBy">
        <ng-template [cdkPortalOutlet]="portal"></ng-template>
      </ng-container>
    </tbody>
    
    // 覆写ngOnInit，调用init方法传入参数useHostAttrs: false
    ngOnInit() {
      this.init({ useHostAttrs: false }); 
    }
    ```
    这样即可做到将element attrs放置到指定位置
