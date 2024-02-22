# 我的 vscode 配置笔记

> 发布日期：2021-09-15 22:25:11

窃钩者诛，窃国者侯。

<!--more-->

![不只是吃螃蟹](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210915-my-vscode-config-note/crab-people.jpg)

## 按不同颗粒度操作（通识操作）
对于这些操作，是每个编辑器/IDE都具备的，也是应该牢记于心的。当然，vscode 也是很好的支持这些操作的。

|      | 字符颗粒度         | 单词颗粒度 | 整行颗粒度 |
| ---- | ------------------ | ---------- | ---------- |
| 移动 | up/down/left/right | up/down/left/right + Option     | up/down/left/right + Command  |
| 选择 | + shift            | + shift    | + shift    |

## 多光标特性
多光标操作给代码的编写提供了极大的方便，下面根据不同场景有三种方法使用多光标操作。

- 按住 `option` 后用鼠标选择
- 用 `cmd + d` 命令进行就近多选
- 首先选择多行代码，然后按下 `option + shift + i`，每一行的最后都会创建一个新的光标

## 必备快捷键（In mac）

- 删除光标前的所有内容：`cmd + delete` 
- 删除光标后面的所有内容：`fn + delete`
- 删除光标前单词颗粒度度的内容：`option + delete`
- 在花括号之间来回跳转：`Cmd + Shift + \`
- 选中代码（或当前行）上下移动快捷键（加入shift就是复制）：`option + up/down`
- 选中代码后，进行多行合并：`ctrl + j`
- 注释代码：`ctrl + /`
- 格式化代码：`option + Shift + f` 
- 全局搜索：`cmd + shift + f`
- 打开隐藏终端：`ctrl + ``
- 智能提示：`ctrl + space`
- 新建空白文件：`cmd + n`
- 打开最近打开项目列表：`ctrl + r`
- 退回光标的上一个位置：`cmd + u`

## 命令行操作
命令行操作是 vscode 的一个重要功能，我们只需要在命令行中输入要操作的关键字，然后回车就可以很好的完成任务。

#### 打开命令行：`cmd + shift + p`
第一个符号代表不通功能的分类，比如：
- `>` 代表用于显示所有命令。
- `@` 用于显示和跳转文件中的“符号”。
- `:` 用于跳转到当前文件中的某一行。
- `?` 用于显示有哪些功能分类（最重要）。

#### 下面是一些常用命令的关键字
- **大小写转换**：搜索 `lower` 和 `upper` 两个关键词
- **用户自定义设置**：搜索 `user setting` 两个关键词
- **用户 snippet**：搜索 `user snippet` 两个关键词
- **排序**：搜索 `sort` 两个关键词

## vscode 的设置

#### 打开设置界面
可以在命令行中通过搜索关键字 `user setting` 来打开用户设置

#### vscode 的设置分为三类
- 第一类是 vscode 本身的设置（不推荐动）
- 第二类是用户设置（推荐个人修改这个配置以适应项目）
- 第三类是项目本身的配置（针对项目的配置，主要在 .vscode 目录下有四个文件）
`extension.json`，`launch.json`，`settings.json`，`tasks.json`。

#### launch.json
`launch.json` 文件是专门为 debugger 调试器准备的，在 vscode 中把调试功能的最终实现交给插件来完成的，因此 vscode 提供了统一的调试接口 Debug Adapter Protocol（DAP）。

vscode 中内置 node 环境的调试插件，但是如果想要调试其他程序，则需要安装其对应的插件，可以在插件仓库中搜索 `Debugger for xxx` 来获得不同的调试环境。

对于 launch.json 文件的配置在不同的环境中是有很多区别的，下面是配置的前端 vue 的调试配置。

```
{

  "version": "0.2.0",
  "configurations": [
    {
      // 命名
      "name": "Launch via NPM",
      // 通过 launch 启动
      "request": "launch",
      // 执行脚本
      "runtimeArgs": ["run", "serve"],
      // 执行命令
      "runtimeExecutable": "npm",
      // 忽律文件
      "skipFiles": ["<node_internals>/**"],
      // 指定调试环境
      "type": "node",
      // 工作目录
      "cwd": "${workspaceFolder}/client",
      // 启动端口
      "port": 9229
    }
  ]
}
```

