# 脚本与配置文件的编写与修改

!!! abstract "导言"

    很多 Linux 的初学者都会对以下这些问题感到迷惑

    - Bash 语法和 C 语言类似吗？和 Powershell 类似吗？

    - Bash script 出 bug 的时候我该如何调试呢？

    下面内容可以解答你的疑问。

## Shell 脚本 {#shell-scripts}

### 什么是 Shell {#what-is-shell}

Shell 是 Linux 的命令解释程序，是用户和内核之间的接口。除了作为命令解释程序外，Shell 同时还提供了一个可支持强大脚本语言的程序环境。

### Bash {#bash}

Bourne Shell (`sh`)，是 Unix 系统的默认 Shell，简单轻便，脚本编程功能强，但交互性差。

Bourne Again Shell，即 Bash，是 GNU 开发的一个 Shell，也是大部分 Linux 系统的默认 Shell，是 Bourne Shell 的扩展。

#### `Bash` 的特点

- 支持 I/O 重定向（`>`, `>>`, `<`）和管道（`|`）等。

- 环境控制，允许用户定制环境以满足自己需要。环境文件 `.bash_profile`、`.bashrc`、`.bash_logout`。通过配置合适的环境变量，可以改变主目录、命令提示符、命令搜索路径等用户工作环境。

- 支持后台运行 `&`。

- 占用资源较少，来自 GNU，与 Linux 相容性高。支持命令行编辑，提供命令补全功能键 Tab。

- 允许应用别名代替命令关键字（`alias name='命令'`）。

### Bash 脚本基础 {#bash-usage}

#### Bash 脚本的运行

可以使用几种方法运行 Bash 脚本：

- 在指定的 Shell 下执行，将脚本程序名作为 Shell 的第一个参数。

  ```shell
  $ bash show.sh [option];
  ```

- 使用「.」命令执行脚本，「.」后要有空格。

  ```shell
  $ . ./show.sh [option];
  ```

- 将脚本设置为可执行，然后当做外部命令执行执行。

  ```shell
  $ chmod a+x showinfo
  $ ./show.sh [option];
  ```

许多 Bash 脚本会在文件首行加上 `#!/bin/bash` 。这里 `#!` 符号的名称是 shebang （也叫 sha-bang，即 sharp `#` 与 bang `!`）。当一个文本文件首行有 shebang，且以可执行模式执行时， shebang 后的内容会看作这个脚本的解释器和相关参数，系统会执行解释器命令，并将脚本文件的路径作为参数传递给该命令。

例如，某个 `foo.sh` 首行为 `#!/bin/bash`，则执行 `./foo.sh` 就等于执行 `/bin/bash ./foo.sh`。

Bash 也支持在同一个行中安排多个命令：

| **分隔符** | **说明**                                                        |
| ---------- | -------------------------------------------------------------- |
| `;`        | 按命令出现的先后，顺序执行                                        |
| `&&`       | 先执行前面的命令，若成功，才接着执行后面命令；若失败，不执行后面命令  |
| `||`       | 先执行前面的命令，若成功，不执行后面命令；若失败，才执行后面命令      |
| 后缀 `&`   | 后台方式执行命令                                                  |

组命令：

- 使用 `{ 命令1; 命令2; … }`，组命令在 shell 内执行，不会产生新的进程，注意花括号和命令之间的空格。

- 使用 `(命令1; 命令2; …)`，组命令会建立独立的 shell 子进程来执行组命令，这里的圆括号周围并不需要空格。

??? example "组命令示例"

    ```shell
    ➜  ~ tree tmp
    tmp
    └── temp
    #------
    ➜  ~ (cd tmp; ls;)
    temp
    ➜  ~
    #------
    ➜  ~ { cd tmp; ls; }
    temp
    ➜  tmp
    ```

    执行组命令 `{ cd tmp; ls; }` 后，当前目录会被修改，但是执行 `(cd tmp; ls;)` 不会修改当前目录。

#### shell 变量 {#bash-variable}

像大多数程序设计语言一样，shell 也允许用户在程序中使用变量。但 shell 不支持数据类型，它将任何变量值都当作字符串。但从赋值形式上看，可将 shell 变量分成四种形式：用户自定义、环境变量、位置变量和预定义特殊变量。

##### 用户自定义变量

变量定义：`name=串`，其中 `=` 两边不允许有空格。如果字串中含空格，就要用双引号括起。在引用时，使用 `$name` 或 `${name}`，后者花括号是为了帮助解释器识别变量边界。

已定义的变量可以通过 `unset name` 来删除。

??? example "变量使用示例"

    变量定义：

    ```shell
    for skill in Ada Coffee Action Java; do
        echo "I am good at ${skill}Script"
    done
    ```

    输出：

    ```
    I am good at AdaScript
    I am good at CoffeeScript
    I am good at ActionScript
    I am good at JavaScript
    ```

    如果不给 `skill` 加花括号，写成 `echo "I am good at $skillScript"` ，解释器就会把 `$skillScript` 当成一个变量（其值为空）。


    删除变量：

    ```shell
    Today=1024
    unset Today
    echo $Today
    ```

    输出为空

