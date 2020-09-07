---
title: 前端代码生成 · TS加持增强版
date: 2020-02-12 23:18:25
tags: [ts, codegen]
---

![codegen-cover.png](https://user-images.githubusercontent.com/10891613/92222651-e96c8400-eed1-11ea-86e7-12a70741e19e.png)
## 为什么要做代码生成？
几乎每个前端项目里大概都是这样的流程，通过模板初始化前端工程，后端同学提供了接口文档后，前端同学就要对照着文档，先是给接口起名字，写上 接口的 path、http method、params、data，勤快的同学会把接口的入参作为函数的注释，就这样完成了一个接口的定义，再看文档，还有几十个接口要这样写，接着 copy，paste，改名字，改path，改method，改参数，可能稍微不注意就改错了某个参数，接着在联调时又浪费不必要的时间。

## 代码生成会带给哪些优势？

- 免去手写大量复制粘贴的代码，接入pipeline，实现自动化
- 具体的方法提示，省去在 vue 组件和 api 文件直接来回切换
- TypeScript 加持，使用 VSCode 类型推导，方便查看参数类型
- 直达 catalog，准确定位 function 位置

## 一、环境搭建
repo 地址
```bash
git clone git@github.com:OpenAPITools/openapi-generator.git
```
项目环境必备条件

- 安装 Java 8
- 安装 maven
```bash
brew install maven
```
以上安装成功后，即可进行项目构建
```bash
mvn clean install
```
配置 maven 私有源移步至 Maven 配置，如果是通过 brew 安装，默认目录为 /usr/local/Cellar/maven/3.x.x/libexec/conf/settings.xml  

## 二、定制化开发
针对前端代码生产需要修改的文件及目录包括
Java 文件：modules/openapi-generator/src/main/java/org/openapitools/codegen/languages/TypeScriptAxiosClientCodegen.java
模板目录：modules/openapi-generator/src/main/resources/typescript-axios/
修改后需要重新构建

```bash
mvn clean package -Dmaven.test.skip=true
```

## 三、代码生成
项目构建完成后，通过以下命令生成代码，其中 typescript-axios-target-es6.json 为配置文件内容
```javascript
{
  "npmRepository": "http://private.npm.com/repository",
  "snapshot": false,
  "supportsES6": true
}
```

```bash
java -jar ./modules/openapi-generator-cli/target/openapi-generator-cli.jar generate -i ~/some-project.yaml \
	-g typescript-axios \
	-c typescript-axios-target-es6.json \
	--group-id groupId \
	--artifact-id some-project \
	--skip-validate-spec \
	-o ./some-project-api
```
> -g 生成模式
> -c 生成所需配置文件
> --group-id 传入 groupId
> --artifact-id 传入 artifactId
> -o 输出项目
> --skip-validate-spec 跳过校验
> -DdebugModels  debug 模式，可选


执行后生成的前端项目结构
```bash
.
├── README.md
├── api.ts
├── base.ts
├── configuration.ts
├── dist
│   ├── api.d.ts
│   ├── api.js
│   ├── base.d.ts
│   ├── base.js
│   ├── configuration.d.ts
│   ├── configuration.js
│   ├── index.d.ts
│   └── index.js
├── git_push.sh
├── index.ts
├── package-lock.json
├── package.json
└── tsconfig.json

1 directory, 17 files
```

## 四、npm 发布
项目生成后，进入项目目录，手动生成 npm publish 需要的配置文件，安装依赖以及构建
**.npmignore**
```git
.*.swp
._*
.DS_Store
.git
.hg
.npmrc
.lock-wscript
.svn
.wafpickle-*
config.gypi
CVS
npm-debug.log
```

```bash
cp .npmignore ./some-project-api && cd some-project-api && npm i && npm run build
```
发布 npm package 到 npm 私服
```bash
# npm login 为前提
npm publish
```

## 五、使用方法
项目中使用

```bash
npm i -S some-project-api
```

在 **`/src/apis/index.js`**  文件中只需要修改项目对应的 npm 名称即可，其他无需操心

```javascript
import request from '@common/request' // 引入 axios 实例
import { ApiFactory } from 'some-project-api'

export default = ApiFactory({}, process.env.VUE_APP_BASE_URL, request)
// 新模板为 VUE_APP_BASE_URL 现用模板为API_BASE_URL
```

1. 在Vue文件中引入** `import API from ''../../apis''`** 即可通过  **`API.getgetBusiness**` 调用 api

```javascript
// import API from '@/api' 这里暂不推荐使用 @ alias 方式引入，使用 @ 会导致 ts 类型推导失败，后续调研下解决方法.
import API from '../../apis'
export default {
  name: 'demo',
  data() {
    return {
      res: ''
    }
  },
  methods: {
    async getBusiness() {
      // 根据接口 operationId 直接调用
      const res = await API.getBusiness()
      this.res = res
    }
  }
}
</script>
```
上面提到 ts 类型推导，其实就是借助 VSCode 先天支持 ts 的特性，根据 interface 声明 跳入 npm package 的源码中，源码中对每个方法的传参进行了详细的说明， 如果不习惯或者觉得声明不详细也可通过 方法注释中的 @catalog 一键打开 catalog 中对应的方法。看图 =。=
![OpenAPI-Generator-Demo-Show1.png](https://user-images.githubusercontent.com/10891613/92222655-eb364780-eed1-11ea-8090-e21286bd4baf.png)
![OpenAPI-Generator-Demo-Show2.png](https://user-images.githubusercontent.com/10891613/92222660-ec677480-eed1-11ea-9cd8-4c6ef7466347.png)
![OpenAPI-Generator-Demo-Show3.png](https://user-images.githubusercontent.com/10891613/92222662-ed000b00-eed1-11ea-80c4-0c0db1e0ed53.png)
![OpenAPI-Generator-Demo-Show4.jpg](https://user-images.githubusercontent.com/10891613/92222671-f12c2880-eed1-11ea-849a-c85e55e7a299.png)

1. 在实际开发过程中，后端同学可能会对 api-spec 修改，当后端修改提交后，pipeline 会重新生成构建发布，这时需要后端同学告知前端同学，前端同学重新安装 api package
```bash
npm i -S some-project-api@latest
```
避免出现问题直接使用 @latest ，这里稍微科普下 package.json version 相关的知识点
```javascript
"dependencies": {
    "vue": "3.0.0",
    "vuex": "*",
    "react": "16.x",
    "typescript": "~5.4.6",
    "redux": "^14.0.0"
 }
```

> 前面三个很容易理解：
> - `"vue": "3.0.0"`: 固定版本号
> - `"vuex": "*"`: 任意版本（`>=0.0.0`）
> - `"react": "16.x"`: 匹配主要版本（`>=16.0.0 <17.0.0`）
> - `"react": "16.3.x"`: 匹配主要版本和次要版本（`>=16.3.0 <16.4.0`）
>
再来看看后面两个，版本号中引用了 `~` 和 `^` 符号：
> - `~`: 当安装依赖时获取到有新版本时，安装到 `x.y.z` 中 `z` 的最新的版本。即保持主版本号、次版本号不变的情况下，保持修订号的最新版本。
> - `^`: 当安装依赖时获取到有新版本时，安装到 `x.y.z` 中 `y` 和 `z` 都为最新版本。 即保持主版本号不变的情况下，保持次版本号、修订版本号为最新版本。
>
在 `package.json` 文件中最常见的应该是 `"yargs": "^14.0.0"` 这种格式的 依赖, 因为我们在使用 `npm install package` 安装包时，`npm` 默认安装当前最新版本，然后在所安装的版本号前加 `^` 号。


## 六、遇到的问题
在测试使用过程中发现几个问题

- 对于 `@/apis`导入 api，不使用相对路径会丢失类型提示，使用 `../../apis`才可以
- VSCode 提示在 .js 文件和 .vue 文件下行为不一致，在 .js 文件中跳转和提示都准确，按 opt + 点击 function name 跳转会准确的定位到这个 function下，在 .vue 文件中提示准确跳转不准确，按 opt + 点击function name，跳转的 function name 会有误差

添加 jsconfig.json 文件，加入相应的配置即可解决以上问题。
**Attention ！jsconfig.json 为 VSCode 特有，当然不排除其他编辑器借鉴，所以一般只有在 VSCode 中才能工作。**

jsconfig.json
```javascript
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": [
        "src/*"
      ],
      "@common/*": [
        "src/common/*"
      ],
      "@apis": [
        "src/apis"
      ],
      "@assets": [
        "src/assets"
      ],
      "@common": [
        "src/common"
      ],
      "@components": [
        "src/components"
      ],
      "@views": [
        "src/views"
      ],
      "@utils": [
        "src/utils"
      ]
    }
  },
  "include": [
    "./src/**/*.vue",
    "./src/**/*.js",
  ],
  "exclude": [
    "node_modules"
  ]
}
```

关于jsconfig.json的更多信息，可以查看 [vscode文档相关说明](https://code.visualstudio.com/docs/languages/jsconfig)。
