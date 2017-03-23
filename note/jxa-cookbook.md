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

## 创建应用实例

必须创建一个应用实例, 然后用其编写脚本.

```javascript
var app = Application.currentApplication();
app.includeStandardAdditions = true;
```

所有代码必须拥有应用实例, 并增加脚本附加内容.

### 使用`Application()`前检查该应用是否运行

某些情况下, `Application()`会永远等待(可能是等待应用启动). 这会阻塞脚本. 这种情况一般会在应用没有启动时出现. 

自OSX 10.11开始, 检测的方法简化为Application类的`.running()`属性:

```javascript
if (Application('TaskPaper').running()) {
    // ...
}
```

在OSX 10.10中, 也有其他间接实现的方法. 大多数应用(也就是在`/Application`中的应用)都拥有一个唯一的bundler识别器. 你可以使用Objective-C bridge来验证应用是否运行:

```javascript
// Look for iTunes
ObjC.import('stdlib')
ObjC.import('AppKit')
var isRunning = false
var apps = $.NSWorkspace.sharedWorkspace.runningApplications // Note these never take () unless they have arguments
apps = ObjC.unwrap(apps) // Unwrap the NSArray instance to a normal JS array
var app, itunes
for (var i = 0, j = apps.length; i < j; i++) {
  app = apps[i]

  // Another option for comparison is to unwrap app.bundleIdentifier
  // ObjC.unwrap(app.bundleIdentifier) === 'org.whatever.Name'

  // Some applications do not have a bundleIdentifier as an NSString
  if (typeof app.bundleIdentifier.isEqualToString === 'undefined') {
    continue;
  }

  if (app.bundleIdentifier.isEqualToString('com.apple.iTunes')) {
    isRunning = true;
    break;
  }
}

if (!isRunning) {
  $.exit(1)
}

itunes = Application('iTunes')
```

你也可以使用`ps`:

```javascript
ObjC.import('Foundation')

var pipe = $.NSPipe.pipe
var file = pipe.fileHandleForReading  // NSFileHandle
var task = $.NSTask.alloc.init

task.launchPath = '/bin/ps'
task.arguments = ['aux']
task.standardOutput = pipe  // if not specified, literally writes to file handles 1 and 2

task.launch // Run the command `ps aux`

var data = file.readDataToEndOfFile  // NSData
file.closeFile

// Call -[[NSString alloc] initWithData:encoding:]
data = $.NSString.alloc.initWithDataEncoding(data, $.NSUTF8StringEncoding)
var lines = ObjC.unwrap(data).split('\n') // Note we have to unwrap the NSString instance
var line
for (var i = 0, j = lines.length; i < j; i++) {
    line = lines[i]
    if (/iTunes$/.test(line)) {
        iTunesRunning = true
        break
    }
}
```

使用`NSWorkspace`你也可以启动应用:

```javascript
ObjC.import('AppKit')
$.NSWorkspace.sharedWorkspace.launchApplication('/Applications/iTunes.app')
```


## 用户交互

### 警告弹窗

AppleScript: `display alert "wow"`

```javascript
app.displayAlert('wow')
//  => {"buttonReturned":"OK"}
```

AppleScript: `display alert "wow" message "I like JavaScript"`

```javascript
app.displayAlert('wow', {message: 'I like JavaScript'})
//  => {"buttonReturned":"OK"}
```

示例:

```javascript
function alert(text, informationalText) {
  var options = { }
  if (informationalText) options.message = informationalText
  app.displayAlert(text, options)
}
```

### 输入弹窗

AppleScript: `display dialog "What is your name?" default answer ""`

```javascript
app.displayDialog('What is your name?', { defaultAnswer: "" })
//    => {"buttonReturned":"OK", "textReturned":"Text you entered"}
// OR !! Error on line 1: Error: User canceled.
```

示例: 

```javascript
function prompt(text, defaultAnswer) {
  var options = { defaultAnswer: defaultAnswer || '' }
  try {
    return app.displayDialog(text, options).textReturned
  } catch (e) {
    return null
  }
}
```

### 确认弹窗

AppleScript: `display dialog "Delete resource forever?"`

```javascript
app.displayDialog('Delete resource forever?')
//    => {"buttonReturned":"OK"}
// OR !! Error on line 1: Error: User canceled.
```

示例:

```javascript
function confirm(text) {
  try {
    app.displayDialog(text)
    return true
  } catch (e) {
    return false
  }
}
```

### 列表选择弹窗

AppleScript: `choose from list {"red", "green", "blue"}`

```javascript
app.chooseFromList(['red', 'green', 'blue'])
//    => ["blue"]
// OR => false
```

带有提示文本:

AppleScript: `choose from list { "red", "green", "blue" } with prompt "What is your favorite color?"`

```javascript
app.chooseFromList(['red', 'green', 'blue'], { withPrompt: 'What is your favorite color?' })
```

多选:

AppleScript: `choose from list { "red", "green", "blue" } with prompt "What is your favorite color?" with multiple selections allowed`

```javascript
app.chooseFromList(['red', 'green', 'blue'],
  { withPrompt: 'What is your favorite color?',
    multipleSelectionsAllowed: true })
```

### 通知横幅

注意: `displayNotification`必须运行在`Application.currentApplication()`, 并且`.includeStandardAdditions = true;`

```javascript
app.displayNotification('The file has been converted',
  { withTitle: 'Success', subtitle: 'Done' })
```


### 设置音量

AppleScript: `set volume output volume 10`

```javascript
app.includeStandardAdditions = true
app.setVolume(null, {outputVolume: 100})
```