#### tasks.json
这个文件是专门用来定制我们专属的执行任务的，我们在命令行中输入关键字 `tasks` 就可以进行任务的创建和执行了。

下面是一个简单 task 配置

```
"version": "2.0.0",
  "tasks": [
    {
      // task 名字
      "label": "test shell",
      // 任务的类型，比如 npm，shell，process 等
      "type": "shell",
      // 执行命令
      "command": "./scripts/test.sh",
      "windows": {
        "command": ".\\scripts\\test.cmd"
      },
      // 分组
      "group": "test",
      // 是用于控制任务运行的时候，是否要自动调出运行的界面
      "presentation": {
        "reveal": "always",
        "panel": "new"
      },
      // 其他配置，比如执行目录，环境变量等
      "options": {
        "cwd": "",
        "env": {},
        "shell": {
          "executable": "bash"
        }
      }
    }
  ]
}
```

## Emmet 支持
Emmet 是一个强大的 html 和 css 书写工具。

下面是开启 Emmet 的关键配置

```
// 对于其他语言，你也可以通过如下的设置来将其打开
"emmet.includeLanguages": {
  "javascript": "javascriptreact",
  "vue-html": "html",
  "razor": "html",
  "plaintext": "jade",
},
// 是否使用 tag 键出触发 emmet abbreviation
"emmet.triggerExpansionOnTab": true,
// 是否启动补全提示
"emmet.showExpandedAbbreviation": "always",
```

同样可以再命令行操作中使用 emmet，关键字就是 `emmet: xxx`

## javascript 和 node 支持
我们知道 javascript 和 typescript 是 vscode 的亲儿子，很多特性都是天然支持这两门语言的。

#### 类型提示
- JSDoc
```
/**
 * bar
 * @param {sring} sr 
 */
```

- typings/d.ts
- `// @ts-check` 开启代码审查

#### 模块引用
由于 js 中模块的规范很多，我们需要通过配置一个 jsconfig.json 来提示 javascript 的语言服务。

```
{
  "compilerOptions": {
    // 使用那种模块规范
    "module": "commonjs",
    // js 语法规范
    "target": "es2016",
    // 根路径
    "baseUrl": ".",
    // 搜索路径
    "paths": {
      "*": [
        "*",
        "common/*"
       ]
     }
   },
   // 相当于给每个文件加上 // @ts-check
   "checkJs": true,
   // 不包括哪些文件
   "exclude": [
     "node_modules",
     "**/node_modules/*"
   ]
}
```

## 文件及文件内容搜索
按 `cmd + p` 可以打开文件命令行：
- 直接通过文件名搜索，`enter` 直接打开文件，`cmd + enter` 在新窗口打开文件
- 先输入`:` 是当前文件跳转行（`ctrl + g`）
- 先输入`@` 是当前文件大纲跳转（`cmd + shift + o`）

定义和实现
- 定义和实现之间的跳转：`cmd + click`，`cmd + u`，`ctrl + -` 为回来
- 引用列表：`shift + F12`

## 其他

#### shell 中使用 vscode 做代码比较
vscode 可以安装其命令行工具 code 到 shell 中，里面会有很多比较实用的功能，比如借助 vscode 来比较两个文件的不同。

```
-d --diff <file> <file>           Compare two files with each other.
```

#### 如何从 vscode 中粘贴纯文本
其他编辑器也适用！

当你从 vscode 中复制文本后，在其他编辑器按下 `cmd + v` 时会带有 vscode 的格式，而你按下 `cmd + shift + v` 则是粘贴纯文本。

#### 自定义代码折叠
基于语言的代码折叠参考下面链接（自定义代码折叠）：
https://code.visualstudio.com/docs/editor/codebasics#_folding

#### 插件分享
你可以通过在项目的 .vscode 文件夹下，创建一个文件 extensions.json。在这个 JSON 文件里，你只需提供一个键（key） recommendations，然后将你想要推荐给这个项目的其他工程师的插件的 ID 们，全部放入到这个数组中。当他们打开这个项目，而且并没有安装这些插件时，VS Code就会给他们提示了。

