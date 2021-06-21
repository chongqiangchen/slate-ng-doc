# Install Slate-Ng

1. First, please create an `angular10+` project first, and then install related dependencies:
```zsh
  npm install slate slate-ng @angular/cdk
```
2. After installation, you can install the following steps to import

   Import related modules in `app.module.ts`:
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
   Introduce relevant content in `app.component.ts`, and create editor and initial value value
    ```
    // ...
    // Import the factory function of the Slate editor.
    import { createEditor } from 'slate' 
    // Import slate-ng related services and Angular plugins
    import {NsDepsService, NsEditorService, RegistryNsElement, withAngular} from 'slate-ng'; 
    
    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.less'],
      providers: [NsEditorService, NsDepsService, RegistryNsElement] // Inject related services
    })
    export class AppComponent {
      // create editor
      editor = withAngular(createEditor());
      // Initial value
      value = [
        {
          type: 'paragraph',
          children: [{text: ''}]
        }
      ]
    }
    ```
    Build relevant rendered dom content in `app.component.html`
    ```
    <div
       ns-editor
       [editor]="editor"
       [value]="value"
       placeholder="enter some"
    ></div>   
    ```
3. After editing, you can run the project to see the effect~
