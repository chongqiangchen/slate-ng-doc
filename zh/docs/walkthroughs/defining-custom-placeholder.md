# 自定义Placeholder

在前面的例子中，我们演示了如何自定义节点，自定义样式，接下来将会带大家自定义placeholder：

1. 首先创建自定义placeholder组件:
    ```
    import { Component, Inject, Injector } from '@angular/core';
    import { BasePlaceholderComponent, PLACEHOLDER_CHILDREN_TOKEN } from 'slate-ng';

    @Component(
        {
            selector: 'div[custom-placeholder]',
            template: `
                <p style="margin: 0">{{children}}</p>
                <pre>Custom placeholder</pre>
            `
        }
    )
    export class CustomPlaceholder extends BasePlaceholderComponent {
        static type = 'placeholder'; // 注意此类型必须为placeholder
        
        constructor(
            @Inject(PLACEHOLDER_CHILDREN_TOKEN) readonly children: any
        ) {
            super();
        }
    }

    ```
   **注意：** 此处children就是一个字符串，并非portal，故可以直接使用。
2. 在`app.component.ts`中使用`RegistryNsElement`注册到slate-ng中，并增加触发条件
   ```
   // 其他保持不变，仅更新registryNsElement
   this.registryNsElement.add([
         CodeComponent,
         LeafComponent,
         CustomPlaceholder
       ]);
   ```
   
## 完整代码：
```
// app.component.ts

import {Component, OnInit} from '@angular/core';
import {NsDepsService, NsEditorService, RegistryNsElement, withAngular} from 'slate-ng';
import {createEditor, Editor, Text, Transforms} from 'slate';
import {withHistory} from 'slate-history';
import {CodeComponent} from './code.component';
import {LeafComponent} from './leaf.component';
import {CustomPlaceholder} from './placeholder.component';

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
    this.registryNsElement.add([
      CodeComponent,
      LeafComponent,
      CustomPlaceholder
    ]);
  }

  onKeyDown = (event: KeyboardEvent) => {
    const editor = this.editor;
    if (!event.ctrlKey) {
      return;
    }

    switch (event.key) {
      case '`': {
        event.preventDefault();
        const [match] = Editor.nodes(editor, {
          match: (n: any) => n.type === 'code',
        });
        Transforms.setNodes(
          editor,
          {type: match ? null : 'code'} as any,
          {match: n => Editor.isBlock(editor, n)}
        );
        break;
      }

      case 'b': {
        event.preventDefault();
        Transforms.setNodes(
          editor,
          {bold: true} as any,
          {match: n => Text.isText(n), split: true}
        );
        break;
      }
    }
  };
}

```
好的，现在你可以看到变更后的placeholder了。
