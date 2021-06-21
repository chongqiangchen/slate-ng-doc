# 创建Slate Ng Plugin

在很多情况下我们会遇见很多node节点的元素是大家都需要的，那么事实上在slate-ng中我们是可以把这个节点组件提取出来变为一个库发布供大家使用，
接下来我将会以[slate-ng-element-table](https://www.npmjs.com/package/slate-ng-element-table)为参照，教大家如何创建一个Slate Ng Plugin(事实上这个一个用Angular Cli创建一个Library的教程)：

1. 首先我们需要一个Angular工作区，可参考[官方](https://angular.io/guide/creating-libraries)
    ```shell
    ng new slate-ng-plugin-workspace --create-application=false
    cd slate-ng-plugin-workspace
    ng generate library slate-ng-element-table 
    ```
2. 创建完后，我会得到如下目录结构：
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
3. 更改package.json依赖，并安装：
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
4. 删除`lib`文件夹,清理`public-api.ts`内容，并新增或更新以下文件：

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
   // 导出
   export * from './ns-element-table.module';
   export * from './table';
   export * from './table-row';
   export * from './table-cell';
   export * from './with-table';
   ```

5. 打包并发布：
```
  ng build --prod
  cd dist/slate-ng-element-table
  npm publish
```

6. 前面已经有了一个插件，那么如何使用呢？具体参考[使用方式](https://github.com/chongqiangchen/slate-ng/blob/master/projects/slate-ng-element-table/README.md)
   
   我们只需要在module中导入`ns-element-table.module`，然后使用`withTables`包裹editor，并进行注册：`this.registryNsElement.add([SlateNgElementTableRow, SlateNgElementTableCell, SlateNgElementTable,]);`，创建对应的value值：
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
最终你便可以看到一个table。
