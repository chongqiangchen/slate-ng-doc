# Defining Custom Placeholder

In the previous example, we demonstrated how to customize the node, customize the style, and then we will take you to customize the placeholder:

1. First create a custom placeholder component:
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
        static type = 'placeholder'; // Note that this type must be a `placeholder`
        
        constructor(
            @Inject(PLACEHOLDER_CHILDREN_TOKEN) readonly children: any
        ) {
            super();
        }
    }

    ```
   **Note：** Here children is a string, not a portal, so it can be used directly.
2. Use `RegistryNsElement` in `app.component.ts` to register to slate-ng
   ```
   this.registryNsElement.add([
         CodeComponent,
         LeafComponent,
         CustomPlaceholder
       ]);
   ```

## Complete code：
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
Okay, now you can see the changed placeholder.
