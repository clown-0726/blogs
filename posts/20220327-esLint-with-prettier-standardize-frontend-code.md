# ESLint 和 Prettier 工程化前端代码

> 发布日期：2022-03-27 12:10:36

大浪淘沙，前段构架工具也在一代代的更迭，最终脱颖而出的是 Prettier 和 ESLint。很多人说这两个有很多重复功能，但其实这两个工具的侧重点是不一样的：

- Prettier 注重的是代码的格式化，也就是让团队有一个统一的代码规范。
- ESLint 注重的则更多是代码的质量，保证的是代码的安全性。

这一点在 Prettier 的官网上有很好的说明：

```
参考链接：https://prettier.io/docs/en/comparison.html

Linters have two categories of rules:

Formatting rules: eg: max-len, no-mixed-spaces-and-tabs, keyword-spacing, comma-style…

Prettier alleviates the need for this whole category of rules! Prettier is going to reprint the entire program from scratch in a consistent way, so it’s not possible for the programmer to make a mistake there anymore :)

Code-quality rules: eg no-unused-vars, no-extra-bind, no-implicit-globals, prefer-promise-reject-errors…

Prettier does nothing to help with those kind of rules. They are also the most important ones provided by linters as they are likely to catch real bugs with your code!

In other words, use Prettier for formatting and linters for catching bugs!
```

## Prettier

Prettier 的设计理念很简单，用最少的配置提供一个统一的格式化配置，在官网上用的词是 “opinionated”，翻译过来就是“固执己见的”，并且 Prettier 很自信提供的代码格式化规则都是最优的。

```
By far the biggest reason for adopting Prettier is to stop all the ongoing debates over styles.
翻译：到现在选择 Prettier 的最主要原因就是停止无休止的代码风格讨论。
```

其实对于大多数项目来说这是够用的，并且很能很好的省下更多的时间放到业务代码上，而不是和团队成员商量我们的代码格式化应该怎么调整。

所有的 Prettier 提供的可配置项只有十几个，并且也是因为历史原因没办法才加上这几个配置的，并且也不会在增加新的了。文章最后的附录提供了 Prettier 配置项的中文说明。

```
Prettier has a few options because of history. But we won’t add more of them.
```

#### 如何配置

本文在此只简单阐述安装的关键步骤，对于安装细节可以参考官网的文档，https://prettier.io/docs/en/install.html。

**安装 prettier**

```
npm install --save-dev --save-exact prettier
```

**生成配置文件**

在项目根目录下手动建立或者通过下面的 shell 命令建立 Prettier 的配置文件

```
echo {}> .prettierrc.json
```

然后参考本文附录的 prettier 的详细配置来配置自己的配置文件。

**将配置文件用于代码格式化**

- 配置文件可以用于 prettier 安装后的 prettier-cli，通过命令行进行文件格式化，运行命令是 `prettier --write .`，表示对当前目录下的所有文件进行格式化。我们可以建立 `.prettierignore` 文件来忽略不需要进行格式化的文件。

- 也可以用于安装好 prettier 插件的 IDE，比如 vscode 或者 webstrom。

## ESLint

ESLint 主要是用于保证代码的质量，如同官网上所说的 `Find and fix problems in your JavaScript code`。我们知道，js 作为弱类型的语言，其灵活性不言而喻，但是这同时会导致很多不规范写法的发生率，而 Eslint 会去静态分析 js 代码，帮助我们规避这些潜在的风险和问题。

我们可以认为 ESLint 主要包含两个部分：

- 第一是代码质量的校验，也就是会导致潜在问题的代码。
- 第二是不符合格式化的代码，但是这部分我们更倾向于让 Prettier 帮我们进行校验。

