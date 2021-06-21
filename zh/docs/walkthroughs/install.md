# 安装Slate-Ng

1. 首先请先创建一个angular10+项目，然后安装相关依赖：
```zsh
  npm install slate slate-ng @angular/cdk
```
2. 安装完后你可以安装下面的步骤进行导入
    
    在`app.module.ts`中导入相关模块：
    ```
    import {SlateNgModule} from 'slate-ng';
    
    @NgModule({
      // ...
      imports: [
        // ...
        SlateNgModule
      ],
      // ...
    })
    export class AppModule {
    }
    ```
    在`app.component.ts`中引入相关内容，并创建editor和初始值value
    ```
      // ...
    import { createEditor } from 'slate' // 导入 Slate 编辑器的工厂函数。
    import {NsDepsService, NsEditorService, RegistryNsElement, withAngular} from 'slate-ng'; // 导入 slate-ng 相关服务和 Angular 插件
    
    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.less'],
      providers: [NsEditorService, NsDepsService, RegistryNsElement] // 注入相关服务
    })
    export class AppComponent {
      // 创建 editor
      editor = withAngular(createEditor());
      // 初始化值
      value = [
        {
          type: 'paragraph',
          children: [{text: ''}]
        }
      ]
    }
    ```
    在`app.component.html`构建相关渲染的dom内容
    ```
    <div
       ns-editor
       [editor]="editor"
       [value]="value"
       placeholder="enter some"
    ></div>   
    ```
3. 编辑完成后即可运行项目查看效果~