```
{
    "recommendations": [
        "eg2.tslint",
        "dbaeumer.vscode-eslint",
        "msjsdiag.debugger-for-chrome",
    ]
}
```

## 设计思想

“三步走”的演变过程，作为一个通用的学习新工具的方法是我在本篇文章中的最大收获。

- 第一步：了解编辑器的快捷键和语言支持，快捷键值得多花时间；

- 第二步：开始挑剔编辑器的其他组件，但凡是跟自己的工作习惯或者工作流不匹配的，就会想办法换掉它，这是个做减法的过程；

- 第三步：最后一步，就是自己学习写插件了，编辑器本身的功能和社区不能够完全满足自己的需求，本着“麻烦别人不如磨炼自己”的精神，我开始自己动手。

很多人都把编辑器等同于IDE，其实从专业角度来讲并非这样。IDE 更为关注开箱即用的编程体验、对代码往往有很好的智能理解，同时侧重于工程项目，为代码调试、测试、工作流等都有图形化界面的支持，因此相对笨重，Java程序员常用的Eclipse定位就是IDE；而编辑器则相对更轻量，侧重于文件或者文件夹，语言和工作流的支持更丰富和自由，VS Code 把自己定位在编辑器这个方向上，但又不完全局限于此。

## 推荐的插件

### 阅读提升系列

**Bracket Pair Colorizer**
该插件可以为你把成对的括号做颜色区分，并且提供一根连接线。方便我们审阅代码结构。

**Rainbow Brackets**
为同一对花括号指定一个单独的配色，这样你就能够轻松地一眼看出花括号的配对了。

**Indent Rainbow**
为你的代码缩进提供颜色上的提示。

**Pigment**
将颜色渲染在段代码的下面。

**vscode-icons**
非常好看的文件图标。

**CSS Peek**
css 样式查看器，可快速查看我们的css样式，非常方便快捷。

**Better Comments**
在代码中通过不同颜色查看注释

![img](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210915-my-vscode-config-note/colourful-comment.png)

**RemoteHub**
在本地查看 github repository 上的代码。

### 操作辅助系列

**Auto Rename Tag / Auto Close Tag**
顾名思义，前端开发必备工具。

**Import Cost**
很好地在代码中给我们以提示，告诉我们引入的某个包，它最终会导致整个项目的大小增加多少。

**Path Intellisense**
自动提示文件路径，支持各种快速引入文件。

**Npm Intellisense**
可自动完成导入语句中的npm模块。

**Code Spell Checker**
css样式查看器，可快速查看我们的css样式，非常方便快捷。

**JavaScript Booster**
这款神器可以在我们代码写的不规范或者有待调整的地方，在光标聚焦后，会有一个小灯泡。会提示对应的不合理原因和改进方案。极大的提高了我们的代码优雅度，强烈推荐。

**Change case**
在各种命名法之间快速切换

使用：在 vs code 命令中输入 prefix: change case...

**Better Align**
以等号作为对其中心的格式化插件，源码中爱用。

使用：选中代码后，在 vs code 的命令中输入：align。

### Snippet 系列

**HTML Snippets / JavaScript (ES6) code snippets**
snippets 系列。语法智能提示，以及快速输入。

### Lint 系列

**ESlint / TSLint**
略

### 实用工具系列

**Partial Diff**
文件比较是一个很常见的场景，如果光凭我们肉眼分别的话，累人不说还容易出错。`Partial Diff`的出现就正好解决了这个问题。

使用：点击右键，选项中 Compare text 开头的就是此插件。

**Live Share** 
Live Share是微软官方出品的非常强大的服务，通过 Live Share service，你可以将你本地的工作区，直 

接分享给你的同伴，然后你的同伴就可以直接编辑你的代码，与你共享代码调试、集成终端等等，而无 

需安装任何环境。

### 有趣的插件

**Rainbow Fart**
萌妹子语音陪你写代码，一个神奇的 VSCode 插件。

## REFERENCE
[1] 33 个提高前端工作效率的 VSCode 实用插件: https://mp.weixin.qq.com/s/8IMjdm1RfzT4DeVW1dOcOg