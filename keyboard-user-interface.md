# CGDB 键盘用户接口

CGDB 通过键盘用户接口从用户那里获取输入。我们通常称键盘用户接口为 KUI。CGDB 仅需要向 KUI 获取 KUI 提供的下一个用户输入的指令。

除了读取用户输入以及提供这些输入给 CGDB 以外，KUI 还有两个主要的责任：它需要检测用户输入自定义的键盘映射与用户按下的特殊键。

用户定义的映射，或是简单映射，是用来改变输入的按键的含义。一些用户可能会称将这种功能称之为 宏 。例如：map a b。当用户输入了\字符，则 KUI 将会检测到并且替换为 \ 然后将 \ 返回给 CGDB。

当用户输入了键盘上的特殊字符时，一个键码会被发往 CGDB。例如 HOME、DEL、\ 等等。当这样的键被按下时，操作系统将会发送几个字符给应用程序，而不是像普通的按键一样仅发送一个字符。这些连结的字符被称之为一个按键序列。KUI 则负责将这些按键序列进行组合，并且向 CGDB 报告：有一个特别的按键被用户按下。ESC 键是比较特殊的，因为大多数的键码都以它为开始。它通常给出了所有的按键序列的通常的头部。KUI 使用了 terminfo 数据库去判断按键序列是由什么键码产生的。有少部分常用的按键序列被硬编码进 CGDB 中。

KUI 主要的挑战是如何判断何时一个映射或者按键序列被输入完成。KUI 有时需要读入不止一个字符去确定映射或者按键序列被输入完成。例如，用户设置了两个映射，`map abc def` 与 `map abd def`，KUI 需要在它能判断用户是否要输入一个映射之前缓存 \ 与 \ 两个字符。当下一个键被按下时，如果用户输入 \ 或是 \ 则 KUI 收到一个映射，然后将 `d e f` 返回给 CGDB。否则，没有映射被接收到，KUI 将会把 `a b` 返回给 CGDB。

选项 timeout，ttimeout，timeoutlen 以及 ttimeoutlen 可以被用来告诉 KUI 是否需要在映射或是按键序列的中间保持超时，以及如果需要的话，需要保持多久。

- **配置 KUI 的超时选项**：KUI 超时选项
- **理解键盘映射**：使用键盘映射
- **理解键码**：理解键码

## KUI 超时选项

KUI 可以被配置为在输入部分的映射或键序列时，经过一定时间间隔后超时。

当 KUI 匹配了一部分的映射或键序列时就可以超时。这意味着它将在前一个按键后等待一段时间，并且在超时后接收这个独立的按键。这是很显而易见的，因为用户在输入映射时必须独立的输入每个字符。对于部分的序列则并不这么明显，因为用户只敲击了一个按键，但是多个字符就会被发送至 CGDB。下表描述了用户可以如何配置 KUI 对键码和映射的超时选项，通过 timeout 与 ttimeout 选项。

## 使用映射

CGDB 完整支持键盘映射，允许用户改变键盘输入的含义。比如，你可以这样的射：

```
map <F2> ip<Space>argc<CR>。
```

在 CGDB 模式下，如果按下 F2 键，就会被替换成它所映射的值。 CGDB 先会收到 i 键，进入插入模式，接着 CGDB 会收到 p argc 以及一个紧跟着的回车键。

CGDB 目前支持两个映射表。用 map 命令添加的映射在 CGDB 模式下会给 CGDB 使用。要删除由 map 命令添加的映射，可以使用 unmap 命令。如果想在 GDB 模式下进行映射，可以使用 imap 命令，而 iunmap 命令则可以删除 imap 所添加的映射。例如：

```
    map ab foo
    unmap ab

    imap ab foo
    iunmap ab
```

## 理解键码

对于不大熟悉 vim 映射的人来说前面的例子需要一点解释。映射包括一个键和一个值，它们以一个空格作为分隔符分开。键和值的表示形式均不能直接含有空格，否则这些空格会被认为是分隔符。如果需要在键或者值之内包含一个空格，可以使用键码记号 <Space\>。下面是以键码记号形式表示的键码表，这些键码记号可以在所有映射命令中使用。

<table>
<tr>
<td><strong>记号</strong></td>
<td><strong>表示含义</strong></td>
</tr>
<tr>
<td>&lt;Esc&gt;</td>
<td>ESC 键</td>
</tr>
<tr>
<td>&lt;Up&gt;</td>
<td>光标向上移动</td>
</tr>
<tr>
<td>&lt;Down&gt;</td>
<td>光标向下移动</td>
</tr>
<tr>
<td>&lt;Left&gt;</td>
<td>光标向左移动</td>
</tr>
<tr>
<td>&lt;Right&gt;</td>
<td>光标向右移动</td>
</tr>
<tr>
<td>&lt;Home&gt;</td>
<td>HOME 键</td>
</tr>
<tr>
<td>&lt;End&gt;</td>
<td>END 键</td>
</tr>
<tr>
<td>&lt;PageUp&gt;</td>
<td>向上翻页</td>
</tr>
<tr>
<td>&lt;PageDown&gt;</td>
<td>向下翻页</td>
</tr>
<tr>
<td>&lt;Del&gt;</td>
<td>DELETE 键</td>
</tr>
<tr>
<td>&lt;Insert&gt;</td>
<td>INSERT 键</td>
</tr>
<tr>
<td>&lt;Nul&gt;</td>
<td>空 (NULL)</td>
</tr>
<tr>
<td>&lt;Bs&gt;</td>
<td>BACKSPACE 回退键</td>
</tr>
<tr>
<td>&lt;Tab&gt;</td>
<td>TAB 键</td>
</tr>
<tr>
<td>&lt;NL&gt;</td>
<td>换行</td>
</tr>
<tr>
<td>&lt;FF&gt;</td>
<td>换页</td>
</tr>
<tr>
<td>&lt;CR&gt;</td>
<td>回车</td>
</tr>
<tr>
<td>&lt;Space&gt;</td>
<td>空格</td>
</tr>
<tr>
<td>&lt;Lt&gt;</td>
<td>小于</td>
</tr>
<tr>
<td>&lt;Bslash&gt;</td>
<td>反斜杆 \</td>
</tr>
<tr>
<td>&lt;Bar&gt;</td>
<td>竖杠</td>
</tr>
<tr>
<td>&lt;F1&gt;-&lt;F12&gt;</td>
<td>功能键 F1 .. F12</td>
</tr>
<tr>
<td>&lt;C-&gt;</td>
<td>Ctrl 控制键</td>
</tr>
<tr>
<td>&lt;S-&gt;</td>
<td>Shift 键</td>
</tr>
</table>
