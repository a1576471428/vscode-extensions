# vscode插件开发教程-helloworld插件

本文参考官方文档：[Your First Extension](https://code.visualstudio.com/api/get-started/your-first-extension)

## 初始化项目

首先确认nodejs和git都已经安装，然后安装必备的工具： `npm install -g yo generator-code`, ubuntu系统下使用`sudo npm install -g yo generator-code`  
然后使用工具初始化一个新的工程：`yo code`  

```bash
xiaochao@xiaochao:~/study/vscode$ yo code

     _-----_     ╭──────────────────────────╮
    |       |    │   Welcome to the Visual  │
    |--(o)--|    │   Studio Code Extension  │
   `---------´   │        generator!        │
    ( _´U`_ )    ╰──────────────────────────╯
    /___A___\   /
     |  ~  |     
   __'.___.'__   
 ´   `  |° ´ Y ` 

? What type of extension do you want to create? New Extension (TypeScript)
? What's the name of your extension? helloworld
? What's the identifier of your extension? helloworld
? What's the description of your extension? helloworld
? Initialize a git repository? Yes
? Bundle the source code with webpack? No
? Which package manager to use? npm
```

待初始化完成，打开该工程： `code ./helloworld`  
**注意：** 在是否使用webpack一栏最好选择否，选择是的话，在`F5`执行后会卡在webpack打包完成的位置，需要`Ctrl+C`来手动退出，才会弹出Extension Development Host的窗口。
***

## 运行项目

打开以后按**F5**,会弹出一个标题为Extension Development Host的窗口，同时在原来的窗口运行栏显示一句*Congratulations, your extension "helloworld" is now active!*  
在新窗口使用快捷键**Ctrl+Shift+P**，然后输入**Hello World**命令，右下角会显示一句*Hello World from helloworld!*，则运行成功！  

## 小试牛刀

### 修改提示信息

修改`src/extension.ts`中的`Hello World from helloworld!`，改成`wow！666！`,然后在Extension Development Host窗口使用快捷键**Ctrl+Shift+P**，执行命令`Developer: Reload Window`（后面执行命令默认使用快捷键**Ctrl+Shift+P**，不再重复），加载完成后再次执行`Hello World`命令，可以看到刚才的修改已生效。

### 新增命令并显示warning

修改`src/extension.ts`的active方法，新增一个命令：

```javascript
let anotherHelloDisposable = vscode.commands.registerCommand('helloworld.anotherHello', () => {
    vscode.window.showWarningMessage('this is my warning!');
});
context.subscriptions.push(anotherHelloDisposable);
```

然后在`package.json`里面的`activationEvents`新加一条`"onCommand:helloworld.anotherHello"`,在`contributes`的`commonds`里面新加一条：

```json
{
    "command": "helloworld.anotherHello",
    "title": "Another Hello"
}
```

结果类似这样：  

```json
    "activationEvents": [
        "onCommand:helloworld.helloworld",
        "onCommand:helloworld.anotherHello"
    ],
    "main": "./out/extension.js",
    "contributes": {
        "commands": [
            {
                "command": "helloworld.helloworld",
                "title": "Hello World"
            },
            {
                "command": "helloworld.anotherHello",
                "title": "Another Hello"
            }
        ]
    },
```

在新窗口重新加载，执行命令`"Another Hello"`，可以看到新添加的`this is my warning!`提示信息。

## helloworld项目分析

下面简单介绍一下我们的插件是怎么跑起来的，以及vscode插件的项目构成。

### hello world插件是如何执行的

初始化好的helloworld插件做了三件事  

* 在`activationEvents`注册`onCommand`: `onCommand:helloworld.helloworld`。我们的插件默认是不激活的，这个注册是为了让在命令Hello World执行的时候，激活我们的插件。onCommand只是一个激活事件，还有onLanguage，onDebug等其他事件，具体可以查看[Activation Event](https://code.visualstudio.com/api/references/activation-events)。
* 使用`contributes.commands` [Contribution Point](https://code.visualstudio.com/api/references/contribution-points)来注册插件，这里使用commands来注册到命令执行，使得在命令列表里面出现`Hello World`。还可以注册到某个快捷键，文件的右键菜单等等各种位置。我们注册的命令是`Hello World`，命令id为`helloworld.helloworld`。
* 在`src/extension.ts`里面使用[vscode API](https://code.visualstudio.com/api/references/vscode-api)注册命令ID对应的实际执行方法。

综上，我们的插件首先在`contributes.commands`被注册到命令执行，在我们执行命令时`Hello World`时，插件被激活，然后根据命令ID`helloworld.helloworld`找到我们的方法，执行`vscode.window.showInformationMessage('wow！666！');`,显示`'wow！666！'`。

### 项目构成

本项目最重要的两个文件时`package.json`和`src/extension.ts`，下面依次说明。

#### package.json

每个插件必须包含`package.json`，除了一些`scripts`等nodejs要求的字段，vscode还有一些要求的字段，全部的字段解释在[这里](https://code.visualstudio.com/api/references/extension-manifest)，下面简单解释一下几个最重要的字段：  

* `name` 和 `publisher`: vscode使用`<publisher>.<name>`作为插件的唯一标识。
* `main`: 插件入口
* `activationEvents` 和 `contributes`： [激活事件](https://code.visualstudio.com/api/references/activation-events)和[切入点](https://code.visualstudio.com/api/references/contribution-points)
* `engines.vscode`: 声明了本插件依赖的vscode api最低版本

#### 插件入口文件

入口文件导出了两个方法，分别是`activate` 和 `deactivate`。 在插件注册的事件发生时（被激活时）执行`activate`方法。`deactivate`可以执行一些清理操作，大部分插件不需要关注本方法，但是如果插件有一些在vscode被关闭时或者插件被禁用或者卸载时执行的东西，可以在这里写。  
vscode API声明在[@types/vscode](https://www.npmjs.com/package/@types/vscode)，我们平时写的时候可以直接ctrl+点击对应的方法来跳转到定义的部分，参考怎么用。

## helloworld 结束

本文演示的代码可以在[这里](https://github.com/a1576471428/vscode-demo)下载，但是推荐按照教程直接创建一个，也很容易。  
系列教程目录：[vscode插件开发教程](https://blog.csdn.net/qq_30794691/article/details/113063788)
