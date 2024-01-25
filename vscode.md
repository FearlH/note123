### vscode调试的时候设置参数
每个单独的参数都使用一个字符串隔开（每个空格都使用字符串隔开）

### 括号匹配
linux:ctrl+shift+\

### vscode的debug
右键点击断点可以设置断点的类型，可以设置为Expression Breakpoint，Hit Count BreakPoint 和Logpoint等.Expression Breakpoint是当满足情况的时候会在此进入断点。Hit Count BreakPoint是当命中次数达到这个次数的时候才会进入断点。Logpoint不会进入断点，但是会在命中断点的时候输出log的内容，同时可以在log里面使用{}去计算表达式的值。