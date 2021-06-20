# 添加事件处理器

现在你已经在页面中看到了渲染后的样子，这个还非常的简单，可能你会需要做更多而不是仅仅输入一些纯文本。

我们可以利用slate的机制进行定制，接下来我们需要做的一个事情就是，按下按键时候，让我们使用 onKeyDown 事件去改变编辑器的内容。

1. 首先我们需要下增加对应的onKeyDown事件
    ```
    // html
    <div
        ns-editor
        [editor]="editor"
        [value]="value"
        [nsOnKeyDown]="onKeyDown"  // <-- 新增
        placeholder="enter some"
    ></div>   
   
    // component.ts 
    
    @Component({
        selector: 'app-root',
        templateUrl: './app.component.html',
        styleUrls: ['./app.component.less'],
        providers: [NsEditorService, NsDepsService, RegistryNsElement]
    })
    export class AppComponent {
        title = 'angular-slate-demo';
        editor = withHistory(withAngular(createEditor()));
        value = [
            {
                type: 'paragraph',
                children: [{text: ''}]
            }
        ];
        
        onKeyDown = (e: KeyboardEvent) => { <-- 新增
            console.log(e.key);
        }
    }
    ```
   很好，它相应的 keyCode 会被打印到控制台中。
2. 现在我们让它实际上去改变内容。就我们的示例而言，让我们实现在输入时将所有的 & 转换为 and。它可能会这样写：
   ```
   // component.ts
   // ... 省略其他，仅更改onKeyDown方法
   onKeyDown = (e: KeyboardEvent) => {
    if (e.key === '&') {
      // 阻止插入 `&` 字符的默认事件。
      e.preventDefault();
      // 执行insertText方法插入某些文本。
      this.editor.insertText('and');
    }
   }
   ```
   添加完成后，到页面中试着输入 & ，你会发现它突然变成了插入 and ！
