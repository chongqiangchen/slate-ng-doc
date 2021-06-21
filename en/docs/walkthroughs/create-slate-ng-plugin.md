# Create Slate Ng Plugin

In many cases, we will encounter a lot of node node elements that everyone needs. In fact, in slate-ng, we can extract this node component into a library for everyone to use.
Next I will use [slate-ng-element-table](https://www.npmjs.com/package/slate-ng-element-table) as a reference to teach you how to create a Slate Ng Plugin (in fact This is a tutorial to create a Library with Angular Cli):

1. First we need an Angular workspace, you can refer to [Angular](https://angular.io/guide/creating-libraries)
    ```shell
    ng new slate-ng-plugin-workspace --create-application=false
    cd slate-ng-plugin-workspace
    ng generate library slate-ng-element-table 
    ```
2. After creation, I will get the following directory structure:
```
├── README.md
├── angular.json
├── node_modules
├── package-lock.json
├── package.json
├── projects
│  └── slate-ng-element-table
│  ├── README.md
│  ├── karma.conf.js
│  ├── ng-package.json
│  ├── package.json
│  ├── src
│  │   ├── lib
│  │   ├── public-api.ts
│  │   └── test.ts
│  ├── tsconfig.lib.json
│  ├── tsconfig.lib.prod.json
│  ├── tsconfig.spec.json
│  └── tslint.json
├── tsconfig.json
└── tslint.json
```
3. Change package.json dependencies and install:
```json
{
  "name": "<u package name>",
  "version": "0.0.1",
  "peerDependencies": {
    "@angular/common": "^10.2.4",
    "@angular/core": "^10.2.4",
    "@angular/cdk": "^10.2.4",
    "slate-ng": "~0.0.2",
    "slate": ">=0.55.0"
  },
  "dependencies": {
    "tslib": "^2.0.0"
  },
   "devDependencies": {
      "slate": "^0.63.0",
      "@angular/cdk": "^10.2.4",
      "slate-ng": "~0.0.2"
   }
}
```
4. Delete the `lib` folder, clean up the contents of `public-api.ts`, and add or update the following files:

   - **index.ts**
   ```
   export * from './public-api';
   ```
   - [**ns-element-table.module.ts**](https://github.com/chongqiangchen/slate-ng/blob/master/projects/slate-ng-element-table/src/ns-element-table.module.ts)
   
   - [**table.ts**](https://github.com/chongqiangchen/slate-ng/blob/3ed150c70214f51c17c569202e9a3dc350c817f5/projects/slate-ng-element-table/src/table.ts#L25)
   
   - [**table-row.ts**](https://github.com/chongqiangchen/slate-ng/blob/master/projects/slate-ng-element-table/src/table-row.ts)
   
   - [**table-cell.ts**](https://github.com/chongqiangchen/slate-ng/blob/master/projects/slate-ng-element-table/src/table-cell.ts)
   
   - [**with-table.ts**](https://github.com/chongqiangchen/slate-ng/blob/master/projects/slate-ng-element-table/src/with-table.ts)
   
   - [**table.css**](https://github.com/chongqiangchen/slate-ng/blob/master/projects/slate-ng-element-table/src/table.css)
   
   - **public-api.ts**
   ```
   // export some files
   export * from './ns-element-table.module';
   export * from './table';
   export * from './table-row';
   export * from './table-cell';
   export * from './with-table';
   ```

5. Package and publish:
```
  ng build --prod
  cd dist/slate-ng-element-table
  npm publish
```

6. There is already a plug-in before, so how to use it? Specific reference [How to use](https://github.com/chongqiangchen/slate-ng/blob/master/projects/slate-ng-element-table/README.md)

   We only need to import in the module `ns-element-table.module`，Then use `withTables` wrap `editor`，then registry：`this.registryNsElement.add([SlateNgElementTableRow, SlateNgElementTableCell, SlateNgElementTable,]);`，创建对应的value值：
   ```
   {
         type: 'table',
         children: [
           {
             type: 'table-row',
             children: [
               {
                 type: 'table-cell',
                 children: [{text: ''}]
               },
               {
                 type: 'table-cell',
                 children: [{text: 'Human', bold: true}]
               },
               {
                 type: 'table-cell',
                 children: [{text: 'Dog', bold: true}]
               },
               {
                 type: 'table-cell',
                 children: [{text: 'Cat', bold: true}]
               }
             ]
           },
           {
             type: 'table-row',
             children: [
               {
                 type: 'table-cell',
                 children: [{text: '# of Feet', bold: true}]
               },
               {
                 type: 'table-cell',
                 children: [{text: '2'}]
               },
               {
                 type: 'table-cell',
                 children: [{text: '4'}]
               },
               {
                 type: 'table-cell',
                 children: [{text: '4'}]
               }
             ]
           },
           {
             type: 'table-row',
             children: [
               {
                 type: 'table-cell',
                 children: [{text: '# of Lives', bold: true}]
               },
               {
                 type: 'table-cell',
                 children: [{text: '1'}]
               },
               {
                 type: 'table-cell',
                 children: [{text: '1'}]
               },
               {
                 type: 'table-cell',
                 children: [{text: '9'}]
               }
             ]
           }
         ]
     }   
   ```
Finally, you can see a table.
