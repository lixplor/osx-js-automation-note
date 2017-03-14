# Wiki

## 使用Javascript实现自动化

### 最简单的方法

使用Script Editor

### 运行REPL

```shell
osascript -il Javascript
```

* `-i`: 交互模式
* `-l Javascript`: 指定Javascript为语言. 不指定的话则使用AppleScript.

### 创建Shebang脚本

```javascript
#!/usr/bin/env osascript -l JavaScript

function run(argv) {
  console.log(JSON.stringify(argv));
}
```

### 创建Mac OSX服务

当你创建一个服务, 它会在菜单栏的"服务"菜单中显示. 
打开 **Automator.app** 创建一个新的"服务". 拖拽"运行JavaScript"动作, 然后编写代码.

### 从Shell脚本中调用JXA

Shell脚本和其他脚本能够使用`osascript`命令行工具运行JXA代码.

```shell
osascript -l JavaScript -e 'Application("iTunes").currentTrack.name()'
```

调用结果会被打印出来, 与调用`eval()`是类似的.


## JXA中的ES6特性

JXA支持部分ES6特性, 具体取决于OSX的版本.

### 展开运算符

允许将数组展开为参数

```javascript
var array = [1, 2, 3];
f(...array);
```

```javascript
var a = [1, 2, 3, 4]
var b = [5, 6, 7, 8]
var concated = [...a, ...b]

concated
#    => [1, 2, 3, 4, 5, 6, 7, 8]
```

### 数组和对象解构

```javascript
var a = 1
var b = 2

[a, b] = [b, a]
#    => [2, 1]

a
#    => 2
b
#    => 1
```

```javascript
var object = { cat: 1, dog: 2 }
var { cat, dog } = object

cat
#    => 1
dog
#    => 2
```

### 字符串模板

可以在JXA中使用ES6字符串模板

```javascript
var name = "Brandon"
console.log(`Hi, ${name}!`)
#    =>  Hi, Brandon!
```


