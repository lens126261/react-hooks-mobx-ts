---
title: "Eslint配置及分享自己的eslint配置包"
layout: post
date: 2020-07-20 17:30
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Eslint
- Prettier
category: blog
author: lens
description: eslint配置
---

## Eslint+Prettier统一代码风格
> 使用Eslint对代码质量进行校验（未使用的变量、==问题等），Prettier对代码样式进行格式化（每行代码长度、结尾加分号等）。
```
npm i -D eslint-plugin-prettier
```
eslint-plugin-prettier会调用prettier对代码风格进行检查

在rules中添加，"prettier/prettier":error"，表示被prettier标记的地方抛出错误信息。
```
//.eslintrc.js
{
  "plugins": ["eslint-plugin-prettier"], // 也可以省略eslint-plugin-
  "rules": {
    "prettier/prettier": "error"
  }
}
```
Eslint规则与Prettier产生冲突怎么办？
```
npm i -D eslint-config-prettier
```
使用eslint-config-prettier配置，可以关闭所有不必要的规则或可能与Prettier冲突的规则。要确保其放在extends数组的最后
```
//.eslintrc.js
{
  extends: [
    "eslint-config-ali", 
    "eslint-config-prettier", // 一定要放在最后，可省略eslint-config-
  ],
}
```
参考官方[文档](https://github.com/prettier/eslint-config-prettier)，这里列出了与prettier冲突的配置项
> **注意：** eslint-plugin-prettier 与 eslint-config-prettier同时使用在`eslint --fix`的时候可能会产生一些问题，比如：缺少闭括号等导致的代码样式错乱，这些必须你手动去修复。

会产生问题的一些规则：
```
prefer-arrow-callback  // 要求回调函数使用箭头函数
arrow-body-style      // 要求箭头函数体使用大括号
```
如果需要批量修复未格式化的代码，可以考虑暂时禁用 prettier/prettier 规则，分别执行eslint --fix 和 prettier --write

我们项目中完整的eslint配置：
```
// .eslintrc.js
module.exports = {
    extends: [
      "eslint-config-ali/react", // 用于校验jsx文件格式
      "eslint-config-ali/typescript/react", // 用于校验ts文件
      "prettier" // eslint-config-prettier 可以关闭所有不必要的规则或可能与Prettier冲突的规则。要确保其放在extends数组的最后
    ],
    plugins: ["prettier"], // eslint-plugin-prettier插件会调用prettier对你的代码风格进行检查
    root: true,
    rules: {
      "prettier/prettier": ["error"], // 表示被prettier标记的地方抛出错误信息
      "import/prefer-default-export": 0, // 当模块只输出一个变量时，是否需要添加default
      "react/jsx-indent": 0, // jsx缩进
      "react/jsx-indent-props": 0, // jsx中的props缩进
      "react/prop-types": 0, // prop 需要 propTypes 验证类型
      "@typescript-eslint/indent": 0, // tsx 缩进
      "@typescript-eslint/no-require-imports": 0, // 是否允许require()导入
      "default-case": 0, // 默认case
      'no-magic-numbers': [ // 是否禁用魔术数字
        'warn',
        {
          ignore: [],
          ignoreArrayIndexes: true, // 忽略数组索引
          enforceConst: true,
          detectObjects: false,
        },
      ],
      "sort-imports": [ // import引入顺序
        "error",
        {
          ignoreCase: true, // 为true表示：忽略 import 语句本地名称的大小写
          ignoreDeclarationSort: false, // 为true表示：忽略 import 声明语句的排序
          ignoreMemberSort: false, // 为true表示：忽略有多个成员的 import 声明的排序。
          memberSyntaxSortOrder: ["all", "multiple", "single", "none"]
        }
      ]
    },
    globals: {
      dd: false,
      window: false,
      global: false,
      __WPO: false,
      IDLAPI: false
    }
};

```
同时，对prettier格式风格也做了统一：
```
// .prettierrc.js
module.exports = {
  printWidth: 80, //一行的字符数，如果超过会进行换行，默认为80
  tabWidth: 2, //一个tab代表几个空格数，默认为80
  useTabs: false, //是否使用tab进行缩进，默认为false，表示用空格进行缩减
  singleQuote: false, //字符串是否使用单引号，默认为false，使用双引号
  semi: true, //行位是否使用分号，默认为true
  trailingComma: 'none', //是否使用尾逗号，有三个可选值"<none|es5|all>"
  bracketSpacing: true //对象大括号直接是否有空格，默认为true，效果：{ foo: bar }
};
```
## webpack配置eslint-loader
> 当同时使用其他loader时，注意加载顺序，eslint-loader一般放在后边。
```
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'eslint-loader',
        options: {
          // eslint options (if necessary)
        },
      },
    ],
  }
};
```
配置完重启项目，即可在编译的时候看到eslint错误提示：
![](https://user-gold-cdn.xitu.io/2020/7/17/1735bcb326f2a789?w=1382&h=506&f=png&s=90050)

## Eslint + pre-commit强制校验
规则配置完，如何保证在团队开发中所有人提交的代码都是符合eslint规范的呢？

Git 为我们提供了一个hooks: `pre-commit` , 他会在我们 `git commit` 时进行校验，不符合eslint规则的代码无法被提交。

1、安装
```
npm install pre-commit -D
```
配置package.json
```
"scripts": {
    "lint": "eslint ./src/ --ext .jsx,.js,.ts,.tsx",
    "precommit-msg": "echo '正在执行eslint验证...' && exit 0"
},
"pre-commit": [
    "precommit-msg",
    "lint"
]
```
配置完后，我们提交代码时会对src文件夹下后缀为.jsx、.js、.tsx、.ts的文件执行检验。

该hooks只在 `git commit` 时有效，如果需要在commit、push、merge……时也起效，则推荐使用git 官方提供的另外一个hooks `husky`，使用方法也很简单，请自行学习。
## 和VSCode集成使用
目前主流的编辑器都提供了一些插件，对不符合eslint规则的代码会在编辑器中给予提示，方便我们在开发过程中及时处理问题。这里以VSCode为例：

#### 集成Eslint
1、安装[eslint-vscode](https://github.com/Microsoft/vscode-eslint)插件
![](https://user-gold-cdn.xitu.io/2020/7/15/173523e43eec91ac?w=630&h=486&f=png&s=62845)
2、settings.json中配置：
```
// 保存时自动修复eslint问题
"eslint.autoFixOnSave": true,   // 扩展版本 < 2.x
"editor.codeActionsOnSave": {   // 扩展版本2.x以上
    "source.fixAll.eslint": true
}
// 指定校验的文件类型
"eslint.validate": [
      "javascript", // 默认校验
      "javascriptreact", // 默认校验
      {
         "language": "typescript", "autoFix": true
      },
      {
         "language": "typescriptreact", "autoFix": true
      }
]
```
2.x版本后`"eslint.autoFixOnSave": true` 不推荐使用，本项目中使用的是1.9.1版本。详细配置规则请查阅[文档](https://github.com/Microsoft/vscode-eslint)
#### 集成Prettier
1、安装[prettier-vscode](https://github.com/prettier/prettier-vscode)插件，安装方法同eslint-prettier

2、settings.json中添加配置:
```
"editor.formatOnSave": false,
 "[typescript]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
 },
 "[typescriptreact]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
 },
 "[javascript]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
 },
 "[javascriptreact]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
 },
```
保证项目中已安装prettier依赖并固定版本`npm install prettier -D --save-exact`
## 可共享的配置
为了方便我们在多个项目中使用这套配置规则，可以将我们配置好的规则发布到npm平台
1、首先新建项目eslint-config-react-lens，目录结构如下：
```
index.js // 你的配置规则
package.json // 项目依赖
README.MD
```
建议在 `package.json` 中用 `peerDependencies` 字段声明你依赖的 ESLint版本
```
"peerDependencies": {
    "eslint": ">= 5"
},
```
2、到[npm平台](https://www.npmjs.com/)注册npm账号

3、登陆
```
npm login
```
4、发布
```
npm publish
```
5、到npm平台搜索你发布的包
![](https://user-gold-cdn.xitu.io/2020/7/15/17351d9beefa0c21?w=2440&h=956&f=png&s=182620)
6、在项目中使用
```
// 安装相关依赖
"eslint-config-react-lens": "^1.0.0",
"eslint-config-ali": "^10.0.0",
"eslint-config-prettier": "^6.11.0",
"eslint-plugin-import": "^2.22.0",
"eslint-plugin-prettier": "^3.1.4",
"eslint-plugin-react": "^7.20.3",
"eslint-plugin-react-hooks": "^4.0.7",
"@typescript-eslint/eslint-plugin": "^3.6.0",
"@typescript-eslint/parser": "^3.6.0",
```
```
// .eslint.js
"extends": ["eslint-config-react-lens"],
```
实例源码地址：https://github.com/lens126261/eslint-config-react-lens