##### 环境变量

每个用户登录系统后，Linux 都会为其建立一个默认的工作环境，由一组环境变量定义，用户可以通过修改这些环境变量，来定制自己工作环境。在 Bash 中，可用 `env` 命令列出所有已定义的环境变量。通常，用户最关注的几个变量是：

- `HOME`：用户主目录，一般情况下为 `/home/用户名`。

- `LOGNAME`：登录用户名。

- `PATH`：命令搜索路径。

- `PWD`：用户当前工作目录路径。

- `SHELL`：默认 shell 的路径名。

- `TERM`：使用的终端名。

##### 位置变量

- Shell 解释用户的命令时，把命令程序名后面的所有字串作为程序的参数。分别对应 `$1`, `$2`, `$3`, ..., `$9`，程序名本身对应 `$0`。

- 可用 `shift n` 命令，改变命令行参数与 `$1`, `$2`, `$3`, ... 的对应关系。

- 可用 `set` 命令，直接给 `$1`, `$2`, `$3`, ... 等赋值。

??? example "范例"

    ```shell
    $ set one two three
    $ echo $1 $2 $3
    one two three
    $ shift 2
    $ echo $1 $2 $3
    three
    ```

##### 特殊变量

Shell 中还有一组有 shell 定义和设置的特殊变量，用户只能引用，而不能直接改变或重置这些变量。

| 特殊变量     | 说明                                           |
| ------------ | ---------------------------------------------- |
| `$#`         | 命令行上的参数个数，不包括 `$0`                |
| `$?`         | 最后命令的退出代码，0 表示成功，其它值表示失败 |
| `$$`         | 当前进程的 PID                                 |
| `$!`         | 最近一个后台运行进程的进程号                   |
| `$*`         | 命令行所有参数构成的一个字符串                 |
| `$@`         | 用双引号括起的命令行各参数拼接构成的一个字符串 |

##### 特殊字符

- 双引号，能消除空格、制表符的特殊含义，但不能消除很多其他特殊字符的特殊含义。

- 单引号，能消除所有特殊字符的特殊含义，包括反斜杠，因此单引号字符串中不能使用反斜杠转义单引号本身。

- 反引号括起的字符串，被 shell 解释为命令，执行时用命令输出结果代替整个反引号对界限部分。

  - 与反引号相同的语法是 `$(command)`，它的好处是界限更明确，且可以嵌套。

- 反斜杠，消除单个字符的特殊含义。

#### 算术运算 {#arithmetic-ops}

在 Bash 中进行算术运算，需要使用 `expr` 计算算术表达式值或 `let` 命令赋值表达式值到变量。基本运算符是 `+`、`-`、`\*` (转义)、`/`、`%`。在 `expr` 中，运算符两边与操作数之间必须有空格，小括号要转义；但 `let` 则没有这个要求，运算符前后有无空格均可，小括号不需转义，但 `=` 前后不能有空格。

另外，所有标准的 shell 都支持另一种语法 `(( 表达式 ))`，其中 `表达式` 是一个 C 风格的数学表达式，可以计算，也可以复制。`(( 表达式 ))` **是一条完整的命令**，命令的返回值为 0 或 1，它的真假性与表达式在 C 中的真假性相同。也就是说，若表达式的结果非零，那么 `(())` 命令返回零，而当表达式结果为零时命令返回 1，这是因为 shell 中用零表示真值，这一点与 C 语言恰好想法（回想一下，C 语言中 `return 0` 表示**正常**退出）。

使用 `$(( 表达式 ))` 可以将计算结果用作为命令行的一部分。

??? example "`expr` 和 `let` 使用示例"

    ```shell
    $ expr length "ustclug"
    7
    $ let a=0
    $ echo $a
    0
    $ let a++
    $ echo $a
    1
    $ ((a+=1))
    $ echo $a
    2
    $ echo $((a+=a/a))
    3
    ```

#### 条件表达式 {#expr}

条件表达式写成 `test 条件表达式`，或 `[ 条件表达式 ]`，注意表达式与方括号之间有空格。

- 字符串比较

| 表达式                            | 含义                                       |
| --------------------------------- | ------------------------------------------ |
| `string1 = string2`               | 如果两个串相等，则结果为真 (true: 0)       |
| `string1 != string2`              | 如果两个串不相等，则结果为真               |
| `string` 或 `-n string`           | 如果字符串 string 长度不为 0，则结果为真   |
| `-z string`                       | 如果字符串 string 长度为 0，则结果为真     |

- 数值比较

表达式：`int1 [option] int2`，其中的参数可以用下列替换。