另外补充一点，TSLint 已经不再维护了。前端代码不管 TS 还是 ES，都用 ESLint 吧。具体看作者在[Medium上的Blog](https://link.zhihu.com/?target=https%3A//medium.com/palantir/tslint-in-2019-1a144c2317a9)和[相关Issue](https://link.zhihu.com/?target=https%3A//github.com/palantir/tslint/issues/4534)。所以说，ESLint 已经成为前端代码的事实规范。

```
As you may have read in this blog post, we plan to deprecate TSLint in 2019 and support the migration to ESLint as the standard linter for both TypeScript & JavaScript. 
```

#### 如何配置

本文在此只简单阐述安装的关键步骤，对于安装细节可以参考官网的文档，https://eslint.org/docs/user-guide/getting-started。

**安装 ESLint**

```
npm install eslint --save-dev
```

**生成配置文件**

在项目根目录下手动建立（.eslintrc.js）或者通过下面的命令建立的配置文件

```
npm init @eslint/config
```

#### 配置 .eslintrc.js

下面是基本的配置文件的解释说明：

```js
module.exports = {
  // 表示当前目录即为根目录，ESLint 将被限制到该目录下
  root: true,
  // 设置解析器，ESLint 本身不具备代码静态分析能力，一般借助 babel 分析 js 静态代码并生成 AST 静态分析树
  parserOptions: {
    parser: "babel-eslint",
    sourceType: "module",
  },
  // env 表示在什么环境下启用 ESLint 检测
  env: {
    browser: true,
    node: true, // 表示在 node 环境下启动 ESLint 检测
    es6: true,
    jquery: true,
  },
  // 表示引入一些全局变量，在检查时对于 ESLint 不识别的全局便令不会报 no-undef 错
  globals: {
    $_: true,
  },
  // 插件，通过扩展 ESLint 的插件可以让 ESLint 分析更多类型的文件
  // 比如 vue 插件可以让 ESLint 识别 vue 文件
  plugins: [],
  // 扩展
  // ESLint 的配置非常多，我们不可能关注到每一个配置，因此很多团队推出了一些标准
  // 我们可以首先继承这些已存在的标准，然后再定义根据项目实际进行扩展
  extends: ["plugin:vue/essential", "eslint:recommended", "plugin:prettier/recommended"],
  /**
   * 错误级别分为 3 种
   * "off" 或 0 - 关闭规则
   * "warn” 或 1 - 开启规则，使用警告级别的错误：warn（不会导致程序退出）
   * "error” 或 2 - 开启规则，使用错误界别的错误；error（当触发的时候，程序会退出）
   */
  rules: {
    // 配置自己的规则，自己的规则拥有最高的优先级
  },
};
```

上面是 ESLint 的基本配置，对于其详细规则配置，参考本文附录的 ESLint 的详细配置文件。

## ESLint 集成 Prettier

作为前端两个必不可少的工具，还需要让彼此能够和谐的工作。对于 ESLint 集成 Prettier，前提是要认同 Prettier 的代码格式化，如果不认同，你可能需要选择其他的方案。

#### 集成思路

Prettier 其实就是做了一件事，也就是代码格式化，我们可以手动在 IDE 中通过快捷键完成或者通过 prettier 的 cli 对指定文件进行格式化，但是这并不会在 IDE 中实时提示哪段代码是不符合 Prettier 格式化规范的。

对于代码实时提示这是 ESLint 的强项，我们可以通过对 ESLint 扩展 Prettier 插件的形式，让不符合 Prettier 格式化规范的代码在 IDE 和项目启动的控制台实时显示出来供我们参考。

由于 ESLint 也有自己的代码格式化规范，对于这部分冲突的代码格式化，我们选择 Prettier，Prettier 也提供了插件可以让我们禁止掉 ESLint 的代码格式化，从而专注于代码的质量。

总结一下，在项目中 Prettier 将会以 ESLint 插件的形式帮我们进行代码的格式化和提示。

#### 集成步骤

在开始之前，要保证已经在项目中安装好了 Prettier 和 ESLint。

**第一步，安装 ESLint 的 Prettier 插件**

```
npm install --save-dev eslint-plugin-prettier
```

`eslint-plugin-prettier` 不会为你安装 Prettier 和 ESLint，你必须手动安装他们。

**第二步，在 ESLint 的配置文件中配置 Prettier**

```
{
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": "error"
  }
}
```

`"prettier/prettier": "error"` 这个自定义规则的意思是，如果任何代码不满足 Prettier 的格式化规范，就会给出这个这个错误提示。

上面这么配置是不是有些麻烦？其实 `eslint-config-prettier` 这个插件可以让配置更加简单。

首先你需要安装安装这个插件，但是 `eslint-plugin-prettier` 也是必须的，

```
npm install --save-dev eslint-plugin-prettier
npm install --save-dev eslint-config-prettier
```

这个插件附带了一个配置 `plugin:prettier/recommended config` 可以一次性设置插件和 `eslint-plugin-prettier`。

因此你的最终的 ESLint 配置是下面这样的。

```
{
  "extends": ["plugin:prettier/recommended"]
}
```

Well, 这个插件其实做了这么几件事情：

```
{
  "extends": ["prettier"],
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": "error",
    "arrow-body-style": "off",
    "prefer-arrow-callback": "off"
  }
}
```

- `"extends": ["prettier"]` 开启了 `eslint-config-prettier `的配置，这些配置管理一些 ESLint 一些与 Prettier 冲突的规则。
- `"plugins": ["prettier"]` 用于在 ESLint 中注册 prettier 插件。
- `"prettier/prettier": "error"` 打开了插件提供规则，让 Prettier 作为 ESLint 的一个规则运行。
- `"arrow-body-style": "off"` 和 `"prefer-arrow-callback": "off"` 关闭了两个与这个插件有冲突的核心规则。

## 参考文档

[1] ESLint 规则说明 https://blog.csdn.net/helpzp2008/article/details/51507428  
[2] Install eslint https://prettier.io/docs/en/install.html  
[3] eslint-plugin-prettier https://github.com/prettier/eslint-plugin-prettier  
[4] 使用ESLint+Prettier来统一前端代码风格 https://segmentfault.com/a/1190000015315545  
[5] eslint-config-prettier https://github.com/prettier/eslint-config-prettier  
[6] Roadmap: TSLint -> ESLint https://github.com/palantir/tslint/issues/4534  
[7] Eslint 超简单入门教程 https://www.jianshu.com/p/ad1e46faaea2  
[8] Prettier看这一篇就行了 https://zhuanlan.zhihu.com/p/81764012?from_voters_page=true

## 附录

#### Prettier 常用配置和中文说明

详细参考官网的配置说明：https://prettier.io/docs/en/options.html

以下是中文配置说明：

```json
{
  // 换行的宽度，其实对于现在普遍的宽屏显示器可以适当调高
  "printWidth": 120,
  // Tab 字符的空格宽度
  "tabWidth": 2,
  // 缩进用 Tab 还是用空格
  "useTabs": false,
  // 是否行尾自动加上分号
  "semi": true,
  // 是否用单引号替代双引号，JSX 会忽略这个配置
  "singleQuote": true,
  // 对象里的key要不要用引号包裹
  "quoteProps": "consistent",
  // JSX 里用单引号替代双引号
  "jsxSingleQuote": true,
  // 多行情况下，是否加上最后元素的逗号
  "trailingComma": "es5",
  // 对象直接量括号之间是否有空格
  "bracketSpacing": true,
  // JSX 里多行元素的 > 是否出现在新的一行。
  "jsxBracketSameLine": false,
  // 箭头函数的单参数是否用括号包裹
  "arrowParens": "avoid",
  // 限制只格式化头部带有 prettier 特殊注释 (pragma) 的文件
  "requirePragma": false,
  // 是否插入 prettier 特殊注释 (pragma)
  "insertPragma": false,
  // 包裹 markdown 的文本
  "proseWrap": "preserve",
  // 处理 HTML 的空白字符
  "htmlWhitespaceSensitivity": "css",
  // 对于 vue 文件的 <script> 和 <style> 标签是否首列缩进
  "vueIndentScriptAndStyle": false,
  // 处理换行行尾字符（因为它跟操作系统相关）
  "endOfLine": "lf",
  // 是否格式化注释的代码，auto 是如果能识别则进行格式化
  "embeddedLanguageFormatting": "auto",
}
```

#### ESLint 常用配置中文说明：

详细参考官网的配置说明：https://eslint.bootcss.com/docs/rules/

```json
"no-alert": 0, // 禁止使用alert confirm prompt
"no-array-constructor": 2, // 禁止使用数组构造器
"no-bitwise": 0, // 禁止使用按位运算符
"no-caller": 2, // 禁止使用 arguments.caller 或 arguments.callee
"no-catch-shadow": 2, // 禁止catch子句参数与外部作用域变量同名
"no-class-assign": 2, // 禁止给类赋值
"no-cond-assign": 2, // 禁止在条件表达式中使用赋值语句
"no-console": 0, // 禁止使用 console
"no-const-assign": 2, // 禁止修改 const 声明的变量
"no-constant-condition": 2, // 禁止在条件中使用常量表达式 if(true) if(1)
"no-continue": 0, // 禁止使用continue
"no-control-regex": 0, // 禁止在正则表达式中使用控制字符
"no-debugger": process.env.NODE_ENV === "production" ? 2 : 0, // 禁止使用debugger
"no-delete-var": 2, // 不能对var声明的变量使用delete操作符
"no-div-regex": 1, // 不能使用看起来像除法的正则表达式 /=foo/
"no-dupe-keys": 2, // 在创建对象字面量时不允许键重复 {a:1,a:1}
"no-dupe-args": 2, // 函数参数不能重复
"no-dupe-class-members": 2, 🤬🤬🤬
"no-duplicate-case": 2, // switch 中的 case 标签不能重复
"no-else-return": 2, // 如果 if 语句里面有 return, 后面不能跟 else 语句
"no-empty": 2, // 块语句中的内容不能为空
"no-empty-character-class": 2, // 正则表达式中的 [] 内容不能为空
"no-empty-label": 2, // 禁止使用空label
"no-empty-pattern": 2, 🤬🤬🤬
"no-eq-null": 2, // 禁止对 null 使用 == 或 != 运算符
"no-eval": 1, // 禁止使用 eval
"no-ex-assign": 2, // 禁止给 catch 语句中的异常参数赋值
"no-extend-native": 2, // 禁止扩展native对象
"no-extra-bind": 2, // 禁止不必要的函数绑定
"no-extra-boolean-cast": 2, // 禁止不必要的 bool 转换
"no-extra-parens": 2, // 禁止非必要的括号
"no-extra-semi": 2, // 禁止多余的冒号
"no-fallthrough": 1, // 禁止switch穿透
"no-floating-decimal": 2, // 禁止省略浮点数中的0.5 3.
"no-func-assign": 2, // 禁止重复的函数声明
"no-implicit-coercion": 1, // 禁止隐式转换
"no-implied-eval": 2, // 禁止使用隐式eval
"no-inline-comments": 0, // 禁止行内备注
"no-inner-declarations": [2, "functions"], // 禁止在块语句中使用声明（变量或函数）
"no-invalid-regexp": 2, // 禁止无效的正则表达式
"no-invalid-this": 2, // 禁止无效的this，只能用在构造器，类，对象字面量
"no-irregular-whitespace": 2, // 不能有不规则的空格
"no-iterator": 2, // 禁止使用 __iterator__ 属性
"no-label-var": 2, // label 名不能与 var 声明的变量名相同
"no-labels": 2, // 禁止标签声明
"no-lone-blocks": 2, // 禁止不必要的嵌套块
"no-lonely-if": 2, // 禁止else语句内只有if语句
"no-loop-func": 1, // 禁止在循环中使用函数（如果没有引用外部变量不形成闭包就可以）
"no-mixed-requires": [0, false], // 声明时不能混用声明类型
"no-mixed-spaces-and-tabs": [2, false], // 禁止混用 tab 和空格
"linebreak-style": [0, "windows"], // 换行风格
"no-multi-spaces": 1, // 不能用多余的空格
"no-multi-str": 2, // 字符串不能用 \ 换行
"no-multiple-empty-lines": [1, {"max": 2}], // 空行最多不能超过2行
"no-native-reassign": 2, // 不能重写native对象
"no-negated-in-lhs": 2, // in 操作符的左边不能有!
"no-nested-ternary": 0, // 禁止使用嵌套的三目运算
"no-new": 1, // 禁止在使用new构造一个实例后不赋值
"no-new-func": 1, // 禁止使用 new Function
"no-new-object": 2, // 禁止使用 new Object()
"no-new-symbol": 2, 🤬🤬🤬
"no-new-require": 2, // 禁止使用 new require
"no-new-wrappers": 2, // 禁止使用 new 创建包装实例，new String new Boolean new Number
"no-obj-calls": 2, // 不能调用内置的全局对象，比如 Math() JSON()
"no-octal": 2, // 禁止使用八进制数字
"no-octal-escape": 2, // 禁止使用八进制转义序列
"no-param-reassign": 2, // 禁止给参数重新赋值
"no-path-concat": 0, // node 中不能使用 __dirnam e或 __filename 做路径拼接
"no-plusplus": 0, // 禁止使用 ++，--
"no-process-env": 0, // 禁止使用 process.env
"no-process-exit": 0, // 禁止使用 process.exit()
"no-proto": 2, // 禁止使用 __proto__ 属性
"no-redeclare": 2, // 禁止重复声明变量
"no-regex-spaces": 2, // 禁止在正则表达式字面量中使用多个空格 /foo bar/
"no-restricted-modules": 0, // 如果禁用了指定模块，使用就会报错
"no-return-assign": 1, // return 语句中不能有赋值表达式
"no-script-url": 0, // 禁止使用javascript:void(0)
"no-self-assign": 2, 🤬🤬🤬
"no-self-compare": 2, // 不能比较自身
"no-sequences": 0, // 禁止使用逗号运算符
"no-shadow": 2, // 外部作用域中的变量不能与它所包含的作用域中的变量或参数同名
"no-shadow-restricted-names": 2, // 严格模式中规定的限制标识符不能作为声明时的变量名使用
"no-spaced-func": 2, // 函数调用时 函数名与()之间不能有空格
"no-sparse-arrays": 2, // 禁止稀疏数组， [1,,2]
"no-sync": 0, // nodejs 禁止同步方法
"no-ternary": 0, // 禁止使用三目运算符
"no-trailing-spaces": 1, // 一行结束后面不要有空格
"no-this-before-super": 0, // 在调用 super() 之前不能使用 this 或 super
"no-throw-literal": 2, // 禁止抛出字面量错误 throw "error";
"no-undef": 1, // 不能有未定义的变量
"no-undef-init": 2, // 变量初始化时不能直接给它赋值为 undefined
"no-undefined": 2, // 不能使用 undefined
"no-unexpected-multiline": 2, // 避免多行表达式
"no-underscore-dangle": 1, // 标识符不能以_开头或结尾
"no-unmodified-loop-condition": 2, 🤬🤬🤬
"no-unneeded-ternary": 2, // 禁止不必要的嵌套 var isYes = answer === 1 ? true : false;
"no-unreachable": 2, // 不能有无法执行的代码
"no-unused-expressions": 2, // 禁止无用的表达式
"no-unused-vars": [2, {"vars": "all", "args": "after-used"}], // 不能有声明后未被使用的变量或参数
"no-unsafe-finally": 2, 🤬🤬🤬
"no-use-before-define": 2, // 未定义前不能使用
"no-useless-call": 2, // 禁止不必要的call和apply
"no-useless-computed-key": 2, 🤬🤬🤬
"no-useless-constructor": 2, 🤬🤬🤬
"no-useless-escape": 0, 🤬🤬🤬
"no-void": 2, // 禁用void操作符
"no-var": 0, // 禁用var，用let和const代替
"no-warning-comments": [1, { "terms": ["todo", "fixme", "xxx"], "location": "start" }], // 不能有警告备注
"no-whitespace-before-property": 2, 🤬🤬🤬
"no-with": 2, // 禁用with

"array-bracket-spacing": [2, "never"], // 是否允许非空数组里面有多余的空格
"arrow-parens": 0, // 箭头函数用小括号括起来
"arrow-spacing": 0, // => 的前 / 后括号
"accessor-pairs": 0,// 在对象中使用 getter/setter
"block-scoped-var": 0, // 块语句中使用 var
"block-spacing": [2, "always"], 🤬🤬🤬
"brace-style": [1, "1tbs"], // 大括号风格
"callback-return": 1, // 避免多次调用回调什么的
"camelcase": 2, // 强制驼峰法命名
"comma-dangle": [2, "never"], // 对象字面量项尾不能有逗号
"comma-spacing": 0, // 逗号前后的空格
"comma-style": [2, "last"], // 逗号风格，换行时在行首还是行尾
"complexity": [0, 11], // 循环复杂度
"computed-property-spacing": [0, "never"], // 是否允许计算后的键名什么的
"consistent-return": 0, // return 后面是否允许省略
"consistent-this": [2, "that"], // this别名
"constructor-super": 0, // 非派生类不能调用 super，派生类必须调用 super
"curly": [2, "all"], // 必须使用 if(){} 中的{}
"default-case": 2, // switch语句最后必须有 default
"dot-location": 0, // 对象访问符的位置，换行的时候在行首还是行尾
"dot-notation": [0, { "allowKeywords": true }], // 避免不必要的方括号
"eol-last": 0, // 文件以单一的换行符结束
"eqeqeq": 2, // 必须使用全等
"func-names": 0, // 函数表达式必须有名字
"func-style": [0, "declaration"], // 函数风格，规定只能使用函数声明 / 函数表达式
"generator-star-spacing": 0, // 生成器函数 * 的前后空格
"guard-for-in": 0, // for in 循环要用 if 语句过滤
"handle-callback-err": 0, // nodejs 处理错误
"id-length": 0, // 变量名长度
"indent": [2, 4], // 缩进风格
"init-declarations": 0, // 声明时必须赋初值
"jsx-quotes": [0, "prefer-single"], 🤬🤬🤬
"keyword-spacing": [2, { before: true, after: true }], 🤬🤬🤬
"key-spacing": [0, { "beforeColon": false, "afterColon": true }], // 对象字面量中冒号的前后空格
"lines-around-comment": 0, // 行前/行后备注
"max-depth": [0, 4], // 嵌套块深度
"max-len": [0, 80, 4], // 字符串最大长度
"max-nested-callbacks": [0, 2], // 回调嵌套深度
"max-params": [0, 3], // 函数最多只能有 3 个参数
"max-statements": [0, 10], // 函数内最多有几个声明
"new-cap": 2, // 函数名首行大写必须使用 new 方式调用，首行小写必须用不带 new 方式调用
"new-parens": 2, // new 时必须加小括号
"newline-after-var": 2, // 变量声明后是否需要空一行
"object-curly-spacing": [0, "never"], // 大括号内是否允许不必要的空格
"object-shorthand": 0, // 强制对象字面量缩写语法
"one-var": 1, // 连续声明
"operator-assignment": [0, "always"], // 赋值运算符 += -= 什么的
"operator-linebreak": [2, "after"], // 换行时运算符在行尾还是行首
"padded-blocks": 0, // 块语句内行首行尾是否要空行
"prefer-const": 0, // 首选 const
"prefer-spread": 0, // 首选展开运算
"prefer-reflect": 0, // 首选 Reflect 的方法
"quotes": [1, "single"], // 引号类型 `` "" ''
"quote-props":[2, "always"], // 对象字面量中的属性名是否强制双引号
"radix": 2, // parseInt必须指定第二个参数
"id-match": 0, // 命名检测
"require-yield": 0, // 生成器函数必须有yield
"semi": [2, "always"], // 语句强制分号结尾
"semi-spacing": [0, {"before": false, "after": true}], // 分号前后空格
"sort-vars": 0, // 变量声明时排序
"space-after-keywords": [0, "always"], // 关键字后面是否要空一格
"space-before-blocks": [0, "always"], // 不以新行开始的块 { 前面要不要有空格
"space-before-function-paren": [0, "always"], // 函数定义时括号前面要不要有空格
"space-in-parens": [0, "never"], // 小括号里面要不要有空格
"space-infix-ops": 0, // 中缀操作符周围要不要有空格
"space-return-throw-case": 2, // return throw case 后面要不要加空格
"space-unary-ops": [0, { "words": true, "nonwords": false }], // 一元运算符的前 / 后要不要加空格
"spaced-comment": 0, // 注释风格要不要有空格什么的
"strict": 2, // 使用严格模式
"template-curly-spacing": [2, "never"], 🤬🤬🤬
"use-isnan": 2, // 禁止比较时使用 NaN，只能用 isNaN()
"valid-jsdoc": 0, // jsdoc规则
"valid-typeof": 2, // 必须使用合法的 typeof 的值
"vars-on-top": 2, // var 必须放在作用域顶部
"wrap-iife": [2, "inside"], // 立即执行函数表达式的小括号风格
"wrap-regex": 0, // 正则表达式字面量用小括号包起来
"yield-star-spacing": [2, "both"], 🤬🤬🤬
"yoda": [2, "never"] // 禁止尤达条件

// 下面是 vue 特有的配置
"vue/max-attributes-per-line": [0, { singleline: 10, multiline: { max: 1, allowFirstLine: false } }],
"vue/html-self-closing": 0,
"vue/singleline-html-element-content-newline": 0,
"vue/multiline-html-element-content-newline": 0,
"vue/no-v-html": 0,
"vue/name-property-casing": [2, "kebab-case"],
```

