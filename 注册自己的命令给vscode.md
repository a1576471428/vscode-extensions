# 注册自己的命令给vscode

## 注册一个命令  

使用`vscode.commands.registerCommand`函数可以注册一个命令给vscode，指定一个命令id并绑定处理方法。这个在前面已经做过几次了，大致的内容如下：

```typescript
import * as vscode from 'vscode';


let myCommand = vscode.commands.registerCommand('myCommandId', () => {
    //balabala   该命令的处理逻辑
});

```

只要myCommand被执行，这个方法就会被激活。command执行有很多种方式，比如之前helloworld插件时使用`Ctrl+Shift+P`执行命令，或者通过某个快捷键执行，或者通过右键菜单执行等，下面将进行详细描述。

## 将命令暴露给用户

在上面创建的命令，只是将一个commandId绑定到了一个方法上，但是用户并没有办法使用它，缺乏使用的入口，该入口在项目的`package.json`文件中contribution属性定义，该属性可以将命令绑定到非常多的位置，比如`commands`(Ctrl+Shift+P)、`keybindings`(快捷键)等，详情请查看官方[Contribution Points指南](https://code.visualstudio.com/api/references/contribution-points)。下面是一个`commands`绑定示例：

```json
{
  "contributes": {
    "commands": [
      {
        "command": "myExtension.sayHello",
        "title": "Say Hello"
      }
    ]
  }
}
```

另外，插件默认是不会执行的，也就是说，里面的`vscode.commands.registerCommand`方法默认不会被执行，插件激活时才会执行该方法，故使用该命令前我们还需要激活插件，激活插件也包括很多种方法，比如在某命令被执行时激活,当当前文件是某指定语言时被激活等等，具体可以查看官方的[Activation Events指南](https://code.visualstudio.com/api/references/activation-events)。下面是一个在命令被执行时激活插件的示例：

```json
{
  "activationEvents": ["onCommand:myExtension.sayHello"]
}
```

## 控制命令展现给用户的时机

我们注册的命令不一定总要展示给用户，如果之前我们开发的是一个python的插件，只有当语言是python时，`Ctrl Shtft P`菜单才会显示我们的命令。那么当用户在python文件编辑时，可以看到插件注册的命令，而当用户在编辑js文件时，就不展示该命令了。在vscode插件的package.json中，`contributes`中的[`menus`](https://code.visualstudio.com/api/references/contribution-points#contributes.menus)可以对这类内容做控制，`menus.commandPalette`里面，可以通过`when`来控制命令的显隐(when提供了很多条件，可以通过[官方指导](https://code.visualstudio.com/api/references/when-clause-contexts)来查看具体细节)。下面是一个当编辑的文件语言为markdown时，才显示指定的命令：

```json
{
  "contributes": {
    "menus": {
      "commandPalette": [
        {
          "command": "myExtension.sayHello",
          "when": "editorLangId == markdown"
        }
      ]
    }
  }
}
```

在配置后，`myExtension.sayHello`命令只会在编辑markdown文件时，才会显示在命令列表里。

## 控制命令的启用与否

有些插件只会在特定的时候启用，比如某个插件是分析**js文件的正则表达式**的，那么，在打开js文件时，该命令并不会启用，只有当用户位于**js的正则表示式**时，才会启用该插件的分析命令，这种情况使用`enablement`来控制，`enablement`的条件和[`when`](https://code.visualstudio.com/api/references/when-clause-contexts)的条件是一样的。下面示例时当编辑的文件为python时，该插件的命令生效。

```json
"contributes": {
        "commands": [
            {
                "command": "helloworld.helloworld",
                "title": "Hello World",
                "enablement": "editorLangId == python"
            }
        ]
    }
```

**注意：** 本条的内容看起来和上面的那条有些类似，但是并不是一个东西。上面那一条只是控制该命令是否展示给用户，但是并不影响该命令的启用与否。比如本条举的例子，如果使用上面的方法，只是用户看不到插件的菜单，但是插件仍然会在打开js文件时启用，进行分析，这样会造成无端的性能损耗。而通过本条的方式禁用该命令，则不会影响性能。

在禁用命令后，vscode会自动控制该命令的显示与否。

## 定制when条件的条件内容

在我们控制插件的时，如果发现官方提供的when里面的条件不能满足需求，我们还可以自定义条件，通过在插件中执行`vscode.commands.executeCommand('setContext', 'myContext', myValue);`就可以设置内容，然后在when条件里面使用。
比如，我们设置了`vscode.commands.executeCommand('setContext', 'myExtension:showMyCommand', true);`，然后在命令显示的条件里面设置 `"when": "myExtension:showMyCommand"`，那么该命令会在执行该段代码后显示。