| 参数   | 说明     |
| ------ | -------- |
| `-eq`  | 等于     |
| `-ne`  | 不等于   |
| `-gt`  | 大于     |
| `-ge`  | 大于等于 |
| `-lt`  | 小于     |
| `-le`  | 小于等于 |

- 文件状态

| 表达式       | 含义                 |
| ------------ | -------------------- |
| `-r file`    | 文件存在且可读       |
| `-w file`    | 文件存在且可写       |
| `-x file`    | 文件存在且可执行     |
| `-f file`    | 文件存在且为普通文件 |
| `-d file`    | 文件存在且为目录     |
| `-s file`    | 文件存在且长度大于 0 |

- 复合逻辑表达式

| 表达式              | 含义      |
| -----------------   | --------- |
| `! expr`            | 否运算    |
| `expr1 –a expr2`    | 与运算    |
| `expr1 –o expr2`    | 或运算    |

#### 流程控制 {#flow-control}

- if

序列中可嵌套 if 语句，在 shell 中也允许有多个 elif ，但 shell 的流程控制不可为空。末尾的 `fi` 就是 `if` 倒过来写，后面还会遇到类似的。

```shell
if condition1
then
  command1
elif condition2
then
  command2
else
  command3
fi
```

- case

选项值必须以右括号 `)` 结尾，若匹配多个离散值，用 `|` 分隔。这里的 `esac` 也是 `case` 倒着写。

```shell
case <variable> in
value1|value2)
  command1
  command2
  ;;
value3)
  command3
  ;;
*)
  command4
  ;;
esac
```

- for

```shell
for var in list
do
  commands $var
done
```

- while

while 循环用于不断执行一系列命令，命令通常为测试条件。

```shell
while condition
do
  commands
done
```

??? example "流程控制样例脚本"

    ```shell
    MAX_NO=0
    read -p "Enter Number between (5 to 9) : " MAX_NO
    if ! [ $MAX_NO -ge 5 -a $MAX_NO -le 9 ] ; then
      echo "I ask to enter number between 5 and 9, Okay"
      exit 1
    fi

    clear

    for (( i=1; i<=MAX_NO; i++ ))
    do
      for (( s=MAX_NO; s>=i; s-- ))
      do
        echo -n " "
      done
      for (( j=1; j<=i;  j++ ))
      do
        echo -n " ."
      done
      echo ""
    done

    for (( i=MAX_NO; i>=1; i-- ))
    do
      for (( s=i; s<=MAX_NO; s++ ))
      do
        echo -n " "
      done
      for (( j=1; j<=i;  j++ ))
      do
        echo -n " ."
      done
      echo ""
    done
    ```

    输出结果：

    ```shell
    Enter Number between (5 to 9) : 9

             .
            . .
           . . .
          . . . .
         . . . . .
        . . . . . .
       . . . . . . .
      . . . . . . . .
     . . . . . . . . .
     . . . . . . . . .
      . . . . . . . .
       . . . . . . .
        . . . . . .
         . . . . .
          . . . .
           . . .
            . .
             .
    ```

除此之外，用于流程控制的还有 `until`（处理与 `while` 恰恰相反）、在 C 语言中同样常见的 `break n` 和 `continute n`（参数 `n` 均表示跳过 n 层循环）。

#### 函数 {#functions}

与其他编程语言类似，shell 也可以定义函数。其定义格式为：

```shell
# POSIX syntax
name() {
    actions
    [return <int>]
}

# Bash syntax
funcion name {
    action
    [return <int>]
}
```

其中 function 和函数参数可以省略。返回参数可以显示用 `return` 返回，或以最后一条命令运行结果作为返回值。在函数中使用 `return` 会结束本次函数执行，而使用 `exit` 会直接结束退出包含函数的当前脚本程序。

函数在使用前必须定义，因此应将函数定义放在脚本开始的部分。在调用函数时仅使用其函数名即可。

??? example "范例"

    某带函数的某脚本程序内容如下：

    ```shell
    #!/bin/sh
    hello() {
      echo "hello, today's date is `date`"
    }
    echo "going to call test function:"
    hello
    ```

    运行脚本，输出结果：

    ```text
    going to call test function:
    hello, today's date is <当前日期显示输出串>
    ```

### Bash 脚本调试 {#bash-debugging}

Bash shell 本身提供了调试方法：

- 命令行中：`$ sh -x script.sh`。

- 脚本开头：`#!/bin/sh -x`。

- 在脚本中用 set 命令调整（`set -x` 启用，`set +x` 禁用）。

其中参数选项可以更改，`-n`：读一遍脚本中的命令但不执行，用于检查语法错误；`-v`：一边执行脚本、一边将执行过的脚本命令打印到标准输出；`-x`：提供跟踪执行信息，将执行的每一条命令和结果依次打印出来。注意避免几种调试选项混用。

除了 Bash shell 内置的选项，还有 [BASH Debugger](http://bashdb.sourceforge.net/)、[shellcheck](https://github.com/koalaman/shellcheck) 等第三方脚本分析工具。
