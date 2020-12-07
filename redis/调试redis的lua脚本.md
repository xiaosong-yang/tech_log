&ensp;&ensp;&ensp;&ensp;有时候我们在程序中需要使用到redis的lua脚本，但是如果lua脚本中报错了，我们该怎么像普通程序一样进行单步调试呢。

## 客户端方式
- 下载Lua编辑器ZeroBrane Studio：https://studio.zerobrane.com/support，没有强制付费https://studio.zerobrane.com/download?not-this-time，可以使用免安装版。
- 下载ZeroBrane Studio的Redis扩展：https://raw.githubusercontent.com/pkulchenko/ZeroBranePackage/master/redis.lua。把它命名为redis.lua，并保存到ZeroBrane Studio执行目录下的packages文件。
- 重启ZeroBrane Studio，执行Project→Lua Interpreter→Redis


## 命令行方式
目前高版本的redis也是支持直接进行调试的，我们下载redis自带的客户端，然后在客户端所在文件内使用cmd命令：
```
redis-cli -h ip -p port  -a password  --ldb --eval year2020_ceremony_shenhao_battle.lua activity:year2020:shenhao_battle:104310:20201202 activity:year2020:shenhao_battle:20201202 , 10000000 {\"1\":18800,\"2\":28800,\"3\":38800,\"4\":48800,\"5\":58800,\"6\":58800,\"7\":58800,\"8\":28800,\"9\":38800,\"10\":48800,\"11\":58800,\"12\":58800,\"13\":58800,\"14\":58800,\"15\":38800,\"16\":48800,\"17\":58800,\"18\":58800,\"19\":58800,\"20\":58800,\"21\":58800,\"22\":48800,\"23\":58800,\"24\":58800,\"25\":58800,\"26\":58800,\"27\":58800,\"28\":58800,\"29\":58800,\"30\":58800,\"31\":58800,\"32\":58800,\"33\":58800,\"34\":58800,\"35\":58800,\"36\":58800,\"37\":58800,\"38\":58800,\"39\":58800,\"40\":58800,\"41\":58800,\"42\":58800,\"43\":58800,\"44\":58800,\"45\":58800,\"46\":58800,\"47\":58800,\"48\":58800,\"49\":58800,\"50\":58800,\"51\":58800,\"52\":58800,\"53\":58800,\"54\":58800} 104310 1606878581000
```
其中keys和参数中间有一个" , "，逗号两边有空格，都不能省略。然后即可进入调试界面，我们可以通过help查看各种调试命令：
```
lua debugger> help
Redis Lua debugger help:
[h]elp               Show this help.
[s]tep               Run current line and stop again.
[n]ext               Alias for step.
[c]continue          Run till next breakpoint.
[l]list              List source code around current line.
[l]list [line]       List source code around [line].
                     line = 0 means: current position.
[l]list [line] [ctx] In this form [ctx] specifies how many lines
                     to show before/after [line].
[w]hole              List all source code. Alias for 'list 1 1000000'.
[p]rint              Show all the local variables.
[p]rint <var>        Show the value of the specified variable.
                     Can also show global vars KEYS and ARGV.
[b]reak              Show all breakpoints.
[b]reak <line>       Add a breakpoint to the specified line.
[b]reak -<line>      Remove breakpoint from the specified line.
[b]reak 0            Remove all breakpoints.
[t]race              Show a backtrace.
[e]eval <code>       Execute some Lua code (in a different callframe).
[r]edis <cmd>        Execute a Redis command.
[m]axlen [len]       Trim logged Redis replies and Lua var dumps to len.
                     Specifying zero as <len> means unlimited.
[a]abort             Stop the execution of the script. In sync
                     mode dataset changes will be retained.

Debugger functions you can call from Lua scripts:
redis.debug()        Produce logs in the debugger console.
redis.breakpoint()   Stop execution as if there was a breakpoint in the
                     next line of code.
```



需要注意的是，不管那种方式进行调试，其实都是不会对redis里真正的key有影响，其次如果需要传入类似我上面那种json，需要传入转义之后的，就是"必须携程\"才行，否则redis无法识别。