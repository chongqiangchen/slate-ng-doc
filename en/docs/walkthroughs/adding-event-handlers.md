# Adding Event Handler

Now that you have seen the rendered look on the page, this is very simple, you may need to do more instead of just input some plain text.

We can use the mechanism of `slate` for customization. The next thing we need to do is to use the `onKeyDown` event to change the content of the editor when the key is pressed.

1. First we need to add the corresponding `onKeyDown` event
    ```
    // html
    <div
        ns-editor
        [editor]="editor"
        [value]="value"
        [nsOnKeyDown]="onKeyDown"  // <-- add
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
        
        onKeyDown = (e: KeyboardEvent) => { <-- add
            console.log(e.key);
        }
    }
    ```
   Very good, its corresponding `keyCode` will be printed to the console.
2. Now we let it actually change the content. As far as our example is concerned, let's realize that all `&` is converted to `and` when inputting. It might be written like this:
   ```
   // component.ts
   // ... Omit the others, only change the `onKeyDown` method
   onKeyDown = (e: KeyboardEvent) => {
    if (e.key === '&') {
      // The default event that prevents the insertion of the `&` character.
      e.preventDefault();
      // Execute the insertText method to insert some text.
      this.editor.insertText('and');
    }
   }
   ```
   After the addition is complete, try to enter & on the page, you will find that it suddenly becomes an insert and!
