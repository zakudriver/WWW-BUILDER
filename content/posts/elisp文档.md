+++
title = "elisp文档"
author = ["zakudriver"]
description = "emacs lisp 指南"
date = 2021-07-25
lastmod = 2022-07-22T10:14:38+08:00
tags = ["emacs"]
categories = ["docs"]
draft = false
+++

## 有三种方式可以加载文件： {#有三种方式可以加载文件}

-   load
-   autoload
-   require


## 使用eval-after-load可以推迟一段代码的执行 {#使用eval-after-load可以推迟一段代码的执行}

(eval-after-load "触发条件的文件" 待执行的代码)
这里，第一个参数的值必须跟上面三种方式加载文件时的值一模一样


## emacs中的变量作用域 {#emacs中的变量作用域}


### buffer-local变量 {#buffer-local变量}


#### 声明buffer-local变量 {#声明buffer-local变量}

-   make-variable-buffer-local

各个缓冲区都有各自的buffer-local变量

-   make-local-variable

当前缓冲区产生一个局部变量,其他缓冲区仍然使用全局变量(推荐使用)


#### buffer相关函数 {#buffer相关函数}

<!--list-separator-->

-  with-current-buffer

    ```elisp
    ;使其中的body表达式在指定的缓冲区里执行(使用指定buffer的配置信息执行body表达式)
    (with-current-buffer buffer
    body)
    ```

<!--list-separator-->

-  get-buffer

    ```elisp
    ;得到缓冲区名字的对应缓冲区对象,如果没有这个名字的缓冲区,返回nil
    (get-buffer buffer-name)
    ```

<!--list-separator-->

-  default-value

    ```elisp
    ;访问符号的全局变量的值
    (default-value symbol)
    ```

<!--list-separator-->

-  setq-default

    ```elisp
    ;修改符号作为全局变量的值
    (setq-default symbol-name)
    ```

<!--list-separator-->

-  local-variable-p

    ```elisp
    ;测试变量是不是buffer-local的
    (local-variable-p symbol [buffer对象])
    ```

<!--list-separator-->

-  buffer-local-value

    ```elisp
    ;得到其他缓冲区内的buffer-local变量值
    (buffer-local-value 符号 buffer对象)
    ```

<!--list-separator-->

-  default-boundp

    ```elisp
    ;判断符号是否有全局值
    (default-boundp 符号)
    ```

<!--list-separator-->

-  makunbound

    ```elisp
    ;使一个变量的值重新为空
    (makunbound 符号)
    ```

<!--list-separator-->

-  kill-local-variable

    ```elisp
    ;清除一个buffer-local变量
    (kill-local-variable 符号)
    ```

<!--list-separator-->

-  kill-all-local-variables

    ```elisp
    ;清除所有的buffer-local变量,但是带有属性permanent-local的不会清除,带有这些标记的变量一般都是和缓冲区模式无关的
    (kill-all-local-variables)
    ```


#### 文件中的本地变量列表 {#文件中的本地变量列表}

-   "本地变量列表"可以位于任何文件的结尾位置,它的形式如下所示

<!--listend-->

```elisp
Local variables:
var1 : value1
var2 : value2
...
eval : form1
eval : form2
...
End:
```

这里定义的var自动是buffer-local的,类似宏一样,value被认为是被引用的
这里的form则会自动执行.

-   可以通过在.emacs中加入

<!--listend-->

```elisp
(setq enable-local-variables 'query)
```

来阻止"本地变量列表"生效


### 变量本地化 {#变量本地化}

-   普通变量通过make-localvariable或make-variable-buffer-local变为buffer－local的
-   make-variable-buffer-local使指定的变量在每个buffer中都是独立的
-   make-local-variable使变量在当前buffer是独立的，而其他buffer依然共享全局变量
-   hook变量智能通过make-local-hook变为buffer-local的


## elisp中的数据类型 {#elisp中的数据类型}


### Number {#number}


#### Integer {#integer}

根据机器的不同,Integer有不同的区间(分别由变量most-positive-fixnum和most-negative-fixnum来指示). 但elisp规定最少的范围是从-536870912到536870911.

任何超过Integer范围的值都认为是Float型

默认,Integer用10进制来表示数,但也可以用

-   \#bNNNN表示二进制数
-   \#oNNNN表示八进制数
-   \#xNNNN表示十六进制数
-   \#MrNNNN表示M进制的数


#### Float {#float}

Elisp中Float的取值范围与C中的double型一样.

Float使用INF表示无穷大. Float还有一个特殊值名为NaN. 当数学函数计算非法表达式时会返回该值.

```emacs-lisp
(/ 0 0.0)                               ;-0.0e+NaN
(/ -0 0.0)                              ;-0.0e+NaN
(/ 1 0.0)                               ;1.0e+INF
(/ -1 0.0)                              ;-1.0e+INF

```

<!--list-separator-->

-  相关函数

    -   判断是否为NAN

        (isnan x)

    -   (frexp x)

        返回一个元组(S . E),使得S\*2^E == x,S的值在[0.5 1)

    -   (ldexp S E)

        返回Float值x 使得x == S\*2^E

    -   (copysign x1 x2)

        假设x1 `= S1*2^E1, x2 =` S2\*2^E2,则返回S1\*2^E2的值

    -   (logb x)

        返回log2(x)的值,向下取整. 例如(logb 10) =&gt; 3


### Sequence {#sequence}

-   获取sequence的长度

    (length mySequence)

需要注意的是length不能对点列表和环形列表求长度,但是可以用safe-length代替


#### Array {#array}

除了char-table之外,创建其他类型的Array都需要指定一个固定长度.

char-table的长度是由character code的范围所决定的,不能人工指定

<!--list-separator-->

-  vector

    vecotr是sequence的一个子类型

    -   如何创建一个vector
        -   使用字面量语法:[val1 val2 val3...]

    -   使用函数(vector val1 val2 val3...)

    -   获取vector的长度

    (length myVector)

    -   获取element

    (elt mySeq index)    # index从0开始

    -   设置element

    (aset mySeq index value)

    -   将多个Sequence组合成一个vector

    (voncat seq1 seq2 ...)

    -   将vector转换成list

    (append myVector nil)

<!--list-separator-->

-  bool-vector

    -   bool-vector只能存放nil或t
    -   bool-vector的输出格式与字符串类似,但在前面加上\`#&amp;长度\`作为标识:
        ```emacs-lisp
        #&长度"内容"                            ;这里的长度表示bool-vector的长度,内容的二进制为vector的内容,内容的现实方式为字符串
        (make-bool-vector 3 t)                  ; => #&3"^G" ?\C-g == 7
        (make-bool-vector 3 nil)                ; => #&3"^@" ?\C-@ == 0
        (equal #&3"\377" #&3"\007")             ; => t

        ```

<!--list-separator-->

-  char-tables

    -   类似vector,但它使用Character作为索引.
    -   char-tables的输出格式与vector类似,但前面加上\`#^\`作为标示
    -   每个char-table对象都有一个symbol类型的"subtype",可以用于标识char-table的用处, 使用函数char-table-subtyple来查询该subtype
    -   char-table的subtype中的属性\`char-table-extra-slots\`决定了该char-table的扩展slot的个数(0-10之间)
    -   每个char-table都可以有一个父char-table,子char-table从父char-table中继承索引的值.
    -   char-table还能够设定一个默认值.若发现char-table指定index的值为nil,则返回该默认值.

<!--list-separator-->

-  string

    -   elisp中的string是不可变的.
    -   string中不能包含?\H-N ?\A-N ?\s-N 这些字符
    -   string中可以包含文本属性,包含文本属性的输出格式为:
        ```emacs-lisp
        #("characters" property-data...)
        ```

        -   Emacs中,对字符串作比较的函数只有string=,string&lt;函数,没有string&gt;函数


#### List {#list}

list是sequence的一个子类型

-   创建一个list

-   使用list函数:(list v1 v2...)

-   使用字面表达式'(v1 v2...)

-   获取element

    | Function       | Purpose                     |
    |----------------|-----------------------------|
    | (car ℓ)        | first element               |
    | (nth n ℓ)      | nth element (start from 0)  |
    | (car (last ℓ)) | last element                |
    | (cdr ℓ)        | 2nd to last elements        |
    | (nthcdr n ℓ)   | nth to last elements        |
    | (butlast ℓ n)  | without the last n elements |

-   获取list的长度

(length mySeq)

-   在list头添加element

(cons x myList)

-   合并两个list

(append list1 list2)

由此可以得出,在list尾部添加element的方法为

(append myList (list myVal))

-   修改list的函数

    | Function       | Purpose                                                                          |
    |----------------|----------------------------------------------------------------------------------|
    | (pop ℓ)        | Remove first element from the variable. Returns the removed element.             |
    | (nbutlast ℓ n) | Remove last n elements from the variable. Returns the new value of the variable. |
    | (setcar ℓ x)   | replaces the first element in ℓ with x. Returns x.                               |
    | (setcdr ℓ x)   | replaces the rest of elements in ℓ with x. Returns x.                            |

<!--list-separator-->

-  association list / alist

    alist中的item是顺序的,且可以有重复键值

    在elisp中,alist中可以存在某些element不是cons cell的情况,alist查询函数会自动略过这些异常的element
    association list类似于C语言中的map,它的结构为

    ```lisp
    (defvar *alist-example* '((key1 value1)
                    (key2 value2)))
    ```

<!--list-separator-->

-  Property list / plist

    plist的作用类似alist,也是用list表示一组键值对.

    plist的key为symbol,它的结构为

    ```emacs-lisp
    (key1 value1 key2 (value2) key3 "value3") ;=>键值对的对应关系为key1->value1;key2->'(value2);key3->"value3"
    ```

    每一个symbol都可以attach一个plist

    需要注意的是,plist中的 **key必须是唯一的** ,相比来说alist就没有这个限制了.


### hashtable {#hashtable}

hash table类似alist一样提供了键值配对的功能. 但比起alist来说,有如下三个方面的不同

-   在搜索大量的键值对集合时,使用hash table的搜索速度比alist快得多
-   hash table中的的item是非排序的,不能有重复键值
-   两个hash table对象无法共享同一个结构体,而两个alist对象之间有可能使用共同的tail

hash table的输出格式以#s开头后接hash table的属性和内容

```text
#s(hash-table size 65 test eql rehash-size 1.5 rehash-threshold 0.8 data
              ())
```


### symbol {#symbol}

-   Lisp中symbol类似于其他语言中的变量,但是它不仅仅只存储一个值.
-   列表的第一个表达式如果是一个符号,解释器会查找这个表达式的函数值.如果函数值是另一个符号,则会继续查找这个符号的函数值
-   symbol中,\\失去了转义的功能,因此'\t就是't而不时'&lt;TAB&gt;
-   一个符号既可以有value,也可以有function,即一个symbol可以同时求值和当作函数用


#### 符号的组成 {#符号的组成}

-   符号名称:使用函数symbol-name获取

-   符号值: 使用函数symbol-value获取

-   使用boundp测试符号是否有值
-   以\`:\`开头的符号的值不能变

-   函数:  使用symobl-function获取
-   使用fboundp测试符号是否设置了函数
-   其实function slot不仅仅可以存放function,还可以存放macro,symbol,keyboard marcro,keympa和autoload object

-   属性列表:使用symbol-plist获取
-   使用get/put来访问/修改某个属性值
-   使用plist-get/plist-put来访问/设置属性列表中的属性
-   属性列表是一个形如(prop1 value1 prop2 value2)的关联列表,但无法删除一个属性
-   可用使用属性列表来存储function的状态


#### 符号的形成 {#符号的形成}

当elisp读取到一个symbol时,它会先在一个名为\`obarray\`的vector中检查是否已经存在这个symbol, 若不存在,则elisp reader创建新的symbol并添加到obarray中(创建并添加symbol的过程被成为"interning",而该symbol被成为是"interned symbol").

如果想避免intern一个symbol,可以在符号名前加上\`#::\`,这些符号被称为\`uninterned symbols\`

```elisp
(intern-soft "abc") ; => nil
'abc ; => abc
(intern-soft "abc") ; => abc
(intern-soft "abcd") ; => nil
'#:abcd ; => abcd
(intern-soft "abcd") ; => nil
```

obarray也是一种类型名称

由于在elisp中,obarray就是一个vector,因此可以使用(make-vecotr LENGTH 0)来创建一个空的obarray, 而要把symbol插入一个obarray,则必须使用如下的intern系列函数来进行.

-   intern
-   intern-soft
-   unintern
-   mapatoms


### Char {#char}

类似C,Elisp中的Char其实就是integer(值从0到(max-char)的范围都可以认为是character). Char类型的字面量结构为\`?字母\`,例如

```emacs-lisp

(message "%d" ?A)      ;65
(message "%d" ?\.)     ;46 标点一般在前面加\
(message "%d" ?我)     ;25105

?\a ;=> 7                 ; control-g, `C-g'
?\b ;=> 8                 ; backspace, <BS>, `C-h'
?\t ;=> 9                 ; tab, <TAB>, `C-i'
?\n ;=> 10                ; newline, `C-j'
?\v ;=> 11                ; vertical tab, `C-k'
?\f ;=> 12                ; formfeed character, `C-l'
?\r ;=> 13                ; carriage return, <RET>, `C-m'
?\e ;=> 27                ; escape character, <ESC>, `C-['
?\s ;=> 32                ; space character, <SPC>
?\\ ;=> 92                ; backslash character, `\'
?\d ;=> 127               ; delete character, <DEL>
?\uNNNN                                 ;这里N为16进制数,表示U+NNNN的Unicode字符
?\U00NNNNNN                             ;

?\x十六进制代码            ; ?\x41 = \?A
?\三个八进制代码           ; ?\001 = `C-a`

```

-   没有charp,因为字符就是整数,但是有char-or-string-p函数
-   使用char-equal比较字符时,需要注意case-fold-search变量的值,t表示忽略大小写


#### Control characters {#control-characters}

Ctrl-N的字面表达式为\`?\\^字母\`(这里的字母不区分大小写)

```emacs-lisp

?\^I ; == ?\^i  == C-i == 9
```

还可以表示为\`?\C-字母\`

```emacs-lisp
?\C-g                                   ;== ?\^g == C-g
```

需要注意的是,?\C-G不时?\C-S-g的意思,它跟?\C-g的意义一样
由于历史的原因,Emacs把&lt;DEL&gt;看成是C-?


#### Meta Character {#meta-character}

M-N的字面表示方式为\`?\M-字母\`


#### Shift Character {#shift-character}

?&sect;-N


#### Alt Character {#alt-character}

?\A-N


#### Hyper Character {#hyper-character}

?\H-N


#### Super Character {#super-character}

?\s-N


### 其他类型 {#其他类型}

-   Buffer
-   Marker
-   Window
-   Frame
-   Terminal
-   Window Configuration
-   Frame Configuration
-   Process
-   Stream
-   nil指\`standard-input\`和\`standard-output\`
-   t指代\`从minibuffer输入\`或输出到\`echo area\`
-   Keymap
-   Overlay

Overlay用来給buffer的一部分内容加上不同的显示风格

-   Font


### 循环结构 {#循环结构}

需要使用\`#N=\`和\`#N#\`来定义循环点和引用循环点,这里N为数字

```emacs-lisp

'(#1=(a) b #1#)
#1='(a #1#)
```


### 数据类型之间的转换 {#数据类型之间的转换}

-   number-to-string / string-to-number
-   concat可以将序列转换成字符串

<!--listend-->

```elisp
(concat '(?a ?b ?c ?d ?e)) ; => "abcde"
(concat [?a ?b ?c ?d ?e]) ; => "abcde"
```

-   vconcat可以把字符串转换成向量

<!--listend-->

```elisp
(vconcat "abdef") ; => [97 98 100 101 102]
```

-   append可以把字符串转换成一个列表

<!--listend-->

```elisp
(append "abcdef" nil) ; => (97 98 99 100 101 102)
```

-   (byte-to-string BYTE)
-   大小写转换

elisp通过case table来确定大小写的对应关系,每个buffer都可以设置自己的case table.

-   (downcase string-or-char)

-   (upcase string-or-char)

-   (capitalize string-or-char)

    string中的所有单词都被格式化为capitalize的格式

    若参数为char类型,则效果跟upcase一样

-   (upcase-initials string-or-char)

    对string中的每个单词的第一个字母转化了大写字母,其他字母不变.

-   转换数字为Float型

(float number)

-   转换数字为Integer型

有8个函数可以用来转换Float到Integer型. 有些函数都带有一个可选参数DIVISOR, 若传入了DIVISOR则返回NUMBER/DIVISOR的整数化的值. 若DIVISOR为0,则Elisp报arith-error

(truncate number &amp;optional divisor) / (ftruncate float)

截断小数位

(floor number &amp;optional divisor) / (ffloor float)

截断成一个更小的整数

(ceiling number &amp;optional divisor) / (fceiling float)

截断成一个更大的整数

(round number &amp;optional divisor) / (fround float)

转换成最近的整数,若小数为为0.5,则转换为偶数,例如(round 1.5)=&gt;2 (round 2.5)=&gt;2

-   字符串与数字之间的相互转换

(string-to-number "N" &amp;optional base)

(number-to-string N)

(format "%d" N)

-   Object转换为String

(format "%s" object)

(prin1-to-string object)

-   String转换为Object

(read-from-string string)

-   symbol转换为string

(symbol-name symbol)

-   string转换为symbol

(intern string)


### 类型判断 {#类型判断}

-   (cl-typep object type)

检查object是否为type类型.

参数type为common lisp形式

-   (type-of object)

返回object的 **基本** 类型


## elisp中的等于 {#elisp中的等于}


### eq {#eq}

eq用于判断两个Object是否为同一个Object


### equal {#equal}

equal用于判断两个Object是否有相同的内部结构(这通常意味着两个Object的类型是一样的,即0 不equal 0.0).

equal在比较两个string时,不会比较他们的属性,需要比较属性的话,使用equal-including-properties来代替

即使两个buffer的内容一样,equal这两个buffer时也不相等

由于equal会对两个Object的各个组成部分递归调用equal,因此在处理循环结构时,可能会陷入死循环.


### = {#43ec3e}

=一般用于数字之间的比较,且0=0.0


### /= {#2c1b5c}

数字间的不等于


### eql {#eql}

eql类似eq,但在处理数字是,会同时比较数字的类型和值.因此(eql 1.0 1)=&gt;nil;(eql 1.0 1.0)=&gt;t


### char-equal {#char-equal}

比较两个character是否相等,默认忽略大小写差异,若变量\`case-fold-search\`nil(默认为t),则比较时区分大小写

```emacs-lisp
(char-equal ?x ?x)                      ; => t
(let ((case-fold-search nil))
  (char-equal ?x ?X))                   ; => nil
```


### string= {#string}

string=接收string或symbol的参数. 使用string=的时候一定会区分大小写,而跟变量\`case-fold-search\`无关. 而且在比较时,string的text properties不参与比较.

```emacs-lisp
(string= "abc" 'abc)                    ;=>t
```


### string-equal {#string-equal}

string=的别名


### equal-includeing-properties {#equal-includeing-properties}


## 变量名命名习惯 {#变量名命名习惯}

-   -hook 一个在特定情况下调用的函数列表，比如关闭缓冲«时，进入某个模式时。
-   -function值为一个函数
-   -functions 值为一个函数列表
-   -flag 值为nil或non-nil
-   -predicate 值是一个作判断的函数，返回nil或non-nil
-   -program 或-command 一个程序或shell名令名
-   -form 一个表达式
-   -forms 一个表达式列表。
-   -map 一个按键映射（keymap）
-   命名以空格开头的缓冲区是临时的,用户不需要关系的缓冲区


## 获取参数的几种方法 {#获取参数的几种方法}


### 变量\`current-prefix-arg\`获取universal-argument {#变量-current-prefix-arg-获取universal-argument}

emacs命令可以使用C-u传递universal-argument.

| Key Input                | Value of current-prefix-arg |
|--------------------------|-----------------------------|
| No universal arg called. | nil                         |
| 【Ctrl+u -】             | Symbol -                    |
| 【Ctrl+u - 2】           | Number -2                   |
| 【Ctrl+u 1】             | Number 1                    |
| 【Ctrl+u 4】             | Number 4                    |
| 【Ctrl+u】               | List '(4)                   |
| 【Ctrl+u Ctrl+u】        | List '(16)                  |


### interactive {#interactive}

-   若interactive的参数以\*开头，则意义是，如果当前buffer是只读的，则不执行该函数
-   interactive可以后接字符串,表示获得参数的方式
-   p 接收C-u的数字参数

    也可以不用P参数,直接在代码中判断current-prefix-arg的值
-   r region的开始/结束位置
-   n 提示用户输入数字参数,n后面可用接着提示符
-   s 提示用户输入字符串参数
-   若函数接收多个input,需要用\n来分隔
-   interactive可以后接一个form,form的求值结果应该是一个list,这个list的值作为参数的实参

在form中一般会用到如下几个函数用于获取用户输入

-   read-string
-   read-file-name
-   read-directory-name
-   read-regexp
-   y-or-n-p
-   read-from-minibuffer
-   使用变量\`current-prefix-arg\`来判断是否有universal-argument


## 光标位置 {#光标位置}


### 函数 {#函数}

-   获取光标当前位置

(point)

-   获取region的开始和结束位置

(region-beginning) / (region-end)

-   当前行的开始/结束位置

(line-beginning-position) / (line-end-position)

-   获取当前buffer的开始/结束位置

(point-min) / (point-max)

-   得到行号

    line-number-at-pos

-   测试是否在buffer头/尾

    bobp(beginning of buffer predicate)和eobp(end of buffer predicate)

-   测试是否在行首/尾

    bolp(beginning of line predicate)和eolp(end of line predicate)


## 光标移动 {#光标移动}


### 函数 {#函数}

-   按单个字符移动

    goto-char /forward-char /backward-char

-   跳转到指定字符串的位置

    (search-forward myStr)  ;光标位于myStr的尾部

    (search-backward myStr) ;光标位于myStr的头部

-   正则查询

(re-search-forward myRegex) / (search-forward-regexp myRegex)

(re-search-backward myRegex) / (search-backward-regexp myRegex)

-   跳到第一个不是"a"-"z"的位置

(skip-chars-forward "a-z")

(skip-chars-backward "a-z")

-   跳到buffer的开头/末尾

    beginning-of-buffer / end-of-buffer

-   按词移动

    forward-word /backward-word

-   按行移动

    没有backward-line,而且每次移动都是移动到行首的,所以(forward-line 0)可以移动到当前行的行首

    (forward-line N) N可以为负数,表示backward-line

-   移动到行首/尾

(beginning-of-line)

(end-of-line)


## 控制结构 {#控制结构}


### 顺序结构 {#顺序结构}

-   (progn bodys)

顺序执行bodys,bodys中的最后一个form的返回值为progn的返回值

-   (prog1 form1 bodys)

类似progn,但form1的返回值为prog1的返回值

-   (prog2 form1 form2 bodys)

类似progn,但form2的返回值为prog2的返回值


### 条件表达式 {#条件表达式}

-   (if condition then-form else-bodys)

注意,then-form只能是一句form,而else-bodys可以为多个form,事实上它包含了一个隐含的progn

-   (when condition then-bodys)

-   (unless condition else-bodys)

-   (cond clauses)

一个clause的格式为(condition bodys)

clause的格式也可以为(condition),这样的话,cond的返回值为非nil的condition的返回值

-   (pcase EXP BRANCH1 BRANCH2 BRANCH3 ...)

一个BRANCH的格式为(UPATTERN BODYS)

pcase先计算EXP的值,并将值与各个BRANCH中的UPATTERN相比较,若相等,则执行相应的BODYS

```emacs-lisp
(pcase (get-return-code x)
  ('success       (message "Done!"))
  ('would-block   (message "Sorry, can't do it now"))
  ('read-only     (message "The shmliblick is read-only"))
  ('access-denied (message "You do not have the needed rights"))
  (code           (message "Unknown return code %S" code)))
```

UPATTERN可以是下面几种格式

-   \`(QPATTERN1 . QPATTERN2)
    该模式匹配一个cons cell,它的car匹配QPATTERN1,而cdr匹配QPATTERN2
    ```emacs-lisp
    (setq form '(1 . 2))
    (pcase form
      (`(,x . ,y) (message "%s + %s = %s" x y (+ x y)))) ;这里x绑定为值1,y绑定为值2
                                            ;=>"1 + 2 = 3"
    ```
-   \`ATOM
    该模式匹配任何\`equal\` ATOM的atom
    ```emacs-lisp
    (pcase (get-return-code x)
      (`success       (message "Done!"))
      (`would-block   (message "Sorry, can't do it now"))
      (`read-only     (message "The shmliblick is read-only")) ;注意,symbol前需要用反引号引起来
      (`access-denied (message "You do not have the needed rights"))) ;这里access-denied为atom,使用equal来进行匹配
    ```
-   \`,UPATTERN
    该模式匹配任何符合UPATTERN的object,并会绑定object到UPATTERN中的变量中
    ```emacs-lisp
    (setq form '(add 1 2))
    (pcase form
      (`(add ,x ,y) (message "%s + %s = %s" x y (+ x y)))) ;这里x绑定为值1,y绑定为值2
    ;=>"1 + 2 = 3"

    ```
-   SYMBOL
    该模式匹配任何object,并且将该symbol绑定到object上.
    ```emacs-lisp
    (pcase (get-code x)
      (code (message "code is %s" code)))   ;这里code为一个symbol,它的值为(get-code x)的结果
    ```
-   -
    该模式匹配任何object,但与SYMBOL不同在于不会将object绑定到任何symbol上
-   (pred PRED)
    返回(PRED object)的值
    ```emacs-lisp
    (pcase x
      ((pred numberp) (message "x is number"))
      ((pred stringp) (message "x is string")))
    ```
-   (or UPATTERN1 UPATTERN2 ...)
    任何一个UPATTERN匹配都行
-   (and UPATTERN1 UPATTERN2 ...)
    所有UPATTERN都必须匹配
-   (guard EXP)
    若EXP的计算结果为非nil,则匹配,否则不匹配


### 组合条件 {#组合条件}

-   (not condition)

-   (and conditions)

-   (or conditions)


### 循环 {#循环}

-   (while condition bodys)

while先判断condition的值,只要condition为非nil,则循环执行bodys,bodys可以为空.

要模拟repeat bodys until condition,可以使用如下的结构

```emacs-lisp
(while (bodys
        (not condition)))
```

-   (dolist (var list [result]]) bodys)

对list的每个element,绑定到变量var中,然后执行bodys中的语句,最后返回result的计算结果(默认为nil).

```emacs-lisp
(defun reverse (list)
  (let (value)
    (dolist (elt list value)
      (setq value (cons elt value)))))
```

-   (dotimes (var count [result]) bodys)

类似dolist,但var的值的范围为[0,count)

-   (do (var initial [step-form]) (end-test [RESULT-FORM] bodys...))

类似for(var=initial;end-test;var=step-form){bodys...}


### 使用catch/throw模拟goto语句 {#使用catch-throw模拟goto语句}

可以在catch中使用throw来跳出循环,throw语句会跳转到catch处,例如

```emacs-lisp
;; (catch tag bodys)
(defun foo-outer ()
  (catch 'foo
  (foo-inner)))

;; (throw tag value)
(defun foo-inner ()
  ...
  (if x
    (throw 'foo t))                   ;这里第一个参数必须与catch的第一个参数匹配. 第二个参数t,作为catch的返回值
  ...)
;; 从这个例子中可以看出,throw可以跨函数间捕获
```

throw会根据其第一个参数来查询匹配的catch,匹配的catch它的第一个参数需要eq throw的第一个参数.

catch语句的返回值由throw的第二个参数决定

若有多个catch可供匹配,则最内层那个catch被匹配.

throw操作退出多个lisp结构时,就好像正常退出这些lisp结构一样.
具体来说,throw操作会这些lisp结构中使解绑用let绑定的变量,退出了save-excursion语句后,会还原buffer和postion. 退出save-restriction语句后,会还原narrowing状态. 它还会调用unwind-protect语句定义的清理动作.

若一个throw的tag没有相应的catch tag来匹配,则会抛出\`no-catch\`错误. 错误内容为throw语句中的\`(tag value)\`


## Elisp中的异常机制 {#elisp中的异常机制}


### 使用singal/error/condition-case模拟try catch语句 {#使用singal-error-condition-case模拟try-catch语句}

elisp中也提供了类似C++中的异常机制,在elisp中,其被称为error.

大多数的error会在调用primitive function时自动抛出. 当然你也可以使用函数\`error\`和\`signal\`手工抛出error.

需要注意的是,C-g触发的quitting,它的处理方式跟error类似,但并不是error

每个error都需要一个错误说明信息,来说明抛出error的原因.


#### error类型 {#error类型}

就好像C++有各种不同类型的异常一样,elisp也有不同类型的error. error的类型使用error symbol来标识. 每个error有且仅有一个error symbol

同样的,跟C++类似,error处于一种被称为error-condition的继承体系内,每个error-condition由condition-name来标识,一个error可以属于多个error-condition.

理论上,\`'error\`处于error-condition的最顶端,但quit是个例外,quit属于一种error-condition但它不是一种error,它的父类就是quit自己

若要定义自己的error,可以使用define-error函数.

-   (define-error symbol message &amp;optional parent)

    定义一个新error,它的error-symbol为参数symbol. 它继承于参数parent所表示的error-condition(默认为error),

    参数message需要是一个字符串,当该error被抛出,而没有handler捕获时,elisp使用该字符串作为error message.

    下面是一个定义error的例子
    ```emacs-lisp
    (define-error 'new-error "A new error" 'my-own-errors) ;error message一般第一个字母是大写的
    ```


#### 抛出error {#抛出error}

-   (error format-string &amp;rest args)

    抛出一个error,该error的错误说明信息为(format format-string args)

-   (signal error-symbol data)

    signal函数抛出一个名为error-symbol的error.

    参数error-symbol必须是由\`define-error\`定义的symbol.
    data参数则是与error环境相关的一系列lisp object,其lisp object中的个数和意义,对不同的error-symbol有不同的要求

    若抛出的error没有被处理,则error-symbol和data这两个参数被用来输出出错信息.

    一般情况下,出错信息由error-symbol的error-message property来提供. data则一般用来提供产生error的上下文环境.
    但若error-symbol为error,则错误信息为(car data),且(car data)必须为string型. file-error类则有其特殊的处理模式.
    ```emacs-lisp
    (signal 'error '("asdbs" (car 1) (cas 1))) ;error--> asdbs: (car 1), (cas 1)
    (signal 'wrong-number-of-arguments '(x y)) ; error--> Wrong number of arguments: x, y
    (signal 'no-such-error '("My unknown error condition")) ; error--> peculiar error: "My unknown error condition"
    ```

-   (user-error format-string &amp;rest args)

    user-error跟error函数类似,但是它使用user-error作为error-symbol而不是error.

    如名称所示,一般用该函数抛出用户级的error,而不是代码级的error,即它不会进入debug模式(即使debug-on-error为非nil)


#### 处理Error {#处理error}

类似C++中的异常机制,elisp中的error也可以定义多个error-handler来捕获它,但只有最靠近error发源地的error-handler会用来处理该error.

若抛出的error,没有对应的error handler来处理它,则根据变量\`debug-on-error\`来决定是调用debug来处理该error(t),还是直接终止程序输出error(nil).

-   (condition-case var protected-form error-handler-bodys)

    可以使用condition-case来定义error handler.例如
    ```emacs-lisp
    (condition-case nil
      (delete-file filename)
    (error nil))
    ```
    condition-case的第一个参数var是一个变量,当参数protected-form正常执行时,该变量只能在error-handler的代码中才能被使用,这时该变量的值为'(error-symbol . data)'. error-handler可以根据该变量中所描述的错误信息来进行操作.
    var参数也可以是nil,表示没有这样一个描述error信息的变量.
    就像写C++代码一样,有时候,需要重新抛出error,以便让外面的代码捕获到该error,则可以这样做:
    ```emacs-lisp
    (signal (car var) (cdr var))
    ```
    我们称呼condition-case的第二个参数为"protected form"(在上例中,就是(delete-file filename))

    "protected form"后面的参数则为定义的error handlers. 每个error-handler的格式为(condition-names handler-bodys).
    这里,conditon-names可以是一个error-condition名称或一个由error-condition名称组成成列表. 在上面的例子中,\`error\`为conditon-name表示所有类型的error.

    捕获到error后,condition-case的返回值为error-handler的执行结果. 若没有error发生,则返回protectd-form的计算结果.
    下面是一些error-handler的例子
    ```emacs-lisp
    (error nil)

    (arith-error (message "Division by zero"))

    ((arith-error file-error)
     (message
      "Either division by zero or failure to open a file"))
    ```

一般情况下,若抛出的error被error-handler所捕获,则不会进入debug模式,但若希望调试那些被condition-case捕获的error,可以设置变量\`debug-on-signal\`为非nil.
你也可以设置某些特定的error在捕获前,先进入debug模式,方法是在error-handler的conditon-name前加上\`debug\`,例如:

```emacs-lisp
(condition-case nil
  (delete-file filename)
((debug error) nil))
```

需要注意的是,这里condition-name前的debug并不意味着一定会进入debug模式,还需要将\`debug-on-error\`设置为非nil才行.

-   (condition-case-unless-debug var protected-form error-handler-bodys)

    类似condition-case,但只在不启用debug的情况下才其作用(即\`debug-on-error\`为nil)

-   (error-message-string error-descriptor)

    输出error-descriptor(即condition-case中的第一个参数var)所表示的字符串.

-   (ignore-errors body)

    执行body语句,并忽略任何抛出的error. 若body执行时不抛error,则返回body的计算结果,否则返回nil

-   (with-demoted-errors format bodys)

    该宏就像是ignore-error的温和版本. 它不会直接忽略掉error的发生,相反,它会使用format来将error转换为一条message输出.

    参数format必须为格式字符串,且必须有且仅有一个"%S"作为占位符.

    需要注意的是,在with-demoted-errors宏中,它是使用conditon-case-unless-debug来捕获error,而不是conditon-case. 因此需要在关闭debug-on-error,才能起作用.


### 使用unwind-protect模拟finally语句 {#使用unwind-protect模拟finally语句}

类似java中的finally语句,elisp也提供了unwind-protect来保证清理动作一定会执行.

-   (unwind-protect body-form cleanup-forms...)

unwind-protect保证执行完body-form后,无论执行过程中是否直接调用throw跳出body-form,还是抛出error,还是正常执行,都会执行cleanup-forms中的语句.

与finally类似,unwind-protect语句只保证body-form执行失败后会执行cleanup-forms中的语句,而不能保证cleanup-forms中如果出了问题,还会执行后面的语句.

与finally不同的是,若body-form正常结束,则unwind-protect的返回值为 **body-form** 的计算结果,而若body-form非正常退出,则不返回任何值(??),而不是返回cleanup-forms的值.


## Elisp中的变量 {#elisp中的变量}

在Elisp中,一个变量就是一个lisp symbol. 变量名为该symbol的名称,变量值为该symbol的value cell中存储的值.

与C++中的变量不同的是,Elisp中的变量可以指向任何类型的数据,而且可以为变量设置一个doc-string,用于说明该变量的用处.


### 全局变量 {#全局变量}

可以使用defvar,defconst,defcustom来定义全局变量.

-   (defvar symbol &amp;optional value doc-string)

定义名为symbol-name的变量,并初始化值为value.

```emacs-lisp
(defvar project-root "~/project/"
  "项目根目录")
```

若省略value的值,则定义出来的变量为空变量.不能直接被访问.

```emacs-lisp
(defvar void-var)
void-var                                ;Lisp error: (void-variable void-var)
```

**需要注意的是:** 若symbol已经有值,则defvar并不会更改symbol的值.

```emacs-lisp
(defvar var 'some-value)
var                                     ;some-value
(defvar var 'other-value)
var                                     ;some-value
```

另外,defvar绑定的是symbol在动态域下的默认值,它并不会影响symbol的buffer-local值,也不改变symbol的静态绑定值.

-   (defconst symbol value &amp;optional doc-string)

与defvardefconst也定义一个名为symbol-name的变量,它的值为value.

另外,defconst绑定的也是symbol在动态域下的默认值,它并不会影响symbol的buffer-local值,也不改变symbol的静态绑定值.

它跟defvar不一样的地方在与,它总是使symbol赋值为value,而不管是否已经有值.

但正如名称所表示的,它表示定义的变量通常应该是一个常量. 不 **建议** 修改它的值(但不是强制性的)


### 常量 {#常量}

在Elisp中,nil,t和任何以\`:\`开头的symbol(我们常常称呼这种symbol为keyword)都是系统的保留常量.

任何对这些系统的保留常量的值进行修改的动作,都会抛出\`setting-constant\` error.

还有一类是用户使用defconst来自定义的常量,对这类常量的值进行修改,并不会抛出error,而且修改行为也能成功


### 局部变量 {#局部变量}

与C++一样,函数中的参数,天生就是局部变量,它的作用范围就是整个函数内部.

而要实现类似C++的代码块内的局部变量({}内定义的局部变量),需要使用let\*语句.

-   (let\* (bings...) forms...)

这里,bingding为定义局部变量的语句,在这里定义的局部变量,只能在后面的bodys中访问.

每个bingding可以是一个symbol,这表示定义一个局部变量,并且该局部变量的值为nil.
也可以是一个(symbol value-form)格式的list,表示定义一个局部变量,并且该局部变量的值为value-form的计算结果. 当然value-form也可以省略,表示nil.

下面是一个例子

```emacs-lisp
(setq y 2)                              ; => 2

(let* ((y 1)
       (z y))
  (list y z))                           ; => (1 1)
```

-   (let (bindings...) bodys...)

类似let\*语句, **但是需要注意的是:** let语句中,在整个(bingings)没有完成之前,所有的局部变量都是不生效的. 举个例子

```emacs-lisp
(setq y 2)                              ; => 2
;; 这里在执行(z y)时,局部变量y还未生效,这时的y是全局变量y,即它的值为2
(let ((y 1)
      (z y))
  (list y z))                           ; => (1 2)
```


### Buffer-Local变量 {#buffer-local变量}

Buffer-Local变量应该说是Elisp所特有的一种变量类型了. 这种变量的作用域仅限于某个buffer.

换句话说,一个变量它在不同作用域下有不同的绑定值. 若它处于buffer作用域下,则该变量的值根据buffer的不同而不同. 而之前提到的与buffer无关的动态作用域的值,我们称它为变量的默认值.

**需要注意:** 即使一个变量被标记为buffer-local变量,当使用defvar和defconst时,改变的依然是它的默认值.而不是buffer作用域下的值.

-   (make-local-variable symbol)

    可以使用命令\`make-local-variable\`来标注一个变量为Buffer-Local变量. 这时,该变量在当前buffer中的值变得跟其他buffer独立开来. 在当前buffer中,该变量处于buffer作用域中,而在其他buffer中则共享该变量的默认值.

该变量在buffer作用域下的值,在创建时与该buffer的默认值是一样的.

若一个变量是terminal-local变量,则该函数会抛出error. terminal-local变量不能有buffer作用域下的值.

**注意:** 不要用该函数来将hook变量设置为buffer-local变量,而应该在使用add-hook和remove-hook时将local参数设为t

-   (setq-local symbol-name value)

将symbol变量标注为buffer-local变量,同时赋值为value. 它等于是先调用make-local-variable后再用setq进行赋值.

-   (make-variable-buffer-local symbol)

    也可以使用命令\`make-variable-buffer-local\`来标注一个变量在所有的buffer中都处于buffer-local作用域下,包括那些还未被创建的buffer. 我们称呼这种变量为automatically buffer-local变量

    所有buffer中的值一开始时默认就是该变量的默认值.

当symbol变量的默认值为空时,该语句会自动为变量在buffer作用域下的值赋值为nil.

-   (defvar-local symbol-name value &amp;optional docstring)

定义以symbol-name为名称的变量,并赋初值为value,并把该变量标注为自动的buffer-local变量.

该红等价于先执行make-variable-buffer-local,然后再执行defvar

-   (local-variable-p symbol &amp;optional buffer)

判断symbol所表示的变量在buffer中是否为buffer-local变量,若省略buffer参数则指的当前buffer.

**注意:该函数在判断automatically buffer-local变量时返回nil**

```emacs-lisp
(defvar-local a 1)                     ;这时a为automatically buffer-local变量
(local-variable-p 'a)                  ;=>nil
```

-   (local-variable-if-set-p symbol &amp;optional buffer)

跟local-variable-p类似,但当symbol为automatically buffer-local变量时,该函数也返回t

```emacs-lisp
(defvar-local a 1)                     ;这时a为automatically buffer-local变量
(local-variable-if-set-p 'a)           ;=> t
```

-   (buffer-local-value symbol buffer)

返回symbol变量在指定buffer中的buffer作用域中的值, 若symbol变量在指定buffer中没有buffer-local绑定值,则返回它的默认值.

-   (buffer-local-variables &amp;optional buffer)

以list的方式返回当前buffer中的所有buffer-local变量. 若buffer参数被省略,则表示当前buffer.

返回的list中的每个元素的格式为'(symbol . value), 但若symbol变量在buffer作用域下的值为空(不是nil),则元素的格式只是单个的symbol

```emacs-lisp
(make-local-variable 'foobar)
(makunbound 'foobar)
(make-local-variable 'bind-me)
(setq bind-me 69)
(setq lcl (buffer-local-variables))
;; =>
    (foobar                             ;foobar为void变量,格式为单个的symbol
    (bind-me . 69))                     ;bind-me变量有值,因此格式为(symbol . value)

```

-   (kill-local-variable symbol)

删除symbol变量在当前buffer中的buffer-local标识,使之在当前buffer中作为一个普通变量来处理.

**要注意的是:** 若对一个automatical buffer-local变量执行该函数,则该变量在当前buffer中访问时会作为一个普通变量来处理,然而, **一旦对这个变量再次赋值,该变量又变成为buffer-local变量**

-   (kill-all-local-variables)

删除当前buffer中所有buffer-local变量(包括函数)的buffer-local标识,但那些标注为"permanent"的变量和"permanent-local-hook"属性为非nil的函数除外.

该函数返回nil

**该函数执行的第一件事就会执行change-major-mode-hook,因为它会把当前buffer的major mode先改为fundamental mode**

<span class="underline">所谓标注为permanent的变量,指的是symbol的permanent-local属性为非nil</span>

当在某buffer中标注某变量问buffer-local变量后,再使用setq来更改变量值时只会更改该变量在该buffer作用域下的值了,要想更改它的默认值,需要使用语句set-default / setq-default了

-   (setq-default symbol1-name value1 symbol2-name value2 ...)

    设置每个变量的默认值

-   (set-default symbol value)

    设置symbol变量的默认值

同样的,使用let对一个buffer-local变量进行局部绑定时,修改的也是该变量在buffer作用域下的值.


### File-Local变量 {#file-local变量}

在文件中指定了某个变量为File-local变量后,当某个buffer访问该文件后,相关变量自动成为buffer-local变量.

处于安全考虑,若某个File-local变量为函数或S表达式,则只有那些明确标记为safe的file-local变量才会自动生效,其他的file-local变量需要用于认可才回生效.

你可以通过修改一个变量的safe-local-variable属性来决定哪些值对于该参数来说是有效的,该属性接收该参数的值,返回非nil则表示该参数safe(有效),nil表示unsafe(无效).

此外,当Emacs读取file-local变量时,\`read-circle\`变量会临时设为nil.

-   变量enable-local-variables

该变量控制了是否让file-local变量生效.

该变量有可以设置为:

1.  t (默认)

    表示自动生效那些标记为safe的变量,而那些unsafe的变量需要提示用户确认后才生效

2.  :safe

    只有标记为safe的变量才生效,其他的unsafe变量不生效

3.  :all

    所有的变量,不管safe或unsafe,都生效

4.  nil

    所有的变量,不管safe或unsafe,都不生效

5.  其他

    素有变量,不管safe或unsafe,都需要用户确认过之后才生效

6.  变量inhibit-local-variables-regexps

该变量是一个由正则表达式组成的list. 如果某个文件名符合list中某元素的个正则表达式,则该文件中的file-loca变量不生效

-   (hack-local-variables &amp;optional mode-only)

启用该buffer所访问file中的file-local变量.

**注意:** 执行该函数时,会按照变量\`enable-local-variables\`的不同值,而有不同的生效方式.

该函数执行前会触发\`before-hack-local-variables-hook\`,执行后会触发\`hack-local-variables-hook\`

若mode-only参数为非nil,则之后名为"mode:"的file-local变量会生效,若文件中指明了"mode:",则该函数返回该函数值,否则返回nil

-   变量file-local-variables-alist

该变量一定为buffer-local变量,它是一个存储了file-local变量信息的alist.

每个file-local-variables-alist中元素的格式为\`(VAR . VALUE)\`,这里VAR为file-local变量,value为变量的值.

当Emacs访问一个文件时,它其实是先将所有的file-local变量收集了起来存入file-local-variables-alist变量中,然后再调用hack-local-variables函数来让他们生效.

-   配置项safe-local-variable-values

该变量是一个由\`(VAR . VALUE)\`组成的list. 这里VAR为变量名,而VALUE为VAR的值,并且该值被认为是safe的.

-   (safe-local-variable-p symbol value)

判断給symbol变量设置为value是否safe

-   (risky-local-variable-p symbol)

该函数判断symbol变量是否认为是risky

risky的变量在生效前,除非明确被设置到\`safe-local-variable-value\`中,佛则一定需要经过用户的确认.

所谓risky变量,指的是它的属性\`risky-local-variable\`为non-nil的变量.

此外,任何以\`-command\`,\`-frame-alist\`,\`-function\`,\`functions\`,\`-hook\`,\`-hooks\`,\`-form\`,\`-forms\`,\`-map\`,\`map-alist\`,\`-mode-alist\`,\`-program\`和\`-predicate\`结尾的变量都自动认为是risky的.

-   变量ignored-local-variables

该变量的值为由变量组成的list, 该list中的对应变量不能被设置为file-local变量,即使在文件中将它设置为file-local变量,也无效果.

-   配置型enable-local-eval

\`:Eval\`是一个明显的潜在漏洞,因此Emacs通常在处理该函数时都要经过用户的确认.

通过设置enable-local-eval值,可以改变这一行为. 该变量可以有三个值:

1.  t

    表示无条件执行

2.  nil

    表示无条件不执行

3.  其他(默认为'mayb)

    表示询问用户.

4.  配置型safe-local-eval-forms

该变量为一个由正则表达式组成的list. 当\`Eval:\`参数的值能够匹配上其中一个正则的S表达式,则认为是安全的.

如果\`Eval:\`参数的值为一个调用函数的S表达式,且调用的函数拥有\`safe-local-eval-function\`属性,则该属性所表示的函数被用来判断该S表达式是否为安全的. \`safe-local-eval-function\`函数得值,可以是一个函数列表,表示其中任何一个函数返回t即为安全,也可以是t,表示所有的S表达式都安全.

由于Text属性值也可能包含要被调用的函数,因此它也认为是一个潜在的漏洞, 因此,若一个变量的值为带有Text属性的String,则该string的Text属性被忽略.


### Directory-Local变量 {#directory-local变量}

在目录中指定了某个变量为Directory-local变量后,当某个buffer访问该目录(极其子目录)下的文件后,相关变量自动成为buffer-local变量.

有两种方式来定义directory-local变量:

1.  把他们放到特定的文件中,该文件名由常量\`dir-locals-file\`决定,默认为\`.dir-locals.el/_dir-locals\`

    基于速度的考虑,一般在访问远程文件时,会禁用该特性,但通过设置变量\`enable-remote-dir-locals\`为t,可以为远程文件也打开该特性.

    dir-locals-file文件的格式为一个list,其中每个元素的格式可以是:

    -   (major-mode . directory-local-variable-value-alist)

    表示当指定major-mode开启时,对应的directory-local变量生效

    -   (nil. directory-local-variable-value-alist)

    表示对所有major-mode,对应的directory-local变量生效

    -   (subdirectory-name-string . dir-locals-file-format-list)

    表示对于指定子目录下的所有文件,directory-local变量生效.

    下面是一个例子:
    ```emacs-lisp
    ((nil . ((indent-tabs-mode . t)
    (fill-column . 80)))
    (c-mode . ((c-file-style . "BSD")
    (subdirs . nil)))  ; =>这里subdirs不是变量名,而是一个关键字,表示该设置,只对当前目录下的文件有效,而对子目录下的文件无效.
    ("src/imported"
    . ((nil . ((change-log-default-name
    . "ChangeLog.local"))))))
    ```
    由于手工修改该文件格式会比较容易出错,因此Emacs提供了命令add-dir-local-variable/delete-dir-local-variable/copy-file-locals-to-dir-locals命令来维护directory-locale变量

2.  为目录定义"project class"

    首先使用函数dir-locals-set-class-variables定义一组变量/值的键值对的集合.
    ```emacs-lisp
    (dir-locals-set-class-variables 'unwritable-directory
                                    '((nil . ((some-useful-setting . value)))))
    ```
    然后使用函数dir-locals-set-directory-class函数为目录分配这组键值对的集合
    ```emacs-lisp
    (dir-locals-set-directory-class
     "/usr/include/" 'unwritable-directory)
    ```


#### 相关函数 {#相关函数}

-   (hack-dir-local-variables)

    为访问当前目录(及子目录)下文件的所有buffer开启directory-local变量

    该函数通过调用函数dir-locals-set-class-variables和dir-locals-set-directory-class来完成此操作.

-   (hack-dir-local-variables-non-file-buffer)

    为当前buffer启用directory-local变量,一般用于那些non-file buffer中.

    对这些non-file buffer开启directory-local变量时会从\`default-directory\`和它的父目录中查找directory-local变量的定义

-   (dir-locals-set-class-variables project-class  dir-locals-file-format-list)

    该函数定义一组directory-local变量及其值,并分配改组变量为project-class
    ```emacs-lisp
    (dir-locals-set-class-variables 'unwritable-directory
                                    '((nil . ((some-useful-setting . value)))))
    ```

-   (dir-locals-set-directory-class directory project-class &amp;optioinal mtime)

    为directory(及其子目录下)下的所有文件分配project-class所表示的directory-local变量.

    当Emacs从\`.dir-locals.el\`文件中读取directory-local变量时,也是通过调用该函数来实现的,这是会带上mtime参数.

    mtime参数存储的是\`.dir-locals.el\`的modification time. Emacs使用该时间来检查已有的directory-local变量是否依然有效.

-   变量dir-locals-class-alist

    该变量是一个alist,它维护了project-class及对应directory-local变量的对应关系.

-   变量dir-locals-directory-cache

    该变量是一个alist,它维护了目录名称,对应的project-class和对应\`.dir-locals.el\`的modification time

-   变量enable-dir-local-variables

    是否启用directory-locall变量特性.


### Terminal-Lock变量 {#terminal-lock变量}


### 空变量 {#空变量}

前面说到,变量的值其实就是取得symbol中的value cell中存储的对象. 当symbol中的value cell没有存储任何对象时(nil也是一个对象),这时访问该变量会抛出\`void-variable\` error. 我们称这种变量为空变量.
(**NOTE:** 上述的情况在Emacs默认的动态作用域下是成立的,若明确指定了静态作用域,则另当别论了,但这种情况比较少用到)

那么创建这种空变量呢? 这就需要用到makeunbound函数了.

-   (makeunbound symbol)

将当前作用域下的局部变量symbol中的value cell清空,使之成为空变量.

若要判断某个变量是否为空变量,则可以使用boundp函数

-   (boundp symbol)

该函数检查symbol的value cell是否有值,若有值则返回t,否则返回nil. 因此我们也可以定义函数

```emacs-lisp
(defun void-variable-p (variable)
  (null (boundp variable)))
```

下面是一些boundp的例子

```emacs-lisp
(boundp 'abracadabra)          ; Starts out void.
;; => nil
(let ((abracadabra 5))         ; Locally bind it.
  (boundp 'abracadabra))
;; => t
(boundp 'abracadabra)          ; Still globally void.
;; => nil
(setq abracadabra 5)           ; Make it globally nonvoid.
;; => 5
(boundp 'abracadabra)
;; => t
```


### 变量别名 {#变量别名}

变量及其别名公用同一个值,修改其中一个也会同时更改另一个值.

-   (defvaralias new-alias base-variable &amp;optional docstring)

为base-variable定义一个名为new-alias的变量别名,可以为这个别名分配一个新的docstring

该函数返回base-variable

-   (indirect-variable alias-variable)

返回别名链中最末端的那个非别名变量

若出现了循环定义的别名,则该函数抛出\`cyclic-variable-indirection\` error


### 废弃变量 {#废弃变量}

-   (make-obsolete-variable obsolete-variable current-variable when &amp;optional access-type)

在编译时警告一个变量即将废弃不用了,其中:

参数obsolete-variable为即将不用的变量

参数current-variable若为symbol,则会提示用新的变量current-variable代替老的变量obsolete-variable. 若current-name为string,则直接警告该string.

参数when指明了obsolete-variable从什么时候开始废弃,通常为一个表示版本号的字符串.

参数access-type指明了对obsolete-variable的哪种操作会触发警告,可以使'get或'set

-   (define-obsolete-variable-alias obsolete-variable current-variable &amp;optional when docstring)

该宏创建obsolete-variable为current-variable的别名,并标记obsolete-variable为即将废弃的变量.

该宏其实等价于:

```emacs-lisp
(defvaralias OBSOLETE-NAME CURRENT-NAME DOCSTRING)
(make-obsolete-variable OBSOLETE-NAME CURRENT-NAME WHEN)
```


### 受限的变量 {#受限的变量}

默认情况下,一个Lisp变量的值可以是任何的Lisp object. 但有些变量不是用Lisp来定义的,而是用C来定义. 这些用C定义的变量有可能只能存储特定类型的值. 如果变量类型为:

-   DEFVAR_LISP

该变量跟在lisp中定义的变量一样,它的值可以是任意的.

-   DEFVAR_INT

该变量的值只能是整型

-   DEFVAR_BOOL

该变量的值只能为t或者nil

其中变量\`byte-boolean-vars\`中列出了所有类型的DEFVAR_BOOL的变量


### 变量的作用域 {#变量的作用域}

与C++不同的是,Elisp中的变量默认情况下是处于动态作用域中. 当然,Elisp也支持静态作用域.


#### 动态作用域 {#动态作用域}

当一个变量处于动态作用域中时,这就意味着,这个变量的值是受到运行环境的影响的. 举个例子:

```emacs-lisp
(setq foo 'outer)                       ;outer
(defun say-foo()
  foo)

(say-foo)                               ;=>outer
(let ((foo 'inner))
  (say-foo))                            ;=>inner,在调用say-foo的运行环境中,foo的值为局部定义的'inner,因此say-foo的返回值为'inner

(say-foo)                               ;=>outer, 在调用say-foo的运行环境中,foo的值为全局值'outer,因此say-foo的返回值为'outer
```

拥有动态作用域值的变量被成为special-variable,可以使用函数special-variable-p来判断一个symbol是否为special variable.

-   (special-variable-p symbol)

    判断symbol所表示的变量是否为special-variable. (由defvar,defconst和defcustom定义的变量都是special variable)

-

elisp实现动态作用域的方法很简单,每个symbol都由一个value cell,这个value cell所持有的值就是该变量在当前动态作用域下的值. 当为该变量创建一个动态局部作用域时,elisp将当前value cell的值压入一个栈中,并将该symbol的value cell存上新值. 当退出该动态局部作用域时,Elisp从栈中弹出以前的值,并重新存入symbol的value cell中.


#### 静态作用域 {#静态作用域}

当一个变量处于静态作用域下时,该变量的值在定义该变量处就已经被确定了,即它的值为定义环境的值. 例如

事实上,Elisp使用一个alist来存储静态作用域中各变量与值的关系. 这种alist的结构为\`'((symbol1 value1)(symbol2 value2)... t)\`.
这种alist可以作为eval函数的第二个参数用来指明eval执行语句的静态作用域环境.

可以使用lexical-let和lexical-let\*来创建静态作用域. 这两个语句的语法跟let和let\*一样,但BODY中的lambda函数会创建闭包.


### 泛化变量(Generalized Variables) {#泛化变量--generalized-variables}

泛化变量(Generalized Variables)或称位置列表(place form)其实就是变量值所被存储的内存地址.

泛化变量可能是: 一个普通的lisp变量或者aref,car,caar,cadr,cdr,cdar,cddr,elt,get,gethash,nth,nthcdr,symbol-function,symbol-plist,symbol-value,default-value,frame-parameter,terminal-parameter,keymap-parent,match-data,overlay-get,overlay-start,overlay-end,process-buffer,process-filter,process-get,process-sentinel,window-buffer,window-display-table,window-dedicated-p,window-hscroll,window-parameter,window-point,window-start函数的返回值.

-   (setf place-form1 value1 place-form2 value2...)

    可以使用setf宏来操作泛化变量. 它的作用类似setq,但setq只能为symbol赋值,而setf可以为任何泛化变量赋值. 例如

<!--listend-->

```emacs-lisp
(setq a '(1 2 3))                       ;(1 2 3)
(setf (cadr a) 'two)                    ;将a中的第二个元素的值改为two
a                                       ;(1 two 3)

```

-   (gv-define-simple-setter name setter-function &amp;optional fix-return)

-   (gv-define-setter name arglist &amp;rest body)


### 取变量值 {#取变量值}

当在静态作用域下,Elisp取变量值时,它会先查看该变量是否存在静态作用域下的绑定值. 然后再查看该变量的动态作用域下的绑定值(即该symbol的value cell所存储的值)

除了直接引用变量可以取得变量值外,还可以使用symbol-value函数来获取变量的动态作用域下的值

-   (symbol-value symbol)

<!--listend-->

```emacs-lisp
(defvar num 123)
(symbol-value 'num)                     ;123
```

**需要注意的是:** 该函数只能用来获取symbol动态绑定的值,而不能用在静态环境下获取它静态绑定的值

```emacs-lisp
(lexical-let ((num 234))
  (symbol-value 'num)                   ;123
  num)                  ;234
```

-   (buffer-local-value symbol buffer)

返回symbol变量在指定buffer中的buffer作用域中的值, 若symbol变量在指定buffer中没有buffer-local绑定值,则返回它的默认值.

-   (default-value symbol)

    取symbol变量的默认值

-   (default-boundp symbol)

    判断symbol变量的默认值是否为不为空

-


## Customization {#customization}

emacs中可以使用\`defcustom\`定义customizable variables, 使用defface定义customizable face,使用defgroup定义cutomization group.


### Common Item Keywords {#common-item-keywords}

defcustom,defgroup,defface这些定义配置项的函数/宏,都接收keyword参数.

所有这些keyword参数,除了\`:tag\`之外,都可以联合使用. 下面是一些通用的参数说明.

-   :tag LABEL

LABEL为一个字符串类型. 该参数表示使用LABEL取代被定义item的名称作为该item的标签.

-   :group GROUP

定义item所属的组别. 一个item可以同时属于多个组别,因此你可以多次使用该参数

-   :link LINK-DATA

在该item的说明文档后面增加一个外部链接. 其中LINK-DATA可以以下格式:

你还可以在LINK_DATA的第一个元素后面加上\`:tag NAME\`用来表示链接显示为NAME. 例如\`(info-link :tag "foo" "(emacs)Top")'会创建一个链接连接到Emacs手册,但是显示为foo

-   (custom-manual INFO-NODE)

    链接到Info node. INFO-NODE为Info文档中某node的名称,像"(emacs)Top"这样的.

    该链接显示为[manual]

-   (info-link INFO-NODE)

    类似custom-manual,只是链接的显示为Info node的名称

-   (url-link URL)

    链接到web页面, 点击它会使用变量\`browse-url-browser-function\`定义的Web浏览器打开

-   (emacs-library-link LIBRARY)

    连接到Emacs Lisp library文件.

-   (file-link FILE)

    连接到某个文件,Emacs会用find-file函数打开它

-   (function-link FUNCTION)

    链接到某个函数的说明文档,当点击它,会使用describe-function来获取函数说明

-   (variable-link VARIABLE)

    连接到某个变量的说明文档

-   (custom-group-link GROUP)

    链接到其他的group

-   :load FILE

在显示item前先加载FILE

-   :require FEATURE

当保存item的值时,执行(require 'FEATURE)

-   :version VERSION

表示该item第一次出现在版本为VERSION的Emacs中,或该item的默认值或意义在版本为VERSION的Emacs中更改了.

VERSION为字符串类型

-   :package-version '(PACKAGE . VERSION)

表示该item第一次出现在版本为VERSION的PACKAGE中,或该item的默认值或意义在版本为VERSION的PACKAGE中更改了.

PACKAGE应该是一个符号,为package的正式名称. 而VERSION应该为字符串类型.

若PACKAGE为Emacs自带的,则PACKAGE和VERSION需要在变量\`customize-package-emacs-version-alist\`中


### customization groups {#customization-groups}

-   (defgroup group members doc [keyword value]...)

定义新的名为group的客户化组. 该客户化组中包含members为内容.

参数group为一个不被quote的symbol.

参数members为由cusomiztion items组成的list,表示这些items属于某个group. 然而实际上一般该参数都为nil,而是定义item时使用:group关键字来标识该item所属的group

members的元素格式为(NAME WIDGET). 这里NAME为表示item的symbol. 而WIDGET为item的类型(custom-variable,custom-face,custom-group)

当对group设置了:version参数,则所有属于该group的其他item,默认继承该参数值

defgroup可以使用:prefix关键字参数

:prefix PREFIX

表示若该group中的item以PREFIX为前缀,而变量\`custom-unlispify-remove-prefixes\`为非nil, 则该item的tag在显示时会忽略掉该PREFIX.

一个group可以设置忽略任意数量的prefix


### customizable variable {#customizable-variable}

defcusomter的语法与defvar有点类似,但是它还可以接收很多keyword参数.

-   (defcustom var standard-value doc [keyword value]...)

参数var为不被quote的symbol. 它表示定义的可配置变量.

参数standard-value为一个表达式,它的计算值作为var的默认值

参数doc为对该变量的说明.

若在defcustom中没有通过:group关键字设置所属的group,则在相同文件中最后defgroup的组会自动作为该item的所属组.

defcustom支持的keyword参数有:

-   :type TYPE

    标注该客户化变量的类型,它指定了哪些值是合理的,如何显示这些值.

    也可以在defcustomer后使用函数(custom-add-frequent-value customization-item value)来增加选值范围

-   :options VALUE-LIST

    指定可选值的范围,该可选值的范围并不具有约束性.

    该keyword只有在type为hook,plist和alist时才有效

-   :set SETFUNCTION

    当使用Customize更改该配置项时,实际上调用的是SETFUNCTION函数,该函数接收两个参数:配置项和新值. 默认SETFUNCTION函数为set-default

-   :get GETFUNCTION

    当获取该配置项的值时,实际上是调用了GETFUNCTION函数. 该函数接收一个参数:配置项, 并返回某个值. 默认GETFUNCTION为\`default-value'

-   :initialize FUNCTION

    当defcustom语句被执行时,实际上是调用了FUNCTION函数. 该函数接收两个参数:配置项和默认值.

    elisp预定义了一些可选的FUNCTION:

    -   \`custom-initialize-set'

    -   \`custom-initialize-default'

    -   \`custom-initialize-reset'

    -   \`custom-initialize-changed'

    -   \`custom-initialize-safe-set'

    -   \`custom-initialize-safe-default'

-   :risky VALUE

    设置该配置项变量的\`risky-local-variable'属性为VALUE

-   :safe FUNCTION

    设置该配置项变量的\`safe-local-variable'属性为FUNCTION

-   :set-after VARIABLES

-   (custom-reevaluate-setting customizable-arg)

可以用该函数在defcustom外,重新对customizablen-arg进行赋值

-   (custom-variable-p arg)

判断arg是否为可配置变量, 这意味着这个变量是带有\`standard-value'属性的symbol或者带有\`custom-autoload'属性的symbol,或者由其他可配置变量组成的alist

-   (custom-set-variables &amp;rest args)

根据arg中的说明,更改配置项

每个arg的格式为'(配置项 配置项的值表达式 [is-NOW [REQUEST-features-list [doc-string]]])


#### Customization Type {#customization-type}

所有的customization type都实现为widget. customization widget可以通过\`C-M-i'或\`M-&lt;TAB&gt;'来补全

<!--list-separator-->

-  Simple Types

    -   'sexp

        任意lisp object

    -   'integer

    -   'number

    -   'float

    -   'string

    -   'regexp

    -   'character

    -   'file

        配置项必须是一个文件名称

    -   '(file :must-match t)

        配置项必须是一个已存在的文件名称

    -   'directory

        配置项必须是目录

    -   'hook

        该配置项必须是一个函数列表

    -   'symbol

    -   'function

        该配置项必须是一个lambda表达式或函数名

    -   'variable

        该配置必须是一个变量名称

    -   'face

    -   'boolean

    -   'key-sequence

    -   'coding-system

    -   'color

<!--list-separator-->

-  Composite Types

    -   '(cons CAR-TYPE CDR-TYPE)

        该配置项必须是cons cell. 并且它的car必须为CAR-TYPE,cdr必须为CDR-TYPE

    -   '(list ELEMENAT1-TYPE ELEMENT2-TYPE ... ELEMENTn-TYPE)

        该配置项为由n个元素组成的list,每个元素都需要跟相应的ELEMENT-TYPE相匹配

    -   '(group ELEMENAT1-TYPE ELEMENT2-TYPE ... ELEMENTn-TYPE)

        类似'(list ELEMENAT1-TYPE ELEMENT2-TYPE ... ELEMENTn-TYPE),区别在于list使用element的tag来作为element value的标签,而group不作标签

    -   '(vector ELEMENAT1-TYPE ELEMENT2-TYPE ... ELEMENTn-TYPE)

        类似'(list ELEMENAT1-TYPE ELEMENT2-TYPE ... ELEMENTn-TYPE)

        区别在于配置项的类型必须是vector

    -   '(alist :key-type KEY-TYPE :value-type VALUE-TYPE)

        配置项为alist类型,且每个cons ceil元素的car必须是KEY-TYPE的,cons ceil的cdr必须是VALUE-TYPE

        :key-type参数与:value-type可以省略,默认为'sexp

    -   '(plist :key-type KEY-TYPE :value-type VALUE-TYPE)

        类似'(alist :key-type KEY-TYPE :value-type VALUE-TYPE),只是配置项为plist类型,且KEY-TYPE默认为symbol类型而不是sexp

    -   '(choice CUSTOMIZE-TYPE1 CUSTOMIZE-TYPE2 ... CUSTOMIZE-TYPEn)

        配置项可以是CUSTOMIZE-TYPES中的任意一种.

        可以在CUSTOMIZE-TYPE中通过:tag关键字来指明配置项为某种TYPE时的label.例如
        ```elisp
        (choice (integer :tag "Number of spaces")
                (string :tag "Literal text"))
        ```

    -   '(radio CUSTOMIZE-TYPE1 CUSTOMIZE-TYPE2 ... CUSTOMIZE-TYPEn)

        类似'(choice CUSTOMIZE-TYPE1 CUSTOMIZE-TYPE2 ... CUSTOMIZE-TYPEn),只是显示时使用radio button的方式显示而不是用菜单显示

    -   '(const VALUE)

        该配置项的值必须为VALUE. 常与choice搭配

    -   '(other VALUE)

        表示配置项可以接收任意的lisp值,但是该配置项实际上总是被赋值为VALUE.

        other主要用在choice中作为最后一个元素使用. 例如:
        ```emacs-lisp
        (choice (const :tag "Yes" t)
                (const :tag "No" nil)
                (other :tag "Ask" foo)
        ```

    -   '(function-item FUNCTION)

        类似const,但是它的值必须是function.

    -   '(variable-item VARIABLE)

        类似const,但是它的值必须是表示某个变量的symbol

    -   '(set TYPE1 TYPE2 ... TYPEn)

        该配置项必须是一个list,且每个list中元素类型必须匹配TYPES中的其中一种

    -   '(repeat ELEMENT-TYPE)

        该配置项必须是一个list,并且每个元素都是ELEMENT-TYPE的

    -   '(restricted-sexp :match-alternatives CRITERIA)

        该配置项的值可以是任一的lisp对象,但是必须匹配CRITERIA中的任一条件.

        CRITERIA是一个list,其中每个元素可以是:一个predicate function或者A quoted constant
        ```emacs-lisp
        ;; allows integers, `t' and `nil' as legitimate values.
        (restricted-sexp :match-alternatives
                         (integerp 't 'nil))
        ```

<!--list-separator-->

-  Type Keywords

    在定义配置项的:type时,可以在customizaton type name symbol后指定以下的keyword-argument对:

    -   :value DEFAULT

        提供默认值. 当某种类型不能包含nil时,特别有用.

    -   :format FORMAT-STRING

        显示配置项值时的格式.

        | 占位符     | 说明                                        |
        |---------|-------------------------------------------|
        | %[BUTTON%] | 以按钮的样式显示文本BUTTON,其:action属性说明了当该按钮被点击时作什么操作 |
        | %{SAMPLE}  | 以\`:sample-face'的样式显示文本SAMPLE       |
        | %v         | 显示为该配置项的value                       |
        | %d         | 显示为该配置项的documentation string        |
        | %h         | 类似%d,但当配置项的doc-string超过一行时,会提供一个按钮隐藏/显示剩下的行 |
        | %t         | 显示为该配置项的tag                         |
        | %%         | 显示为%                                     |

    -   :action ACTION

        当点击button时执行的操作. 这里ACTION为一个函数,它会接收两个参数:点击的按钮所在widget和点击事件

    -   :button-face FACE

        提供:button-face的显示样式,它会用于显示FORMAT-STRING中的%[...%]中的内容

    -   :button-prefix PREFIX / :button-suffix SUFFIX

        指明在显示button的前后文本. 他们的值可以是:

        nil: 不显示多于的文本

        string: 显示文本

        symbol: 显示symbol的值

    -   :tag TAG

        指定TAG(字符串类型)作为配置项为该类型时的tag

    -   :doc DOC

        指定DOC作为配置项为该类型时的doc-string

    -   :help-echo MOTION-DOC

    -   :match FUNCTION

        使用FUNCTION判断配置项的值是否匹配该类型,FUNCTION为一个函数,它接收两个参数:表示CUSTOMIZATION TYPE的widget和配置项的值

    -   :validate FUNCTION

        使用FUNCTION校验配置项的值是否有效. FUNCTION函数接收一个参数:表示CUSTOMIZATION TYPE的widget. 若该函数判断widget是当前值是有效的,则返回 **nil** ,否则返回包含无效数据的widget,并设置该widget的\`:error'属性为出错描述.

<!--list-separator-->

-  Defining New Types

    定义新Type就是为一个Composite Types命一个名字. 由于一个type就是一个widget,因此使用define-widget来实现

    ```emacs-lisp
    (define-widget 'binary-tree-of-string 'lazy
      "A binary tree made of cons-cells and strings."
      :offset 4
      :tag "Node"
      :type '(choice (string :tag "Leaf" :value "")
                     (cons :tag "Interior"
                           :value ("" . "")
                           binary-tree-of-string
                           binary-tree-of-string)))
    ```

    这里define-widget的第一个参数为表示新widget type的symbol.

    第二个参数为一个已经存在的widget,表示新widget type的类别,一般用'lazy

    第三个参数为doc-string

    :type参数描述了所代表的composite type说明


### customizable face {#customizable-face}

-   (custom-set-faces &amp;rest args)

根据arg,更改face配置项.

一个arg的格式为'(FACE SPEC [is-Now [doc-string]])


## Loading {#loading}


### Load命令 {#load命令}

-   (load filename &amp;optional missing-ok nomessage nosuffix must-suffix)

load先查找filename.elc文件,再查找filename.el文件,再查找filename文件

若开启了Auto-Compression-mode(默认开启),则load在查找后一个文件前还会查找前一个文件的压缩版本(参见变量\`jka-compr-load-suffixes').

若nosuffix参数为非nil,则load不会查找filename.elc和filename.el,但不影响auto-compression-mode的作用

若must-suffix参数为非nil,则load认为加载的文件名后缀必须为.el和.elc(或他们的压缩版本),除非filename中包含了明确的目录名称.

若参数missing-ok,则在找不到要加载的文件时,不抛出错误,只是返回nil

若\`load-prefer-newer'配置项为非nil,则load会挑选filename.elc和filename.el中最近较新的那个来加载

若filename为相对路径,则load会在变量\`load-path'中定义的路径中查询,查询到的第一个存在文件作为要加载的文件. 需要注意的是:需要在\`load-path'中添加nil才回在当前路径搜索要load的文件

load在加载文件时,会同时设置变量\`load-file-name'的值为加载文件的文件名.

若load顺利加载文件,则返回t

-   (load-file filename)

加载filename所明确指定的文件,并不会为它添加.el会.elc后缀(但不影响Auto Compression Mode的作用)

若filename为相对路径,则认为是相对于当前路径来说的. (该函数并不涉及到load-path变量)

-   (load-library library)

类似load

-   变量\`load-file-name'

存储的是load时实际加载的文件名称.

-   变量\`load-in-progress'

若Emacs正在加载某个文件,则该值为非nil,否则为nil

-   变量\`load-read-functioin'

该变量指明的函数,用于替代\`load'和\`eval-region'中的\`read'函数

该变量默认为nil,表示\`read'

-   变量\`load-suffixes'

load在搜索文件时,会根据该变量中设置的后缀,添加到filename参数后面来寻找文件.

默认为'(".elc" ".el")

-   变量\`load-file-rep-suffixes'

This is a list of suffixes that indicate representations of the same file.

该变量一般以""开头,若开了Auto Compression Mode则会把\`jka-compr-load-suffixes'的内容也加进去.

-   (get-load-suffixes)

返回load函数尝试添加的文件后缀. 它的值一般是\`load-suffixes'与\`load-file-rep-suffixes'的集合.

-   配置项load-prefer-newer

若该配置项为非nil,则load会检测所有可能的加载文件,并挑选最新的那个来加载

-   变量load-path

load函数搜索加载文件的路径列表,nil表示当前工作目录

可以在运行emacs时用-L选项指定load-path的值

对于每个load-path中的目录,emacs都会去检查是否有subdirs.el这个文件,若存在该文件,则加载它. 由emacs自动生成的subdirs.el会自动将该目录下的所有以 **字母与数字结尾** 的子目录路径添加到load-path中.

-   命令(locate-library library &amp;optional nosuffix path interactive-call)

找到指定library所表示的精确文件名. 它的搜索方式与load一致.

参数nosuffix与load参数中的一样. 非nil表示不添加.elc和.el后缀

若PATH为非nil,则表示用参数PATH的值代替load-path的值

当作为命令运行locate-library时,参数interactive-call的值为t,则会在echo area中显示file name,否则函数直接返回文件名称

-   命令(list-load-path-shadows &amp;optional stringp)

该命令列出隐藏的Emacs Lisp文件的列表.

所谓隐藏文件指的是这样一些文件,虽然在load-path中有定义其目录,但是由于在搜索到其目录之前已经发现了符合条件的加载文件,因此load命令无法加载到这些文件.

参数stringp指定是以字符串的形式返回文件列表,还是显示在buffer中.


### Autoload {#autoload}

autoload让你在一开始只是记录函数/宏所对应的加载文件路径. 当第一次用到该函数/宏(或查看其帮助文档)时才开始加载对应的文件.

有两种方法设置一个autoload函数:使用autoload函数和在源代码中使用特定的注释

-   (autoload function-or-macro filename &amp;optional docstring interactive type)

该函数指定function-or-macro为autoload函数/宏. 其源代码定义在filename中.

若filename中不包含目录名称或.el/.elc的后缀,该函数会自动在加载时添加后缀,并且该函数不会加载不带后缀的文件.

参数doc-string使得在不加载实际文件前,也能够查看function-or-macro的对应说明.

参数interactive为非nil,则表示function-or-macro为命令. 这使得Emacs能够为M-x提供该命令的补全而不用加载function-or-macro的真实定义.

当参数function-or-macro为macro/keymap时,可以将type参数设置为'macro或'keymap

若function-or-macro已经有一个非autoload的非空函数,则autoload什么也不做,只是返回nil

-   (autoloadp object)

判断object是否为autoload类型的对象

-   使用特殊注释定义autoload对象

在定义真实的函数定义前,加上注释\`;;;###autoload'(这种特殊的注释,被称为autoload cookie)

随后执行M-x update-file-autoloads/update-directory-autoloads命令,会将autoload的调用命令写道生成的loaddefs.el中.

-   变量generate-autoload-cookie

该变量指定了定义autoload对象的特殊注释格式,默认为\`;;;###autoload'

-   变量generate-autoload-file

该变量定义了将生成的autoload语句放到哪个文件中,默认为\`loaddefs.el'

-   (autoload-do-load autoload-object &amp;optional name macro-only)

加载autoload-object所在的源代码文件.

参数name若为非nil则需要时一个表示autoload-function的symbol. 这时它的返回值为该symbol的实际定义函数.

若参数macro-only为'macro,则autoload-do-load不加载函数,只加载macro


### Features {#features}

features是除autoload外推迟加载的另一种方式.

一个feature是一个表示函数与变量的集合的symbol,可以在文件中用provide声明一个feature,同时使用require来加载一个feature

需要注意的是: 不要在let内使用require,否则可能会产生不可预知的后果.

Although top-level calls to \`require' are evaluated during byte compilation, \`provide' calls are not.  Therefore, you can ensure that a file of definitions is loaded before it is byte-compiled by including a \`provide' followed by a \`require' for the same feature, as in the following example.

```emacs-lisp
(provide 'my-feature)  ; Ignored by byte compiler,
                                        ;   evaluated by `load'.
(require 'my-feature)  ; Evaluated by byte compiler.
```

-   (provide feature &amp;optional subfeatures)

该函数声明已经加载了feature,下次再require该feature时,不会去重新加载该feature所在的文件

这里参数subfeatures应该而我一个由symbol组成的list,表示该版本的feature,提供了一系列的subfeatures

-   (require feature &amp;optional filename noerror)

该函数检查该Emacs Session是否已经加载了feature,若没有,则使用load加载filename.

若参数filename为nil,则使用feature的字符串表示作为load的参数. 但要注意的是,这种情况下,require只会加载带有.el/.elc为后缀的文件(auto comression mode也有效果). 一个名为feature而不带任何后缀的文件不会被加载.

若noerror参数为非nil,则当load文件失败时,只返回nil,而不抛出error.否则返回参数feature.

若加载filename成功,而该文件没有provide feature,则require抛出error:\`Required feature FEATURE was not provided'

**require语句会在编译阶段得到执行.**

-   (featurep feature &amp;optional subfeature)

若feature已经加载到该Emacs Session(即feature是否为\`features'中的member)则返回t.

若subfeature为非nil,则只有在subfeature也被provided了的情况下才返回t

-   变量features

该变量为一个由symbol组成的list,每个symbol都是加载到该Emacs Session中的feature


### 查找定义所在的文件 {#查找定义所在的文件}

-   (symbol-file symbol &amp;optional type)

查找定义symbol的文件路径.

参数type指定了symbol的类型,可以是nil,'defun,defvar或defface

symbol-file实际是从\`load-history'变量中查找symbol所在的文件的.


### Unloading {#unloading}

-   (unload-feature feature &amp;optional force)

回收feature所定义的函数/变量,恢复之前的symbol定义.

若变量\`FEATURE-unload-function'的值为某个函数,则unload-feature会在执行清理前执行该函数. 若该函数返回nil,则unload-feature接着执行正常的清理过程,否则,unload-feature不再进行下一步的清理.

默认情况下,unload-feature不会unload被其他库依赖的feature, 但若force参数为非nil,则unload-feature不会检查依赖关系.

unload-feature函数也是根据变量\`load-history'的内容来行动的.

-   变量unload-feature-special-hooks

该变量为一个hooks列表,在执行unload操作前,会先删除这些hooks中的定义在library中的函数.


### Hooks {#hooks}

-   after-load-functions

load完某个文件后,会执行该hook,每个hook函数会接收一个参数:刚加载文件的绝对路径

-   宏(with-eval-after-load library-or-feature bodys...)

若library-or-feature为library,则在每次加载完library文件后,都执行一次bodys代码

若在执行该宏的时候,library已经被加载过了,则该宏会立刻执行一次bodys

```emacs-lisp
(with-eval-after-load "edebug" (def-edebug-spec c-point t)
```

若library-or-feature为feature,则在执行(provide feature)之后才回执行bodys的内容

若执行bodys时抛出error,不会unload已加载的文件,但是会阻止bodys中的下面语句的执行.

一般该宏没什么用.


## Byte Compilation {#byte-compilation}

Elisp的Byte Compilation为伪编译,它将lisp编译为字节码格式,由特定的字节码解释器解释,而不是编译为与硬件相关的代码. 这使得它的速度会稍微慢点,但同时也保证了不同硬件平台之间的可移植性

若希望某个lisp file不被编译,设置file-local变量no-byte-compile的值为t
格式为:\`;; -**-no-byte-compile: t; -**-'

当编译的文件中包含宏时要特别注意,因为在编译阶段,宏会被展开,这时可能宏的定义还未加载到Emacs中. 为了应付这种情况,一般使用require语句指定包含所需宏的文件(require在编译阶段会被执行). 为了防止用户在执行编译后程序时依然执行require语句,可以使用\`eval-when-compile'包含\`require'语句


### 相关函数 {#相关函数}

-   (byte-compile symbol)

编译symbol的函数定义成字节码格式.

参数symbol的函数定义必须是函数的真实代码,而不能是引用函数.

参数symbol也可以是lambda表达式,但这种情况下,byte-compile只是返回对应的编译后代码,而并不存储它

若symbol的函数定义是一个已经编译为字节码格式的函数,则该函数什么也不做,只是返回nil

-   命令(copmile-defun &amp;optional arg)

编译并执行当前top-level form,并将结果输出到echo area中.

若参数arg为非nil,则将结果插入到当前buffer中,执行的form位置后

-   命令(byte-compile-file filename &amp;optional load)

该命令将lisp格式的filename编译为字节码格式的文件,生成的文件名称是原filename的.el后缀改为.elc后缀,若filename不带.el后缀,则生成的文件名为filename.elc

若load参数为非nil,则在编译完filename后,还是加载编译后的文件.

若byte-compile-file作为命令执行时,会提示输入要编译的文件,这时参数load额值为prefix argument

-   命令(byte-recompile-directory directory &amp;optional flag force)

该命令重新编译directory及其子目录中的所有需要重新编译的.el文件(存在.elc文件比.el文件旧的.el文件)

若存在没有对应.elc文件的.el文件,则由参数flag来说明该如何处理,nil表示不编译这些文件,0表示编译他们,其他值表示询问用户

若参数force为非nil,则命令在重编译所有有对应.elc文件的.el文件.

-   (batch-byte-compile &amp;optional noforce)

该函数调用\`byte-compile-file'编译命令行中指定的文件.

该函数必须当Emacs处于batch状态时才能使用,因为当编译完成后,它会关闭Emacs.

```sh
emacs -batch -f batch-byte-compile *.el
```

编译一个文件出错,不会妨碍其编译下一个文件,但这时,Emacs退出时会设置非0的状态码.

若参数noforce为非nil,则,该函数不会编译那些已经有更新版本的.elc文件的.el文件

-   配置项byte-compile-dynamic-docstrings

默认情况下,Emacs从字节码文件中加载函数和变量时不会加载 他们的doc-string,当需要时才动态的从字节码文件中加载进来. 这就产生了一个后果:若此时字节码文件被更新了,那么原来的doc-string就被覆盖了.

设置该配置项为nil,可以静止Emacs动态加载doc-string的行为.

可以在lisp文件中添加一行"-**-byte-compile-dynamic-docstrings: nil;-**"

-   变量byte-compile-dynamic

若为非nil,则表示开启"dynamic function loading"功能. 这时加载该文件并不会读取其中函数的真实定义,只有在正在调用该函数时采取临时读取该函数的定义.

-   (fetch-bytecode function)

若function为byte-code function object,则立即从字节码文件中加载function的字节码.

返回参数function


### 编译期执行语句 {#编译期执行语句}

-   (eval-and-compile bodys...)

在编译期间和执行期间都执行bodys

想过类似于将bodys放入file中,然后(require file)

autoload和require在编译期和执行期都会执行.

-   (eval-when-compile bodys...)

只在编译期才计算bodys的值. **这时bodys的值被作为常量存储起来,当正常执行该段代码时,直接返回该常量**

```emacs-lisp
(defvar my-regexp
  (eval-when-compile (regexp-opt '("aaa" "aba" "abb"))))
```


### Compiler Errors {#compiler-errors}

-   (with-no-warning bodys...)

执行bodys时,不出现warnning警告

-   变量\`byte-compile-warnings'

控制编译时什么样的警告会被抛出


### Disassembly {#disassembly}

-   (disassemble object &amp;optional buffer-or-name)

显示object的反编译代码


## Reading and Printing Lisp Objects {#reading-and-printing-lisp-objects}

Reading是将文本转换为lisp object的过程

Printing是将lisp object转换为文本的过程


### Input Stream {#input-stream}

输入流可以是以下几种类型

-   Buffer

从buffer中光标所处位置开始读取

-   Maker

从buffer中指定Maker处开始读取

-   string

从string的第一个字母开始读取

-   function

由function产生要读取的字符. 这种函数必须支持两种调用模式:

1.  当无参数调用时,返回下一个要读取的字符

2.  当带一个参数(通常是一个字符)调用时,保存该参数,并在下一次无参数调用时返回它,及实现unreading功能.

3.  t

表示从minibuffer中读取,若Emacs运行在batch mode状态下,则使用stdin

-   nil

表示使用\`standard-input'的值作为输入流

-   symbol

表示使用symbol的函数定义作为输入,类似function


#### Input Functions {#input-functions}

-   (read &amp;optional stream)

    从stream中读取一个S表达式,并转换为Lisp Object返回

-   (read-from-string string &amp;optional start end)

    从string中读取一个S表达式,并返回'(lisp-object . postion)

    其中lisp-object为S读到的表达式,postion是string中剩余字符的位置(第一个未读字符的位置)
    ```emacs-lisp
    (read-from-string "(setq x 55) (setq y 5)") ; => ((setq x 55) . 11)
    (read-from-string "\"A short string\"")     ; => ("A short string" . 16)

    ;; Read starting at the first character.
    (read-from-string "(list 112)" 0)       ; => ((list 112) . 10)
    ;; Read starting at the second character.
    (read-from-string "(list 112)" 1)       ; => (list . 5)
    ;; Read starting at the seventh character, and stopping at the ninth.
    (read-from-string "(list 112)" 6 8)     ; => (11 . 8)
    ```

-   变量standard-input

    当stream参数为nil时,使用该变量的值作为steam参数的实参

-   变量read-circle

    若为非nil,则允许读取循环结果的S表达式,默认为t


### Output Stream {#output-stream}

输出流参数可以是以下类型:

-   buffer

输出字符插入到Buffer中的光标处,光标会随着字符的插入而向前移动.

-   Maker

输出字符插入Buffer中Maker处

-   Function

Elisp使用输出字符作为参数调用function,该function应该存储这些输出字符

-   t

输出结果到echo area

-   nil

使用\`standard-output'的变量值

-   symbol

使用symbol的function定义


#### Output Functions {#output-functions}

-   (print object &amp;optional stream)

    输出object的文本表示到stream中.

    输出时,在object的前后都会增加一个回车. 并且会输出引用字符
    ```emacs-lisp
    (progn (print 'The\ cat\ in)
           (print "the hat")
           (print " came back"))
    ;; -|
    ;; -| The\ cat\ in
    ;; -|
    ;; -| "the hat"
    ;; -|
    ;; -| " came back"
    ;; => " came back"
    ```
    该函数返回object的文本表示字符串

-   (prin1 object &amp;optional stream)

    类似print,但是不会在object的文本表示前后添加回车
    ```emacs-lisp
    (progn (prin1 'The\ cat\ in)
           (prin1 "the hat")
           (prin1 " came back"))
    ;; -| The\ cat\ in"the hat"" came back"
    ;; => " came back"
    ```

-   (princ object &amp;optional stream)

    该函数输出object的文本表示到stream中,并返回参数object.

    该函数一般用来输出对人可读的信息(而不是对read函数可以读),因此该函数并不会插入引用字符,也不会在字符串两边加上双引号,更不会自动插入空格分隔两次调用间的内容
    ```emacs-lisp
    (progn
      (princ 'The\ cat)
      (princ " in the \"hat\""))
    ;; -| The cat in the "hat"
    ;; => " in the \"hat\""
    ```

-   (terpri &amp;optional stream)

    输出newline到stream中

-   (write-char char &amp;optional stream)

    输出char到stream中,返回参数char

-   (prin1-to-string object &amp;optional noescape)

    该函数返回一个字符串,该字符串的内容就是(prin1 object)的输出
    ```emacs-lisp
    (prin1-to-string 'foo) ;; => "foo"
    (prin1-to-string (mark-marker)) ;; => "#<marker at 2773 in strings.texi>"
    ```
    若参数noescape为非nil,则输出时不使用引用字符
    ```emacs-lisp
    (prin1-to-string "foo")                 ; => "\"foo\""
    (prin1-to-string "foo" t)               ; => "foo"
    ```
    也可以使用format函数实现该功能

-   宏(with-output-to-string bodys...)

    该宏在将\`standard-output'设置为一个字符串的环境下执行bodys,然后返回该字符串
    ```emacs-lisp
    (with-output-to-string
      (princ "The buffer is ")
      (princ (buffer-name)))                ;=>"The buffer is foo"
    ```

-   (pp object &amp;optional stream)

    类似prin1,但是输出的格式更方便阅读.


#### Output Variables {#output-variables}

-   standard-output

    当参数stream为nil时,使用该变量的值

-   print-quoted

    若该值为非nil,表示使用简写形式输出quoted forms.例如
    (quote foo)输出为'foo, (function foo)输出为#'foo

-   print-escape-newlines

    若该值为非nil,则表示字符串中的newline字符,会被输出为\n,formfeed符会被输出为\f

    该参数影响prin1和print函数的输出方式,但是不能影响prnc的输出
    ```emacs-lisp
    (prin1 "a\nb")
    -| "a
    -| b"
    => "a
       b"

    (let ((print-escape-newlines t))
      (prin1 "a\nb"))
    -| "a\nb"
    => "a
       b"
    ```

-   print-escape-nonascii

    若该变量值为非nil,则字符串中的unibyte格式非ascii字符输出为\XXX的格式.

    该参数影响prin1和print函数

-   print-escape-multibyte

    若该变量值为非nil,则字符串中的mutibyte格式非ascii字符输出为\XXX的格式.

    该参数影响prin1和print函数

-   print-length

    该变量指明了输出list,vector或bool-vector时,能输出最多多少个元素. 若超过这么多个元素,则使用引号缩写
    ```emacs-lisp
    (setq print-length 2)                   ; => 2
    (print '(1 2 3 4 5))                    ; => (1 2 ...)
    -| (1 2 ...)
    ```
    nil表示无限制

-   print-level

    该变量值指明了输出时()和[]能够嵌套的最大深度,超过这个深度会用省略号代替,nil表示无限制

-   配置项eval-expression-print-length/eval-expression-print-level

    eval-expression中使用的print-length/print-level版本

-   print-circle

    若该值为非nil,则在输出时开启探测object是否有循环结构

-   print-gensym

    若该值为非nil,则输出时开启探测symbol是否是uninterned.

    这时,uninterned symbol输出时会带有前缀#:

-   print-continuous-numbering

    If non-\`nil', that means number continuously across print calls.
    This affects the numbers printed for \`#N=' labels and \`#M#' references.
    Don't set this variable with \`setq'; you should only bind it temporarily to \`t' with \`let'.
    When you do that, you should also bind \`print-number-table' to \`nil'

-   print-number-table

    This variable holds a vector used internally by printing to implement the \`print-circle' feature.
    You should not use it except to bind it to \`nil' when you bind \`print-continuous-numbering'.

-   float-output-format

    该变量指明了输出float类型数字时的格式. 默认为nil,表示在不丢失精度的情况下,使用最短的格式输出.


## Documentation {#documentation}


### 获取doc-string {#获取doc-string}

-   (documentation-property symbol property &amp;optional verbatim)

查看symbol的property属性中存储的doc-string,它会自动从DOC文件中或编译的字节码代码中抽取出对应的doc-string

若参数verbatim为nil,则,找到的doc-string会传入函数\`substitute-command-keys'进行键绑定说明的转换

```emacs-lisp
(documentation-property 'command-line-processed
                        'variable-documentation) ; => "Non-nil once command line has been processed"
(symbol-plist 'command-line-processed)           ; => (variable-documentation 188902)
(documentation-property 'emacs 'group-documentation) ; => "Customization of the One True Editor."
```

-   (documentation function &amp;optional verbatim)

该函数返回function的doc-string,其中function可以是macro,named keyboard macro,special forms,oridnary function

若参数verbatim为nil,则,找到的doc-string会传入函数\`substitute-command-keys'进行键绑定说明的转换

若function没有函数定义,则抛出\`void-function'错误,若函数定义没有doc-string,则返回nil

-   (face-documentation face)

返回face的doc-string

-   doc-directory

DOC文件的存放路径,Emacs可能要从DOC文件中读取doc-string


### 替换doc-string中的key binding {#替换doc-string中的key-binding}

当doc-string中要引用绑定的键序列时,使用特殊的引用形式可以通过函数\`substitute-command-keys'转换为指定命令真实的绑定键序列.

-   \`\\[COMMAND]'

显示调用COMMAND时的键序列,若COMMAND没有绑定键序列,则显示为M-x COMMAND

-   \`\\{MAPVAR}'

使用函数\`describe-bindings'显示MAPVAR所表示的keymap中的summary

-   \`\\&lt;MAPVAR&gt;'

转换为空值,该形式的说明会产生一个副作用:it specifies MAPVAR's value as the keymap for any following \`\\[COMMAND]' sequences in this documentation string.

-   \`\\='

引用接下来的那个字符;例如\`\\=\\['在显示时显示为\`\\[',而\`\\=\\='显示为\`\\='


### 将键序列输出为文本格式 {#将键序列输出为文本格式}

-   (key-description sequence &amp;optional prefix)

将sequence中的input event转换为文本格式

```emacs-lisp
(key-description [?\M-3 delete])        ; => "M-3 <delete>"
(key-description [delete] "\M-3")       ; => "M-3 <delete>"
```

-   (single-key-description event &amp;optinal no-angles)

将input event转换为文本形式字符串.

若参数no-angle为非nil,则在包围在function keys和event symols的尖括号会被忽略,这是为了与旧版本的Emacs兼容.

```emacs-lisp
(single-key-description ?\C-x)          ; => "C-x"
(key-description "\C-x \M-y \n \t \r \f123") ; => "C-x SPC M-y SPC C-j SPC TAB SPC RET SPC C-l 1 2 3"
(single-key-description 'delete)             ; => "<delete>"
(single-key-description 'C-mouse-1)          ; => "<C-mouse-1>"
(single-key-description 'C-mouse-1 t)        ; => "C-mouse-1"
```

-   (text-char-description character)

返回描述character的字符串,类似\`single-key-description'的作用

```emacs-lisp
(text-char-description ?\C-c)           ; => "^C"
(text-char-description ?\M-m)           ; => "\xed"
(text-char-description ?\C-\M-m)        ; => "\x8d"
(text-char-description (+ 128 ?m))      ; => "M-m"
(text-char-description (+ 128 ?\C-m))   ; => "M-^M"
```

-   命令(read-kbd-macro string &amp;optional need-vector)

\`key-description'的逆操作

参数string中包含了用空格分隔的key descriptions,该函数会返回一个string或vector,包含了对应的events.

若参数need-vector,则总是返回vector


## Elisp中的函数 {#elisp中的函数}

Elisp中的函数,是跟C++不同的. C++中的函数必须有一个函数名,然而Elisp中的函数没有函数名,只是你可以把它与一个symbol相连接,这样这个symbol的名字就暂时作为该函数的函数名了.

此外,Elisp中的函数可以通过与多个symbol相关连的方式而为同一个函数提供多个名称,而C++中的函数只有一个函数名称.


### Elisp中函数的分类 {#elisp中函数的分类}

函数的特性在与能够接收参数,然后返回计算结果,并可能产生一定的副作用. 在Elisp中符合这些特性的类函数对象有以下几种类型:

-   lambda expression

匿名函数

-   primitive

使用C语言编写的内置类函数对象,special form都认为是primitive的一种.

-   special form

类似C语言中的固定语法的语句

-   macro

宏,跟function类似,但它并不对参数进行预运算且返回的结果必须是一段S表达式.

-   command

可以通过\`command-execute\`这个primitive调用的对象,通常是一个带了interactive声明的函数(也可能是keyboard macro).

-   closure

闭包,带有静态作用域下变量的函数.

-   byte-code function

编译为字节码的函数

-   autoload object

它指向一个真实的函数的位置. 当真正调用到autoload object时,Emacs载入包含真正函数定义的那个文件,并且调用那个真正的函数.


### 获取函数信息 {#获取函数信息}

-   (functionp object)

判断object是否为函数. 若为函数则返回t,但macro和special form会返回nil

object也可以为symbol类型,会自动判断它所指向的function.

```emacs-lisp
(functionp 'goto-line)                  ;=>t
```

-   (subrp object)

判断object是否为primtive.

**注意:** 与functionp不同.object为symbol的话,会返回nil.

```emacs-lisp
(subrp 'message)            ; `message' is a symbol,
=> nil                 ;   not a subr object.
(subrp (symbol-function 'message))
=> t
```

-   (byte-code-function-p object)

判断object是否为byte-code function. object为symbol类型则返回nil

```emacs-lisp
(byte-code-function-p 'next-line) ; => nil
(byte-code-function-p (symbol-function 'next-line)) ; => t
```

-   (subr-arity subr)

这里subr需要为一个primitive对象(不能为symbol),该函数返回subr的最少参数个数和最大参数个数.

返回格式为'(MIN . MAX).

若参数有&amp;rest,则MAX为many.

若subr为special form,则该函数返回'unevalled

-   (interactive-form function)

获取function的interactive信息


### 匿名函数 {#匿名函数}


#### 获取匿名函数 {#获取匿名函数}

获取匿名函数,主要有三种方法:

-   使用lambda宏
    ```text
    (lambda (参数列表...)
        [函数描述字符串]
        [交互模式声明]
        函数体...)
    ```

-   使用function函数

    (function function-object)

    类似quote函数,它直接返回 **未计算** 的参数function-object.
    ```emacs-lisp
    (function 3)                            ;=>3,function的参数可以不为lambda表达式
    (function (lambda add-1(x) (1+ x)))     ;=>(lambda add-1(x) (1+ x)),但一般function的参数都是lambda表达式
    ```
    所不同的是,该函数告诉Emacs evaluator和byte-compiler,function-object为函数.

    具体来说,若function-object为lambda表达式,则有两个附加效果:

    1.  When the code is byte-compiled, FUNCTION-OBJECT is compiled into a byte-code function object

    2.  When lexical binding is enabled, FUNCTION-OBJECT is converted into a closure

-   使用\`#'\`标识

    \#'f是(function f)的缩写形式
    ```emacs-lisp
    ;; 一下三种写法是等价的
    (lambda (x) (* x x))
    (function (lambda (x) (* x x)))
    #'(lambda (x) (* x x))
    ```


#### 参数列表 {#参数列表}

参数列表的格式为:(必须参数列表...[&amp;optional 可选参数列表] [&amp;rest 剩余参数])

使用&amp;optional表示之后的参数是可选的.

使用&amp;rest表示之后的参数为不定参数. 它是实际参数的一个列表.

若在实际调用函数时,没有为可选参数和剩余参数提供实际参数值,则这些参数值为nil.


### 命名函数 {#命名函数}

使用fset/defalias将匿名函数与一个symbol想结合,就为这个匿名函数分配了一个名称.

-   (fset symbol lambda函数)

<!--listend-->

```emacs-lisp
(fset 'plus-one (lambda (x) (+ x 1)))
(plus-one 10)                           ;11
```

-   (defalias alias-name lambda-function-or-symbol &amp;optional doc-string)

为函数设定名字或别名,一般很少用到

```emacs-lisp
(defalias 'add-one '1+)
(add-one 11)                            ;12
(defalias 'add-two (lambda (x) (+ x 2)))
(add-two 11)                            ;13
```

实际上,更常见的定义命名函数的方法是使用defun宏

-   (defun 函数名 (参数列表) [函数说明字符串] [declare-form] [交互模式声明] 函数体...)

定义一个带名字的函数.

```emacs-lisp
(defun add-one (x)
  "return value after plus one"
  (+ x 1))

(add-one 10)                            ;11
```

**注意:** 它可以比lambda函数多一个declare-form部分,这个declare-form通常用于提供Elisp编译器一些函数的信息,以便进行优化.

-   (fboundp symbol)

判断symbol是否可以作为函数使用

-   (fmakunbound symbol)

删除函数,使symbol不再作为函数使用.


#### declare form {#declare-form}

declare form常用来为函数或宏添加一些关于属性的元标签. 它的语法是:

-   (declare specs...)

    其中spec的格式为(PROPERTY ARGS...),spec可以是以下几种说明

    -   (advertised-call-convention new-arg-list when)

    new-arg-list为正确的调用函数的方法,其他的调用方法都被认为是废弃的.

    when为一个表示什么时候废弃的字符串.

    -   (debug EDEBUG-FORM-SPEC)

    只能在定义宏时使用. 当用Edebug来调试该宏时,使用EDEBUG-FORM-SPEC

    -   (doc-string N)

    This is used when defining a function or macro which itself will be used to define entities like functions, macros, or variables

    它表示,第N个参数作为doc-string来看待

    -   (ident indent-spec)

    Indent calls to this function or macro according to INDENT-SPEC.

    虽然可以用在函数上,但一般还是用在宏定义中

    -   (obsolete current-name when)

    类似(make-obsolete),表示该函数被废弃了

    -   (compiler-macro EXPANDER)

    只能用在函数定义时,告诉编译器在编译时,使用EXPANDER代替该函数.

    这样的话,所有的(function args...)都实际上调用的是(EXPANDER args...)

    -   (gv-expander EXPANDER)

    Declare EXPANDER to be the function to handle calls to the macro (or function) as a generalized variable, similarly to \`gv-define-expander'.

    EXPANDER can be a symbol or it can be of the form \`(lambda (ARG) BODY)' in which case that function will additionally have access to the macro (or function)'s arguments.

    -   (gv-setter SETTER)

    Declare SETTER to be the function to handle calls to the macro (or function) as a generalized variable.
      SETTER can be a symbol in which case it will be passed to \`gv-define-simple-setter', or it can be of the form \`(lambda (ARG) BODY)' in which case that function will additionally have access to the macro (or function)'s arguments and it will passed to \`gv-define-setter'.


### 调用函数 {#调用函数}

最常用的调用函数的方式是将函数作为一个list的第一个参数. 这样当计算这个list时,会把地一个元素作为函数,其他作为参数来调用.

但是有的时候,需要在运行期间决定要执行的函数,这时候就需要使用以下函数的帮助:

-   (funcall function &amp;rest arguments...)

使用参数arguments调用函数function.

```emacs-lisp
(setq f 'list)                          ; => list
(funcall f 'x 'y 'z)                    ; => (x y z)
(funcall f 'x 'y '(z))                  ; => (x y (z))
```

参数function必须是一个lisp function或primitive function,而不能是macro或special form

-   (apply function &amp;rest arguments...)

类似funcall函数,但apply的arguments中,最后一个参数 **必须** 是list. 而这个list中的元素会被打散为独立的参数来作为function的实参.

```emacs-lisp
(setq f 'list)                          ; => list
(apply f 'x 'y 'z)                      ; error--> 最后一个参数z不是list类型
(apply '+ 1 2 '(3 4))                   ; => 10
(apply '+ '(1 2 3 4))                   ; => 10

(apply 'append '((a b c) nil (x y z) nil)) ; => (a b c x y z)
```

-   (apply-partially func &amp;rest args)

apply-partially使用参数args绑定func中的前(length args)个参数,并由此产生一个新的函数.

返回的新函数接受剩余的参数,并在内部调用原func函数.

```emacs-lisp
(defalias 'add-1 (apply-partially '+ 1)
  "Increment argument by one.")
(add-1 10)                              ; => 11
```

-   (identity arg)

该函数返回参数arg,没有任何其他处理

-   (ignore &amp;rest args)

该函数忽略args,直接返回nil

若要对某个集合(包括list)中的每个元素都调用某个函数(注意,不能是宏和special form),需要使用到map系列的函数.

**需要注意的是**,char-table比较特殊,只能用map-char-table函数调用.

-   (mapcar function sequence)

mapcar将sequence中的每个元素都调用一次function方法,并将结果组成一个list返回.

```emacs-lisp
(mapcar 'car '((a b) (c d) (e f)))      ; => (a c e)
(mapcar '1+ [1 2 3])                    ; => (2 3 4)
(mapcar 'string "abc")                  ; => ("a" "b" "c")
```

-   (mapc function sequence)

类似mapcar,但不收集个函数的运算结构. mapc的返回值为参数sequence

-   (mapconcat function sequence separator)

对sequence中的每个元素都调用function方法,其function方法计算的结果必须为string类型. 然后将这个string类型的结果用separator结合起来.

```emacs-lisp
(mapconcat 'symbol-name
           '(The cat in the hat)
           " ")                         ; => "The cat in the hat"

(mapconcat (function (lambda (x) (format "%c" (1+ x))))
           "HAL-8000"
           "")                          ; => "IBM.9111"
```

-   (cl-maplist function list...)

类似mapcar,但调用function的参数为(cdr list)

```emacs-lisp
(maplist #'(lambda (x) x)
         '(a b c))
;; ((a b c) (b c) (c))

```


### 废弃函数 {#废弃函数}

类似变量一样,函数也可以被标注为废弃的.

-   (make-obsolete obsolete-name current-name &amp;optional when)

该函数标注obsolete-name为废弃的. 其中

obsolete-name可以为表示函数或宏的symbol,也可以为函数或宏的别名.

current-name可以是一个symbol,表示使用current-name代替obsolete-name. 也可以是一个字符串表示废弃的警告说明. 也可是nil.

when应该是一个日期或版本号的字符串,用于表示什么时候开始废弃该函数.

-   (define-obsolete-function-alias obsolete-name current-name &amp;optional when doc)

该宏定义obsolete-name为函数current-name的别名,同时标注obsolet-name为废弃的函数.

该宏等价于:

```emacs-lisp
(defalias OBSOLETE-NAME CURRENT-NAME DOC)
(make-obsolete OBSOLETE-NAME CURRENT-NAME WHEN)
```

-   (set-advertised-calling-convention function new-arg-list when)

该函数与上面两个函数不同点在于,它不是标注某个函数为废弃的,它只标注某个函数的某种用法为废弃的.

任何不使用new-arg-list表示的实参调用function函数都会被警告为废弃的.

when表示什么时候开始废弃function的原用法,一般为表示版本号的字符串.

```emacs-lisp
;; 在老版本中sit-for函数可以接受三个参数
(sit-for seconds milliseconds nodisp)

;; 然而用这种调用方法在Emacs22.1版本之后就被废弃掉了,因此可以这样设置
(set-advertised-calling-convention
 'sit-for '(seconds &optional nodisp) "22.1") ;表示新的sit-for函数签名为(defun sit-for (seconds &optional nodisp))
```


### 内联函数 {#内联函数}

要定义内联函数,只需要将定义函数的defun,换成defsubst即可

```text
(defsubst name (arg-list)
   [doc-string]
   [declare-form]
   [interactive]
   bodys)
```

注意:虽然内联函数会加快函数的执行速度,但它会增加文件和内存的消耗量,而且对debugging,tracing和asdising支持不够好,因此除非速度真的很重,否则不要用内联函数.


### 函数声明 {#函数声明}

-   (declare-function function file &amp;optional arglist fileonly)

该宏告诉编译器,function函数在文件file中定义,且参数签名为arglist.

编译器会检查文件file中是否包含了function函数,且参数签名是否为arglist,若想让编译器不检查函数的参数签名,需要将arglist设置为t

若参数fileony为非nil,表示只检查file存在,而不检查文件中是否定义了function.


### 函数描述字符串(docstring) {#函数描述字符串--docstring}

-   一般来说,函数描述字符串的第一行为对该函数作用的总结.
    -   docstring的第一行最好独立的,因为apropos命令只显示第一行的文档
    -   docstring中参数最好用大些字母
    -   docstring以\*开头的defvar变量被认为是用户选项（user option）
    -   用户选项可以通过命令set-variable交互设置
    -   可以使用edit-options命令编辑\*scratch\*
    -   \`符号名'生成一个链接
    -   \\\\{major-mode-map}可以显示扩展成按键的说明
    -   docstring最后那个的\\[ command ]会被command的绑定键所代替
    -   如果不想要这种代替，需要用\\=转义，当然，在Emacs的docstring中，真正的写法应该是
        ```elisp
        "\\=\\{major-mode-map}"
        "\\=\\[command]"
        ```
-   将\`\n(fn ARGLIST)\`放在最后一行,会自动扩展为该函数的实际参数列表.


### 交互模式声明 {#交互模式声明}

若一个函数带了交互模式声明,则它也就是一个命令了,即可以通过M-x(execute-command)来调用了.

交互模式声明的格式为(interactive code-string),其中:

-   若interactive的参数以\*开头，则意义是，如果当前buffer是只读的，则不执行该函数

-   interactive可以后接字符串,表示获得参数的方式
-   p 接收C-u的数字参数

    也可以不用P参数,直接在代码中判断current-prefix-arg的值
-   r region的开始/结束位置
-   n 提示用户输入数字参数,n后面可用接着提示符
-   s 提示用户输入字符串参数
-   若函数接收多个input,需要用\n来分隔

-   interactive可以后接一个form,form的求值结果应该是一个list,这个list的值作为参数的实参

在form中一般会用到如下几个函数用于获取用户输入

-   read-string
-   read-file-name
-   read-directory-name
-   read-regexp
-   y-or-n-p
-   read-from-minibuffer
-   使用变量\`current-prefix-arg\`来判断是否有universal-argument


### declare-form {#declare-form}

使用\`declare'宏能够为函数或宏增加元属性. 它的语法为:

```emacs-lisp
(declare specs...)
```

其中spec的格式为\`(property args...)'. 目前支持以下几种spec

-   (advertised-calling-convention SIGNATURE WHEN)

    该spec的功能类似调用\`set-advertised-calling-convention'函数

    其中,SIGNATURE为参数列表,指定了调用函数或宏的正确用法. WHEN为一个字符串指明了什么时候开始废除原函数用法.

-   (debug edebug-form-spec)
    该spec只对宏有效. 当使用edebug调试宏时,使用edebug-form-spec. 参见[Instrumenting Macro Calls](https://www.gnu.org/software/emacs/manual/html_node/elisp/Instrumenting_002520Macro_002520Calls.html "Emacs Lisp: (info \"(elisp) Instrumenting%20Macro%20Calls\")")

-   (doc-string n)
    指明第n个参数为documentation string

-   (indent indent-spec)
    对当前函数或宏缩进时,根据indent-spec来缩进. 该功能通常用在宏中,但也对函数生效. 参见[Indenting Macros](https://www.gnu.org/software/emacs/manual/html_node/elisp/Indenting_002520Macros.html "Emacs Lisp: (info \"(elisp) Indenting%20Macros\")")

    indent-spec可以是:

    -   nil
        使用标准缩进模式

    -   defun
        将该宏看成是一个\`def'结构: 这种结构中会将第二行看成是body的起始行

    -   an integer,number
        The firstnumberarguments of the function aredistinguishedarguments; the rest are considered the body of the expression.
        A line in the expression is indented according to whether the first argument on it is distinguished or not.
        If the argument is part of the body, the line is indented lisp-body-indent more columns than the open-parenthesis starting the containing expression.
        If the argument is distinguished and is either the first or second argument, it is indented twice that many extra columns.
        If the argument is distinguished and not the first or second argument, the line uses the standard pattern.

    -   表示函数名称的symbol
        该函数接收两个参数\`pos'和\`state',并且应该返回一个表示应该缩进到多少列的数字,或一个car表示缩进列的list. 其中
        -   **pos:** 光标当前行缩进的起始位置(The position at which the line being indented begins.)

        -   **state:** 函数\`parse-partial-sexp'解析从光标当前位置到当前行行首之间内容的返回结果

            The difference between returning a number and returning a list is that a number says that all following lines at the same nesting level should be indented just like this one; a list says that following lines might call for different indentations. This makes a difference when the indentation is being computed byC-M-q; if the value is a number, C-M-qneed not recalculate indentation for the following lines until the end of the list

<!--listend-->

-   (obsolete current-name when)
    该spec功能类似调用\`make-obolete'函数

    current-name为一个symbol,或string,或nil

    when为一个字符串用来指定什么时候开始废弃该函数/宏


### 判断function是否安全 {#判断function是否安全}

使用unsafep来判断一个form是否是安全的

-   (unsafep form &amp;optional unsafep-vars)

若判断form为安全的可以执行,则返回nil. 否则返回一个list描述为什么form是不安全的.

The argument UNSAFEP-VARS is a list of symbols known to have temporary bindings at this point;

The current buffer is an implicit argument, which provides a list of buffer-local bindings.


## Advising Emacs Lisp Functions {#advising-emacs-lisp-functions}

Emacs's advice system provides two sets of primitives for that:
the core set, for function values held in variables and object fields (with the corresponding primitives being \`add-function' and \`remove-function') and
another set layered on top of it for named functions (with the main primitives being \`advice-add' and \`advice-remove').


### Core Advising Primitives {#core-advising-primitives}

-   (add-function where function-place advise-function &amp;optional props)

为存储function的place(泛化变量)加上advise-function,使之称为一个组合了原始函数和advise函数的组合函数.

-   where参数指明了advise-function与function-place处函数的整合方式.

-   :before

在调用原function(function-place所存放的function)前调用advise-function.

原function与advise-function接收同样的参数调用,并以原function的返回结果为组合函数的返回结果

```emacs-lisp
(add-function :before 'old-function 'advise-function)
;; 等价于
(lambda (r) (advise-function r) (old-function r))
```

-   :after

在原function调用之后调用advise-function.

原function与advise-function接收同样的参数调用,并以原function的返回结果为组合函数的返回结果

```emacs-lisp
(add-function :after 'old-function 'advise-function)
;; 等价于
(lambda (r) (prog1(advise-function r) (old-function r)))
```

-   :override

用advise-function代替原function

-   :around

使用advise-function代替原function,但原function会作为第一个参数传递給advise-function. 这样advise-function内可以调用原函数.

```emacs-lisp
(add-function :around 'old-function 'advise-function)
;; 等价于
(lambda (r) (apply 'advise-function 'old-function r))
```

-   :before-while

先执行advise-function,若advise-function返回nil,则不再调用原function.

advise-function与原function公用一样的参数,使用原function的结果作为组合函数的返回值

```emacs-lisp
(add-function :before-while 'old-function 'advise-function)
;; 等价于
(lambda (r) (and (apply 'old-function r) (appy 'advise-function r)))
```

-   :before-until

先执行advise-function, 只有当advise-function返回nil,才调用原function.

advise-function与原function共用一样的参数,使用原function的结果作为组合函数的返回值

```emacs-lisp
(add-function :before-while 'old-function 'advise-function)
;; 等价于
(lambda (r) (or (apply 'old-function r) (appy 'advise-function r)))
```

-   :after-while

先调用原function,若function返回nil,则不调用advise-function.

原function和advise-function共用同样的参数. 组合函数的返回结果为 **advise-function** 的返回结果

```emacs-lisp
(add-function :after-while 'old-function 'advise-function)
;; 等价于
(lambda (rest r) (and (apply 'old-function r) (apply advise-function r)))
```

-   :after-until

先调用原function,只有当function返回nil时,才调用advise-function.

原function和advise-function共用同样的参数. 组合函数的返回结果为 **advise-function** 的返回结果

```emacs-lisp
(add-function :after-while 'old-function 'advise-function)
;; 等价于
(lambda (rest r) (or (apply 'old-function r) (apply advise-function r)))
```

-   :filter-args

先用原始参数调用advise-function,再将advise-function返回的结果(advise-function必须返回一个list)作为参数,来调用原function.

```emacs-lisp
(add-function :filter-args 'old-function 'advise-function)
;; 等价于
(lambda (reset& r) (apply 'old-function (funcall 'advise-function r)))
```

-   :filter-return

先调用old-function,将结果作为参数调用advise-function.

```emacs-lisp
(add-function :filter-return 'old-function 'advise-function)
;; 等价于
(lambda(rest& r) (funcall 'advise-function (apply 'old-function r)))
```

-   function-place为被添加advise-function的函数位置. 它同时也决定了该advise是全局都有用,还是只在当前buffer生效.

    若function-place是一个symbol,则该advise全局生效

    若function-place为'(local SYMBOL-expression),这里SYMBOL-experssion表示一个expression,它的计算结果为一个表示变量的symbol. 则该advise只在当前buffer生效

    若要对静态作用域下的变量提出advise,则function-place的格式应为(var VARIABLE)

    -   props参数为一个代表属性的alist,目前只支持两个属性:

    name属性,表示该advice的名字,当remove-function时有用. 尤其是当advise-function为匿名函数时,特别有用.

    depth属性,表示优先级,用于决定多个advise-function的执行顺序.
    他的取值范围从-100(表示最接近原始函数的执行顺序)到100(表示里原始函数的执行顺序最远). 默认为0
      当两个advise-function用了同一个优先级,则最后添加的advise-function会覆盖前面的.

    -   advise-function参数

    若advise-function没有interactive声明,则advise后的组合函数会继承原始函数的interactive声明.

    若advise-function有interactive声明,则advise后的组合函数使用advise-function的interactive声明.

    上述关于advised后的组合函数的interactive声明,在某一种情况下不成立:
      if the interactive spec of FUNCTION is a function (rather than an expression or a string), then the interactive spec of the combined function will be a call to that function with as sole argument the interactive spec of the original function.  To interpret the spec received as argument, use \`advice-eval-interactive-spec'.

-   (remove-function function-place advise-function)

删除通过add-function添加到function-place的advise-function

-   (advice-function-member-p advice-function function-def)

判断advice-function是否已经function-def的advice

-   (advice-function-mapc f function-def)

用添加到function-def的每个advicse-function和对应的propos作为参数,都调用一次f函数.

-   (advice-eval-interactive-spec interactive-spec)

根据interactive-spec所声明的interactive方式,返回对应的获取值.


### Advising Named Functions {#advising-named-functions}

advice的最常用法是給命名函数或宏添加advice

这种方法会引入一些问题,最好在没有办法的时候,使用下面的方法添加advice

-   (advice-add function-symbol where advice-function &amp;optional props)

    为function-symbol添加名为advice-function的advice. where和props参数与add-function一致

-   (advice-remove function-symbol advise-function)

删除function-symbol上的advise-function

-   (advice-member-p advise-function function-symbol)

判断advise-function是否已经是function-symbol的advice了

-   (advice-mapc f function-symbol)

使用function-symbol中的每个advise-function及其对应的props作为参数,用f来调用.


## 宏 {#宏}


### 宏与函数的不同 {#宏与函数的不同}

-   宏的参数在传递給宏前并不会作计算处理,也就是说宏看到的是传递给它的原始参数. 而函数参数传递給函数时是已经经过计算的结果,也就是说函数看到的是参数的计算结果.
-   宏的计算结果需要是一个S表达式(这个过程被称为宏扩展),Elisp会再计算这个返回的S表达式以算出最终结果.


### 定义宏 {#定义宏}

定义宏的格式与定义函数的格式一样,只是用defmacro替代defun

```text
(defmacro macro-name (参数列表...)
        [函数描述字符串]
        [declare-form]
        [交互模式声明]
        函数体...)
```

destructuring-bind宏接受一个匹配模式,一个求值到列表的实参,以及一个表达式体，然后在求值表达式时将模式中的参数绑定到列表的对应元素上.例如

```emacs-lisp
(destructuring-bind (x (y) . z) '(a (b) c d)
  (list x y z))
```

(a b (c d))

defmacro宏允许任意列表结构作为参数列表.当宏调用被展开时,宏调用中的各部分将会以类似destructuring-bind的方式被赋值到宏的参数上面
例如,可以这样定义dolist宏

```emacs-lisp
(defmacro our-dolist ((var list &optional result) &body body)
  `(progn
      (mapc #'(lambda (,var) ,@body)
               ,list)
      (let ((,var nil))
        ,result)))
```


### 宏扩展 {#宏扩展}

调用宏会将传递給宏的参数扩展成一个S表达式,这个过程称为宏扩展过程.

-   (macroexpand macro-form &amp;optional environment)

    递归扩展macro-form直到结果中不再为宏调用为止(不代表结果中就不包含宏了,只是第一个元素不为宏而已). 然后返回扩展结果.
    ```emacs-lisp
    (defmacro inc (var)
      (list 'setq var (list '1+ var)))

    (macroexpand '(inc r))                  ; => (setq r (1+ r))

    (defmacro inc2 (var1 var2)
      (list 'progn (list 'inc var1) (list 'inc var2)))

    (macroexpand '(inc2 r s))               ; => (progn (inc r) (inc s))  ; `inc'并没有扩展
    ```
    environment参数为一个包含宏定义的alist. macroexpand在扩展宏时会使用environment中的宏定义替代当前环境下的宏定义.

-   (macroexpand-all macro-form &amp;optional environment)

    类似macroexpand,但会递归扩展直到结果中不再包含宏为止.
    ```emacs-lisp
    (defmacro inc (var)
      (list 'setq var (list '1+ var)))

    (macroexpand '(inc r))                  ; => (setq r (1+ r))

    (defmacro inc2 (var1 var2)
      (list 'progn (list 'inc var1) (list 'inc var2)))

    (macroexpand-all '(inc r s))           ; => (progn (setq r (1+ r)) (setq s (1+ s))) inc也扩展了
    ```

当对程序进行编译时,编译器在遇到宏调用时,会对宏进行扩展,因此:

-   要注意分清哪些操作应该放在宏扩展的过程中完成,哪些操作放在宏扩展后的结果中进行. 例如
    ```emacs-lisp
    (defmacro my-set-buffer-multibyte (arg)
      (if (fboundp 'set-buffer-multibyte)
          (set-buffer-multibyte arg)))      ;在编译期执行该操作其实是没有意义的,应该改为`(set-buffer-multibyte ,arg)
    ```

-   不要在宏中对宏参数进行eval操作. 因为这时候宏参数还并未绑定任何实际参数.

-   由于编译器只对宏进行一次扩展,在其他使用宏的地方不再进行扩展动作,而在解释执行时会在每次宏调用时都对宏进行扩展. 因此宏扩展的过程,不能产生副作用,否则就会发生编译和解释执行结果不一致的情况. 例如:
    ```emacs-lisp
    (defmacro empty-object ()
      (list 'quote (cons nil nil)))

    ;; 上面的宏在解释执行时,每次都会生成一个新的(nil).
    ;; 但在编译执行时,会在编译器生成一个(nil),然后每次都使用它
    ```

-   如果我们在主调函数编译以后，重定义那个宏. 由于对最初的宏调用的无迹可寻，所以函数里的展开式无法更新。该函数的行为将继续反映出宏的原来的定义

    此外,如果在定义宏之前，就已经编译了宏的调用代码，也会发生类似的问题.

    为了避免这类问题,我们需要

    1.  在调用宏之前，先定义它。

    2.  一旦重定义一个宏，就重新编译所有直接(或通过宏间接)调用它的函数(或宏)。


### 宏的工作模式 {#宏的工作模式}

下面是一个宏的模拟实现

```emacs-lisp
(defmacro our-expander (name) `(get ,name 'expander))
(defmacro our-defmacro (name parms &body body)
  (let ((g (gensym)))
    `(progn
        (setf (our-expander ',name)
              #'(lambda (,g)
                   (block ,name
                     (destructuring-bind ,parms (cdr ,g)
                       ,@body))))
        ',name)))
(defun our-macroexpand-1 (expr)
  (if (and (consp expr) (our-expander (car expr)))
      (funcall (our-expander (car expr)) expr)
    expr))
```


## 调试ELisp程序 {#调试elisp程序}

有以下几种调试Elisp程序的方法

-   若运行程序时抛出异常,可以使用Emacs内置的debugger或edebug
-   通过查看编译器定位问题.
-   使用ERT包来写回归测试
-   使用profile来定义性能关注点


### debugger {#debugger}


#### 配置何时进入debugger {#配置何时进入debugger}

当Elisp程序运行时,若发生error,则根据配置项\`debug-on-error\`决定是否进入debugger.

-   配置项debug-on-error

若值为t,则任何种类的error都会进入debugger.

若值为nil,则任何种类的error都不会进入debugger

若值为error conditon的列表,则只有指定种类的error会进入debugger

-   配置项debug-ignored-errors

该变量在debug-on-error的基础上,屏蔽指定种类的error不触发debugger.

该变量的值为一个由error condition和正则表达式组成的list. 任何符合error condtion的error,和error message匹配指定正则表达式的error,都不会触发debugger

-   配置项eval-expression-debug-on-error

该配置项的值为t时,使用eval-expression命令时,会将debug-on-error的值临时改为t.

若该配置项为nil,则不会修改debug-on-error的值.

-   变量debug-on-signal

默认情况下,若error被condition-case所捕获,则不会进入debugger,但若该变量为非nil,则会先进入debugger,再被condition-case所捕获.

当然,是否进入debugger,还要看debug-on-error和debug-ignored-errors的值

一般不会使用该变量,而使用condition-case-unless-debug代替.

-   debug-on-event

当Emacs捕获到指定event发生时,进入debugger

-   debug-on-message

该变量为一个正则表达式,当在echo area显示了符合该正则的message时,进入debugger.

该变量常用于寻找引起特定message的原因.

-   配置项debug-on-quit

    当按下C-g时,会产生一个quit,quit和error不是一回事,因此quit默认是不进入debugger的.

    通过设置该值为非nil,则当quit发生时,进入debugger

-   命令(debug-on-entry function-symbol)

    该命令标注当指定的function被调用时,主动进入debugger(无论有没有error/quit发生)
    ```emacs-lisp
    (defun fact (n)
      (if (zerop n) 1
        (* n (fact (1- n)))))               ; => fact
    (debug-on-entry 'fact)                  ; => fact
    (fact 3)                                ; => 进入debugger
    ```

-   命令(cancel-debug-on-entry &amp;optional function-symbol)

    该函数取消debug-on-entry对指定function的操作.

    若function-symbol为nil,则表示debug-on-entry对所有函数的操作.

-   (debug &amp;rest debugger-args)

    显式调用debugger. 程序执行到该语句,会立刻进入debugger.

    通常debugger只是把debugger-args的值显示在backtrace buffer的首部,以方便用户可以看到它.

    然而,有些debugger-arg有其特殊的意义:

    -   第一个参数为lambda

    当debugger是由于参数\`debug-on-next-call\`设置为非nil的情况下进入了函数而触发的,则该变量会显示为\`Debugger entered--entering a function:\`

    -   第一个参数为debug

    当debugger是由于设置了参数\`debug-on-entry\`的情况下进入了函数而触发的,则该变量会显示为\`Debugger entered--entering a function\`

    -   第一个参数为t

    当debugger是由于参数\`debug-on-next-call\`设置为非nil的情况下进入了函数而触发的,则该变量会显示为\`Debugger entered--beginning evaluation of function call form\`

    -   第一个参数为exit,第二个参数为debug

    当debugger是由于之前被b标记过的stack frame退出而触发的情况下,会显示为\`Debugger entered--returning value:\`加上返回的值

    -   第一个参数为error

    当debugger是由于error/quit未捕获而触发时,会显示为\`Debugger entered--Lisp error:\`加上error的信息

    -   当第一个参数为nil

    TODO 不知道什么意思.


#### debugger使用说明 {#debugger使用说明}

当进入debugger后,会有一个名为\*Backtrace\*的buffer出现.

在该buffer的第一行显示了进入debugger的原因,下面是backtrace.

backtrace由一系列的stack frame组成,每行一个stack frame. 其中

-   光标所在的stack frame为当前frame,有些debugger命令会对当前frame进行操作
-   若某stack frame以\*开头,表示离开这个stack frame会再次调用debugger.
-   若stack frame中的函数名带了下划线,表示debugger可以找到该函数的源代码.

当进入debugger后,会根据\`eval-expression-debug-on-error\`的值来临时更改\`debug-on-error\`的值.若\`eval-expression-debug-on-error\`的值为t,则会设置debug-on-error的值为t. 这意味着在debug中触发的任意error都会产生一个backtrace. 若不想这样,可以设置\`eval-expression-debug-on-error\`为nil,或在\`debugger-mode-hook\`中设置\`debug-on-error\`为nil

Debugger中的命令:

-   c

    continue. 退出debugger,并且继续向下执行.

-   d

    步进一个S表达式,然后看步进的那个S表达式做了什么操作.

-   b

    为当前frame加上断点标记. 加了标志的frame,在行头会加上一个\*

-   u

    取消b命令为frame加上的断点标志.

-   j

    像b命令一样为当前frame加上标记,然后像命令c一样继续执行程序,但是在执行时会临时屏蔽\`debug-on-entry\`标记

-   e

    执行输入的S表达式. 通过该命令,可以修改debugger的变量.

-   R

    类似e,但是会把执行S表达式的结果,保存到一个名为\*Debugger-record\*的buffer中.

-   q

    退出debugger,退出程序的执行过程

-   r

    读取一个S表达式,并将其计算结果作为当前frame的返回值.

    当debugger是由于捕获到异常而触发时,无法使用该命令.

-   l

    列出会debug-on-entry的函数列表. 该列表根据\`debug-on-entry\`的值来过滤函数.

-   v

    显示/不现实当前stack frame中的局部变量


#### debugger内部实现使用到的变量与函数 {#debugger内部实现使用到的变量与函数}

-   debugger

    该变量的值需要是一个函数,当触发debugger时,实际上是通过调用该函数来实现的.

    该变量默认值为\`debug\`

-   (backtrace)

    This function prints a trace of Lisp function calls currently active.
    ```emacs-lisp
    (defun show-back-trace()
      (backtrace))

    (show-back-trace)

    ;; ==================>
    backtrace()
    show-back-trace()
    eval((show-back-trace) nil)
    eval-last-sexp-1(nil)
    eval-last-sexp(nil)
    call-interactively(eval-last-sexp nil nil)
    command-execute(eval-last-sexp)


    ```

-   变量debug-on-next-call

    若该值为nil,则在执行下一个eval,apply或funcall之前,先会调用debugger.

    进入debugger后,会将该值设置为nil,但debugger中的d命令会将该变量设置为t

-   (backtrace-debug level flag)

-   变量command-debug-status

    该变量记录了当前interctive command的debug状态.

-   (backtrace-frame frame-number)


### edebug {#edebug}


#### 使用Edebug的一般步骤 {#使用edebug的一般步骤}

1.  引入函数/宏到edebug中来调试

    将光标移动到要debug的函数/宏定义上,按下C-u C-M-x(eval-defun)

    一旦函数/宏被引入到edebug来调试,任何调用该函数/宏的操作都会触发edebug执行.

2.  Edebug跳转到定义Elisp源代码的buffer,且该buffer暂时变为只读的.

3.  使用Edebug命令开始调试,可以使用\`?\`来显示Edebug命令

4.  若不需要在用Edebug调试了,需要将函数/宏引出Edebug,方法是再执行一边函数/宏的定义即可.


#### Edebug中的命令 {#edebug中的命令}

<!--list-separator-->

-  Execution Mode

    Edebug在调试程序时,也支持各种execution modes. 但它并不是实际上的major-mode或minor-mode

    execution mode决定了Edebug下一次在哪里暂停,以及在暂停时显示多少执行的信息.

    | 命令        | 说明                                                                                |
    |-----------|-----------------------------------------------------------------------------------|
    | S           | Stop:不再往下执行程序,等待用户输入更多的Edebug命令(edebug-stop)                     |
    | &lt;SPC&gt; | Step:步进下一个语句(edebug-step-mode)                                               |
    | n           | Next:跳到下一个Form(edebug-next-mode)                                               |
    | t           | Trace:每执行一个语句(会在echo area显示每个语句执行的结果)就暂停一段时间(默认为1s,由参数\`edebug-sit-for-seconds\`确定) |
    | T           | Rapid trace:类似t,但并不暂停(edebug-Trace-fast-mode)                                |
    | g           | Go:继续执行直到下一个端口(edebug-go-mode)                                           |
    | c           | Continue:继续执行,在每个断点处都停顿一下,然后继续执行(edebug-continue-mode)         |
    | C           | Rapid continue:类似c,但在断点处并不停顿(edebug-continue-fast-mode)                  |
    | G           | Go non-stop:忽略断点的存在,继续执行程序.                                            |

    在程序执行过程中,可以用S或其他命令暂停程序的执行

<!--list-separator-->

-  Jumping命令

    Jumping系列命令告诉Edebug,让程序执行直到指定的位置

    | 命令 | 说明                             |
    |----|--------------------------------|
    | h  | 执行到光标所在位置(edebug-goto-here) |
    | f  | 执行一个sexp(edebug-forward-sexp) |
    | o  | 执行完(跳出)当前的sexp(edebug-step-out) |
    | i  | 进入form所调用的函数/宏定义(edebug-step-in) |
    |    |                                  |

<!--list-separator-->

-  Breaks

    在三种情况下,Edebug会暂停程序的执行:

    -   设置断点

        | 命令                         | 说明                                                                                                  |
        |----------------------------|-----------------------------------------------------------------------------------------------------|
        | b                            | 设置断点(edebug-set-breakpoint),若带prefix argument,则该断点为临时断点                                |
        | u                            | 取消断点(edebug-unset-breakpoint)                                                                     |
        | x CONDITION-FORM &lt;RET&gt; | 设置条件断点,当运行CONDITION-FORM的结果为非nil时,断点生效(edebug-set-conditional-breakpoint). 若带prefix argument,则断点为零时断点 |
        | B                            | 光标跳转到下一个断点处(edebug-next-breakpoint)                                                        |

        re-evaluting/reinstrumenting函数定义会移除之前的所有断点

        设置条件断点时,CONDITION-FORM抛出的error会被忽略,当成返回nil来看待.

        一般情况下,Edebug会在断点处暂停程序的执行. 然而,当Edebug处于Go-nonstop mode下时,会完全忽略断点.

    -   当某个条件(Global Condition)匹配时

        若变量\`edebug-global-break-condtion\`的计算结果为非nil则暂停程序的执行. 同样的,若计算过程中抛出error,则当返回nil处理.

        可以在edebug模式下使用X命令来设置该条件. 也可以在任何buffer的任何时候,通过C-x X X来调用(edebug-set-global-break-condition)设置该条件

    -   插入断点代码

        使用(edebug)主动调用edebug,进入断点模式.

        若执行(eedbug)时,该函数并未引入到edebug中,则该函数其实调用的是(debug)

<!--list-separator-->

-  Evaluation

    | 命令                | 说明                                                    |
    |-------------------|-------------------------------------------------------|
    | e EXP &lt;RET&gt;   | 在Edebug的外部上下文环境中计算EXP(edebug-eval-expression) |
    | M-: EXP &lt;RET&gt; | 在Edebug的上下文环境中计算EXP(eval-expression)          |
    | C-x C-e             | 在Edebug的外部上下文环境中计算光标前的expression(edebug-eval-last-sexp) |

<!--list-separator-->

-  Evalution List Buffer

    在Edebug中可以按E命令,进入名为\*edebug\*的"evaluation list buffer".

    在该buffer中可以交互式的运行SEXP,且在该buffer中计算的SEXP,处于Edebug外部的上下文环境中.

    可以在该buffer中使用Lisp Interaction mode中的命令. 还可以执行以下命令:

    -   C-j (edebug-eval-print-last-sexp)

        在edebug的外部上下文环境中,计算光标前的expression,并将结果插入到当前buffer

    -   C-x C-e (edebug-eval-last-sexp)

        在edebug的外部上下文环境中,计算光标前的expression

    -   C-c C-u (edebug-update-eval-list)

        基于当前buffer的内容,创建新的evaluation list

        evalution list由多个evalutation list groups组成. 每个groups由多个Lisp expression组成,group之间使用注释行来区分.

        当edebug每次暂停程序执行时,每个evaluation list group中的地一个Lisp expression都会自动执行一遍.
        ```emacs-lisp
        (current-buffer)
        #<buffer *scratch*>
        ;---------------------------------------------------------------
        (selected-window)
        #<window 16 on *scratch*>
        ;---------------------------------------------------------------
        (point)
        196
        ;---------------------------------------------------------------
        bad-var
        "Symbol's value as variable is void: bad-var"
        ;---------------------------------------------------------------
        (recursion-depth)
        0
        ;---------------------------------------------------------------
        this-command
        eval-last-sexp
        ;---------------------------------------------------------------

        ```

    -   C-c C-d (edebug-delete-eval-list)

        从evalution list中删除光标所在的group

    -   C-c C-w (edebug-where)

        切换回当前暂停点的原代码buffer处

<!--list-separator-->

-  其他命令

    | 命令 | 说明                                                                         |
    |----|----------------------------------------------------------------------------|
    | ?   | 显示Edebug的帮助信息(edebug-help)                                            |
    | C-] | Abort one level back to the previous command level(\`abort-recursive-edit')  |
    | q   | 终止程序运行并退出edebug,但\`unwind-protect\`和\`condition-case\`中的代码还是会执行 |
    | Q   | 类似q,但\`unwind-protect\`和\`condition-case\`中的代码不会执行(edebug-top-level-nonstop) |
    | r   | 重新在echo area中显示上次expression的运算结果(edebug-previous-result)        |
    | d   | 显示backtrace(但是不显示Edebug自己的function,并且此时处于标准debugger模式下),(edebug-backtrace) |
    |     |                                                                              |

<!--list-separator-->

-  捕获Errors

    默认情况下,若某函数被引入edebug中,则当该函数抛出error时,会自动激活edebug.

    但可以通过配置变量\`edebug-on-error\`和\`edebug-on-quit\`来改变这一情况.

<!--list-separator-->

-  Edebug Views


#### Edebug中的输出格式 {#edebug中的输出格式}

当Edebug输出循环list结构时,可能会出错,这时需要设置一下几个变量

-   配置项edebug-print-length

-   配置项edebug-print-level

-   配置项edebug-print-circle


#### Trace Buffer {#trace-buffer}

通过设置变量\`edebug-trace\`的值为非nil,可以使得Edebug将每次执行的过程都记录下来.

记录存储在名为\`\*edebug-trace\*\`的buffer中. 它记录了使用什么参数调用那个函数,返回值是什么.


### test coverage {#test-coverage}


#### 使用步骤 {#使用步骤}

通过testcover库,能够对代码进行铺盖面测试. 方法是:

1.  载入testcover库

    (require 'testcover)

2.  执行命令testcover-start

    M-x testcover-start &lt;RET&gt; FILE &lt;RET&gt;

3.  然后对你的代码进行测试

4.  测试完成后执行命令testcover-mark-all命令会高亮出覆盖面不完全的地方

    M-x testcover-mark-all

5.  使用命令testcover-next-mark跳转到下一个高亮点


#### 高亮说明 {#高亮说明}

一般来说,红色的高亮表示这个地方从来没有测试过.

棕色的高亮表示这个地方的计算结果每次都一样的,而这往往意味着测试得还不够充分.


#### give advice to the test coverage too {#give-advice-to-the-test-coverage-too}

可以通过将代码包裹进一些宏(这些宏本身不会改变代码的执行结果)中,来告诉testcover一些信息.

-   (1value form)

    该宏告诉testcover,form的计算结果每次都应该一样的.

-   (noreturn form)

    该宏告诉testcover,form不应该返回. 若form返回了,会收到一个run-time error


### Profiling {#profiling}

-   使用M-x profiler-start开启性能监控,然后选择监控cpu还是mem还是两者都监控.

-   执行操作,运行待测试的函数

-   执行M-x profiler-report显示性能检测结果.

在报告中

-   按j可以跳转到函数定义处.

<!--listend-->

-   按d可以显示函数的documentation

-   可以使用C-x C-w保存检测报告

-   使用=比较两个检测结果

-   执行M-x profiler-stop结束监控过程


### Trace {#trace}


#### (trace-function FUNCTION &amp;optional BUFFER CONTEXT) {#trace-function-function-and-optional-buffer-context}

可以追踪函数FUNCTION的执行过程. 当调用到FUNCTION函数时,会在trace buffer中输出FUNCTION的参数以及返回值.

参数BUFFER,指明了在哪个buffer中输出trace信息. 默认buffer名由变量\`trace-buffer'决定

**使用该函数追踪FUNCTION时,总会弹出\`trace-buffer' buffer.因此不要用该函数追踪哪些会切换buffer的函数,对于这种函数使用\`trace-function-background'代替**

使用\`untrace-function'或\`untrace-all'停止对FUNCTION的追踪.


#### (trace-function-background FUNCTION &amp;optional BUFFERCONTEXT) {#trace-function-background-function-and-optional-buffercontext}

类似\`trace-function',但追踪函数FUNCTION时,不会弹出buffer也不会改变window configuration


#### (untrace-function FUNCTION) {#untrace-function-function}

取消对FUNCTION的追踪


#### (untrace-all) {#untrace-all}

取消对所有的函数的追踪


## Command Loop {#command-loop}

当进入Emacs后,Emacs会循环读取key sequences,读取对应的命令,并显示结果. 这个过程称为Command Loop.


### Command Loop概述 {#command-loop概述}

1.  command loop第一步是调用函数\`read-key-sequence'来读取key sequence,并转换为一个command或keyboard macro.

2.  在执行command前,调用\`undo-boundary'来保存undo信息

3.  执行\`pre-command-hook'中的函数

    在执行command前,会触发\`pre-command-hook',这时变量\`this-command'的值为将要运行的command,而\`last-command'的值为上一次运行的command

    执行该hook参数时,不抛出quitting,也不抛出error,但会把抛出error的函数移除该hook

4.  Emacs通过调用\`command-execute'来读取传递給command的参数列表
    1.  若command为keyboard macro,则Emacs通过\`execute-kbd-macro'来执行command

    2.  若command为命令(interactively callable function),则通过\`call-interactively'来读取参数并执行command

5.  执行\`post-command-hook'中的函数

    在执行command后,会触发\`post-command-hook',这时变量\`this-command'的值为刚运行的command,而\`last-command'的值为再上一次运行的command

    执行该hook参数时,不抛出quitting,也不抛出error,但会把抛出error的函数移除该hook


### Command {#command}

所谓Command,不仅仅指的带有top-level \`interactive' form的函数. 还可以是声明为interactive的autoload object,某些primitive functions,以及strings和vectors(被当成是keyboard macro来看待),

-   (commandp object &amp;optional for-call-interactively)

判断object是否为command

若参数for-call-interactively为非nil,则只有在object能被\`call-interactively'调用时才返回t,这时keyboard macro返回nil

-   (command-execute command &amp;optional record-flag keys special)

执行command

若command为string或vector,则被认为是keyboard macro,会使用\`execute-kbd-macro'执行command. 否则连同参数keys和record-flag一起传递給\`call-interactively'一起调用

参数special,若为非nil,则表示忽略preifix argument但是不clear它. 常用来执行特殊event

-   命令(execute-extended-command prefix-argument)

该命令使用\`completing-read'从minibuffer中读取命令,并用\`command-execute'执行该命令,并返回执行结果

若读取的命令需要prefix argument,则会将参数prefix-argument传递給它.

当用interactively的方式运行该命令时,参数prefix-argument的值就是传递給\`execute-extended-command'的prefix-argument

默认情况下,M-x执行的就是该命令


### Disabling Commands {#disabling-commands}

"Disabling a Command"給一个command加上标记,运行command时实际上会调用\`disabled-command-function'所指向的函数,默认的行为是会要求用户确认是否执行该函数.

被disable的command,其symbol的\`disabled'属性被设置为非nil,若\`disabled'属性为一个string,则警告信息中会包含该string

-   (enable-command command)

允许command(symbol格式)不经确认就执行该command

**该命令会同时修改init文件,使之在以后的session中也生效**

-   (disable-command command)

使得command在之前,需要经过确认.

**该命令会同时修改init文件,使之在以后的session中也生效**

-   变量disabled-command-function

该变量的值为一个function. 当用户交互式的调用一个disabled command时,实际上调用的是该函数.

在该function中可以使用\`this-command-keys'来探测用户实际的输入的key sequence,并以此来找到需要执行的函数.

若值为nil,则disabled function跟普通function一样执行.


### Command History {#command-history}

command loop会记录执行过的complex command的历史记录.

所谓complex command指的是其 **interactive argument** 会从minibuffer中读取参数值的command.(在执行command的过程中明确使用到minibuffer的,不算)

-   command-history

记录了最近执行过的complex command的list

-   命令(repeat-complex-command N)

编辑并重新执行最后执行过的/倒数第N个complex command

-   命令(list-command-history)

列出在minibuffer中输入过的command的历史


### 如何分辨Command是否通过Interactive方式调用 {#如何分辨command是否通过interactive方式调用}

一个比较好的方法是在interactive form中设置某个标识为非nil. 例如

```emacs-lisp
(defun foo (&optional print-message)
  (interactive "p")
  (when print-message
    (message "foo")))
```

另一种方法是使用函数\`called-interactively-p'

-   (called-interactively-p kind)

若正在执行的function是通过\`call-interactively'调用的,则返回t

参数kind只能是'interactive或'any

若参数kind为'interactive,则只有当function是直接由用户调用的情况下,才返回t(例如if the user typed a key sequence bound to the calling function, but <span class="underline">not</span> if the user ran a keyboard macro that called the function)

```emacs-lisp
(defun foo ()
  (interactive)
  (when (called-interactively-p 'any)
    (message "Interactive!")
    'foo-called-interactively))

;; Type `M-x foo'.
-| Interactive!

(foo)
=> ni
```

若参数kind为'any,则包括keyboard macro在内,也返回t

```emacs-lisp
(defun bar ()
  (interactive)
  (message "%s" (list (foo) (called-interactively-p 'any))))

;; Type `M-x bar'.
-| (nil t)
```


### generic command {#generic-command}

第一次执行用M-x COMMAND&lt;RET&gt;来执行generic command,Emacs会提示你选择哪一种具体实现,并保存选择信息,下一次就不会询问了. 若执行时带了prefix argument,则又会重复该过程.

COMMAND的不同实现存储在变量\`COMMAND-alternatives'中,只有在该变量存在时,才能使用宏\`define-alternatives'定义COMMAND的另一个实现方式.

If CUSTOMIZATIONS is non-\`nil', it should consist of alternating \`defcustom' keywords (typically \`:group' and \`:version') and values to add to the declaration of \`COMMAND-alternatives'.

-   宏(define-alternatvies comand &amp;rest customizations)

定义新命令COMMAND,参数COMMAND为一个symbol


### 获取Command Loop中的信息 {#获取command-loop中的信息}

-   last-command

上次运行的command名称

当一个command从command loop中退出时,会从\`this-command'中复制该值

-   real-last-command

类似\`last-command',但不会被Lisp程序所修改

-   last-repeatable-command

类似\`last-command'但不保存input event. 这里面保存的是\`repeat'命令会重复执行的command

-   this-command

正在执行的命令名称,但有些命令会在执行时人工修改该值

-   this-original-command

类似this-command,但当command remapping发生时,\`this-command'存储的是实际运行的command名称,而\`this-original-command'存储的是触发的原始command的名称

-   (this-command-keys)

返回调用command的key sequence,返回类型为string或vector

但若command调用了\`read-key-sequence',则返回的是最后读取到的key sequence

```emacs-lisp
(this-command-keys)
;; Now use `C-u C-x C-e' to evaluate that.
=> "^U^X^E"
```

-   (this-command-keys-vector)

类似\`this-command-keys',只是返回的值总是vector

-   (clear-this-command-keys &amp;optional keep-record)

This function empties out the table of events for \`this-command-keys' to return.

Unless KEEP-RECORD is non-\`nil', it also empties the records that the function \`recent-keys' will subsequently return.

一般常用于读取一个密码后

-   last-nomenu-event

该变量斥候最后发生的input event(不包括mouse menu event)

使用该变量的一个场景是用来告诉\`x-popup-menu'在哪里弹出一个menu. 在\`y-or-n-p'内也用到了该变量值

-   last-command-event

This variable is set to the last input event that was read by the command loop as part of a command

在\`self-insert-command'中用到该变量来决定插入哪个character

```emacs-lisp
last-command-event
;; Now use `C-u C-x C-e' to evaluate that.
=> 5

;; The value is 5 because that is the ASCII code for `C-e'.
```

-   last-event-frame

该变量记录了最后的input event是在哪个frame中调用的.

Usually this is the frame that was selected when the event was generated, but if that frame has redirected input focus to another frame, the value is the frame to which the event was redirected.


### Command的prefix argument {#command的prefix-argument}

prefix argument有两种表现形式:"raw"和"numeric". coomand loop内部,和lisp变量使用raw表现形式

Here are the possible values of a raw prefix argument:

-   \`nil', meaning there is no prefix argument.
    Its numeric value is 1, but numerous commands make a distinction between \`nil' and the integer 1.

-   An integer, which stands for itself.

-   A list of one element, which is an integer.
    This form of prefix argument results from one or a succession of \`C-u's with no digits.
    The numeric value is the integer in the list, but some commands make a distinction between such a list and an integer alone.

-   The symbol \`-'.
    This indicates that \`M--' or \`C-u -' was typed, without following digits.
    The equivalent numeric value is -1, but some commands make a distinction between the integer -1 and the symbol \`-'.

-   (prefix-numeric-value arg)

这里的arg为raw格式的prefix argument. 该函数将arg转换为对应的numeric格式.

若arg为nil,则返回1

若arg为-,则返回-1

若arg为数字,则返回该数字

若arg为list,则返回(car arg)

-   current-prefix-arg

该变量存储的是当前命令的raw prefix argument.

command可以直接检查该变量的值,但大多数是使用\`(interactive "P")'来获取该参数的值

-   prefix-arg

下一个命令使用的raw prefix argument.

类似\`universal-argument'这样的函数,通过设置该变量来为接下来要执行的命令设置prefix argument

-   last-prefix-arg

上一个command的raw prefix argument

-   (universal-argument)

该命令读取用户的输入,并将用户的输入值设置为变量\`prefix-arg'的值,这样就为下一个待执行的command设置了prefix argument

小心使用该函数

-   (digit-argument arg)

This command adds to the prefix argument for the following command.
  The argument ARG is the raw prefix argument as it was before this command;
  it is used to compute the updated prefix argument.

-   (negative-argument arg)

This command adds to the numeric argument for the next command.
  The argument ARG is the raw prefix argument as it was before this command;
  its value is negated to form the new prefix argument


### Quitting {#quitting}

当Lisp函数正在运行时,可以按下\`C-g',让Emacs退出当前的工作.

然而当command loop在等待keyboard input时,按下C-g并不会引发quitting,Emacs只是把它当成一个普通的input character.

when \`C-g' follows a prefix key, they combine to form an undefined key. The effect is to cancel the prefix key as well as any prefix argument.

而当在minibuffer中输入时,\`C-g'的意义又不一样,它中断并退出minibuffer

\`C-g'通过设置变量\`quit-flag'为t来表示要quit,Emacs检查该变量的值并产生quitting

若想在执行Lisp函数时阻止quitting的产生,只需要将变量\`inhibit-quit'绑定为非nil值即可.这时,即使\`quit-falg'设置为t,依然不会产生quitting

-   quit-flag

该值为非nil,除非\`inhibit-quit'被设置为非nil,否则Emacs立即退出当前执行的任务.

-   inhibit-quit

当\`quit-flag'设置为非nil时,该变量决定Emacs是否执行退出

-   宏(with-local-quit body...)

该宏执行body,执行过程中,即使\`inhibit-quit'设置为非nil,也运行产生quitting.

若body运行过程中,被quitting中断,则返回nil,否则返回最后语句的执行结果.

若进入\`with-local-quit'时,\`inhibit-quit'为nil,则执行body时若产生quitting,则Emacs设置\`quit-flag'并产生一个普通quit.

若进入\`with-local-quit'是,\`inhibit-quit'为非nil,这时普通的quitting被推迟. 非nil的\`quit-flag'会触发一种特殊的quit--local quit. 它会终止body的执行并退出\`with-local-quit'. **退出\`with-local-quit'后的\`quit-flag'依然为非nil**.

-   (keyboard-quit)

该函数设置产生quit的条件,默认绑定到C-g


### Keyboard Macro {#keyboard-macro}

一个"keyboard macro"指的是一系列的input event,这一系列的input event可以认为是一个command.(A "keyboard macro" is a canned sequence of input events that can be considered a command and made the definition of a key)

keyboard macro的lisp表现形式为一个string或由event组成的vector

-   (execute-kbd-macro kbdmacro &amp;optional count loopfunc)

把kbdmacro当作一系列的event来执行.

若kbdmacro为string或vector,则就好像用户直接输入一样.  The sequence is <span class="underline">not</span> expected to be a single key sequence

若kbdmacro为symbol,则执行该symbol的函数定义,若该symbol的函数定义还是一个symbol,则不断的递归下去. 最终,其函数定义应该是一个string或vector,否则会抛出error

参数count表示重复执行多少次kdbmacro,若count为nil,则执行一次,若count为0,表示执行无穷多次

若loopfunc为非nil,则配次循环都会不带参数调用该函数,或函数返回nil,则停止执行该macro

-   executing-kbd-macro

当前正在执行的kbd-macro,nil表示没有在执行kbd-macro

-   defining-kbd-macro

只有当正在定义的kbd-macro的情况下,该值才为非nil.

该值为'append表示正在为现有的macro添加定义( The value is \`append' while appending to the definition of an existing macro)

命令\`start-kbd-macro',\`kmacro-start-macro'和\`end-kbd-macro'会设置该值--尽量不要自己去设置该值

The variable is always local to the current terminal and cannot be buffer-local

-   last-kbd-macro

最近所定义的kbd-macro

-   kbd-macro-termination-hook

当keyboard macro执行完成后触发该hook(不管是正常结束还是异常结束都触发)


## InputEvent {#inputevent}

Emacs Command Loop读取一系列的"input event"来表示键盘/鼠标的动作,或发送给Emacs的系统事件.

表示键盘动作的event,用char或symbol的格式表示. 其他类型的event统一用list来表示.

-   (eventp object)

若object为input event或event type,则返回非nil

需要注意的是,任何symbol既可以用来作为event,也可以作为event type,因此eventp不能区分一个symbol是被用于作为event,还是event type.


### Keyboard Event {#keyboard-event}

键盘输入可以分为两类:普通的按键和功能键.


#### 普通按键事件 {#普通按键事件}

普通按键产生的event,在lisp中用character来表示.

The event type of a character event is the character itself (an integer)

一个input character event由"basic code"(取值范围从0到524287)加上"modifier bits"组成

modifier bits包括:

| 说明    | 值      | 说明                                                                        |
|-------|--------|---------------------------------------------------------------------------|
| meta    | 2\*\*27 |                                                                             |
| control | 2\*\*26 | C-a这样的已经定义在ASCII中的控制字符,由于已经有了特定的basic code了,因此Emacs不需要使用special bit来指示它 |
| shift   | 2\*\*25 | 对于字符,数字和标点来说,basic code中已经定义相关的shift key按下后的对应键值,对于这些按键,Emacs不使用special bit |
| hyper   | 2\*\*24 |                                                                             |
| super   | 2\*\*23 |                                                                             |
| alt     | 2\*\*22 |                                                                             |

最好不要直接在程序中使用specific bit(因为这些bit的位置可能会改变)

应该使用\`event-modifiers'函数来测试specific bit是否被设置

| 简写形式 | 说明    |
|------|-------|
| A-   | alt     |
| C-   | control |
| H    | hyper   |
| M-   | meta    |
| S-   | shift   |
| s-   | super   |


#### 功能键事件 {#功能键事件}

功能键event在elisp中用symbol来表示. 一般来说,symbol的名称就是功能键的label(全小些形式). 例如&lt;F1&gt;产生的input event表示为符号'f1

The event type of a function key event is the event symbol itself

还有一些功能键event的表示与功能键的label不一致的情况:

-   \`backspace',\`tab',\`newline',\`return',\`delete'

-   \`left',\`up',\`right',\`down'

    光标箭头按键

-   \`kp-add',\`kp-decimal',\`kp-divide',...

    右边小键盘的加减乘除

-   \`kp-0',\`kp-1',...

    右边小键盘的数字键

-   \`kp-f1',\`kp-f2'...

    右边小键盘的fn

-   \`kp-hoome',\`kp-left',\`kp-up',\`kp-right',\`kp-down'

    右边小键盘的对应功能键

-   \`kp-prior',\`kp-next',\`kp-end',\`kp-begin',\`kp-insert',\`kp-delete'

    右边小键盘的对应功能键


#### 以字符串表示keyboard event {#以字符串表示keyboard-event}

现在一般不建议使用string来表示keyboard event,最好使用vector代替.

可以使用函数\`listify-key-sequence'来讲string格式的keyboard event转换为list,方便解析出其中的内容.

需要注意:当使用字符串来表示keyboard event时,只有Meta modifier才能以'\M-'的格式表示在string中,其他modifier都无法表示.

下面是一些转换规则:

-   若keyboard character的值范围为0到127,则可以直接写进string
-   若上面的keyboard character同时按下了meta键(即2\*\*27 到 2\*\*27+127),则需要转换为(2\*\*7到2\*\*7+127)
-   若是大于256的非ASCII字符,可以包含进multibyte string中
-   其他字符(128-255范围的字符)无法用string表示.


### Mouse Events {#mouse-events}

Emacs支持4种鼠标事件:click event,drag event,button-down event和motion event.

所有的鼠标事件都用list来表示,且(car list)为event type(提供了按下的是哪个鼠标按键,同时有哪个modifier key被按下了,这些信息). (cdr list)则提供了位置与时间的信息

需要注意的是,鼠标事件是由鼠标所在buffer的keymap来处理的,而不是光标所在的buffer的keymap来处理.


#### 点击事件 {#点击事件}

点击事件的结果为'(EVENT-TYPE PSITIION CLICK_COUNT)

其中:

-   EVENT-TYPE

    该symbol标识鼠标的哪个按键被点击,可选值为'mouse-1,'mouse-2,'mouse-3

    当然,也可以通过添加前缀\`A-',\`C-',\`H-',\`M-',\`S-'和\`s-'来标识点击时同时按下了哪个modifier key

    该symbol同时也作来标识event的event type

-   POSTION

    POSTION具体的格式,根据点击的位置而不同.

    当点击在text area,mode-line,header-line或area的边界时,POSTION的格式为:
    ```text
    (WINDOW POS-OR-AREA (X . Y) TIMESTAMP
      OBJECT TEXT-POS (COL . ROW)
      IMAGE (DX . DY) (WIDTH . HEIGHT))
    ```
    其中:

    -   WINDOW

        表示点击的那个window

    -   POS-OR-AREA

    若点击的位置在text area内,则表示点击处的buffer postion

    否则,它的值为表示window area的symbol:'mode-line,'header-line,'vertical-line,'left-margin,'right-margin,'left-fringe,'right-fringe

    -   X,Y

    点击的位置相对text area左上角的坐标

    -   TIMESTAMP

    事件发生的时间

    -   OBJECT

    若点击的位置没有string类型的text property,则为nil.

    否则为'(点击位置的带属性string . string的位置)

    -   TEXT-POS

    对于点击在marginal area或fringe上时,该值为对应行第一个字符的buffer postion.

    其他情况下,则就是当前buffer position

    -   COL,ROW

    点击位置相对text area左上角的行列数

    -   IMAGE

    若点击的位置是一个IMAGE,则该值为\`find-image'返回的image object

    否则为nil

    -   DX,DY

    若OBJECT为nil,为点击的位置相对点击到的字符左上角的坐标

    否则,为点击的位置相对OBJECT左上角的坐标

    -   WIDTH,HEIGTH

    OBJECT的宽度与高度,若OBJECT为nil,则为点击处文本的宽度与高度

    若点击的地方为scroll bar,则POSTION的格式为
    ```text
    (WINDOW AREA (PORTION . WHOLE) TIMESTAMP PART
    ```
    其中:

    -   WINDOW

    点击到的scroll bar所属的window

    -   AREA

    为'vertical-scroll-bar

    -   PORTION

    从scrollbar的最顶端到点击位置的长度,以像素为单位

    On some toolkits, including GTK+, Emacs cannot extract this data, so the value is always \`0'.

    -   WHOLE

    scrollbar的整个长度,以像素为单位

    On some toolkits, including GTK+, Emacs cannot extract this data, so the value is always \`0'.

    -   TIMESTAMP

    事件发生的时间

    -   PART

    点击在了scrollbar的哪个位置,可以为'handle,'above-handle,'below-handle,'up,'down

-   CLICK-COUNT

    快速点击的次数


#### 拖拽事件 {#拖拽事件}

拖拽事件的格式为:

```emacs-lisp
(EVENT-TYPE
 (WINDOW1 START-POSITION)
 (WINDOW2 END-POSITION))
```

EVENT-TYPE以\`drag-'为前缀,例如\`drag-mouse-1'表示按下mouse button 1来拖动

根据是否按下了Modifier Key,还可以在\`drag-'前添加\`C-',\`M-'...等前缀.

WINDOW和POSTION的值,则跟点击事件定义一样

若\`read-key-sequence'接收到一个拖拽事件,但发现并没有相应的key binding绑定到这个事件上,而相应的点击事件有binding. 则会自动将拖拽事件转换为点击事件.


#### Button-Down事件 {#button-down事件}

Button-Down事件的格式与Click事件格式一样,都是

```emacs-lisp
(EVENT-TYPE PSITIION CLICK_COUNT)
```

不同点在于EVENT-TYPE是以\`down-'作为前缀的,根据是否按下Modifier key,在\`down-'前还有\`C-'和\`M-'前缀

\`read-key-sequence'忽略任何没有command binding的buton-down event.


#### Repeat Event {#repeat-event}

若快速点击同一个mouse botton而不移动mouse位置的话,则Emacs产生"repeat event"

最常见的"repeat event"就是"double-click" event(双击事件)

双击事件的EVENT-TYPE以\`double-'为前缀,根据modifier key是否按下,可能在\`double-'前添加\`M-',\`S-'等前缀

当用户执行双击时,其实产生了两个事件,第一个是普通的单击事件,第二个为双击事件. 因此在处理双击事件前单击事件的相关命令已经执行了.

同理,若点击一次鼠标之后立即按下鼠标并拖动鼠标,则产生了\`double-drag' event.

而,在\`double-click' event和\`double-drag' event产生前,Emacs还会产生\`double-down' event.

**总结起来,一次双击动作会产生4个事件** :down event-&gt;click event-&gt;double-down event-&gt;double-click event.

**一次double-drag动作也会产生4个事件** :down event-&gt;click event-&gt;double-down event-&gt;double-drag event.

同理,还有\`triple-down',\`triple-click'和\`triple-drag'

**Emacs最多只产生triple-click event**.

若想知道精确的点击几次button,使用函数\`event-click-count'

-   (event-click-count event)

    获取event中鼠标点击的次数

-   配置项double-click-fuzz

    定义了两次双击之间,位置不能超过的像素数

-   配置项double-click-time

    定义了两次双击之间,不超过的时间,以毫秒为单位

    nil表示直接不探测multi-click

    t表示无限时间.


#### Motion Events {#motion-events}

在运行\`trace-mouse'的body时,不按mouse botton的情况下移动mouse,会产生"mouse motion" event,它的格式为:

```emacs-lisp
'(mouse-movement POSITION)
```

这里的POSITION跟点击事件中的POSITION一样

在trace-mouse之外的情况下,emacs不产生mouse motion event


#### Focus Events {#focus-events}

当切换frame时,会产生Focus Event. 它的格式为:

```emacs-lisp
'(switch-frame NEW-FRAME)
```

这里NEW-FRAME为新切换到的frame

由于在一系列按键序列中间产生一个focus event会扰乱原按键序列的执行,因此Emacs不会在key sequence中间产生focus event.

若用户在key sequence中间更改了focus,则Emacs会重新排列event,将focus event放在multi-event key sequence的最前面或最后面.


#### 其他System Event {#其他system-event}

若用户在key sequence中间发生了下面的那些system event,则Emacs会重新排列event,将这些system event放在multi-event key sequence的最前面或最后面.

-   '(delete-frame (FRAME))

    表示用户对FRAME发送了关闭命令

-   '(iconify-frame (FRAME))

    iconify某个FRAME,默认的定义为\`ignore'

-   '(make-frame-visible (FRAME))

    表示用户deiconified FRAME,默认为\`ignore'

-   '(wheel-up POSITION) / '(wheel-down POSITION)

    滚动鼠标wheel时产生的event.

    POSITION的结构跟Click Event的POSITION一样,标识了event发生时的鼠标位置

    这种event只在某些操作系统上会产生,有些操作系统上产生的是\`mouse-4'和\`mouse-5' event.

    因此,为了可移植性,建议使用定义在\`mwheel.el'中的变量\`mouse-wheel-up-event'和\`mouse-wheel-down-event'来代替

-   '(drag-n-drop POSITION FILES)

    当从Emacs外选择了一些文件,并拖到Emacs frame中时产生\`drag-n-drop' event

    这里POSITION的格式跟click event中的POSITION一样.

    FILES为文件名称的列表.

-   '(help-echo FRAME HELP WINDOW OBJECT POS)

    但光标移动到buffer中带有\`help-echo' text property的文本时,产生该event

-   'sigusr1 / 'sigusr2

    当Emacs收到信号\`SIGUSR1'和\`SIGUSR2'时触发该event. 一般用于调试时使用

    要捕获user signal,绑定相应的event到\`special-event-map'中的命令. 这时会不带参数地执行该命令,而signal event可以通过变量\`last-input-event'来获得. 例如
    ```emacs-lisp
    (defun sigusr-handler ()
      (interactive)
      (message "Caught signal %S" last-input-event))

    (define-key special-event-map [sigusr1] 'sigusr-handler)
    ```

-   '(language-change FRAME CODEPAGE LANGUAGE-ID)

    在MS-Windows下才更改input language会产生该event.

    这里FRAME表示改变input language时的当前frame.

    CODEPAGE为更改为的新codepage number

    LANGUAGE-ID为新input language的数字id

    例如:
    ```emacs-lisp
    ;; Get the abbreviated language name, such as "ENU" for English
    (w32-get-locale-info language-id)
    ;; Get the full English name of the language,
    ;; such as "English (United States)"
    (w32-get-locale-info language-id 4097)
    ;; Get the full localized name of the language
    (w32-get-locale-info language-id t)
    ```


### 特殊Events {#特殊events}

特殊Event在非常底层的地方被处理--as soon as they are read.

\`read-event'函数内部就会消化掉这些event,而不会返回这种event. 事实上,\`read-event'会一直等待并返回地一个非特殊event

特殊event不会被显示出来,不会被纳入key sequence中,不会存入\`last-command-event'和\`(this-command-keys)'.
特殊event They do not discard a numeric argument, they cannot be unread with \`unread-command-events', they may not appear in a keyboard macro, and they are not recorded in a keyboard macro while you are defining one.

然而在读取到特殊event时,会记录在\`last-input-event'中,and this is the way for the event's definition to find the actual event.

常见的特殊event type有\`iconify-frame',\`make-frame-visible',\`delete-frame',\`drag-n-drop',\`language-change'以及用户信号\`sigusr1',\`sigusr2'...

定义如何处理特殊event的keymap为变量\`special-event-map'


### 区分Events {#区分events}

每个event都有一个"event type",用于区分event.

对于keyboard event,event type就是event value

对于list格式的event,event type为(car list)

相同的event type运行相同的命令. 键序列与event type绑定

-   (event-modifiers event)

返回一个list,包含了该event中所有的modifiers,modifier的类型为symbol,
  它的值可能是'shift,'control,'meta,'alt,'hyper和'super
对于mouse event来说,还可能包括'click,'drag,'down,'double,'triple

参数event可以是一个event对象,也可以是个event type.

If EVENT is a symbol that has never been used in an event that has been read as input in the current Emacs session, then \`event-modifiers' can return \`nil', even when EVENT actually has modifiers.

```emacs-lisp
(event-modifiers ?a)                    ; => nil
(event-modifiers ?A)                    ; => (shift)
(event-modifiers ?\C-a)                 ; => (control)
(event-modifiers ?\C-%)                 ; => (control)
(event-modifiers ?\C-\S-a)              ; => (control shift)
(event-modifiers 'f5)                   ; => nil
(event-modifiers 's-f5)                 ; => (super)
(event-modifiers 'M-S-f5)               ; => (meta shift)
(event-modifiers 'mouse-1)              ; => (click)
(event-modifiers 'down-mouse-1)         ; => (down)
```

-   (event-basic-type event)

返回去掉modifier标志之后的event描述. 例如

```emacs-lisp
(event-basic-type ?a)                   ; => 97
(event-basic-type ?A)                   ; => 97
(event-basic-type ?\C-a)                ; => 97
(event-basic-type ?\C-\S-a)             ; => 97
(event-basic-type 'f5)                  ; => f5
(event-basic-type 's-f5)                ; => f5
(event-basic-type 'M-S-f5)              ; => f5
(event-basic-type 'down-mouse-1)        ; => mouse-1
```

-   (mouse-movement-p object)

    object是否为mouse movent event

-   (event-convert-list list)

    This function converts a list of modifier names and a basic event type to an event type which specifies all of them.
      The basic event type must be the last element of the list.
    例如:
    ```emacs-lisp
    (event-convert-list '(control ?a))      ; => 1,C-a
    (event-convert-list '(control meta ?a)) ; => -134217727
    (event-convert-list '(control super f1)) ; => C-s-f1
    ```


### 获取Mouse Events中的信息 {#获取mouse-events中的信息}

要想获得mouse event中的position list,可以使用以下两个函数

-   (event-start event)

若为drag event,则返回start-postion

若为click或button-down event,则返回唯一的那个postion

-   (event-end event)

若为drag event,则返回end-postion

若为click或button-down event,则返回唯一的那个postion

-   (posnp object)

判断object是否为mouse position

下面的函数,一mouse postion list为参数,返回相应部分的值

-   (posn-window postion)

返回postion list中的window

-   (posn-area position)

返回position中的window area标志

若event发生在text-area则返回nil,否则返回表示area的symbol

-   (posn-point position)

返回position中的buffer位置信息.

当event发生在text-area,marginal area或fringe上时,返回一个表示buffer位置的整数

其他情况下,返回值意义不明确

-   (posn-x-y position)

以'(X . Y)的形式返回相对(posn-window postion)的坐标,单位为像素.

下面的例子,演示了如何将相对window的坐标转换为相对frame的坐标

```emacs-lisp
(defun frame-relative-coordinates (position)
  "Return frame-relative coordinates from POSITION.
          POSITION is assumed to lie in a window text area."
  (let* ((x-y (posn-x-y position))
         (window (posn-window position))
         (edges (window-inside-pixel-edges window)))
    (cons (+ (car x-y) (car edges))
          (+ (cdr x-y) (cadr edges)))))
```

-   (posn-col-row postion)

以'(COL . ROW)的格式返回buffer postion在text area中的列与行

它是根据(postion-x-y postion)的信息与frame的默认字符的宽度和默认行的高度,计算出来的.

需要注意的是,ROW是从text area的最顶端开始计算的,也就是说,如果(position-window positon)拥有header line,则它不会计算如ROW中

-   (posn-actual-col-row postion)

以'(COL . ROW)的形式返回真正的相对(posn-window postion)的列数与行数

-   (posn-string positiion)

返回position中的string object,可能为nil或'(STRING . STRING-POS)

-   (posn-image position)

获得position中的image object,可能为nil或'(image...)

-   (posn-object position)

返回position中的string object或image object,可能为nil或'(STRING . STRING-POS)或'(image...)

-   (posn-object-x-y position)

获取相对POSITION list中的'(DX . DY). 即位置相对(posn-object position)的坐标. 单位为像素

若position为buffer position,则返回相对该处字符左上角的坐标.

-   (posn-object-width-height position)

以'(WIDTH. HEIGHT)格式,返回(posn-object position)的宽度和高度,单位为像素

若position为buffer position,则返回位置处字符的宽度和高度

-   (posn-timestamp position)

返回position中的timestamp信息,表示事件发生的时间戳,以毫秒为单位

以下函数根据buffer position或screen position,计算出position list

-   (posn-at-point &amp;optional pos window)

该函数返回position list用于表示参数pos在参数window中的位置. 若pos在window中不可见,则返回nil

参数pos默认为参数window中光标的位置

参数window默认为选中的window

-   (posn-at-x-y x y &amp;optional frame-or-window whole)

该函数返回position list用于表示(x . y)在参数frame-or-window中的相对坐标,

参数x,y是相对frame-or-window的以像素为单位的位置.

参数frame-or-window,默认为当前window

若参数为nil,则坐标是相当与window text area来计算的. 否则计算包括整个window area(text-rea+scroll bar+margin+fringe)


### 获取scroll bar event中的信息 {#获取scroll-bar-event中的信息}

-   (scroll-bar-event-ratio event)

以格式'(PORTION .WHOLE)返回event在scroll-bar中的位置.

-   (scroll-bar-scale ratio total)

该函数事实上将参数ratio与total相乘,并将结果约为整数.

这里参数ration不是数字,而是格式为'(NUM . DENOM)的cons ceil. 一般该值由函数scroll-bar-event-ration返回.

该函数用于将scroll bar position转换为buffer postion是很方便:

```emacs-lisp
(+ (point-min)
   (scroll-bar-scale
    (posn-x-y (event-start event))
    (- (point-max) (point-min))))

```


### 捕获Input Event {#捕获input-event}

-   (read-key-sequence prompt &amp;optional continue-echo dont-downcase-last switch-frame-ok command-loop)

该函数读取key sequence并以string或vector的形式返回.

该函数会一直读取key sequence直到获取到一个完整的key sequence为止(即在当前keymap下能定位到某个command)

需要注意的是: **以mouse event开头的key sequence,是在mouse所在的window中keymap中查找对应command的**

参数prompt为提示信息,nil表示没有提示

参数continue-echo表示当key sequence不完整时,是否显示已经输入的key sequence

默认情况下,任何upper case event在找不到对应command时,会转换为lower case event再去查找一遍(这是会设置变量\`this-command-keys-shift-translated'为t),参数dont-downcase-last禁止这种转换

参数swith-frame-ok表示在输入key sequence的过程中,是否能切换frame

参数command-loop若为非nil,则表示可以一次输入多个key sequence(一个key sequence与command想对应). nil表示只读取表示一个key sequence

\`read-key-sequence'会压抑住quitting,也就是说,在输入\`C-g'时,就好像其他普通的字符一样,不会去设置quit-flag

**当使用\`read-key-sequence'读取mouse event时,若mouse event发生在window的非text area中,则会添加prefix-key来表示该area**:'header-line,'horizontal-scroll-bar,'menu-bar,'mode-line,'vertical-line,'vertical-scroll-bar

```emacs-lisp
(read-key-sequence "Click on the mode line: ")
          => [mode-line
              (mouse-1
               (#<window 6 on NEWS> mode-line
                (40 . 63) 5959987))]
```

-   (read-key-sequence-vector prompt &amp;optional continue-echo dont-downcase-last switch-frame-ok command-loop)

与\`read-key-sequence'类似,只是肯定以vector类型返回

-   num-input-keys

当前Emacs session目前为止处理过的key sequence的数量.

-   (read-event &amp;optional prompt inherit-input-method seconds)

该函数只读取一个event,而不像\`read-key-sequence'一样可能读取多个event.

The returned event may come directly from the user, or from a keyboard macro.
  It is not decoded by the keyboard's input coding system

参数prompt为提升信息

若参数inherit-input-method为非nil,则支持用当前输入法输入non-ASCII字符. 否则会禁用输入法

参数seconds表示等待输入的超时秒数,若超时还未有event发生,则返回nil.
  若参数seconds为nil,则Emacs在等待用户输入时被认为处于idle状态,若设置了值,则等待期间不会认为处于idle状态.

If \`read-event' gets an event that is defined as a help character, then in some cases \`read-event' processes the event directly without returning.

Certain other events, called "special events", are also processed directly within \`read-event'

-   (read-char &amp;optional prompt inherit-input-method seconds)

读取并返回输入的character. 若用户产生的event不是character(例如点击事件或功能键事件),则\`read-char'会抛出一个错误

-   (read-char-exclusive &amp;optional prompt inherit-input-method seconds)

类似\`read-char',只是当读到的event不是character时,会忽略这个event,接着读取下一个event,而不是抛出错误

-   num-nonmacro-input-events

该变量存储了到目前为止从terminal读取到的input events总数(那些由keyboard macro)产生的不算.

-   (read-key &amp;optional prompt)

该函数读取single key. 它处于\`read-key-sequence'和\`read-event'之间.

跟\`read-key-sequence'不同之处在于,它读取single key而不是完整的key sequence

跟\`read-event'不同之处在于,它会根据\`input-decode-map',\`local-function-key-map'和\`key-translation-map'解码并转换用户的输入.

-   (read-char-choice prompt chars &amp;optional inhibit-quit)

该函数使用\`read-key'读取并返回一个character. 它会忽略任何不是参数chars中的member的character.

chars为一个由characters组成的list. 表示可接受的character范围.

-   (read-quoted-char &amp;optional prompt)

类似\`read-char',只是当读取的地一个character是一个8进制数时(0-7),它会读取接下来输入的所有8进制数,并返回由这些8进制numeric character code所表示的character.

```emacs-lisp
(read-quoted-char "What character")

  ---------- Echo Area ----------
  What character 1 7 7-
  ---------- Echo Area ----------

  => 127
```


### Modifying and Translating Input Events {#modifying-and-translating-input-events}

在使用\`read-event'时,Emacs会根据\`extra-keyboard-modifiers'的值对读取到到的event做改变(modify),然后根据\`keyboard-translate-table'的值做转换(translate)

-   extra-keyboard-modifiers

该变量允许Lisp程序模拟按下键盘上的modifier key.

该变量的至必须为一个设置了modifier bit位的character(例如'?\C-\M-a'). 真正其作用的是其中的modifier bit(?\C-\M-).

若变量的值为\`?\C-@',不会设置Ctrl被按下,相反, **这个值表示取消所有的modification**

另外,需要注意的是, **该变量只会修改从keyboard读取到的event,而对mouse event或其他类型的event无效**

-   keyboard-translate-table

该变量为terminal-local variable. 它允许你将一个keyboard event重新映射成另一个keyboard event

一般情况下,它的值为一个char-table或nil.

Note that this translation is the first thing that happens to a character after it is read from the terminal.
  Record-keeping features such as \`recent-keys' and dribble files record the characters after translation.

-   (keyboard-translate from to)

该函数通过修改\`keyboard-translate-table'的值来达到将character code FROM转换为character code TO的目的.


### Event Input的其他特性 {#event-input的其他特性}

-   unread-command-events

该变量存储的值为一个由event组成的list,表示待读取的event.

该list中的event,以显示的顺序(即最前面的最先被使用)被读取,并且在使用后被删除

一般情况下,从该list中读取的event不会添加到当前命令的key sequence中(即不会被\`this-command-keys'),因为该event在第一次读取时已经添加过一次了.
但若list中的element格式为'(t . EVENT)则表示强制将该event放入当前command的key sequence中

-   (listify-key-sequence key)

该函数将key(string或vector)转换为由单独event组成的list,可以很容易的将这个list放入\`unread-command-events'中

-   (input-pending-p &amp;optional check-timers)

该函数检查是否有command input可以被读取了.

若参数check-timers为非nil,则若没有input可以被读取时,运行Emacs运行已经ready的timers

-   last-input-event

该变量存储最后的terminal input event. whether as part of a command or explicitly by a Lisp program.

在下面的例子中,这段Lisp程序读取字符'1'(ASCII码为49). 假设我们用C-x C-e来执行这段代码,则会发现\`last-input-event'的值为'1'(49),而\`last-command-event'只为?\C-e(5)

```emacs-lisp
(progn (print (read-char))
           (print last-command-event)
           last-input-event)
         -| 49
         -| 5
         => 49
```

-   宏(while-no-input body...)

当运行body的过程中没有输入时,正常执行body并返回body的值.

但若执行body的过程中有输入到来,则会中断body的执行(类似quit),并返回t

若执行body的过程中,被真正的quit所打断,则返回nil

若BODY中某部分绑定\`inhibit-quit'为非nil,则即使有输入到来,也不会中断该部分代码的执行,直到该部分代码被执行完毕.

-   (discard-input)

该函数丢弃terminal input buffer的内容并取消正在处理的keyboard macro,并返回nil

例如:

```emacs-lisp
(progn (sleep-for 2)                    ;在等待期间,用户可能输入了一些东西
       (discard-input))                 ;会丢弃用户在等待期间所输入的东西
       => nil

```


## 关于输入法 {#关于输入法}

读取event的函数会调用当前使用的输入法. 但\`read-event'读取一个print character(包括&lt;SPC&gt;)时,会以该character作为参数,调用\`input-method-function'所表示的函数

-   input-method-function
    \`read-event'读取一个print character(包括&lt;SPC&gt;)时,会以该character作为参数,调用\`input-method-function'所表示的函数

该input-method-function的返回值应该是一系列由event组成的list. 若返回nil表示没有输入,这样\`read-event'会等待下一个event产生.

若input-method-function中又调用了\`read-event'或\`read-key-sequence',则需要记住把\`input-method-function'的值绑定为nil,否则会发生无限递归.

The input method function is not called when reading the second and subsequent events of a key sequence.
  Thus, these characters are not subject to input method processing.
  The input method function should test the values of \`overriding-local-map' and \`overriding-terminal-local-map';
  if either of these variables is non-\`nil', the input method should put its argument into a list and return that list with no further processing.


## Keymaps {#keymaps}

input event与command的对应关系被记录在名为"keymaps"的结构体中. 每个event type对应一个command或另一个keymap.

当一个event type绑定到另一个keymap时,这个keymap被用于查找下一个input event的对应关系.

若某个event sequence找到的是一个keymap,则我们称该key sequence为"prefix key". 否则则为"complete key".

这个从event sequence一直找到某个command的过程,被成为"key lookup"

Emacs中有三类keymap:

-   global map

由所有buffer所共享

-   local keymap

由buffer指定的major mode所决定,会覆盖global map的key sequence

-   minor mode keymaps

由buffer所开启的minor mode所决定,会覆盖global map和local keymap的key sequence


### Key Sequences {#key-sequences}

"Key sequence",简称为"key",是指一个或多个input event做组成的序列集合. input event包括characters,function keys,mouse actions或system events

当用vector来表示key sequences时,每个vector的元素都是character,每个character表示一个input event. 例如\`[?\C-x ?l]'表示key sequence \`C-x l'

-   (kbd keyseq-text)

将string类型的keyseq-text转换为key sequence.

```emacs-lisp
(kbd "C-x") => "\C-x"
(kbd "C-x C-f") => "\C-x\C-f"
(kbd "C-x 4 C-f") => "\C-x4\C-f"
(kbd "X") => "X"
(kbd "RET") => "\^M"
(kbd "C-c SPC") => "\C-c "
(kbd "<f1> SPC") => [f1 32]
(kbd "C-M-<down>") => [C-M-down]
```


### Keymaps的内部结构 {#keymaps的内部结构}

keymap是一个list,它的car为'keymap. 剩下的元素可以是如下格式:

-   '(EVENT-TYPE . BINDING)

该EVENT-TYPE对应着BINDING

-   '(EVENT-TYPE ITEM-NAME . BINDING)

该event-type对应着binding,同时该binding也是一个名为ITEM-NAME的simple menu item

-   '(EVENT-TYPE ITEM-NAME HELP-STRING . BINDING)

该event-type对应着binding,同时该bingding也是一个名为ITEM-NAME的菜单项,而带有help string

-   '(EVENT-TYPE menu-item . DETAILS)

EVENT-TYPE对应着的绑定,同时也是一个extended menu item

-   '([t] . BINDING)

设置BINDING为"default key binding". 任何不能找到对应项的event-type都执行该binding

-   某char-table

-   某string

-   某parent-keymap

若元素为keymap,则表示该keymap所定义的绑定关系都嵌入到包含该keymap的keymap中来.

有一点需要注意:key \`M-a'在keymap中被分拆表示为\`&lt;ESC&gt; a'(只有在meta与另一个普通字符关联时,才被分拆),而\`M-&lt;end&gt;'直接存储为\`M-&lt;end&gt;'

-   (keymapp object)

判断object是否为keymap,若object为symbol,则判断该symbol对应的function definition是否为keymap

```emacs-lisp
(keymapp '(keymap))                     ; => t
(fset 'foo '(keymap))
(keymapp 'foo)                          ; => t
(keymapp (current-global-map))          ; => t
```


### 创建keymap {#创建keymap}

-   (make-sparse-keymap &amp;optional prompt)

创建一个空的keymap

若传递了参数prompt,则其称为keymap的overall prompt string. 你只能为menu keymap设置该值,因为任何被设了overall prompt string的keymap都被认为是menu

-   (make-keymap &amp;optional prompt)

类似(make-spare-keymap),但是它所创建的不是空keymap,而是包含了一个char-table,这个char-table包含了所有的不带modifier的characters

新产生的keymap绑定所有这些characters到nil

-   (copy-keymap keymap)

深拷贝keymap.  However, recursive copying does not take place when the definition of a character is a symbol whose function definition is a keymap; the same symbol appears in the new copy.

```emacs-lisp
(setq map (copy-keymap (current-local-map)))
    => (keymap
         ;; (This implements meta characters.)
         (27 keymap
             (83 . center-paragraph)
             (115 . center-line))
         (9 . tab-to-tab-stop))

    (eq map (current-local-map))
        => nil
    (equal map (current-local-map))
        => t

```


### keymap的继承 {#keymap的继承}

若keymap中的element为某keymap,则该被包含的keymap的内容会被内嵌到包含的keymap中,即实现了keymap的继承机制.

我们称被包含的keymap为parent-keymap

如果parent-keymap中的绑定关系被更改了,则这些改变也会影响到继承keymap(因为keymap中的元素为指向parent-keymap的引用). 反过来则不会影响parent-keymap,这是因为使用define-key更改的绑定是直接记录在继承的keymap中的

-   (keymap-parent keymap)

获取keymap中的parent-keymap

-   (set-keymap-parent keymap parent-keymap)

设置参数parent-keymap为keymap的parent keymap,若参数parent-keymap为nil,则清空keymap的所有parent keymap

该函数返回参数parent-keymap

若keymap本身为其他sub-keymap的parent keymap,则该操作也会影响到sub-keymap

-   (make-composed-keymap maps &amp;optional parent)

若希望创建一个keymap,这个keymap继承于多个keymap, 则需要使用该函数

该函数创建一个keymap,该keymap集合了maps中的所有keymap的绑定信息. 还可以为它设置一个parent keymap

参数maps可以是单个的keymap,或者一个由keymap组成的list.

当Emacs在创建的这个keymap中搜索event type的绑定信息时,Emacs会按顺序搜索maps中的keymap,最后搜索parent. 以找到的第一个符合条件的绑定为准.

需要注意的是: **maps中的nil绑定会覆盖parent中的绑定信息,但是不会覆盖maps中的其他keymap的绑定信息**

```emacs-lisp
(defvar help-mode-map
  (let ((map (make-sparse-keymap)))
    (set-keymap-parent map
                       (make-composed-keymap button-buffer-map special-mode-map))
    ... map) ... )

```


### 标准Emacs prefix key keymap {#标准emacs-prefix-key-keymap}

-   esc-map

global keymap. 默认绑定到&lt;ESC&gt;

-   help-map

global keymap. 默认绑定到C-h

-   mode-specific-map

global keymap. 默认绑定到C-c

-   ctl-x-map

global keymap. 默认绑定到C-x

-   mule-keymap

global keymap. 默认绑定到C-x &lt;RET&gt;

-   ctl-x-4-map

global keymap. 默认绑定到C-x 4

-   ctl-x-5-map

global keymap. 默认绑定到C-x 5

-   2C-mode-map

global keymap. 默认绑定到C-x 6

-   vc-prefix-map

global keymap. 默认绑定到C-x v

-   goto-map

global keymap. 默认绑定到M-g

-   search-map

global keymap. 默认绑定到M-s

-   facemenu-keymap

global keymap. 默认绑定到M-o

-   (define-prefix-command symbol &amp;optioinal mapvar prompt)

该函数创建一个sparse keymap并存储在symbol的function definition中, 这样绑定到该symbol的key sequence就称为了"prefix key"

该函数返回symbol

该函数同时也会设置symbol的value为该keymap,但若参数mapvar为非nil,则symbol的值为mapvar

若参数prompt为非nil,则它称为该keymap的overall prompt string. 同样的,请只有当该keymap为menu时才这么做


### Active Keymaps {#active-keymaps}

active keymaps按照优先级从高到底以此为:

1.  由光标/鼠标点所在的string的keymap property指定的keymap

2.  minor mode开启的keymap

    由变量\`emulation-mode-map-alists',\`minor-mode-overriding-map-alist'和\`minor-mode-map-alist'决定

3.  当前buffer的local keymap

    一般由buffer的major mode决定

4.  global keymap

    由变量\`global-map'决定

除了上面的常见keymaps外,Emacs还提供了一些方法让程序员激活其他的keymaps.

1.  变量\`overriding-local-map'指定的keymap会替代除了globa keymap之外的其他三种常见keymap

2.  \`overriding-terminal-local-map'指定的keymap的优先级高过其他任何keymaps(包括\`overriding-local-map'指定的keymap),
    **常用于临时修改keybinding(参见函数\`set-transient-map')**

总结来说,搜索active keymap的顺序为:

```emacs-lisp
(or (if overriding-terminal-local-map
        (FIND-IN overriding-terminal-local-map))
    (if overriding-local-map
        (FIND-IN overriding-local-map)
      (or (FIND-IN (get-char-property (point) 'keymap))
          (FIND-IN-ANY emulation-mode-map-alists)
          (FIND-IN-ANY minor-mode-overriding-map-alist)
          (FIND-IN-ANY minor-mode-map-alist)
          (if (get-text-property (point) 'local-map)
              (FIND-IN (get-char-property (point) 'local-map))
            (FIND-IN (current-local-map)))))
    (FIND-IN (current-global-map)))

;; In the above pseudo-code, if a key sequence starts with a mouse event (*note Mouse Events::), that event's position is used instead of point, and the event's buffer is used instead of the current buffer.
```

下面是一些相关函数与变量:

-   (current-active-maps &amp;optional olp position)

当前环境下处于激活状态的keymap的list.

正常情况下,该函数的返回值会忽略掉\`overriding-local-map'和\`overriding-terminal-local-map'的值,但若参数olp为非nil,则不会忽略掉这俩个变量的值.

参数position可以是\`event-start'函数返回的event position或buffer position,表示使用position所在的string的keymap property指定的keymap代替光标或鼠标点所在的stirng的keymap property

-   (key-binding key &amp;optional accept-defaults no-remap position)

该函数根据key在当前active 的keymaps中查找对应的binding.

参数accept-default控制是否检查default binding,即keymap中格式为(t . binding)的元素

当command被重映射过,key-binding默认会查找被重新映射过的binding,但若参数no-remap为非nil,则\`key-binding'会忽略重映射

参数position可以是\`event-start'函数返回的event position或buffer position,表示使用position所在的string的keymap property指定的keymap代替光标或鼠标点所在的stirng的keymap property

```emacs-lisp
(key-binding "\C-x\C-f")                ; => find-file
```

-   global-map

改变量额值为默认的global keymap

-   (current-global-map)

返回当前global keymap的 **引用而不是拷贝**.

-   (current-local-map)

该函数返回当前buffer的local keymap的 **引用而不是拷贝**

-   (current-minor-mode-maps)

返回当前minor mode开启的keymaps的list

-   (use-global-map keymap)

设置keymap为global keymap

-   (use-local-map keymap)

设置keymap为当前buffer的local keymap(大多数major mode命令都使用该函数来设置local keymap)

参数keymap可以为nil,表示不设置local keymap

-   minor-mode-map-alist

该变量是一个由元素'(variable . keymap)组成的alist.

它表示,当变量variable的值为非nil时,对应的keymap处于激活状态,否则若变量variable的值为nil,则keymap被禁用. 这里keymap可以是一个keymap或者一个function definition为keymap的symbol

通常情况下变量variable被用于控制是否启用某个minor mode

-   minor-mode-overriding-map-alist

该变量运行major mode覆盖特定minor mode的key binding.

该alist的元素跟\`minor-mode-map-alist'类似,也是(VARIABLE . KEYMAP)

若统一变量variable,同时在\`minor-mode-overriding-map-alist'和\`minor-mode-map-alist'中出现,则使用\`minor-mode-overriding-map-alist'中的对应keymap覆盖\`minor-mode-map-alist'中的keymap

-   overriding-local-map

该值若为nil,则该值所指定的keymap会覆盖除了global map外的所有keymap

-   overriding-terminal-local-map

若为非nil,则该变量所表示的keymap,会优先于\`overriding-local-map',buffer的local keymap,text property或overlay keymap和其他所有的minor mode keymap

改变了为terminal local variable,不能设置为buffer-local

-   overriding-local-map-menu-flag

若该值为非nil,则表示\`overriding-local-map'和\`overriding-terminal-local-map'的值可以影响到menu bar的显示, 默认为nil

需要注意的是,无论是否影响menu bar的显示,\`overriding-local-map'和\`overriding-terminal-local-map'依然会影响到通过menu bar产生的key sequence的执行情况

-   special-event-map

存放special event与command对应关系的keymap.

若某event type在该keymap中有binding,则该event type被认为是特殊的,并且该event所对应的绑定command会直接被函数\`read-event'所执行

-   emulation-mode-map-alist

This variable holds a list of keymap alists to use for emulation modes.
  It is intended for modes or packages using multiple minor-mode keymaps.
  Each element is a keymap alist which has the same format and meaning as \`minor-mode-map-alist', or a symbol with a variable binding which is such an alist.
  The "active" keymaps in each alist are used before \`minor-mode-map-alist' and \`minor-mode-overriding-map-alist'.

-   (set-transient-map keymap &amp;optional keep-pred on-exit)

该函数临时增加keymap作为优先级最高的keymap

正常情况下,keymap只会被使用一次,来查找紧接着的下一个key sequence的binding. 但若参数keep-pred为t则keymap一直有效,直到某个key sequence在keymap中找不到binding为止.

	 参数on-exit,若为非nil,则需要是一个不带参数的回调函数,会在map失效时调用该函数.
.


### Key Lookup {#key-lookup}

根据key sequence在指定keymap中查找binding的过程,称为"Key lookup"

key lookup的过程只使用key sequence中各event的event type作为查找的条件,而忽略event的其他部分. 当event为mouse event时,甚至可以使用symbol类型的mouse event type来代替list类型的整个event.

通过key lookup找到的binding,我们称之为"keymap entry".

这里的BINDING可以是以下几种类型的对象

-   nil

-   command/lambda表达式

-   string/vector

bingding为keyboard macro

-   keymap

-   list

这时list的格式应该为'(other-keymap . other-event-type),表示为"indirect entry"

即,它会在other-keymap中搜索other-event-type表示的binding

该功能重用与为某个key创建其他key的引用.

-   symbol

使用symbol的function definition

-   其他类型

上面几种类型的binding找到回,都会当成command被execute-command执行.

但若不是上面几种类型的其他类型,则不会被当成command看待


#### 相关函数 {#相关函数}

-   (lookup-key keymap key &amp;optional accept-defaults)

    返回key在keymap中的binding
    ```emacs-lisp
    (lookup-key (current-global-map) "\C-x\C-f") ; => find-file
    (lookup-key (current-global-map) (kbd "C-x C-f")) ; => find-file
    (lookup-key (current-global-map) "\C-x\C-f12345") ; => 2
    ```
    If the string or vector KEY is not a valid key sequence according to the prefix keys specified in KEYMAP, it must be "too long" and have extra events at the end that do not fit into a single key sequence.
    Then the value is a number, the number of events at the front of KEY that compose a complete key.

    若参数accept-defaults不为nil,则\`lookup-key'在找不到key的binding是,会使用default binding

    若key为meta character+普通character则会被分拆为一个由两个character组成的sequence:\`meta-prefix-char'表示的值+普通character
    ```emacs-lisp
    (lookup-key (current-global-map) "\M-f") ; => forward-word
    (lookup-key (current-global-map) "\ef")  ; => forward-word
    ```
    Unlike \`read-key-sequence', this function does not modify the specified events in ways that discard information .
    In particular, it does not convert letters to lower case and it does not change drag events to clicks.

-   (undefined)

    该函数常用于keymap中用于undefine某个key.

    它调用\`ding',但是不会引发error

-   (local-key-binding key &amp;optional accept-default)

    该函数在当前的local keymap(由major-mode决定的)中查找key的binding

    参数accept-defaults决定了是否检查default binding

-   (global-key-binding key &amp;optional accept-defaults)

    该函数在global keymap中查找key的binding

    参数accept-defaults决定了是否检查default binding

-   (minor-mode-key-binding key &amp;optional accept-defaults)

    该函数返回所有active minor mode中key所对应的binding.

    返回的值为由'(MODENAME . BINDING)组成的alist

    若第一个找到的binding不是prefix definition(即binding不为keymap或function definition不为keymap的symbol),则接下来的其他minor mode中的binding都会忽略掉,因为他们都会被找到的该binding所屏蔽掉.
    Similarly, the list omits non-prefix bindings that follow prefix bindings.

    参数accept-defaults决定了是否检查default binding

-   配置项meta-prefix-char

    该变量为meta作为prefix时的character code,用于将代meta的character转换为一个由两个character组成的sequence. 默认为27,表示&lt;ESC&gt;的character code
    ```emacs-lisp
    meta-prefix-char                    ; The default value.
         => 27
    (key-binding "\M-b")
         => backward-word
    ?\C-x                               ; The print representation
         => 24                          ;   of a character.
    (setq meta-prefix-char 24)
         => 24
    (key-binding "\M-b")
         => switch-to-buffer            ; Now, typing `M-b' is
                                        ;   like typing `C-x b'.

    (setq meta-prefix-char 27)          ; Avoid confusion!
         => 27                          ; Restore the default value!
    ```


### Changing Key Binding {#changing-key-binding}

-   (define-key keymap key binding)

该函数设置keymap中的key的对应binding.

若参数key为\`[t]',则表示设置default binding

-   (substitute-key-definition olddef newdef keymap &amp;optional oldmap)

该函数扫描keymap中的所有binding为olddef的key,并rebind这些key到newdef. 该函数返回nil

若参数oldmap为非nil,则表示 **只有在oldmap中包含的key才参与替换.**

-   (suppress-keymap keymap &amp;optional nodigits)

该函数将keymap中的所有binding为\`self-insert-command'的key都重映射到\`undefined'上,使得插入文本变得不可能.

若nodigits为nil,则\`suppress-keymap'重新映射数字的binding为\`dgit-argument',重新映射\`-'为\`negative-argument'

若nodigits为非nil,则数字和\`-'都跟字符一样映射为\`undefined'

由于该函数会更改keymap的结构,因此当该keymap被其他keymap引用时,很容易产生问题,因此,一般参数keymap都为新创建的keymap

-   (global-set-key key binding)

设置global keymap中key的绑定

```emacs-lisp
(global-set-key KEY BINDING)
;; ==
(define-key (current-global-map) KEY BINDING)
```

-   (global-unset-key key)

删除global-keymap中key的绑定

```emacs-lisp
(global-unset-key KEY)
;; ==
(define-key (current-global-map) KEY nil)
```

-   (local-set-key key binding)

设置local keymap中key的绑定

```emacs-lisp
(local-set-key KEY BINDING)
;; ==
(define-key (current-local-map) KEY BINDING)
```

-   (local-unset-key key)

取消local keymap中key的绑定

```emacs-lisp
(local-unset-key KEY)
;; ==
(define-key (current-local-map) KEY nil)
```


### Remapping Commands {#remapping-commands}

A special kind of key binding can be used to "remap" one command to another, without having to refer to the key sequence(s) bound to the original command.

To use this feature, make a key binding for a key sequence that starts with the dummy event \`remap', followed by the command name you want to remap;

例如,要用\`my-kill-line'替代\`kill-line',可以这样:

```emacs-lisp
(define-key my-mode-map [remap kill-line] 'my-kill-line)
```

需要注意的是: **remap只对active keymap才生效**
另外, **不能对已经remap的binding再做remap**. 例如

```emacs-lisp
(define-key my-mode-map [remap kill-line] 'my-kill-line)
(define-key my-mode-map [remap my-kill-line] 'my-other-kill-line) ;不会生效
```

要取消remap,只需要remap到nil即可

```emacs-lisp
(define-key my-mode-map [remap kill-line] nil)
```

-   (command-remapping command &amp;optional position keymap)

该命令返回当前active-keymap中被remap的command(a symbol)被remap到什么binding

若command没有被remap,或者不是symbol类型,则该函数返回nil.

参数position可以通过指定buffer position或mouse event position来决定当前active的keymap

若指定了参数keymaps,则使用参数keymaps中的keymap代替但钱active keymaps. **该参数在position为非nil时被忽略**


### 用于转换event sequence的keymap {#用于转换event-sequence的keymap}

当\`read-key-sequence'函数读取key sequence时,它使用"translation keymaps"来转换特定的event sequence为其他的event sequence.

这些translation keymap按优先级排列分别为:\`input-decode-map',\`local-function-key-map'和\`key-translation-map'

当读取key sequence时,Emacs会针对其中的每个event作一次检查,若在translation keymap中发现有对应的binding,则将该event转换为绑定的event

> For example, VT100 terminals send \`&lt;ESC&gt; O P' when the keypad key
> &lt;PF1&gt; is pressed.  On such terminals, Emacs must translate that
> sequence of events into a single event \`pf1'.  This is done by
> "binding" \`&lt;ESC&gt; O P' to \`[pf1]' in \`input-decode-map'.  Thus, when you
> type \`C-c &lt;PF1&gt;' on the terminal, the terminal emits the character
> sequence \`C-c &lt;ESC&gt; O P', and \`read-key-sequence' translates this back
> into \`C-c &lt;PF1&gt;' and returns it as the vector \`[?\C-c pf1]'.

Translation keymaps take effect only after Emacs has decoded the keyboard input (via the input coding system specified by \`keyboard-coding-system').

-   input-decode-map

该变量的keymap转换的是由功能键产生的key sequence(This variable holds a keymap that describes the character sequences sent by function keys on an ordinary character terminal.)

-   local-function-key-map

This variable holds a keymap similar to \`input-decode-map' except that it describes key sequences which should be translated to alternative interpretations that are usually preferred.
  It applies after \`input-decode-map' and before \`key-translation-map'.

注意: **若\`local-function-key-map'中的某个key与minior mode keymap,local keymap或global keymap有冲突,则该key会被忽略而不会作转换**

-   key-translation-map

类似\`input-decode-map'. 区别在于它的优先级比较低.

与\`input-decode-map'类似,与\`local-function-key-map'不同,该keymap不存在与其他普通keymap冲突就不生效的情况.

You can use \`input-decode-map', \`local-function-key-map', and \`key-translation-map' for more than simple aliases, by using a function, instead of a key sequence, as the "translation" of a key. Then this function is called to compute the translation of that key.

The key translation function receives one argument, which is the prompt that was specified in \`read-key-sequence'--or \`nil' if the key sequence is being read by the editor command loop.  In most cases you can ignore the prompt value.


### Scanning keymaps {#scanning-keymaps}

-   (accessible-keymaps keymap &amp;optional prefix)

该函数返回指定keymap中所有可以到达的'(key . binding)的alist

-   (map-keymap function keymap)

该函数对keymap中的每个binding都调用function执行一次. function接收两个参数:event type和binding

若keymap包含parent-keymap,该parent-keymap中的binding也被执行,这话似一个不断递归的过程.

-   (where-is-internal comand &amp;optional keymap firstonly noindirect no-remap)

该函数返回一个list,包含了在keymaps中绑定到command的所有key sequence.

**参数command可以任意object,该函数使用\`eq'与keymap的binding进行比较**

若参数keymap为nil,则搜索范围为当前active的keymaps(不管\`overriding-local-map'的值).

若参数keymap为keymap,则搜索范围为参数keymap和global keymap

若参数keymap为keymap组成的list,则搜索范围仅仅是指定的这些keymaps

-   命令(describe-binding &amp;optional prefix buffer-or-name)

该命令列出当前key binding并显示在\*Help\*

若参数prefix为非nil,则只列出以prefix为prefix key的key sequence

默认绑定为\`C-h b'


### Menu Keymaps {#menu-keymaps}


#### Defining Menus {#defining-menus}

一个menu其实就是带有prompt-string的keymap. 这个prompt-string是\`make-keymap',\`make-sparse-keymap'和\`define-prefix-command'函数的最后一个参数.

需要注意的是,使用define-key为menu创建新绑定时,会将新绑定放在最前面,因此若关心menu项的顺序的话,要注意从最后项开始向最前项定义.

当然也可以使用\`define-key-after'来指定新绑定的位置.

-   (keymap-prompt keymap)

    获取keymap中的prompt-string

<!--list-separator-->

-  Simple Menu Items

    定义menu item的原始方法是绑定event type(具体是什么event type无所谓)到如下格式的list

    -   (ITEM-STRING . REAL-BINDING)
    -   (ITEM-STRING HELP-STRING . REAL-BINDING)

    例如:

    ```emacs-lisp
    (defvar menu-lancher-keymap (make-sparse-keymap "menu-lancher"))

    (define-key menu-lancher-keymap "C-c C-a"
      '("menu-item" "menu-item-help" (lambda ()
                                       (message "show menu-item"))))
    ```

    需要注意的是, 使用lookup-key来查找键绑定时,只会返回real-binding这部分内容.

    ```emacs-lisp
    (lookup-key menu-lancher-keymap "C-c C-a")
    ;; => ((lambda nil (message "show menu-item")))
    ```

    若real-binding为nil,则menu-item无法被选中.

    若real-binding为symbol,且该symbol具有非nil的menu-enable属性(该属性需为一个S-表达式),则每次显示menu都会执行该S-表达式以决定该item是否可选.

<!--list-separator-->

-  Extended Menu Item

    定义Extended menu item的方法是绑定event type(具体是什么event type无所谓)到如下格式的list(**这些list的car都是menu-item**)

    -   (menu-item ITEM-NAME)

        定义不可选中的menu item.

    -   (menu-item ITEM-NAME REAL-BINDING . ITEM-PROPERTY-LIST)

        定义可选中的menu item. 其中

        -   ITEM-NAME为一个表达式,该表达式的运行结果必须返回一个string.

        -   REAL-BINDING为要执行的command

        -   ITEM-PROPERTY-LIST定义了menu item的其他信息

            | :enable FORM               | FORM的计算结果决定了该menu item是否有用                                                                                                                                                                                                                    |
            |----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
            | :visible FORM              | FORM的计算结果决定了是否显示该menu item                                                                                                                                                                                                                    |
            | :helper HELP-STR           | HELP-STR为固定字符串,其指定了help-echo的显示内容                                                                                                                                                                                                           |
            | :button (TYPE . SELECTED)  | 该属性提供了一种定义radio button或toggle button的方式                                                                                                                                                                                                      |
            |                            | TYPE表示button的类型,可以是:toggle或:radio                                                                                                                                                                                                                 |
            |                            | SELECTED需要为一个FORM,该FORM的计算结果决定了是否选中该button                                                                                                                                                                                              |
            |                            | 对于toggle button来说,SELECTED的返回值决定了该button为on还是off                                                                                                                                                                                            |
            |                            | 对于radio button来说,The SELECTED form for each radio button in the group should check whether the variable has the right value for selecting that button.  Clicking on the button should set the variable so that the button you clicked on becomes selected. |
            | :key-sequence KEY-SEQUENCE | 该属性指定该menu item对应的command可能被分配到哪个键序列上.                                                                                                                                                                                                |
            |                            | 若分配的键序列与该command实际分配的键序列相同,则能够加快menu的显示速度                                                                                                                                                                                     |
            |                            | 若分配的键序列与实际分配的键序列不同,则无效果                                                                                                                                                                                                              |
            | :key-sequence nil          | 表示该menu item对应的command可能没有对应的键绑定,这使得Emacs不用去搜索对应的键序列,从而加快menu的显示速度                                                                                                                                                  |
            | :keys STRING               | STRING被显示为触发该menu item的键序列,可以在STRING中使用\\\\[...]格式                                                                                                                                                                                      |
            | :filter FILTER-FN          | 该属性提供了一种动态产生menu item的途径.                                                                                                                                                                                                                   |
            |                            | FILTER-FN为一个函数,该函数接收REAL-BINDING作为唯一的参数,该函数的返回值会作为该menu item的真正READL-BINDING.                                                                                                                                               |

<!--list-separator-->

-  Menu Separator

    所谓menu separator是一种特殊的menu item. 它被显示为一个横线,因此被用来将menu分隔成几个部分.

    menu separator也是一个以symbol menu-item开头的list. 它的格式如下:

    -   (menu-item SEPARATOR-TYPE)
        ```emacs-lisp
        (menu-item "--")
        ```

    -   (menu-item SEPARATOR-TYPE nil . ITEM-PROPERTY-LIST)
        ```emacs-lisp
        (menu-item "--" nil :visible (boundp 'foo))
        ```

    这里SEPARATOR-TYPE为一个以"--"开头的字符串. 它有如下几种类型

    -   "" / "-" / "--"

        默认的seprator风格

    -   "--no-line" / "--space"

        以空格分隔

    -   "--single-line"

        以menu前景色着色的单行线

    -   "--double-line"

        以menu前景色着色的双行线

    -   "--single-dashed-line"

        以menu前景色着色的单行间断线

    -   "--double-dashed-line"

        以menu前景色着色的双行间断线

    -   "--shadow-etched-in"

        3D的凹入线

    -   "--shadow-etched-out"

        3D的凸出线

    -   "--shadow-etched-in-dash"

        3D的凹入间断线

    -   "--shadow-etched-out-dash"

        3D的凸出间断线

    -   "--shadow-double-etched-in"

        3D的凹入双行线

    -   "--shadow-double-etched-out"

        3D的凸出双行线

    -   "--shadow-double-etched-in-dash"

        3D的凹入双行间断线

    -   "--shadow-double-etched-out-dash"

        3D的凸出双行间断线


#### Menu and the Mouse {#menu-and-the-mouse}

若一个menu keymap是由鼠标事件触发的,则会弹出一个menu,使得用户可以通过鼠标来选择菜单项.

the event generated is whatever character or symbol has the binding that brought about that menu item.

**比较常见的方式是使用button-down事件来触发menu,这样一释放鼠标就能触发menu item了**

一般情况下,若menu keymap的binding中有另一个keymap,则该keymap被认为是submenu. **但若被包含的keymap中无其他menu item,则会直接显示该keymap的内容,而不是作为submenu**

However, if Emacs is compiled without X toolkit support, or on text terminals, submenus are not supported.
Each nested keymap is shown as a menu item, but clicking on it does not automatically pop up the submenu.
If you wish to imitate the effect of submenus, you can do that by giving a nested keymap an item string which starts with ‘@’.
This causes Emacs to display the nested keymap using a separate "menu pane"; the rest of the item string after the ‘@’ is the pane label.
If Emacs is compiled without X toolkit support, or if a menu is displayed on a text terminal, menu panes are not used; in that case, a ‘@’ at the beginning of an item string is omitted when the menu label is displayed, and has no other effect.


#### Menus and the Keyboard {#menus-and-the-keyboard}

若menu是由键盘事件触发的,则我们称这种menu为keyboard menu.

Emacs使用文本的方式,在echo area中显示菜单. 若在一次无法显示完整的菜单,可以通过按&lt;SPC&gt;(由变量\`menu-prompt-more-char'决定)来显示下一屏菜单.


#### The Menu Bar {#the-menu-bar}

要添加item到menu bar上,需要先创建自己的"function key"(例如KEY),然后为 `[menu-bar KEY]` 这样一个键序列添加绑定.

若为同一个 `[menu-bar KEY]` 绑定了多个不同的menu item,则只会创建一个menu,但包含了多个menu item的内容.

下面是一个例子:

```emacs-lisp
;; Make a menu keymap (with a prompt string)
;; and make it the menu bar item’s definition.
(define-key global-map [menu-bar words]
  (cons "Words" (make-sparse-keymap "Words")))

;; Define specific subcommands in this menu.
(define-key global-map
  [menu-bar words forward]
  '("Forward word" . forward-word))
(define-key global-map
  [menu-bar words backward]
  '("Backward word" . backward-word))
```

local keymap可以取消global keymap创建的menu bar item. 方法是重新绑定该item的key为'undefined.
例如:

```emacs-lisp
(define-key dired-mode-map [menu-bar edit] 'undefined)
```

-   menu-bar-final-items

    默认情况下,menu bar先显示global-map中定义的menu item,再显示local-map中定义的menu item.

    而该变量中包含着的多个item对应的fake function key. 这些item会显示在menubar的最后面.

-   menu-bar-update-hook

    在重新刷新menu bar之前会触发该hook.


#### Tool bars {#tool-bars}

Tool-bar的定义与menu-bar的定义及其类似,只是fake "function key"由"menu-bar"变为了"tool-bar". 例如:

```emacs-lisp
(define-key global-map [tool-bar KEY] ITEM)
```

同样,也可以通过为major mode的local map添加\`[tool-bar FOO]'绑定来设置特定major-mode的tool-bar

你还可以为Modifier-key(shift,control,meta...)+鼠标点击定义不同的意义. 例如:

```emacs-lisp
;; if the original item was defined this way,
(define-key global-map [tool-bar shell]
  '(menu-item "Shell" shell
              :image (image :type xpm :file "shell.xpm")))

;; here is how you can define clicking on the same tool bar image with the shift modifier:
(define-key global-map [tool-bar S-shell] 'some-command)
```

**但是ITEM中的REAL-BINDING必须是一个command,而不能是keymap**

ITEM中除了能使用一般的extended menu item中的属性外,还能使用\`:image'属性来指定显示的图标.

-   \`:image IMAGE'

    这里IMAGE可以是一个image对象,或一个由4个image对象组成的vector.

    若IMAGE是一个由4个image对象组成的vector,则在不同的条件下会展示不同的image对象:

    | item 0 | Used when the item is enabled and selected.    |
    |--------|------------------------------------------------|
    | item 1 | Used when the item is enabled and deselected.  |
    | item 2 | Used when the item is disabled and selected.   |
    | item 3 | Used when the item is disabled and deselected. |

    若IMAGE为单个的image对象,则当该menu item disable时,Emacs自动使用edge-detection算法修改image的显示

-   \`:rtl IMAGE'

    表示当使用right-to-left语言时,显示的图像内容.

相关函数与变量

-   tool-bar-map

    默认情况下,\`[tool-bar]'的定义是通过如下代码实现的:
    ```emacs-lisp
    (global-set-key [tool-bar]
                    `(menu-item ,(purecopy "tool bar") ignore
                                :filter tool-bar-make-keymap))
    ```
    这里函数\`tool-bar-make-keymap'会从变量\`tool-bar-map'中动态派生出实际的tool-bar map.
    因此可以通过修改该变量的值来修改默认的global tool-bar.
    在某些Major mode中(例如Info mode)是通过将\`tool-bar-map'设为buffer-local,再设置该值的方式来代替global tool bar的.

-   (tool-bar-add-item icon def key &amp;rest props)

    通过修改\`tool-bar-map'的方式,添加item到tool bar中. 其中

    ICON为XPM,XBM或PBM文件的 **base name**. 例如若ICON为"exit",则emacs会依次查找exit.xpm,exit.pbm和exit.xbm

    DEF为实际调用的命令

    KEY为the fake function key symbol in the prefix keymap.

    PROPS则为tool bar menu item的其他属性.

    To define items in some local map, bind ‘tool-bar-map’ with ‘let’ around calls of this function:
    ```emacs-lisp
    (defvar foo-tool-bar-map
      (let ((tool-bar-map (make-sparse-keymap)))
        (tool-bar-add-item …)
        …
        tool-bar-map))
    ```

-   (tool-bar-add-item-from-menu command icon &amp;optional map &amp;rest props)

    用该函数可以将menu bar中的绑定添加到tool bar中来.

    各参数说明与tool-bar-add-item icon类似.

    The binding of COMMAND is looked up in the menu bar in MAP (default ‘global-map’)

    MAP must contain an appropriate keymap bound to ‘[menu-bar]’.

    该函数会修改\`tool-bar-map',因此最好只在修改global tool bar item时才使用该函数

-   (tool-bar-local-item-from-menu command icon in-map &amp;optional from-map &amp;rest prop)

    该函数可以用来创建非全局的tool bar items.

    使用方式类似\`tool-baar-add-item-from-menu',只是多了个IN-MAP参数用来指明新加的tool bar item放在哪个keymap中.

-   auto-resize-tool-bars

    若该变量值为非nil,则tool bar会自动改变大小以便在一个frame中能显示出所有的tool bar item

    若变量值为'grow-only,则tool bar只自动增大,而不自动缩小. 要想缩小tool bar,必须用户手工输入\`C-l'

-   auto-raise-tool-bar-buttons

    若该变量为非nil,则当鼠标移到item上时,该item会凸起

-   tool-bar-button-margin

    指定了tool bar item的extra margin. 单位为像素

-   tool-bar-button-relief

    该变量值指定了tool bar item的阴影宽度,单位为像素

-   tool-bar-border

    This variable specifies the height of the border drawn below the tool bar area.  An integer specifies height as a number of pixels.

    If the value is one of ‘internal-border-width’ (the default) or ‘border-width’, the tool bar border height corresponds to the corresponding frame parameter.


#### Modifying Menus {#modifying-menus}

当使用\`define-key'往menu中添加item时,会把新加的item放到menu的第一项的位置.

若想指定新加item的位置,需要使用\`define-key-after'

-   (define-key-after map key binding &amp;optional after)

    与\`define-key'类似,在MAP中将KEY绑定到BINDING. 但是位置由AFTER决定.

    The argument KEY should be of length one—a vector or string with just one element.

    But AFTER should be a single event type—a symbol or a character, not a sequence.

    先加入的item位置在AFTER之后,若AFTER为t或nil,则表示新item会放在最后的位置.

    下面是一些例子
    ```emacs-lisp
    ;; makes a binding for the fake function key <DRINK> and puts it right after the binding for <EAT>.
    (define-key-after my-menu [drink]
      '("Drink" . drink-command) 'eat)


    ;; Here is how to insert an item called ‘Work’ in the ‘Signals’ menu
    ;; of Shell mode, after the item ‘break’:
    (define-key-after
      (lookup-key shell-mode-map [menu-bar signals])
      [work] '("Work" . work-command) 'break)
    ```


#### Easy Menu {#easy-menu}

使用宏\`easy-menu-define symbol maps doc menu'可以很容易的定义pop-up-menu / menu-bar-menu

-   (easy-menu-define symbol maps doc menu)

    该宏定义一个pop-up-menu / menu-bar-menu,其内容由MENU决定.

    若参数SYMBOL非空,则会生成一个名为SYMBOL的函数,该函数用来弹出该menu. 且该函数使用DOC作为其的doc-string.

    参数SYMBOL不需要被引用.

    MAPS可以为一个keymap,也可以为一个由keymap组成的list.

    -   if MAPS is a keymap, the menu is added to that keymap, as a top-level menu for the menu bar
    -   It can also be a list of keymaps, in which case the menu is added separately to each of those keymaps.

    MENU参数的第一个元素必须为字符串,该字符串被作为menu label.

    MENU参数中紧跟着的是以下的键值对的任意组合:

    -   :filter FUNCTION

        FUNCTION会接收唯一的一个函数:其他menu item组成的列表. 其返回值作为menu中显示的真实item

    -   :visible VISIBLE-FORM

        VISIBLE-FORM的执行结果决定了该item是否可见

    -   :active ENABLE-FORM

        ENABLE-FORM的执行结果决定了该item是否可选.

    参数MENU可以是如下几种形式:

    -   [NAME CALLBACK ENABLE]

        NAME为一个字符串,表示menu item的名字

        CALLBACK为点击该item时要运行的命令或S-表达式

        ENABLE为一个S-表达式,其执行结果决定了该item是否可用.

    -   [MENU CALLBACK [KEYWORD ARG]...]

        这里KEYWORD与ARG对必须是如下几种:

        -   :keys KEYS

            KEYS为与menu item等价的keyboard键序列,它需为字符串格式

        -   :key-sequence KEYS

            该KEYS用来加速Emacs第一次显示menu.

            若你知道该menu item没有对应的keyboard键序列,则KEYS应该设成nil,否则它应该是表示该键序列的string或vector

        -   :active ENABLE

            ENABLE为一个S表达式,其计算结果决定了该item是否可用

        -   :visible INCLUDE

            INCLUDE为一个S表达式,其计算结果决定了该item是否可见

        -   :label FORM

            FORM是一个S表达式,其计算结果作为该menu item的标签(默认为参数NAME)

        -   :suffix FORM

            FORM是一个S表达式,其计算结果会作为后缀自动加到menu item标签后

        -   :style STYLE

            STYLE为一个symbol,它决定了该item的类型.

            它可以是以下几个值:

            | 'toggle | checkbox           |
            |---------|--------------------|
            | 'radio  | radio button       |
            | 其他    | ordinary menu item |

        -   :selected SELECTED

            SELECTED为一个S表达式,其计算结果决定了该checkbox item或radio button item是否被选中

        -   :help HELP

            HELP为一个字符串,描述了menu item

    -   menu-string

        MENU也可以是一个字符串,则该字符串在menu中显示为一段不可用的文本.

        若字符串以\`-'开头,则解释为menu-separator

    -   (MENU1 MENU2 ...)

        MENU还可以使上述MENU格式组成的list,则表示为submenu

下面是一个例子:

```emacs-lisp
(easy-menu-define words-menu global-map
  "Menu for word navigation commands."
  '("Words"
    ["Forward word" forward-word]
    ["Backward word" backward-word]))
```


## Major Mode and Minor Mode {#major-mode-and-minor-mode}


### Hooks {#hooks}

Emacs中大多数的hook为"normal hooks",这表示该hook中的函数会被不带参数调用,且他们的返回值会被忽略. 以\`-hook'结尾的hook都是"normal hook".

非\`-hook'结尾的hook可能为"abnormal hook",这表示该hook中的函数会被带参数调用,或者他们的返回值会被使用. 一般这种\`abormal hook'的名称是以\`-functions'结尾的.

以\`-function'结尾的hook表示它的值为单个的函数,而不是函数列表,此时 **不能用\`add-hook'来添加hook函数,而需要使用\`add-function'代替**

hook触发式,排在前面的hook函数优先被调用


#### 调用Hook中的函数 {#调用hook中的函数}

-   (run-hooks &amp;rest hookvars)
    该函数依次调用hookvars中的hook函数. 每个hookvar都是一个 **符号**. 且每个hook都需为一个"normal hook"

    若hookvar为buffer-local变量,则调用的hook函数以hookvar的buffer-local的值为准,但 **若此时hook值中有元素t,则表示全局的hook值中定义的函数也会被调用**

-   (run-hook-with-args hook &amp;rest args)
    该函数以参数args调用hook中的函数,此时hook为"abnormal hook"

-   (run-hook-with-args-until-failure hook &amp;rest args)
    以参数args依次调用abnormal hook中的函数,直到某个函数返回nil为止

    该函数返回最后那次调用函数的返回值. 即若由于hook中某个函数返回nil而退出时也返回nil,否则返回非nil值

-   (run-hook-with-args-until-success hook &amp;rest args)
    以参数args依次调用abnormal hook中的函数,直到某个函数返回非nil为止

    该函数返回最后那次调用函数的返回值. 即若由于hook中某个函数返回非nil而退出时则返回该值,否则返回nil值


#### 设置Hook变量 {#设置hook变量}

-   (add-hook hook function &amp;optional append local)
    为hook添加function为hook函数

    若function已经存在(使用equal进行比较),则不再重复添加

    若function的\`permanent-local-hook'属性非nil,则 **\`kill-all-local-variables'或更改major mode都不会将该函数从hook的当前值中被删除**

    默认情况下,function会被放在hook的最前方优先调用,但\`append'参数可以让function添加到hook的最后方,最后被调用

    参数\`local'表示将function加入buffer-local hook中,该标志会使hook变为buffer-local变量,并在buffer-local变量值中添加元素\`t'(表示同时也执行该hook的global value)

-   (remove-hook hook function &amp;optional local)
    该函数从hook中移除function

    若参数\`local'为非nil,则表示将该hook变为buffer-local hook,然后移除function


### Major Modes {#major-modes}

每个Major Mode都需要有一个"mode hook". Major Mode在完成初始化过程的最后一个步骤都应该是调用"mode hook"

每个Major Mode都需要一个以\`-mode'结尾的命令用来进入该Major Mode

-   命令(fundamental-mode)
    fundamental-mode不包含任意的与该mode相关的函数定义和变量设置. 它 **甚至没有mode hook**

-   配置项major-mode
    该变量为buffer-local变量,其值为一个symbol,标识了当前处于哪个major mode. 其标准的默认值为\`fundamental-mode'

    若默认值为nil,则当Emacs使用命令(例如\`switch-to-buffer')创建新buffer(??是创建吗??)时, **新buffer的major mode与之前buffer保持不变**
    但是作为一个特例,若前一个buffer的major-mode的\`mode-class'属性为\`special',则 **新buffer的Major Mode为Fundamental mode** (具体参见Major Mode Conventions)


#### Major Mode Conventions {#major-mode-conventions}

定义Major Mode时,有很多惯例需要遵守,因此推荐使用\`define-derived-mode'从一个现成的Major Mode中继承新Major Mode. 该宏会自动维护这些惯例.

下面是一些常见的惯例

-   进入Major Mode的命令以\`-mode'结尾

    且当不带参数调用时,该命令通过设置 **keymap,syntax table和buffer-local变量** 将当前buffer切换到新的Major Mode.

    该命令不能修改buffer的内容

-   为进入Major Mode的命令添写doc-string,简要描述一下该mode有哪些特殊命令

    doc-string中可以使用\`\\[COMMAND]',\`\\[KEYMAP]'和\`\\&lt;KEYMAP&gt;'来自动显示用户自定义的键绑定.

-   进入Major Mode的命令的第一个动作应该是调用\`kill-all-local-variables'

    \`kill-all-local-variables'会先触发\`change-major-mode-hook',然后清理之前的Major Mode设置的buffer-local变量

-   Major Mode Command需要设置buffer-local变量\`major-mode'的值为major mode command的symbol.

    命令\`describe-mode'会根据该变量输出帮助文档

-   Major Mode Command需要设置buffer-local变量\`major-name'的值为该major mode的"pretty" name.

    \`major-name'通常为一个字符串,且它的值用来会显示在mode line上

-   由于Emacs只有一个命名空间,因此所有与Major Mode相关的变量,常量和函数,应该以该major mode名称为前缀

-   当Major Mode是用来编辑特定结构的文本或编程语言时,具备根据结构自动缩进文本是很有用的一项功能,因此这类major mode一般都会有缩进函数,并将其设置为变量\`indent-line-function'的值. 同时也可能设置其他一些关于缩进的变量的值

-   Major Mode通常有其自己的keymap,该keymap的名字一般为\`mode名称-mode-map'

    Major Mode Command应该调用\`use-local-map'函数来安装自己的keymap

-   Major Mode Keymap中的键绑定,一般以\`C-c'+控制字符或数字或\`{}&lt;&gt;:;'为前缀.

    \`C-c'+标点符号是留给minor mode使用的.

    \`C-c'+普通字母留给用户使用的

    major mode也可以重新绑定\`M-n',\`M-p'但应该表示某种向前向后移动的命令

-   编辑文本的major mode不应该重定义&lt;RET&gt;为任何非换行的命令.

    非编辑文本的major mode无此显示

-   Major mode不应该修改哪些会严重影响用户性能的配置项(例如是否开启Auto-Fill mode)

-   若major mode有自己的syntax table,则该syntax table变量的名称规范为\`mode名称-mode-syntax-table'

-   若major mode希望支持某种编程语言的注释语法,则需要设置与注释语法相关的变量.

    具体参见\`comment-column',\`comment-start-skip',\`comment-start',\`comment-end'

-   若major mode有自己的缩写表,则需要存放到名为\`MODENAME-mode-abbrev-table'的变量中.

    If the major mode command defines any abbrevs itself, it should pass ‘t’for the SYSTEM-FLAG argument to ‘define-abbrev’. 详情参见[Defining Abbrevs](https://www.gnu.org/software/emacs/manual/html_node/elisp/Defining_002520Abbrevs.html "Emacs Lisp: (info \"(elisp) Defining%20Abbrevs\")")

-   major mode通过设置buffer-local变量\`font-lock-defaults'来设置高亮

-   major mode中用到的每个face都应该尽可能的从已有的Emacs face中继承

-   major mode应该告诉Imenu如何找出buffer中的各个定义和章节的位置.

    方法是通过设置\`imeu-generic-expression',\`imenu-prev-index-position-function',\`imenu-extract-index-name-function',\`imenu-create-index-function'. 具体参见[Imenu](https://www.gnu.org/software/emacs/manual/html_node/elisp/Imenu.html "Emacs Lisp: (info \"(elisp) Imenu\")")

-   major mode可以定义buffer local变量\`eldoc-documentation-function'以便eldoc能支持该mode

-   major mode可以通过设置\`completion-at-point-functions'来指定如何实现补全. 具体参见[Completion in Buffers](https://www.gnu.org/software/emacs/manual/html_node/elisp/Completion_002520in_002520Buffers.html "Emacs Lisp: (info \"(elisp) Completion%20in%20Buffers\")")

-   在major mode command中使用\`make-local-variable'来创建buffer-local变量.

    **不要使用\`make-variable-buffer-local'来创建buffer local变量** ,因为该函数会是的即使不是该mode的buffer中的变量也变成buffer-local变量.

-   每个major mode都应该有一个名为\`MODENAME-mode-hook'的normal hook.

    major mode command的最后步骤应该是 **使用\`run-mode-hooks'依次调用\`change-major-mode-after-body-hook',\`MODENAME-mode-hook'和\`after-change-major-mode-hook'这三个hook**

-   若major mode为子mode,则在major mode command开始时还需要调用父mode的major mode command

    通过宏\`define-derived-mode'定义的mode会自动完成这种设计,但若没有使用\`define-derived-mode'宏,则需要手工调用\`delay-mode-hooks'中的父mode command

-   若从major mode切换成其他major mode,则会触发\`change-major-mode-hook',可以进行一些特殊处理

-   若该major mode仅用来管理由major mode自己产生的文本(而不是用户输入的内容),则该major command symbol的\`mode-class'属性应该为\`special',像下面所示:
    ```emacs-lisp
    (put 'funny-mode 'mode-class 'special)
    ```
    默认情况下,若\`major-mode'的默认值为\`nil',则新创建的buffer会继承当前buffer的major mode. 但对于属性\`mode-class'为\`special'的major mode来说, **新创建的buffer使用Fundamental Mode代替**,像Dired,Rmail,Buffer List这些Major Mode都开启了该特性

    同时,在这些special major mode中调用\`view-buffer'函数并不能启用\`view-mode' minor mode,因为这类的mode通常都提供了他们自己的类似view-mode的键绑定

    这类major mode,推荐使用\`define-derived-mode'直接从\`special-mode'中继承

-   通过配置\`auto-mode-alist'变量,可以让Emacs打开特定规则的文件名时自动选中Major Mode.

    If you define the mode command to autoload, you should add this element in the same file that calls ‘autoload’.

    If you use an autoload cookie for the mode command, you can also use an autoload cookie for the form that adds the element (参见[autoload cookie](https://www.gnu.org/software/emacs/manual/html_node/elisp/autoload_002520cookie.html "Emacs Lisp: (info \"(elisp) autoload%20cookie\")"))

-   定义mode的代码可能会被重复执行

    因此在定义与mode相关的变量时,推荐使用\`defvar'和\`defcustom'


#### How Emacs Chooses a Major Mode {#how-emacs-chooses-a-major-mode}

当Emacs打开文件时,会自动根据文件名称和文件内容选择合适的major mode

-   命令(normal-mode &amp;optional find-file)

    让Emacs为当前buffer选择合适的major-mode

    该函数先调用\`set-auto-mode'函数,然后运行\`hack-local-variables'来使file local变量生效. 参见[Local Variables in Files](https://www.gnu.org/software/emacs/manual/html_node/emacs/Local_002520Variables_002520in_002520Files.html "Emacs Lisp: (info \"(emacs) Local%20Variables%20in%20Files\")") 和[File Local Variables](https://www.gnu.org/software/emacs/manual/html_node/elisp/File-Local-Variables.html "Emacs Lisp: (info \"(elisp) File Local Variables\")")

    若参数\`find-file'为非nil,则normal-mode假设是被\`find-file'函数调用的,这种情况下,它会根据\`enable-local-variables'的值来决定是否应用file local变量的值.

    若参数\`find-file'为nil,则无条件应用file local变量值

    该函数内部调用\`set-auto-mode'来选择major mode,若选择失败,则根据\`major-mode'的默认值来决定应用major mode

-   函数(set-auto-mode &amp;optional keep-mode-if-same)

    该函数为当前buffer选择合适的major-mode,其选择的依据依次为

    1.  根据\`_\*\_'行或文件结尾处的\`mode:' file local变量值.

        注意: **若\`enable-local-variables'为nil,或文件名称匹配\`inhibit-local-variables-regexps'中的元素,则Emacs不使用file local变量**

    2.  根据\`interpreter-mode-alist'变量值和\`#!'行推测

    3.  根据\`magic-mode-alist'变量值和buffer开头的内容推测

    4.  根据\`auto-mode-alist'变量值和文件名称推测

    若参数keep-mode-if-same为非nil,则若该buffer已经处于合适的mode状态时,并不再此调用major mode coomand. **这样是为了防止用户自定义的buffer loal 变量值被重设**

-   (set-buffer-major-mode buffer)
    设置指定buffer的major mode为\`major-mode'的默认值.

    若默认值为nil,则表示使用当前buffer的major mode.

    作为特例,\`\*scratch' buffer的值会被设置为\`initial-major-mode'

    低层的原始创建buffer的函数不会调用该函数,但中层的创建buffer的函数(例如\`switch-to-buffer'和\`find-file-noselect')在创建buffer时会使用该函数

-   配置项initial-major-mode

    该值决定了初始的\`\*scratch\*' buffer的major mode. 该变量的值应该为major mode command的symbol

-   变量interpreter-mode-alist

    该变量告诉Emacs如何根据\`#!'行的内容判断major mode

    它一个alist,其元素格式为\`(REGEXP . MODE)' 表示\`#!'行内容匹配REGEXP的其Major Mode为MODE

-   变量majic-mode-alist

    该变量告诉Emacs如何根据buffer内容判断major mode

    该值为一个alist,其元素格式为\`(REGEXP . FUNCTION)', 若buffer开头部分的内容匹配REGEXP,且FUNCTION为非nil,则Emacs会通过调用该FUNCTION切换Major Mode

    若FUNCTION为nil,则通过变量\`auto-mode-alist'判断Major Mode

-   变量\`majic-fallback-mode-alist'
    该变量类似\`magic-mode-alist',但 **只有在\`auto-mode-alist'中没有相应配置时才生效**

-   变量\`auto-mode-alist'
    该变量告诉Emacs如何根据文件名称判断major mode

    该值为一个alist,其元素格式 **一般** 为\`(REGEXP . MODE-FUNCTOIN)'表示文件名称匹配REGEXP的话,调用MODE-FUNCTION来选择Major Mode(若访问的文件是[扩展过的文件名](https://www.gnu.org/software/emacs/manual/html_node/elisp/File_002520Name_002520Expansion.html "Emacs Lisp: (info \"(elisp) File%20Name%20Expansion\")") ,则文件名会先经过\`file-name-sans-versions'过滤掉版本号或备份标志)

    元素还可能为格式\`(REGEXP FUNCTION t)',表示调用FUNCTION后,Emacs继续搜索\`auto_mode-alist'并选择合适的Major Mode. 该功能在处理压缩的文件时特别有用.


#### Mode Help {#mode-help}

-   命令(describe-mode &amp;optional buffer)

    该命令显示指定buffer(默认为当前buffer)的major mode和minior mode的相关文档.

    该函数使用\`documentation'函数从major mode command和minior mode command中获取doc-string


#### Derived Modes {#derived-modes}

创建一个新的major mode的推荐方法是使用\`define-derived-mode'从一个现有的major mode中继承出来.

即使没有接近的major mode,那也应该从\`text-mode',\`special-mode'或\`prog-mode'这三大基本major mode中选一个来继承.

实在不行,那就从\`fundamental-mode'中继承

-   宏(define-derived-mode VARIANT PARENT NAME DOCSTRING KEYWORD-ARGS... BODY...)

    该宏定义VARIANT为新Major Mode Command,该Major Mode,继承自PARENT,且以NAME为Mode Name

    参数VARIANT和PARENT为不被引用的symbol

    新的Major Mode覆盖了PARENT Mode的以下几个方面:

    -   新Major Mode拥有自己的keymap,名为\`VARIANT-map'.
        除非\`VARIANT-map'在调用\`define-derived-mode'前已经被设置并且定义了父keymap,否则PARENT mode的keymap为\`VARIANT-map'的父keymap

    -   新Major Mode拥有自己的syntax table,名为\`VARIANT-syntax-table',但该名字可以通过\`:syntax-table' keyword关键字修改
        除非\`VARIANT-syntax-table在调用\`define-derived-mode'前已经被设置并且定义了父syntax-table,否则PARENT mode的syntax-table为\`VARIANT-syntax-table的父syntax-table

    -   新Major Mode拥有自己的abbrev table,存在名为\`VARIANT-abbrev-table'的变量中,但该变量名可以通过\`:abbrev-table' keyword关键字修改

    -   新Major Mode有自己的mode hook,名为\`VARIANT-hook'.
        该hook会在运行完其祖先的mode hook后,通过\`run-mode-hooks'调用

    可以在参数BODY指定了如何覆盖PARENT mode的其他设置. **注意不要加\`interactive'语句,\`define-derived-mode'会自动添加该语句**

    若PARENT有一个非nil的\`mode-class'属性,则\`define-derived-mode'会设置VARIANT的\`mode-class'属性为相同的值.

    参数PARENT也可以为nil,表示新Major Mode没有父mode

    参数DOCSTRING为对新Major Mode的说明,\`define-derived-mode'会在此基础上增添一些关于mode hook,mode keymap的信息,该参数可以省略

    \`define-drived-mode'支持以下几种KEYWORD-ARG

    -   \`:syntax-table'

        为新Major Mode指定syntax table变量.

        若参数为nil,则表示使用PARENT mode的syntax table变量,若参数PARENT为nil,则使用标准syntax-table

    -   \`:abbrev-table'

        为新Major Mode指定abbrev-table变量.

        若参数为nil,表示使用PARENT mode的abbrev-table变量,若参数PARENT为nil,则使用\`fundamental-mode-abbrev-table'

    -   \`:group'

        指定了该mode所属的customization group.

-   函数(derived-mode-p &amp;rest modes)

    当前Major Mode是否继承自modes中的任意一个mode, modes为symbol列表


#### Basic Major Modes {#basic-major-modes}

除了Fundamental mode外,还有三个普遍被继承的mode:Text mode,Prog mode和Special mode

-   命令(text-mode)

    Text-mode用于编辑自然语言.

    It defines the ‘"’ and ‘\\’ characters as having punctuation syntax (参见 [Syntax Class Table](https://www.gnu.org/software/emacs/manual/html_node/elisp/Syntax_002520Class_002520Table.html "Emacs Lisp: (info \"(elisp) Syntax%20Class%20Table\")"))

    该mode下绑定\`M-&lt;TAB&gt;'为\`ispell-complete-word'

-   Prog-mode

    Prog-mode用于编辑编程语言. 大u偶数的编程语言major mode都继承自该mode

    Prog-mode设置\`parse-sexp-ignore-comments'为\`t',设置\`bidi-paragraph-direction'为\`left-to-right'

-   Special-mode

    Special-mode常用于那些内容由Emacs自动产生(而不是人工输入或从文件读取)的buffer中.

    从Special mode继承的mode会设置\`mode-class'属性为\`special'

    Special mode设置mode为只读的.且会重新绑定很多通用绑定,例如\`q'绑定为\`quit-window',\`g'绑定为\`revert-buffer'


#### Mode Hooks {#mode-hooks}

每个Major Mode Command最后三条指令应该是调用\`change-major-mode-after-body-hook',自己的\`MODE-hook',和\`after-change-major-mode-hook'. **通常是通过调用函数\`run-mode-hooks'来自动完成上面的三个步骤.**

当major mode为某个父mode的子mode,则在body中调用父mode command时,应该放入\`delay-mode-hooks'结构内,这样才能保证父mode的hook不会立即被触发,而统一等到子mode调用\`run-mode-hooks'时再触发.

若不使用\`define-derived-mode'宏,而选择手工定义Major Mode,则可能会需要用到下列函数

-   函数(run-mode-hooks &amp;rest hookvars)

    Major Mode应该使用该函数来运行自己的mode hook.

    该函数类似\`run-hooks',但它还会调用\`change-major-mode-after-body-hook'和\`after-change-major-mode-hook'.

    **若在\`delay-mode-hooks'宏的body中调用该函数,它不会立即执行这些hook,而是推迟到下一次调用\`run-mode-hooks'时再执行**

-   宏(delay-mode-hooks &amp;rest body)
    该宏执行BODY中的语句,但BODY中的所有\`run-mode-hooks'调用都会延迟运行他们的hook,直到下次不在\`delay-mode-hooks'结构中的调用\`run-mode-hooks'才运行.

-   变量\`change-major-mode-after-body-hook'
    触发的时机在major mode hook之前

-   变量\`after-change-major-mode-hook'
    触发时机正常应该为major mode command的最后一步!


#### Generic Modes {#generic-modes}

"Generic Modes"是指的那些只支持基本的注释语法和Font Lock Mode的简单Major Mode.

使用宏\`define-generic-mode'来定义generic mode,更多例子参见\`generic-x.el'中的内容

-   (define-generic-mode mode comment-list keyword-list font-lock-list auto-mode-list function-list &amp;optional docstring)

    若参数docstring为nil,则\`define-generic-mode'会自动生成一个


### Minor Modes {#minor-modes}

"minor mode"提供了一系列的与major mode无关的特性.

-   变量minor-mode-list
    该变量存储了所有的minor mode commands


#### Minor Mode Conventions {#minor-mode-conventions}

定义minor mode也有一些惯例要遵循,这些惯例有:

-   每个minor mode都应该有一个以\`-mode'结尾的指示变量,用于判断该minor mode是否启用.

    若minor mode是buffer-local的,则该指示变量也应该是buffer-local的

    该指示变量常与变量\`minor-mode-alist'结合来在mode line上显示minor mode name.

    该指示变量还与变量\`minor-mode-map-alist'结合来判断是否激活minor mode keymap.

-   定义一个与上面的指示变量同名的命令,该命令用于开启/关闭minor mde

    该命令需要能够接收一个可选参数.

    1.  当以interactive方式调用该命令时,若不带参数调用该命令,则切换minor-mode的状态,若参数为正数,则开启minor-mode,若为负数则关闭minor-mode

    2.  当在lisp中调用该命令时,若参数为nil或正数,则开启minor mode,参数若为'toggle,则切换minor-mode,参数为负数则关闭minor-mode

    下面是一个实现模板
    ```emacs-lisp
    (interactive (list (or current-prefix-arg 'toggle)))
    (let ((enable (if (eq arg 'toggle)
    (not foo-mode) ; this mode’s mode variable
    (> (prefix-numeric-value arg) 0))))
    (if enable
    DO-ENABLE
    DO-DISABLE))
    ```

-   若想在mode line显示该minor mode,则需要往变量中\`minor-mode-alist'中添加相应的元素.

    \`minor-mode-alist'中的元素格式应该为\`(MODE-VARIABLE STRING)'. 其中

    -   \`MODE-VARIABLE'为指示minor-mode是否开启的哪个变量名称

    -   \`STRING'为在mode line上的显示文本.

    -   **要注意 minor-mode-alist中不要出现重复的MODE-VARIABLE**

-   还有一些类似Major Mode的惯例
    -   those regarding the names of global symbols

    -   the use of a hook at the end of the initialization function

    -   the use of keymaps

    -   other

-   另外,尽可能允许用户通过\`customization'来开闭minor mode.

    因此应该尽量使用\`defcustom'来定义minor mode的标识变量. 并且要记得 **給该标识变量加上autoload cookie并指定\`:require'定义minor mode的库**:
    ```emacs-lisp
       ;;;###autoload
    (defcustom msb-mode nil
      "Toggle msb-mode.
         Setting this variable directly does not take effect;
         use either \\[customize] or the function `msb-mode'."
      :set 'custom-set-minor-mode
      :initialize 'custom-initialize-default
      :version "20.4"
      :type    'boolean
      :group   'msb
      :require 'msb)
    ```


#### Keymaps and Minor Modes {#keymaps-and-minor-modes}

每个minor mode都可以有自己的keymap. 要为minor mode设置自己的keymap,需要往变量\`minor-mode-map-alist'中添加元素. 具体参见[Definition of minor-mode-map-alist](https://www.gnu.org/software/emacs/manual/html_node/elisp/Definition_002520of_002520minor_002dmode_002dmap_002dalist.html "Emacs Lisp: (info \"(elisp) Definition%20of%20minor-mode-map-alist\")")


#### Defining Minor Modes {#defining-minor-modes}

-   (define-minor-mode MODE DOC [INIT-VALUE [LIGHTER [KEYMAP]]] KEYWORD-ARGS... &amp;rest BODY)

    该宏定义一个新的名为MODE的minor mode,并生成一个名为MODE的minor mode command. 参数DOC为该minor mode的说明文档

    该宏还定义了一个名为MODE的指示变量,通过设置该变量为t或nil可以开启/关闭该minor mode. 该变量的默认值为INIT-VALUE.

    参数LIGHTER为一个字符串或nil,当开启了该minor mode后就会在mode-line中显示该字符串, 若为nil则表示不显示在mode-line上

    参数KEYMAP为nil,值为keymap的变量名,keymap,或元素为\`(KEY-SEQUENCE . DEFINITION)'的alist. 它指定了该minor mode所使用的keymap,并生成名为\`MODE-map'的变量用于存放keymap
    这里的KEY-SEQUENCE和DEFINITION参数格式要匹配\`define-key'函数的参数格式.

    目前参数KEYWORD-ARGS支持如何keyword参数:

    -   \`:group GROUP'

        指定BODY中的\`defcustom'语句定义配置项时所属的默认组为GROUP, 若不指定该参数,则所属的默认组为参数MODE

        **使用该参数前,请确保已经预先用\`defgroup'定义了分组**

    -   \`:global GLOBAL-P'

        指明该minor-mode作用于为global还是buffer-local, 默认为nil即为buffer-local的.

        将一个minor-mode变为global的,意味着它的指示变量\`MODE'被定义为用户配置项, 通过配置项界面改变该变量的值会同时关闭/开启该minor mode.

        这种情况下,需要 **保证在配置该配置项时有运行相应的minor mode代码**,最简单的方法是使用\`:require'关键字

    -   \`:init-value INIT-VALUE'

        设置指示变量\`MODE'的初始值

    -   \`:lighter LIGHTER'

        设置显示在mode-line上的内容

    -   \`:keymap KEYMAP'

        设置minor mode的KEYMAP

    -   \`:variable PLACE'

        使用PLACE作为minor mode的指示变量(默认为参数MODE).

        这里PLACE可以是一个变量名称,或者可以被\`setf'赋值的泛型变量(参见[Generalized Variables](https://www.gnu.org/software/emacs/manual/html_node/elisp/Generalized_002520Variables.html "Emacs Lisp: (info \"(elisp) Generalized%20Variables\")")).

        PLACE还可以是格式为\`(GET . SET)'的cons cell. 其中GET为获取minor mode状态的表达式, SET为接收一个参数并设置minor mode的函数

    -   \`:after-hook AFTER-HOOK-FORM'

        AFTER-HOOK-FORM为一个S表达式(不需要被引用), 它会在minor mode hook触发后运行.

    -   其他任意的keyword参数

        **这些keyword参数直接被传递給\`defcustom'用来作为生成minor mode指示变量时的参数**. 比较常见的有\`:require'参数.

    下面是一个使用\`define-minor-mode'的例子
    ```emacs-lisp
    (define-minor-mode hungry-mode
    "Toggle Hungry mode.
    Interactively with no argument, this command toggles the mode.
    A positive prefix argument enables the mode, any other prefix
    argument disables it.  From Lisp, argument omitted or nil enables
    the mode, `toggle' toggles the state.

    When Hungry mode is enabled, the control delete key
    gobbles all preceding whitespace except the last.
    See the command \\[hungry-electric-delete]."
    ;; The initial value.
    nil
    ;; The indicator for the mode line.
    " Hungry"
    ;; The minor mode bindings.
    '(([C-backspace] . hungry-electric-delete))
    :group 'hunger)
    ```

-   宏(define-globalized-minor-mode GLOBAL-MODE MODE TURN-ON KEYWORD-ARGS)
    创建一个与MODE对应的名为GLOBAL-MODE的minor mode.

    \`GLOBAL-MODE'的意义在于同时启用/关闭所有buffer中的名为\`MODE'的minor mode

    它会使用函数\`TURN-ON'来开启buffer中的minor mode,使用\`(MODE -1)'来关闭buffer中的minor mode

    该宏会定义一个名为\`GLOBAL-MODE'的配置项,用户可以通过customize通过更改该配置项的值来开启/关闭该minor mode. **当更改该配置项时,请保证已经执行了该\`define-globalized-minor-mode'代码,最简单的方法是使用\`:require'关键字参数**

    使用\`:group GROUP' keyword 参数指定了\`GLOBAL-MODE'所属的组别.

    **一般情况下,定义了一个global minor mode的同时,需要定义一个buffer local minor buffer,这样才能允许某些buffer不开启该minor mode**


### Mode Line Format {#mode-line-format}

每个Emacs window的底部一般都会有一个mode line,用来显示buffer的信息.

每个Emacs Window的顶部也可以有一个header line,其作用与mode line类似.


#### Mode Line基础说明 {#mode-line基础说明}

mode line显示什么内容由buffer local变量\`mode-line-format'决定.

header line显示什么内容则由buffer local变量\`header-line-format'决定

当前选中的window的mode line使用名为\`mode-line'的face显示, 其他为选中的mode line使用名为\`mode-line-inactive'的face显示

出于效率考虑,Emacs不会不停地更新mode line和header line. 只有当你进行如下操作时才回去更新mode-line和header-line

1.  更改window configuration
2.  切换buffer
3.  narrow或widen buffer
4.  scroll buffer
5.  修改了buffer内容
6.  调用了函数\`force-mode-line-update'

7.  (force-mode-line-update &amp;optional all)
    该函数强制更新当前buffer的mode-line和header-line

    但若参数all为非nil,则表示强制更新所有buffer的mode-line和header-line


#### mode-line-format,header-line-format和frame-title-format的格式 {#mode-line-format-header-line-format和frame-title-format的格式}

mode-line-format,header-line-format和frame-title-format的格式可能是以下几种类型:

-   字符串
    类似\`format'函数中的格式说明符.

    除了字符串中的"%-Constructs"会被替换为其他数据外,其他的内容原样显示

    若部分子串包含\`face'属性,则该部分字串显示时使用\`face'属性指定的face来显示, 其他没有\`face'属性的字串,使用\`mode-line'或\`mode-line-inactive'来显示.

    **字串中的\`help-echo'和\`keymap'属性具有特殊的意义**,具体参见[Properties in Mode](https://www.gnu.org/software/emacs/manual/html_node/elisp/Properties_002520in_002520Mode.html "Emacs Lisp: (info \"(elisp) Properties%20in%20Mode\")")

-   SYMBOL类型的变量

    显示为symbol的值.

    但 **即使symbol的值为带"%-constructs"的字符串,其中的"%-constructs"也不会被转义!!**

    此外,若symbol的值为t或nil,或者该symbol被标记为"risky"(该symbol的\`risky-local-variable'属性非nil),则这些symbol的值会被忽略

-   (字符串 其他数据...) / (列表 其他数据...)

    若数据为一个list,且该list的第一个元素为字符串类型或list类型,则表示递归处理所有的列表元素,并将结果合并显示.

-   (:eval FORM)

    若数据为一个list,且第一个元素为\`:eval',则表示运行\`FORM',并将结果作为显示内容

    **注意 FORM中不要load任何文件,因为这样可能导致无穷循环**

-   (:propertize ELT PROPS...)

    若数据为一个list,且第一个元素为\`:propertize',则表示递归处理ELT,并为结果加上PROPS提供的字符串属性.

    参数PROPS应该由0到多个"TEXT-PROPERTY" "VALUE"组成

-   (SYMBOL THEN ELSE)

    若数据为一个list,且第一个元素为非条件语句的关键字,则表示若SYMBOL的值为非nil则显示递归处理THEN的处理结果,否则显示ELSE的处理记过.

    ELSE可以省略,则表示若symbol值为nil,什么也不显示

-   (WIDTH REST...)

    若数据为一个list,且第一个元素为整数,则表示限定REST的结果显示宽度为WIDTH.

    若WIDTH为正数,表示向左对齐,WIDTH为负数,表示向右对齐.


#### Mode-line-format中常用到的变量 {#mode-line-format中常用到的变量}

下面所列举的变量,都是mode-line-format的组成部分

-   mode-line-mule-info

    该变量在mode-line-format中可以显示语言环境,buffer的字节编码,当前的输入法

-   mode-line-modified

    该变量会显示当前buffer是否被修改. 默认情况下:

    1.  若buffer被修改显示'\*\*'

    2.  若buffer未被修改显示为'--'

    3.  若buffer只读显示为'%%'

    4.  若buffer只读单被修改,显示为'%\*'

-   mode-line-frame-identification

    该变量可以用来标识当前frame

    在图形界面下,该变量默认为" ". 在字符界面下,该变量默认为"-%F "

-   mode-line-buffer-identification

    该变量可以用来标识当前buffer.

    它的默认值为"%12b",附带一些text properties

-   mode-line-position

    该变量标识光标所在当前buffer的位置.

    它默认为"%p",显示当前所在位置是整个buffer的百分之多少处.

    但也可以给它添加注入buffer大小,行号,列号等信息

-   vc-mode

    该变量指示buffer所关联的文件纳入的版本控制信息.

    若文件未纳入版本控制系统中,则该变量值为nil

-   mode-line-modes

    该变量显示buffer所开启的major mode,minor mode,递归编辑层次,buffer所关联进程的状态,是否出于narrow状态

    该变量值中又包含了三个变量:

    -   mode-name

        Major Mode的名称

    -   mode-line-process

        与当前buffer相关的process的状态.

        该变量紧挨着major-name显示. 默为nil

    -   minor-mode-alist

        该变量是一个buffer local变量,它指示了如何显示minor-mode.

        该变量的元素组成格式为\`(MINOR-MODE-VARIABLE MODE-LINE-STRING)'

        当MINOR-MODE-VARIABLE的值为非nil时,则显示MODE-LINE-STRING,否则什么也不显示

<!--listend-->

-   mode-line-remote

    该变量用来显示当前buffer的\`default-directory'是否是远程目录

-   mode-line-client

    该变量用来标识哪些frame为\`emacsclient'的frame

-   global-mode-string

    若开启了该minor mode的话,该变量的内容会显示在\`which-func-mode' minor mode后,否则会显示在\`mode-line-modes'后


#### %-constructs说明 {#constructs说明}

类似format函数中的格式字符串,%-constructs的格式为\`%[整数]标识',这里的整数指定了最小的长度,标识指定了替换为何值

| %%     | %字符                                                                                                                                                                                                                                            |
|--------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| %b     | 当前buffer名称                                                                                                                                                                                                                                   |
| %c     | 当前光标所在的列数                                                                                                                                                                                                                               |
| %e     | 当Emacs接近内存耗尽时,显示警告信息,否则为空                                                                                                                                                                                                      |
| %f     | 当前buffer访问的文件名称                                                                                                                                                                                                                         |
| %F     | 当前frame的title或name                                                                                                                                                                                                                           |
| %i     | 当前buffer可访问部分的size,单位为字节                                                                                                                                                                                                            |
| %I     | 类似%i,但以更人性化的方式显示,例如会转换为多少k,M,G                                                                                                                                                                                              |
| %l     | 当前光标所在的行数                                                                                                                                                                                                                               |
| %n     | 若当前buffer出于narrow状态,则显示"Narrow",否则显示空                                                                                                                                                                                             |
| %P     | The percentage of the buffer text above the **top** of window, or ‘Top’, ‘Bottom’ or ‘All’.  Note that the default mode line construct truncates this to three characters.                                                                       |
| %p     | The percentage of the buffer text that is above the **bottom** of the window (which includes the text visible in the window, as well as the text above the top), plus ‘Top’ if the top of the buffer is visible on screen; or ‘Bottom’ or ‘All’. |
| %s     | 与当前buffer相关的process的状态                                                                                                                                                                                                                  |
| %z     | 键盘,中断,buffer编码格式的信息                                                                                                                                                                                                                   |
| %Z     | 类似%z,但还包括换行符的信息                                                                                                                                                                                                                      |
| %\*    | 若buffer只读,显示"%",若buffer被修改过,显示"\*",否则显示"-"                                                                                                                                                                                       |
| %+     | 类似%\*,但若一个read-only buffer被修改了,它显示"\*",而%\*会显示"%"                                                                                                                                                                               |
| %&amp; | 若buffer被修改则显示"\*",否则显示"-"                                                                                                                                                                                                             |
| %[     | 显示递归编辑的层次,多少层就有多少个"["                                                                                                                                                                                                           |
| %]     | 显示递归编辑的层次,多少层就有多少个"]"                                                                                                                                                                                                           |
| %-     | 使用"-"填充剩余的mode line                                                                                                                                                                                                                       |


#### Mode Line中的text properties {#mode-line中的text-properties}

某些text properties在mode line中有其特殊的意义:

-   \`face'属性影响text的显示
-   \`help-echo'属性提供了光标指向他text后弹出的帮助内容
-   \`keymap'属性是的text能处理鼠标点击事件


#### 模拟mode line的显示结果 {#模拟mode-line的显示结果}

-   (format-mode-line format &amp;optional face window buffer)
    该函数,将format当成是\`mode-line-format'的值,模拟当指定的WINDOW显示指定BUFFER时会显示怎样的mode-line.

    参数face指定了那些没有指定\`face'属性的text应该如何显示


#### Window Header Line {#window-header-line}

head-line与mode-line极其类似,它的显示是由变量\`header-line-format'指定的. 且\`header-line-format'的格式与\`mode-line-format'的格式一样

当一个Window只能显示一行内容时,则它不会显示header line.

若一个window只能显示两行内容时,它无法同时显示mode-line和header-line. 这时若mode-line不为nil,则会显示mode-line而不是header-line

-   (window-header-line-height &amp;optional window)
    该函数返回指定WINDOW的header line的高度,单位为像素


### Imenu {#imenu}

Imenu会在imenu菜单(参见[Imenu](https://www.gnu.org/software/emacs/manual/html_node/emacs/Imenu.html "Emacs Lisp: (info \"(emacs) Imenu\")"))中列出buffer中的语法定义名称或章节名称,然后通过点击菜单中的定义名称或章节名称就能直接跳转到相应位置上了.

-   命令(imenu-add-to-enubar NAME)

    该命令会产生一个名为NAME的imenu菜单

当然,使用Imenu的前提是,能够产生一个定义/章节名称与buffer位置之间关系的索引.


#### 通过设置\`imenu-generic-expression'定义Imenu {#通过设置-imenu-generic-expression-定义imenu}

通过设置\`imenu-generic-expression'定义Imenu,是最常用的方式

-   imenu-generic-expression

    该变量为一个list,它指定了使用那种正则表达式来定位定义/章节的位置. 它的元素可以是以下两种格式

    -   (SUB-MENU-TITLE REGEXP INDEX)

        这里SUB-MENU-TITLE表示通过REGXP找到的项目应该放到名为\`SUB-MENU-TITLE'的子菜单中.
        若其值为nil,则表示找到的项目直接放到Imenu菜单中.

        参数REGEXP则表示buffer中任何匹配REGEXP的内容都被认为是定义/章节.

        参数INDEX则说明了,REGEXP中哪个匹配的子组为定义/章节的名称

    -   (SUB-MENU-TITLE REGEXP INDEX FUNCTION ARGUMENTS...)

        这种格式的\`SUB-MENU-TITLE',\`REGEXP'和\`INDEX'与上面格式的意义相同.

        不同之处在于,当点击Imenu中的相关项是,会调用\`FUNCTION'函数,且传入参数为:菜单项的名称,定义的位置和参数ARGUMENTS

-   imenu-case-fold-search

    该变量决定了使用\`imenu-generic-expression'中的正则表达式匹配buffer内容时,是否大小写敏感.

    默认为t,表示大小写敏感.

-   imenu-syntax-alist

    该变量是一个元素为'(CHARACTER-OR-STRING . SYNTAX-DESCRIPTION)的alist.

    当Imenu处理\`imenu-generic-expression'时,会使用该变量来修改当前buffer的syntax table(通过调用\`modfy-syntax-entry'来实现,具体参见[Syntax Table Functions](https://www.gnu.org/software/emacs/manual/html_node/elisp/Syntax_002520Table_002520Functions.html "Emacs Lisp: (info \"(elisp) Syntax%20Table%20Functions\")")).


#### 通过设置\`imenu-prev-index-position-function'和\`imenu-extract-index-name-function'来定义Imenu {#通过设置-imenu-prev-index-position-function-和-imenu-extract-index-name-function-来定义imenu}

-   imenu-prev-index-position-function

    该变量的函数,需要从光标开始处向前扫描,并将光标定位到发现定义/章节的位置,然后返回非nil值.

    若向前再找不到任何定义/章节了,则该函数需要返回nil

-   imenu-extract-index-name-function

    该变量的函数,需要从光标所在位置出抽取出定义/章节的名字


#### 通过设置\`imenu-create-index-function'来定义Imenu {#通过设置-imenu-create-index-function-来定义imenu}

该变量的函数不接受参数,并且返回当前buffer的index alist.

该变量的默认值为\`imenu-default-create-index-function',该函数通过调用\`imenu-prev-index-position-function'和\`imenu-extract-index-name-function'的值来产生index alist. 但若这两个参数有一个为nil,则根据\`imenu-generic-expression'来产生index alist

这里index alist的元素有三种格式

-   '(INDEX-NAME . INDEX-POSITION)

    表示选择INDEX-NAME项,则跳转到INDEX-POSITION位置

-   '(INDEX-NAME INDEX-POSITION FUNCTION ARGUMENTS...)

    表示选择INDEX-NAME项,则执行\`(funcall FUNCTION INDEX-NAME INDEX-POSITION ARGUMENTS...)'

-   '(SUB-MENU-TITLE . SUB-INDEX-ALIST)

    表示根据SUB-INDEX-ALIST创建子菜单SUB-MENU-TITLE


## Font Lock Mode {#font-lock-mode}

Font Lock Mode根据Major Mode的语法规则来对buffer内容进行作色.

Font Lock Mode通过两种途径来寻找要高亮作色的文本:

-   根据syntax table的语法规则来作色,这种方法优先进行
-   通过查询(通常通过正则表达式),这种方法随后运行


### Font Lock Mode的基础说明 {#font-lock-mode的基础说明}

很多变量用来控制Font Lock Mode如何高亮显示. 但Major Mode不应该直接设置这些变量,它应该通过设置\`font-lock-defaults'变量来实现这一目的.

Font Lock minor mode启动时,会根据\`font-lock-defaults'的值来自动设置其他变量.

-   font-lock-defaults

    该变量的值可以为nil,表示Font Lock mode不做任何高亮,需要用户手工通过Edit-&gt;Text Properties-&gt;Faces菜单手工指定那些文本高亮.

    若该变量的值为非nil,则其格式应该为:
    ```emacs-lisp
    (KEYWORDS [KEYWORDS-ONLY [CASE-FOLD [SYNTAX-ALIST [SYNTAX-BEGIN OTHER-VARS...]]]])
    ```
    其中:

    -   KEYWORDS

        会被设置为\`font-lock-keywords'的值. 该变量会影响基于查询的高亮.

        KEYWORDS可能是一个symbol,一个变量或一个返回值为list的函数

        KEYWORDS也可以是一个由多个symbol组成的list,每个symbol对应高亮的一个等级. 其中第一个symbol对应'mode default'等级的高亮,下一个symbol对应'level 1'级的高亮,再下一个symbol对应'level2'级别的高亮依次类推.

        'mode default'级别的高亮通常与'level 1'级别的高亮一样,它在当变量\`font-lock-maximum-decoration'为nil时被使用.

    -   KEYWORD-ONLY

        会被用来设置\`font-lock-keyword-only'的值. 该变量标识了是否只采用基于搜索的高亮.

        若该变量为nil,则Font Lock mode会使用基于语法的高亮,否则Font Lock mode不会使用基于语法的高亮

    -   CASE-FOLD

        被用来设置\`font-lock-keywords-case-fold-search'的值. 该变量表明了在基于搜索的高亮时,是否忽略大小写的差异.

        若为非nil,则表示Font Lock mode在搜索时忽略大小写

    -   SYNTAX-ALIST

        被用来作为Font Lock mode基于语法高亮时使用的syntax table.

        该变量的值若为nil,则其应该为由格式\`(CHAR-OR-STRING . STRING)'组成的list.

        若变量的值为nil,则Font Lock mode使用函数\`syntax-table'返回的syntax table作为依据.

    -   SYNTAX-BEGIN

        该变量用于设置\`font-lock-beginning-of-syntax-function'的值.

        一般设置该值为nil,而使用\`syntax-begin-function'代替

    -   OTHER-VARS

        剩下的所有元素都统一叫做OTHER-VARS, 其中每个元素的格式应该为(VARIABLE . VALUE),表示将VARIABLE设置为buffer local变量,然后设置值为VALUE

        一样用于设置影响Font Lock mode高亮的其他变量


### 基于搜索的高亮 {#基于搜索的高亮}

直接控制基于搜索高亮的变量是\`font-lock-keywords',它一般由\`font-lock-defautls'的'KEYWORDS'元素指定


## Auto-Indentation {#auto-indentation}


## Tabulated List Mode {#tabulated-list-mode}

Tabulated List Mode是一种用列表来显示数据的Major Mode,这种数据往往由一条条的记录组成,每条记录的内容又可以划分到多个列中.

Tabulated List Mode继承至Special mode,且同时被多个子mode继承,例如Process Menu mode和Package Menu Mode

要想继承Tabulated List Mode,需要在\`define-derived-mode'中的BODY中通过设置一些变量(下面会讲到)来定义数据的结构,然后调用函数\`tabulated-list-init-header'来初始化header line
同时,继承的子mode还需要定义一个"listing command",该命令不是major mode command,它是給用户使用的命令封装,该命令通常完成以下几个动作

1.  创建或切换到某个特定buffer
2.  开启新的Major mode
3.  指定要以列表方式显示的数据.
4.  调用\`tabulated-list-print'函数,以弹出该buffer


### 指定tabulated data的格式 {#指定tabulated-data的格式}

通过定义以下变量来指定tabulated data的格式

-   tabulated-list-format
    该变量为buffer-local变量,它定义了数据的格式.

    该变量为一个vector,且vector中的每个元素都表示一个数据列,其格式为\`(NAME WIDHT SORT)'. 其中:

    -   NAME为列的名称

    -   WIDHT为每列的保留宽度,该参数对最后一列来说无意义

    -   SORT指定了根据该列排序记录的方式.

        | 值     | 意义                                                         |
        |-------|------------------------------------------------------------|
        | nil    | 表示不排序                                                   |
        | t      | 表示根据字符串排序                                           |
        | 其他判断函数 | 使用该判断函数传入\`sort'进行排序. 该判断函数的参数为\`tabulated-list-entries'中的元素 |

-   tabulated-list-entries

    该变量为buffer-local变量,它指定了要显示的记录数据. 它的值 **可以为一个list,或一个函数**

    若该值为list,则list中的每个元素格式应该为\`(ID CONTENTS)',其中

    -   ID可以为nil,或一个标识某条记录的lisp object

    -   CONTENT为一个表示记录内容的vector,其中vector中的每个元素表示记录的一列内容.

        vector中的元素可能为字符串或者格式为\`(LABEL . PROPERTIES)'的list,

        若元素为字符串则表示直接显示该字符串

        若元素为\`(LABEL . PROPERTIES)'格式的list,则表示用\`LABEL'和\`PROPERTIES'作为参数调用\`insert-text-button'的方式来插入一个text button(参见[Making Buttons](https://www.gnu.org/software/emacs/manual/html_node/elisp/Making_002520Buttons.html "Emacs Lisp: (info \"(elisp) Making%20Buttons\")"))

    若\`tabulated-list-entries'的值为为一个函数,则不带参数调用该函数应该返回上面格式的list

-   tabulated-list-revert-hook

    该hook在重载入Tabulated List Buffer前会被触发, **常用于对\`tabulated-list-entries'的值作预处理.**

-   tabulated-list-printer

    该变量的值应该为一个函数, 该函数接收\`tabulated-list-entries'中的ID和CONTENTS两个参数, **其返回值才作为真正的列内容显示**

    默认该变量的函数,直接插入CONTENTS

-   tabulated-list-sort-key

    该变量指定了根据那几列对记录进行排序.以及是从大到小还是从小到大排序

    -   nil表示不排序

    -   (NAME . FLIP)格式的cons表示根据名为NAME的列排序,FLIP表示是否反转排序顺序

-   函数(tabulated-list-init-header)
    该函数为Tabulated List buffer生成并设置\`header-line-format'(参见[Header Lines](https://www.gnu.org/software/emacs/manual/html_node/elisp/Header_002520Lines.html "Emacs Lisp: (info \"(elisp) Header%20Lines\")")),并为header line分配keymap以便允许通过点击header列来排序

    该函数依赖上面所述的变量

-   函数(tabulated-list-print &amp;optional remember-pos)
    该函数刷新tabulated list buffer. **该函数通常被list command所调用**

    该函数会作以下动作:

    1.  清空buffer

    2.  根据\`tabulated-list-sort-key'对\`tabulated-list-entries'中的数据进行排序

    3.  调用\`tabulated-list-printer'所表示的函数输出记录

    若参数REMEMBER-POS为非nil,则该函数在刷新buffer前会记住当前行的ID,并在刷新后自动定位到该ID行

-   (tabulated-list-print-entry id cols)

    在光标处插入新entry,entry信息由ID和COLS决定

    其中ID是一个用于标示entry的lisp对象,而COLS为一个由各列信息组成的vector

-   (tabulated-list-print-col n col-desc x) ??

    为光标处entry插入一个特定的entry列

    其中N为列编号,COL-DESC为列描述,X为光标处的列编号

    该函数在插入后,返回列编号


### 其他函数 {#其他函数}

-   (tabulated-list-get-id &amp;optional POS)

    返回POS位置entry的ID. POS默认为光标当前位置

-   (tabulated-list-get-entry &amp;optional POS)

    返回POS位置entry的信息. 返回的格式为一个由列信息组成的vector

    POS默认为光标当前位置

-   (tabulated-list-delete-entry)

    删除光标所在位置的entry. 并返回一个(ID COLS...)的列表来暂时被删除entry的信息.

    **该函数同时会将光标移动到entry的最开头位置**

    **该函数只会更改buffer内容,而不会更改\`tabulated-list-entries'的值**

-   (tabulated-list-set-col COL DESC &amp;optional CHANGE-ENTRY-DATA)

    更改当前位置的entry中第COL列的内容为DESC

    COL可以是列的位置,也可以是列的名称.

    CHANGE-ENTRY-DATA指示了是否同时更改\`tabulated-list-entries'的值

-   (tabulated-list-put-tag tag &amp;optional advance)

    将TAG放入当前entry的padding区域

    TAG为字符串,且大小不能大于\`tabulated-list-padding'的值

    若ADVANCE为非nil,则光标同时移动到下一行

-   tabulated-list-padding

    每个entry前预留于padding的字符个数


## Desktop Save Mode {#desktop-save-mode}


## Emacs Display {#emacs-display}


### The Display Property {#the-display-property}


### Images {#images}


## Number相关函数 {#number相关函数}

-   判断是否为自然数(0+正整数)

(natnump object)

-   判断是否为0

(zerop number)

-   取余数

(% dividend divisor)

%的参数必须是整数,对于任何两个整数dividend和divisor都有(+(% DIVIDEND DIVISOR) (\* (/ DIVIDEND DIVISOR) DIVISOR)) == DIVIDEND

(mod dividend divisor)

mode的参数可以是Float型,它的返回值的正负号与DIVISOR一致,并且mod的商值使用floor进行截断到整数,然后再计算返回的余值.

(+ (mod DIVIDEND DIVISOR) (\* (floor DIVIDEND DIVISOR) DIVISOR)) == DIVIDEND

-   三角函数
-   (sin arg)

-   (cos arg)

-   (tan arg)

-   (asin arg)

-   (acos arg)

-   (atan y &amp;optional x)

-   指数计算
-   (exp arg)
    e的arg次方

-   (expt x y)
    x的y次方

-   (log arg &amp;optional base)
    base默认为e,取arg的指数

-   (aqrt arg)
    取arg的平方根,若arg&lt;0,则返回NaN

-   常量float-e

-   常量float-pi

-   获取随机值
    (random &amp;optional limit)
    -   若limit为正整数，则返回0到limit的随意整数

    -   若limit为t，则表示使用当前时间和Emacs的进程号重新选择一个种子

    -   若limit为一个字符串，它表示使用string的内容作为种子


## 二进制函数 {#二进制函数}

二进制函数只能作用于Integer型参数.

-   逻辑位移

(lsh Integer count)

lsh是logical shift的缩写,它向左移动count位(若count为负数,则表示向右移动), 使用0填充

-   算术位移

(ash Integer count)

ash是arithmetic shift的缩写,它跟lsh类似,但当对负数进行右移时,使用符号位进行填充,即该函数不会改变Integer的正负号

-   and操作

(logand &amp;rest ints-or-markers)

对所有Integer的所有位做and操作,若没有参数,则返回-1,即所有位都是1的Integer

-   or操作

(logior &amp;rest ints-or-markers)
对所有Integer的所有位做or操作,若没有参数,则返回0,即所有位都是0的Integer

-   xor操作

(logxor &amp;rest ints-or-markers)
对所有Integer的所有位做xor操作,若没有参数,则返回0,即所有位都是0的Integer

-   not操作

(lognot integer)
对Integer的所有位做not操作


## 字符串处理相关函数 {#字符串处理相关函数}

  Emacs has only very few functions that takes a string as argument. Any non-trivial string processing is done with a buffer. Use with-temp-buffer, then insert your string, process it, then use
buffer-string to get the whole buffer content.


### 判断函数 {#判断函数}

-   (stringp object)
-   (string-or-null-p object)
-   (char-or-string-p object)
-   (string-prefix-p prefix str &amp;optional ignore-case)
-   (string-suffix-p suffix str &amp;optional ignore-case)
-   (compare-strings str1 start1 end1 str2 start2 end2 &amp;optional ignore-case)

比较的区间为[start end),start为nil则默认为0,end为nil则表示字符串的结尾.

该函数会把unibyte string先转换为multibyte string再进行比较.

若比较的区间,两个string是相等的,则返回t.
  若str1&lt;str2,则返回负数,若str1&gt;str2则返回正数.
  该整数的绝对值指示了不同点开始的地方


### 获取字符串的函数 {#获取字符串的函数}

-   (make-string count character)

    count必须为整数，返回由count个character组成的字符串

-   (string &amp;rest characters)

    返回由characters组成的字符串

-   带text properties截取子字符串

(substring myStr startIndex &amp;optional endIndex)

startIndex和endIndex若为负数,则表示从尾部开始往回数.

因此(substring str 0)表示返回str的一个copy,但推荐用copy-sequence代替

substring中的参数str也可以是一个vector,例如

```emacs-lisp
(substring [a b (c) "d"] 1 3)           ; => [b (c)]
```

若string带有text properties,则新产生的string也会带有text properties

-   不带text properties截取字符串

    (substring-no-peroperties string &amp;optional start end)

-   组合字符串

(concat &amp;rest sequences) 返回连接多个字符串的字符串

```emacs-lisp
(concat "abc" (list 120 121) [122])     ; => "abcxyz"
```

(mapconcat function sequence separator)

对sequence的每个元素都调用function,然后将结果用separator concat起来作为字符串返回.当function为'identity时则起到join的作用

```emacs-lisp
(mapconcat 'identity '("abc" "123" "one" "two" "three") " ") ;=>"abc 123 one two three"
```

-   从buffer获取字符串

(buffer-substring myStartPos myEndPos)

(buffer-substring-no-properties myStartPos myEndPos)

-   获取光标所在位置的内容

(thing-at-point THING),这里THING指明了各种类型:

(thing-at-point 'word) // 光标所在位置的单词

(thing-at-point 'symbol) // 光标所在位置的单词(包括连字符和下划线)

(thing-at-point 'line) // 光标所在位置的行, **一般情况下它获取一行及行结束符,然而若光标处于行的最后一个字符,则不获取行结束符**

(thing-at-point 'sexp) // 光标所在位置的s表达式

(thing-at-point 'sentence) // 光标所在位置的句子

(thing-at-point 'defun) // 光标所在位置的函数

(thing-at-point 'url) // 光标所在位置的url, **若url不以http开头,则会自动加上http**

...还有很多类型

-   获取光标所在THING的开始/结束位置

(bounds-of-thing-at-point THING)

-   从报文中截取字符串

(setq myStr (buffer-substring startPos endPos))


### 字符串操作 {#字符串操作}

-   获取字符串长度

(string-width myStr)

这里要注意的是，不能用(lenth myStr)来获取字符串长度

```emacs-lisp
(length "我的")                         ;=>2
(string-width "我的")                   ;=>4
```

-   修改字符串内容

    由于字符串是character的数组,因此最基础的修改字符串内容的函数是使用(aset str idx char)来将str的地idx位置的内容替换为char.
    由于字符串是数组,而数组的长度是不可变的,因此若替换的character和被替换的character的字节数不相同,则会报错
    ```emacs-lisp
    (setq str "我的")
    (aset str 0 ?\m)                        ;str变为了"m的"
    ```
    (store-substring str idx string-or-char)

    使用string-or-char从idx开始替换str的内容,若替换的内容过长,则会报错
    ```emacs-lisp
    (setq str "我的")
    (store-substring str 0 ?n)              ;str变为"n的"
    (store-substring str 1 "nm")            ;会raise args-out-of-range error
    ```

-   清空字符串

    (clear-string str)
    该函数会使得str变为二进制字符串,并且将内部结构清空为0
    ```emacs-lisp
    (setq str "我的")
    (clear-string str)                      ;=>str现在为"      "
    ```

<!--listend-->

-   判断字符串是否匹配某正则表达式

(string-match myRegex myStr)

-   获取捕获到的分组内容

(match-string N myStr) ,这里myStr可以忽略,表示在buffer中作的查询. 但若上一次的匹配使用string-match函数作的,则需要该参数

-   对字符串进行正则替换

(replace-regexp-in-string myRegex myReplacement myStr)  ;这里myReplacement可以是一个函数,该函数接收匹配的字符串,然后返回要替换的字符串

-   split字符串

(split-string myStr &amp;optiional mySepeartor omit-nulls)

根据separator拆分myStr,默认值为变量\`split-string-default-separators\`的值(默认为"[ \f\t\n\r\v]+")

若omit-nulls为t则在组成list时,会忽略空字符串.

若希望把字符串分解为(split-string-and-unquote "cd 'abc edf'")

-   字符串正则替换

(replace-regexp-in-string myRegxp myReplace myStr)

-


### Format函数 {#format函数}

-   %s

object的输出格式,但是不带引号. 若object为带text properties的string,则text properties也被复制进去了.

-   %S

object的输出格式,带引号

-   %.Ns / %.NS

截取字符串的前N位显示

-   %o

8进制的Integer型

-   %d

十进制的Integer型

-   %x

十六进制的Integer型,字母用小些形式

-   %X

十六进制的Integer型,字母用大写形式

-   %c

输出Character

-   %e

使用指数形式表示Float型

-   %f

使用小数形式表示Float型

-   %g

使用指数或小数形式表示Float型,使用哪种表示方式就看哪个更简短.

-   %%

%


## list相关函数 {#list相关函数}


### 判断函数 {#判断函数}

-   (consp object)

object是否为cons cell. 一般情况下,list就是cons cell,但nil比较特殊,虽然它是list,但不是cons clell

-   (atom object)

nil即是atom,也是list

-   (listp object)

object是否为list

-   (nlistp object)

object是否不是list

-   (null object)
-   (memq object list)

判断list中是否包含object,它返回list中第一次出现object的位置.函数名中的q表示使用 **eq** 作为比较函数.

-   (memql object list)

类似memq,但是函数名中的ql表示使用 **eql** 作为比较函数.

-   (member object list)

类似memq,但是使用 **equal** 作为比较函数

-   (member-ignore-case str list)

类似member,但参数str必须是string类型,并且比较时不区分大小写,单字节字符串也被转换为多字符串进行比较.


### 获取list中的元素 {#获取list中的元素}

-   (car-safe object)

它与car不同点在于,若object不是cons ceil,则car会报错,而car-safe只是返回nil

-   (cdr-safe object)

它与cdr不同点在于,若object不是cons ceil,则cdr会报错,而cdr-safe只是返回nil

-   (pop list)

弹出list中的第一个元素

-   (nth n list)

返回list中的第n个元素(从0开始计算), 若n为负数,则返回list中的第一个元素.

-   (nthcdr n list)

返回第n个cdr后的list,若n为0或负数,则返回原list

```emacs-lisp
(nthcdr 2 '(1 2 3 4))                   ;=>(3 4)
(nthcdr 10 '(1 2 3 4))                  ;=>nil
(nthcdr -3 '(1 2 3 4))                  ;=>(1 2 3 4)
```

-   (last list &amp;optional n)

返回list中最后n个元素组成的list,默认n=1

```emacs-lisp
(last '(1 2 3 4 5) )                    ;=>(5)
(last '(1 2 3 4 5) 3 )                  ;=>(3 4 5)
```

-   (safe-length list)

对于circular list,只能表示list的长度最大不会超过safe-length返回的值,而不是实际值.

-   (butlast l &amp;optional n)

返回l去掉了后面n个元素后的列表,默认n为1

```emacs-lisp
(butlast '(1 2 3 4) 1)                  ;=>(1 2 3)
(butlast '(1 2 3 4) 2)                  ;=>(1 2)
```

-   (nbutlast l &amp;optional n)

类似(butlast,但是会同时更改l的值


### 创建cons cell和list {#创建cons-cell和list}

-   (cons obj1 obj2)

cons一般用来将一个元素添加到某个list中的头部.

```emacs-lisp
(cons 1 '(2 3 4))                       ;=>(1 2 3 4)
```

-   (list &amp;rest objects)
-   (make-list n object)

返回由n个object组成的list

```emacs-lisp
(make-list 3 2)                         ;=>(2 2 2)
```

-   (append &amp;rest sequences)

将所有的sequences中的元素串在一起组成一个list, 需要注意的是,出了最后一个参数以外,其他的参数都被copy一份,用于与最后那个参数进行连接.

```emacs-lisp
(setq trees '(pine oak))                ;=>(pine oak)
(setq more-trees (append '(maple birch) trees)) ;=>(maple birch pine oak)
(eq trees (last more-trees 2))                  ;=>t
```

通过在最后的参数后添加nil,可以让所有的参数都强制copy

最后的参数在处理上会有些特殊,最后一个参数只是单纯的被安置到前面参数组成list的cdr位置,例如

```emacs-lisp
(append '(x y) [z])                     ;=>(x y . [z]),最后的参数只是放在cdr的位置而已,因此最终结果不是(x y z)
```

-   (reverse list)
-   (copy-tree tree \*optional vecp)
-   (number-sequence from &amp;optional to step)

返回一个number list,值的范围为从from开始到to结束,步进为step(默认为1)

```emacs-lisp
(number-sequence 4 9)                   ;=>(4 5 6 7 8 9)
```

若to为nil或与from相等,则返回单元素列表(from)

若step为0,且to不为nil或与from的值相等,则elisp会报出错误,因为这会产生一个死循环

若step不为0,而from累加step永远不会超过或到达to,则函数返回nil

```emacs-lisp
(number-sequence 8 5 1)                 ;nil
(number-sequence 5 8 -1)                    ;nil
```


### 修改list变量 {#修改list变量}


#### 会破坏原参数中的值 {#会破坏原参数中的值}

这里的函数都会直接修改参数中的list

-   (push element list)

    把element放在list的第一位,作用类似cons

-   (add-to-list listname element &amp;optional append compare-fn)

    类似push,但若list中已经有了element,则保持list不变

    若append为非nil,则将element放在list的最后位置

    compare-fn默认使用equal进行比较

-   (add-to-ordered-list listname element &amp;optional order)

    这里的order需要是number类型,

    order的值决定了element的位置,若order为nil,则将element放在list中最后带order的元素后面
    ```emacs-lisp
    (setq foo nil)
    (add-to-ordered-list 'foo 'a 1)         ;=>(a)
    (add-to-ordered-list 'foo 'c 3)         ;=>(a c)
    (add-to-ordered-list 'foo 'b 2)         ;=>(a b c)
    (add-to-ordered-list 'foo 'b 4)         ;=>(a c b)  若element已经存在,且设定了order,则使用新orde放置element的位置
    (add-to-ordered-list 'foo 'd)           ;=>(a c b d)
    (add-to-ordered-list 'foo 'e)           ;=>(a c b e d)
    (add-to-ordered-list 'foo 'f)           ;=>(a c b f e d)  若order为nil,则将element放在list中最后带order的元素后面

    ```

-   (setcar cons-ceil object) / (setcdr cons-ceil object)

    修改cons-ceil的car/cdr部分,并返回参数object作为返回值

    通过setcdr可以实现删除/添加list中element的目的
    ```emacs-lisp
    (setq x1 '(a b c d))
    (setcdr x1 (cddr x1));=>'(a c d)
    (setcdr x1 (append '(1 2) (cdr x1)))    ;=> (a 1 2 c d)
    ```

-   (nconc &amp;rest lists)

    类似append,所不同的是它直接修改所有参数的last element的cdr,而不会先做copy-sequence操作.

    由于除了最后一个参数不用被修改之外,其他参数的结构都会被修改,因此除了最后一个参数可以是const list外,其他参数必须是个变量.

    跟append一样的,最后一个参数可以不是list
    ```emacs-lisp
    (setq x '(1 2 3))                       ; => (1 2 3)
    (nconc x 'z)                            ; => (1 2 3 . z)
    x                                       ; => (1 2 3 . z)
    ```

-   (nreverse list)

    翻转参数list

-   (sort list predicate-less)

    使用predicate-less进行从小到大的排序,若存在相等的值,则保持相等值的位置不变
    ```emacs-lisp
    (setq x1 '(1 2 4 3 7 6 5));=>(1 2 4 3 7 6 5)
    (sort x1 '<)     ;=>(1 2 3 4 5 6 7)
    ```
    这里的predicate-less为比较函数,它接收两个参数,并判断第一个参数是否小于第二个参数.

    predicate-less函数必须有下面两个特性:

    1.  A&lt;B,则B!&lt;A

    2.  若A&gt;B,B&gt;C,则A&gt;C

    **注意:**

    sort有一个很变态的特性:sort函数会让参数list依然指向原来的哪个cons-cell的位置,而不管这个cons-cell是否在sort后依然是处于第一个元素的位置. 例如
    ```emacs-lisp
    (setq x1 '(9 8 7 2 3))
    (sort x1 '<)                            ;=>(2 3 7 8 9)
    x1                                      ;=>(9),注意,x1实际上依然指向了9这个cons cell的位置
    ```
    **因此一般情况下,都需要把sort的返回结果赋值回参数list**

-   (delq object list)

    移除list中所有与object相等的element,函数中的q表示使用eq作为比较函数.

    由于delq在删除list头部的element时,仅仅是返回跳过头部element的cdr位置,而不会改变list参数所指向的位置.
    ```emacs-lisp
    (setq x1 '(a b c))
    (delq 'a x1)                            ;=>(b c)
    x1                                      ;=>(a b c)
    ```
    **因此使用delq,一般我们也需要将返回值赋值回参数list**

-   (delete object seq)

    delete函数比较特殊,它根据seq的类型不同而有不同的行为.

    当seq为list类型时,它跟delq一样会修改seq的值,所不同的是它使用equal作为比较函数.

    当seq为vector或string,则delete不会修改原seq的值
    ```emacs-lisp
    (setq l '((2) (1) (2)))
    (delete '(2) l)                         ; => ((1))
    l                                       ; => ((2) (1))
    ;; If you want to change `l' reliably,
    ;; write `(setq l (delete '(2) l))'.
    (setq l '((2) (1) (2)))
    (delete '(1) l)                         ; => ((2) (2))
    l                                       ; => ((2) (2))
    ;; In this case, it makes no difference whether you set `l',
    ;; but you should do so for the sake of the other case.
    (setq v [(2) (1) (2)])
    (delete '(2) v)             ; => [(1)]
    v                           ; => [(2) (1) (2)]
    ```

-   (delete-dups list)

    删除list中所有重复的元素,使用equal作为比较函数.

    当list中有多个重复元素时,delete-dups保留第一个元素.


#### 不破坏原参数的值 {#不破坏原参数的值}

-   (remq object list)
    类似delq,但不改变原参数list的值. 这里的q也表示使用eq作为判断函数.

-   (remove object seq)
    类似函数delete,但它保证不修改参数seq的值


### alist相关函数 {#alist相关函数}


#### 获取alist {#获取alist}

-   (copy-alist alist)

拷贝alist的一个副本(two-level deep copy)


#### 根据key取key-value键值对 {#根据key取key-value键值对}

-   (assoc key alist)

当使用assoc在alist中取键值对时,只会取发现的第一个符合条件的键值对. 因此可以直接用push命令将要修改为的新键值对放到alist的前面,以此来模拟对老键值对的修改,同时保留了老键值对的历史.

```lisp
(assoc 'garden *nodes*)
```

assoc使用equal函数作为比较函数

-   (assq key alist)
    类似assoc,但函数名中的q表示使用eq作为比较函数. 一般用于key为symbol类似时,因为eq速度比equal快得多.


#### 根据value查找key-value键值对 {#根据value查找key-value键值对}

-   (rassoc value alist)

    rassoc也使用equal作为比较函数.

-   (rassq value alist)

    类似rassoc,但是函数名中的q标识用eq作为比较函数

-   (assoc-default key alist &amp;optional test default)

    assoc-default与其他assoc系列函数不同之处在于, **它直接返回key部分,而且它也比较不为cons cell的element**.

    若alist中某element为atom,则该element整个被用于与可以进行比较,且使用default作为返回值.
    否则若element为cons cell,则使用(car element)进行比较. 使用(cdr element)作为返回值.
    ```emacs-lisp
    (setq x1 '(0 (1 "one") (2 "two")(3 "three" )))
    (assoc-default 1 x1);=>("one")
      (assoc-default 0 x1 'equal "zero") ;=>"four"
      (assoc-default 4 x1) ;=>nil
    ```
    test为比较函数,默认为equal


#### 更新新值/添加新值 {#更新新值-添加新值}

使用push将新键值对放入alist中即可

```elisp
(push '(old-key new-value) *alist*)
(push '(new-key new-value) *alist*)
```


#### 删除键值对 {#删除键值对}

-   根据key删除
    (assq-delete-all key alist)

    类似delq的alist版,他可能会也可能不会修改原alist的结构,因此 **需要将返回值赋值回原参数alist**

    函数名中的q表示比较时使用eq函数

-   根据value删除
    (rassq-delete-all value alist)

    类似assq-delete-all,但是它比较value而不是key


### Property list相关函数 {#property-list相关函数}

-   判断plist中是否存在指定的property

(plist-member plist property)

返回plist中指向property的位置. 使用eq作为

```emacs-lisp
(plist-member '((1) "one" 2 two) 2)     ;=>(2 two)
```

-   返回plist中匹配参数property的value值
    (plist-get plist property)

    该函数使用eq作为比较函数

<!--listend-->

```emacs-lisp
(plist-get '(foo 4) 'foo)               ; => 4
(plist-get '(foo 4 bad) 'foo)           ; => 4
(plist-get '(foo 4 bad) 'bad)           ; => nil
(plist-get '(foo 4 bad) 'bar)           ; => nil
```

(lax-plist-get plist property)

类似plist-get,但是该函数使用equal来作为比较函数

-   增加/修改plist中的键值对

(plist-put plist peroperty value)

该函数可能/可能不会修改原参数plist的结构.因此你需要把返回值重新赋值回原plist中

```emacs-lisp
(setq my-plist '(bar t foo 4))               ; => (bar t foo 4)
(setq my-plist (plist-put my-plist 'foo 69)) ; => (bar t foo 69)
(setq my-plist (plist-put my-plist 'quux '(a))) ; => (bar t foo 69 quux (a))
```

(lax-plist-put plist property value)
类似plist-put函数,但是使用equal作为比较函数来决定是添加还是更新值


## 环状结构体相关函数 {#环状结构体相关函数}

有时我们会使用一种环状结构体来存储数据,我们可以插入数据到环状结构体中,也可以删除,旋转环状结构中的数据,还可以遍历数据或者根据索引的模值访问数据(在这种环状结构体中,最新插入的数据索引为0,然后从新到就以此累加).

elisp提供了名为\`ring\`的package,供我们方便操作这种环状结构体.

-   (make-ring size)

创建一个大小为size的环状结构

-   (ring-p object)

判断object是否为环状结构体

-   (ring-size ring)

获取该ring上实际包含了多少数据

-   (ring-elements ring)

把ring中的数据,从新到旧,以list的形式返回

-   (ring-emplty-p ring)

判断ring是否为空

-   (ring-ref ring index)

根据索引,查找ring中相应的值.

最新插入的数据索引为0,然后从新到旧依次累加.

索引也可以为负数,-1表示最旧插入的数据,-2表示第二旧的数据,以此类推.

-   (ring-insert ring object)

往ring中插入object,该object作为最新插入的数据,它的索引为0.

若ring中的数据已经满了,则该操作会同时删除最旧的那个数据

该函数的返回值为object

-   (ring-insert-at-beginning ring object)

往ring中插入数据,但该数据被作为最旧的数据项来处理.

若ring中的数据已经满了,则该操作会删除最旧的那个数据.

-   (ring-remove ring &amp;optional index)

除并返回ring中的数据. index若为nil则表示最旧的数据

-   若能够保证不超出ring的容量,则可以使用ring结构体作为先进先出队列来使用.例如

<!--listend-->

```emacs-lisp
(let ((fifo (make-ring 5)))
  (mapc (lambda (obj) (ring-insert fifo obj))
        '(0 one "two"))
  (list (ring-remove fifo) t
        (ring-remove fifo) t
        (ring-remove fifo)))
;; => (0 t one t "two")

```


## Sequences相关函数 {#sequences相关函数}

-   (sequencep object)

判断object是否为sequence

-   (length sequence)

获取sequence中的element个数

需要注意的是length不能对点列表和环形列表求长度,但是可以用safe-length代替

-   (elt sequence index)

返回sequence中index所标示的element(index从0开始标示)

若index的值不为0到(1- (length sequence)),则若参数sequence不是list,则elisp报\`args-out-of-range\`错误

```emacs-lisp
(string (elt "1234" 2))                 ; => "3"
(elt [1 2 3 4] 4)                       ; error--> Args out of range: [1 2 3 4], 4
(elt [1 2 3 4] -1)                      ; error--> Args out of range: [1 2 3 4], -1
```

-   (copy-sequence sequence)

创建sequence的一个新的引用.

这里引用的意思在于新的copy与原sequence共同指向一个结构,即:若对拷贝添加信新的元素,并不会对原sequence进行修改,但若对原element进行修改会同时影响原sequence,反之依然.

```emacs-lisp
(setq y (copy-sequence x))              ; => [foo (1 2)]
(eq x y)                                ; => nil
(equal x y)                             ; => t
(eq (elt x 1) (elt y 1))                ; => t

;; Replacing an element of one sequence.
(aset x 0 'quux)                        ; x => [quux (1 2)] ; y => [foo (1 2)]

;; Modifying the inside of a shared element.
(setcar (aref x 1) 69)                  ; x => [quux (69 2)] ; y => [foo (69 2)]

```

copy-sequence不能用于点列表和环形列表,可以用copy-tree函数拷贝点列表,但无法复制环形列表

-   (append aVector nil)
    append函数提供了一种将sequence转换为list的方法
    ```emacs-lisp
    (setq avector [1 two (quote (three)) "four" [five]]) ; => [1 two (quote (three)) "four" [five]]
    (append avector nil)                                 ; => (1 two (quote (three)) "four" [five])

    ```


### Array相关函数 {#array相关函数}

-   (arrayp object)

判断object是否为array

-   (aref array index)

获取array中index所指的element的引用

-   (aset array index object)

设置array中第index的元素为object,该函数返回object

-   (fillarray array object)

将array中所有元素都填充为object

```emacs-lisp
(setq a [a b c d e f g])                ; => [a b c d e f g]
(fillarray a 0)                         ; => [0 0 0 0 0 0 0]
a                                       ; => [0 0 0 0 0 0 0]
(setq s "When in the course")           ; => "When in the course"
(fillarray s ?-)                        ; => "------------------"
```


#### Vector相关函数 {#vector相关函数}

-   (vectorp object)

    判断object是否为vector

-   (vector &amp;rest objects)

    创建由objects组成的vector

-   (make-vector N object)

    创建由N个object组成的vector

-   (vconcat &amp;rest sequences)

    将sequences中的element,转换到vector中
    ```emacs-lisp
    (setq a (vconcat '(A B C) '(D E F)))    ; => [A B C D E F]
    (eq a (vconcat a))                      ; => nil
    (vconcat)                               ; => []
    (vconcat [A B C] "aa" '(foo (6 7)))     ; => [A B C 97 97 foo (6 7)]
    ```


#### Char-Table相关函数 {#char-table相关函数}

-   (char-table-p object)
    判断object是否为char-table类型的

-   (make-char-table SUBTYPE &amp;optional init)
    创建一个新char-table对象,该char-table的subtype为参数SUBTYPE(必须为symbol类型). 该char-table的所有element初始化为参数init(默认为nil).

    一旦创建了char-table,就不能再修改其subtype了.

-   (char-table-subtype char-type)
    返回char-table的subtype

-   (char-table-parent char-table)
    获得char-table的父级char-table对象,若没有,则返回nil

    char-table从父级char-table中继承值

-   (set-char-table-parent char-table new-parent-char-table)
    为char-table设置父级char-table

-   (char-table-extra-slot char-table n)
    返回char-table中第n个slot上的值

    一个char-table有多少个slot,由它的subtype的属性\`char-table-extra-slots\`决定

-   (set-char-table-extra-slot char-table n value)
    设置char-table的第n个slot的值为value

-   (char-table-range char-table range)
    该函数返回char-table中某个由参数range指定的范围内的值

    range参数可以是:

    -   nil

    获取默认值

    -   char

    获取char对应的值

    -   '(from . to)

    获取[from,to]这个范围内char的相应值

-   (set-char-table-range char-table range value)
    设置char-table中由range所指范围的对应值.

    这里的参数range可能是:

    -   nil

    设置char-table的默认值为value

    -   t

    设置整个char-table的值为value

    -   char

    设置char对应的值为value

    -   (from. to)

    设置范围[from,to]之间的值为value

-   (map-char-table function char-table)
    针对char-table中所有值非nil的键值对都作为参数调用function函数.

    该function函数必须接收两个参数(一个key,一个value).
    其中key的类型可以是某个char,或这类似'(from . to)这样的标示范围的cons ceil(不是很明白什么时候会用到(from.to)这样类型的参数呢??)
    而value的值为(char-table-range char-table key)的返回值

    map-char-table的返回值必定为nil,因此我们通常只使用function的副作用.
    ```emacs-lisp
    (let (accumulator)
      (map-char-table
       #'(lambda (key value)
           (setq accumulator
                 (cons (list
                        (if (consp key)
                            (list (car key) (cdr key))
                          key)
                        value)
                       accumulator)))
       (syntax-table))
      accumulator)
    ;; =>
    ;; (((2597602 4194303) (2)) ((2597523 2597601) (3))
    ;;  ... (65379 (5 . 65378)) (65378 (4 . 65379)) (65377 (1))
    ;;  ... (12 (0)) (11 (3)) (10 (12)) (9 (0)) ((0 8) (3)))


    ```


#### Bool-vector相关函数 {#bool-vector相关函数}

-   创建bool-vector
    (make-bool-vector length initial)

    创建长度为length的bool-vector,每个值初始化为initial
    ```emacs-lisp
    (setq a (make-bool-vector 10 nil))      ;=>#&10"  "

    ```

-   类型判断

    (bool-vector-p object)

    object是否为为bool-vector类型

-   集合运算

    (bool-vector-exclusive-or a b &amp;optional c)

    求a和b的异或计算结果,若有参数c,则将结果存入c中.
    所有参数都都必须为bool vector类型且具有相同的长度

    (bool-vector-union a b &amp;optional c)

    求a&amp;b的计算结果,若有参数c,则将结果存入c中.
    所有参数都都必须为bool vector类型且具有相同的长度

    (bool-vector-intersection a b &amp;optional c)

    求a|b的运算结果,若有参数c,则将结果存入c中.
    所有参数都都必须为bool vector类型且具有相同的长度

    (bool-vector-set-difference a b &amp;optional c)

    求a-b的运算结果,若有参数c,则将结果存入c中.
    所有参数都都必须为bool vector类型且具有相同的长度

    (bool-vector-not a &amp;optional b)

    求!a的运算结果,若有参数b,则将结果存入b中.
    所有参数都都必须为bool vector类型且具有相同的长度

    (bool-vector-subsetp a b)

    判断a是否为b的子集, 所谓a是b的子集指的是所有a中值为t的位置,在b中也为t.
    所有参数都都必须为bool vector类型且具有相同的长度

    (bool-vector-count-consecutive bool-vector a index)

    统计bool-vector中从index开始,连续值等于a的个数

    (bool-vector-count-population bool-vector)

    统计bool-vector中值为t的个数

    (aref bool-vector index)

    要修改/获取bool-vector的值,需要使用array的相关函数来进行:

    获取bool-vector中index的值

    (aset bool-vecotr index value)

    设置bool-vector在index位置的value

    下面是一些例子
    ```emacs-lisp
    (setq bv (make-bool-vector 5 t))        ; => #&5"^_"
    (aref bv 1)                             ; => t
    (aset bv 3 nil)                         ; => nil
    bv                                      ; => #&5"^W"
    ```


## HashTable相关函数 {#hashtable相关函数}


### 判断函数 {#判断函数}

-   (hash-table-p table)

判断table是否为hash-table


### 创建hash-table {#创建hash-table}

(make-hash-table &amp;rest keyword-args)

常用的keyword-args有:

-   :test TEST

    指定了用于测试相等的函数. 默认使用eq

-   :weakness WEAK

    指定了hash-table什么时候被垃圾回收机制回收. WEAK的可选参数为:

    -   nil(默认值)

    表示key和value都不是弱引用,hash-table会保证key和value不会被垃圾回收机制回收

    -   key

    表示key为弱引用,即若除了hash-table其他地方没有引用key的变量,则key变量所指的内存块被回收. 该key-value键值对从hash-table中被删除

    -   value

    表示value为弱引用,即若除了hash-table其他地方没有引用value的变量,则value变量所指的内存块被回收. 该key-value键值对从hash-table中被删除

    -   key-or-value

    表示若除了hash-table其他地方 **同时** 没有引用key或value的变量,则key和value变量所指的内存块被回收. 该key-value键值对从hash-table中被删除

    -   key-and-value / t

    表示key和value都为弱引用,即若除了hash-table其他地方没有引用key或value的变量,则key或value变量所指的内存块被回收. 该key-value键值对从hash-table中被删除

-   :size SIZE
    指示大致上会有SIZE个数据存入hash-table,用于优化初始容量,默认为65

-   :rehash-size REHASH-SIZE
    指示当hash-table容量爆满后,怎么进行扩容.

    若REHASH-SIZE为正整数,则每次扩容都增加REHASH-SIZE个容量

    若REHASH-SIZE为正浮点数,则每次扩容都按照REHASH-SIZE的倍数来调整容量(因此REHASH-SIZE需要&gt;1)

    REHASH-SIZE默认为1.5

-   :rehash-threshold THRESHOLD
    该参数指明了什么时候hash-table进行扩容. 默认为0.8

    THRESHOLD是一个不大于1的浮点数, 当hash-table中的元素个数&gt;THRESHOLD乘与hash-table容量时,进行扩容

    (copy-hash-table table)

    创建table的副本,但 **它的key和value与原table共享**


### 添加item / 修改item的值 {#添加item-修改item的值}

(puthash myKey myVal myHash)


### 删除item {#删除item}

(remhash myKey myHash)

删除myHash中索引为myKey的键值对. 该函数总是返回nil

(clrhash myHash)

清空myHash中的所有内容,该函数总是返回nil


### 获取某item的值 {#获取某item的值}

(gethash myKey myHash &amp;optional default)

若没key为myKey的item,则返回default,默认为nil


### 获取hash中的属性 {#获取hash中的属性}

-   获取hash中的key-value键值对个数
    (hash-table-count myHash)

-   获取hash中的查询机制(即:test属性的值)
    (hash-table-test myHashTable)

-   获取hash-table中:weak属性的值

(hash-table-weakness myHash)

-   获取hash-table中:rehash-size参数的值

(hash-table-rehash-size table)

-   获取hash-table中:rehash-threshold参数的值

(hash-table-rehash-threshold table)

-   获取hash-table中的:size参数的值

(hash-table-size table)


### 为hash-map中的所有键值对调用函数处理 {#为hash-map中的所有键值对调用函数处理}

(maphash myFunc myHash)

myFunc接收两个参数,一个key,一个value.该函数总是返回nil


### 获取hash-map中的所有key值 / value值 {#获取hash-map中的所有key值-value值}

-   在emacs24.4之后,可以使用

<!--listend-->

```elisp
;; get all keys
(require 'subr-x)
(hash-table-keys myHash) ;
(hash-table-values myHash) ;
```

-   在emacs24.3可以自定义函数

<!--listend-->

```elisp
(defun get-hash-keys (hashtable)
"Return all keys in hashtable."
(let (allkeys)
  (maphash (lambda (kk vv) (setq allkeys (cons kk allkeys))) hashtable)
  allkeys
)
)

(defun get-hash-values (hashtable)
"Return all values in HASHTABLE."
(let (allvals)
  (maphash (lambda (kk vv) (setq allvals (cons vv allvals))) hashtable)
  allvals
)
)
```


### 修改Hash-table的比较方法 {#修改hash-table的比较方法}

要修改Hash-table中的查询机制,需要同时修改计算Hash Code的方法和比较key值的方法.

-   (define-hash-table-test name test-fn hash-fn)

定义一个名为name的hash-table查询机制.

当定义了查询机制后,该查询机制就可以传给make-hash-table中的:test参数用于新生成的hash-table了.

test-fn需要接收两个key作为参数,并在认为两个key相等时返回非nil

hash-fn则需要接收一个key作为参数,并返回一个整数(可以为负数)作为它的hash值.

elisp提供了一个函数用于根据object的内容来生成hash值:sxhash

```emacs-lisp
(defun case-fold-string= (a b)
  (eq t (compare-strings a nil nil b nil nil t)))
(defun case-fold-string-hash (a)
  (sxhash (upcase a)))

(define-hash-table-test 'case-fold
  'case-fold-string= 'case-fold-string-hash)

(make-hash-table :test 'case-fold)
```

-   (sxhash obj)

根据obj的内容生成hash code,若两个obj是equal的,则该函数返回相等的hashcode

```emacs-lisp
(define-hash-table-test 'contents-hash 'equal 'sxhash)

(make-hash-table :test 'contents-hash)

```


## Symbol相关函数 {#symbol相关函数}


### symbol组成部分 {#symbol组成部分}

-   (symbol-name symbol)

获取symbol的名称

```emacs-lisp
(symbol-name 'foo)
```

-   (symbol-function symbol)

获取symbol的函数cell

-   (indirect-function symbol-or-function &amp;optional noerror)

类似symbol-function,但若symbol的function cell为另一个symbol,则它会返回(indirect-function 另一个symbol)的值

参数symbol-or-function表示它也可以是function类型的,若类型为function,则直接返回该function参数

-


-   (fset symbol definition)

设置symbol的函数cell为definition


### 获取symbol {#获取symbol}

-   (make-symbol name)

创建uninterned symbol

```emacs-lisp
(setq sym (make-symbol "foo"))          ; => foo
(eq sym 'foo)                           ; => nil ,uninterned symbol和interned symbol是不一样的
```

-   (intern name &amp;optoinal obarray)

返回obarray中名为name的symbol,若obarray中不存在名为name的symbol,则新建一个symbol.

obarray默认为全局变量\`obarray\`

参数name必须是字符串类型

```emacs-lisp
(setq sym (intern "foo"))               ; => foo
(eq sym 'foo)                           ; => t
(setq sym1 (intern "foo" other-obarray)) ; => foo
(eq sym1 'foo)                           ; => nil
```

-   (intern-soft name &amp;optional obarray)

类似intern,但若obarray中不存在名为name的symbol,则返回nil

且参数name可以为symbol类型和字符串类型

```emacs-lisp
(intern-soft "frazzle")                 ; No such symbol exists. => nil
(make-symbol "frazzle")                 ; Create an uninterned one. => frazzle
(intern-soft "frazzle")                 ; That one cannot be found. => nil
(setq sym (intern "frazzle"))           ; Create an interned one. => frazzle
(intern-soft "frazzle")                 ; That one can be found! => frazzle
(eq sym 'frazzle)                       ; And it is the same one. => t
```


### 其他函数 {#其他函数}

-   (mapatoms func &amp;optional obarray)

对obarray中包含的每个symbol,都调用func来处理,然后返回nil.

obarray默认为全局的obrray变量

```emacs-lisp
(setq count 0)

(defun count-syms (s)
  (setq count (1+ count)))

(mapatoms 'count-syms)                  ; => nil
count                                   ; => 54972
```

-   (unintern symbol-or-string obarray)

从obarray中删除指定的symbol

删除成功返回t,否则返回nil


### symbol中的property {#symbol中的property}

-   获取symbol的property

(get symbol property)

从symbol的property list中出去出与参数property eq的值,若没有找到,则返回nil

(symbol-plist symbol)

返回symbol相应的property list

(function-get symbol property)
该函数与get类似,但若参数symbol为另一个函数的别名时,该函数返回实际函数的property的值,而不是别名的property的值

-   设置symbol的property

(put symbol property value)

设置symbol中property list的相应property的值.

若原property不存在则添加新property和value,否则更新其value

```emacs-lisp
(put 'fly 'verb 'transitive)            ; =>'transitive
(put 'fly 'noun '(a buzzing little bug)) ; => (a buzzing little bug)
(get 'fly 'verb)                         ; => transitive
(symbol-plist 'fly)                      ; => (verb transitive noun (a buzzing little bug))
```

(setplist symbol plist)

覆盖symbol的原property list为参数plist

该函数的返回值为参数plist

```emacs-lisp
(setplist 'foo '(a 1 b (2 3) c nil))    ; => (a 1 b (2 3) c nil)
(symbol-plist 'foo)                     ; => (a 1 b (2 3) c nil)
```


### 标准symbol property说明 {#标准symbol-property说明}

-   :advertised-binding

symbol所表示的函数的建议绑定键

-   char-table-extra-slots

若symbol所表示的值为char-table类型,则该值指明了能有多少个额外slot

-   customized-face,face-defface-spc,saved-face,theme-face

定义了face的方方面面的性质,不要去直接修改这些属性,而应该使用deffac及相关函数

-   customized-value,save-vlaue,standard-value,theme-value

这些属性用来记录customizable variable的方方面面的属性,不要去直接修改这些属性,而应该使用defcustom及相关函数

-   disabled

若为非nil,则表示该函数不为command

-   face-documentation

指定face的说明,该属性由defface自动设置

-   history-length

指明了minibuffer中,对指定的history list variable的最大历史存储数量

-   interactive-form

symbol表示函数的interactive form. 不要直接修改该属性,使用定义函数时的interactive特殊form

-   menu-enable

该属性是一个表达式,根据该表达式的值来决定是否在menu中显示该菜单项

-   mode-class

若该属性为\`special\`,则指定的major mode是\`special\`的

-   permanent-local

若该属性值为非nil,则表示symbol表示的变量是buffer-local的变量, 当更换major mode时,这种变量的值不会被重置

-   permanent-local-hook

若该属性值为非nil,则表示当鞥该major mode时,该symbol表示的hook变量不被重置

-   pure

该属性值为非nil告诉编译器,该symbol所表示的方法为纯函数方法(即没有副作用). 因此在使用const参数来调用该方法时,在编译期就能执行该方法. 这会让一些本该在运行期报的错误,在编译器就被检查出来.

-   risky-local-variable

若该属性值为非nil,表示该symbol表示的变量,不建议设置为file-local variable

-   safe-function

标示该symbol标示的函数,是否可以安全的执行

-   safe-local-eval-function

标示该symbol标示的函数,是否可以安全的在file-local evaluation form中执行

-   safe-local-variable

该属性值应该是一个函数,该函数用来决定该symbol表示的变量是否为safe file-local的

-   side-effect-free

非nil表示symbol表示的函数无副作用,该属性用来决定函数的安全性和字节编译优化, **不要尝试手工修改它**

-   variable-documentation

该属性为指定变量的说明


## Region/Mark相关函数 {#region-mark相关函数}

-   设置mark

(set-mark-command)

-   删除region

(kill-region)

-   注释region

(comment-region)

-   重新格式化region

(fill-region)

-   缩进region

(indent-region)

-   判断Region是否是active的

(region-active-p) 该函数还要求Transient-mark-mode


## Evaluation {#evaluation}


### \`(反引号) {#反引号}

\`类似', 但当object前带了\`,'时则会对该object进行求值, 当object前带了\`,@\`,则会将object的求值结果中的各个元素打散插入.

对于,@的使用，还另有限制:

1.  为了确保其参数可以被拼接，,@必须出现在序列(sequence)Ʈ中。形如\`,@b的说法是错误的,因为无处可供b的值进行拼接。

2.  要进行拼接的对象必须是个列表，除非它出现在列表最后。

    表达式 `` `(a ,@1) `` 将被求值成 `(a . 1)`.

    但如果尝试将原子拼接到列表的中间位置,例如 `` `(a ,@1 b) ``,将导致一个错误.

3.  反引用并不是只能使用在.你可以在任何需要构造序列的场合使用反引用.


### 函数 {#函数}

-   (eval form &amp;optional lexical)

在当前环境下执行form

参数lexical决定了form中的本地变量采用的是静态作用域还是动态作用域.

-   nil 表示使用动态作用域

-   t 表示使用静态作用域

-   某个非空的alist 表示alist中所指定变量采用静态作用域,其他变量为动态作用域

-   (eval-region start end &amp;optional stream read-function)

运行由start和end所标识的region

默认情况下,eval-region并不产生任何输出,但通过传递一个非nil的stream参数,可以将期间产生的输出输出到stream中

若read-function为非nil,则使用指定的read-function来取代read函数读取表达式. 该函数需要接收一个参数:读取输入的stream

-   (eval-buffer &amp;optioanl buffer-or-name stream filename unibyte print)

运行指定buffer的可见部分所组成的region.

buffer-or-name可以为buffer或string,也可以为nil表示当前buffer

stream的作用跟eval-region中的stream类似,但若stream为nil而print为非nil,则执行的结果依然会被丢弃,但执行的过程中所产生的哪些输出会被输出到echo area中.

filename是給load-history使用的文件名称.

若inibyte为非nil,则elisp reader尽可能的将string转换为unibyte格式.


### 选项 {#选项}

-   max-lisp-eval-depth

-   max-specpdl-size

-   values

该变量的值为一个list,每个list元素都是执行form的结果,最近的结果放到第一位.

```emacs-lisp
(setq x 1)                              ; => 1
(list 'A (1+ 2) auto-save-default)      ; => (A 3 t)
values                                  ; => ((A 3 t) 1 ...)
```


## 缓冲区 {#缓冲区}


### 获取buffer对象 {#获取buffer对象}

-   得到当前buffer对象

    current-buffer(**当前buffer不一定是在屏幕上显示的哪个缓冲区,另外,当命令执行完成后,光标所在的buffer自动成为当前buffer** )

-   若缓冲区存在则返回该缓冲区对象,否则新建缓冲池对象返回

    get-buffer-create

-   若有同名缓冲区存在,则新建的缓冲区后会加上后缀

    generate-new-buffer

-   获得所有buffer的列表

    buffer-list

-   获得窗口对应的buffer

    window-buffer


### buffer操作 {#buffer操作}

-   返回当前buffer的文件全路径

    buffer-file-name

-   获得指定/当前buffer的名字

    buffer-name

<!--listend-->

```elisp
(buffer-name [buffer对象])
```

-   重命名缓冲区

    rename-buffer

-   产生一个唯一的缓冲区名
    generate-new-buffer-name

-   设置指定buffer为当前buffer

    (set-buffer myBuffer)

-   保存当前buffer到文件

    save-buffer

-   在不改变当前状态下,临时用另一buffer的变量代替现有变量执行语句

    with-current-buffer

<!--listend-->

```elisp
(with-current-buffer buffer对象或buffer名称
  表达式)
```

-   关闭缓冲区

    (kill-buffer myBuffer) ;如果要用户确认是否要关闭缓冲区，可以加到kill-buffer-query-functions里。如果要作善后处理，可以用kill-buffer-hook

-   关闭当前buffer

    (kill-this-buffer)

-   确认缓冲区是否存在

    buffer-live-p

-   使用临时buffer执行相应代码

    with-temp-buffer

<!--listend-->

```elisp
;; use a temp buffer to manipulate string
(with-temp-buffer
  (insert myStr)
  ;; manipulate the string here
  (buffer-string) ; get result
)
```

-   创建新的空标记

    make-marker

<!--listend-->

```elisp
(make-marker)
```

-   设置标记的位置和缓冲区

    set-marker

<!--listend-->

```elisp
(set-marker foo (point))
```

-   得到point处的标记

    point-marker

<!--listend-->

```elisp
(point-marker)
```

-   复制标记或直接用位置生成一个标记

    copy-marker

<!--listend-->

```elisp
(copy-marker 位置/marker对象)
```

-   得到一个marker的内容

    maker-position / marker-buffer

<!--listend-->

```elisp
(maker-position marker对象)
(marker-buffer marker对象)
```

-   buffer大小

    buffer-size

-   mark-marker

    point /point-max /point-min

返回当前缓冲区的mark(**注意mark与marker的区别,mark是用来与point一起定义一个region的,而marker是一个标记位置**)

-   设置mark的值,并激活mark

    set-mark

-   加入/删除mark-ring的元素

    push-mark / pop-mark

-   取得region的起点和终点

    region-beginning / region-end

-   让某个缓冲区可见

    display-buffer

-   判断buffer是否被修改

(buffer-modified-p)

-   选中的window切换到上一个/下一个buffer

(previous-buffer)

(next-buffer)

-


### 获取缓冲区内容 {#获取缓冲区内容}

-   得到整个缓冲区的文本

    buffer-string
-   得到buffer某个区间的文本

    buffer-substring
-   得到point附件的字符

    char-after / char-before
-   point处的词

    current-word
-   得到point处的其他类型的文本

    thing-at-point


### buffer内容处理相关函数 {#buffer内容处理相关函数}


#### 删除操作 {#删除操作}

-   删除从当前光标开始的N个字符

(delete-char N)

-   删除光标前的N个字符

    (delete-backward-char N)

    -   删除region

    (delete-region sartPos endPos)

    -   清空整个buffer

    (erase-buffer)


#### 插入操作 {#插入操作}

-   在光标处插入文字

(insert str)

-   在光标处插入某buffer的一部分文本

(insert-buffer-substring-no-properties myBuffer myStartPos myEndPos)

-   插入文件中某部分到当前缓冲区中

    (insert-file-contents myPath)
    ```elisp
    (insert-file-contents filename &optional visit beg end replace)
    ```
    如果指定visit则会标记缓冲区的修改状态并关联缓冲区到文件，一般是不用的。
    replace是指是否要删除缓冲区里其它内容，这比先删除缓冲区其它内容后插入文件内容要快一些，但是一般也用不上。
    insert-file-contents会处理文件的编码，如果不需要解码文件的话，可以用insert-file-contents-literally。


#### 查找/替换操作 {#查找-替换操作}

-   改变大小写

(capitalize-region startPos endPos)

-   替换操作

变量\`case-fold-search\`决定是否大小写敏感

replace-match,需要与其他的search类函数配合,它替代上次search匹配的文本

(replace-match 字符串) 表示用字符串替代上次search匹配的文本呢

-   获取上次正则查询的分组内容

    (match-string N) 返回上次正则查询的第N个分组的内容

-   获取上次正则查询分组的起始/结束电

    (match-beginning N)

    (match-end N)

-


### 保存现场 {#保存现场}

-   保存当前buffer,执行其中的表达式,然后回复为原来的buffer

    save-current-buffer

<!--listend-->

```elisp
(save-current-buffer
  表达式)
```

-   保存narrow-to-region

(save-restriction
     (narrow-to-region pos1 pos2)
     lisp代码)

-   保存buffer状态

(save-excursion
    reset body
      )


## Window {#window}


### 基本概念 {#基本概念}


#### live window {#live-window}

有buffer显示的window被称为live window, 可以通过 `(window-livep object)` 来判断


#### valid window {#valid-window}

valid window可能是live window或internal window. 要注意它与live window的区别.
一个valid window可能被删除,则变成invalid window,但它仍可能被其他lisp对象所引用.
并且一个被删除的window可能通过恢复之前报错的window configuration变回valid window

使用 `(window-valid-p object)` 来判断是否为valid window


#### selected-window {#selected-window}

任何时候,不管有多少个frame,只有唯一一个selected window

一般来说,selected window的buffer就是"current buffer",但有一种情况例外,就是使用 `set-buffer` 之后.


### window与frame的关系 {#window与frame的关系}

一个window只可能属于一个frame.

-   (window-frame &amp;optional window)

    该函数获取指定window所属的frame

-   (window-list &amp;optional frame minibuffer window)

    该函数获取指定frame的所有 **live window** 列表.

    MINIBUFFER参数指定了返回的live window列表是否包含minibuffer window.

    -   **t:** 包含minbuffer window

    -   **nil:** 当minibuffer被激活时才包含

    -   **其他:** 不包含

    WINDOW参数指定了live window列表中的第一个window,它应该是指定frame中的一个live window.

    若未指定WINDOW参数,则selected window作为列表的第一个window

window在每个frame中都是以window tree的形式被组织起来的.

```text
______________________________________
| ______ ____________________________ |
||      || __________________________ ||
||      |||                          |||
||      |||                          |||
||      |||                          |||
||      |||____________W4____________|||
||      || __________________________ ||
||      |||                          |||
||      |||                          |||
||      |||____________W5____________|||
||__W2__||_____________W3_____________ |
|__________________W1__________________|
```

每个window tree的叶节点都是live window组成的.
window tree的中间节点则是由internal window组成,即哪些不显示buffer的window
internal window存在的目的是为了组织live window之间的关系.
window tree的root节点被称为root window. 它可以是live window也可以是internal window

**一般来说,window tree并不包含minibuffer window,除非整个frame只有这个minibuffer window**

当分割一个window后,分割出来的两个live window中有一个 **就是之前被分割的那个window对象**.
emacs会新建两个window对象:一个是分割出来的另一个live window,还有一个是internel window作为分割出两个live window的父window

每个internal window最少具有两个子窗口,若某internal window的子窗口数降为1,则Emacs自动删除该internal window.


### 获得窗口对象 {#获得窗口对象}

-   (frame-root-window &amp;optional frame-or-window)

    该函数返回FRAME-OR-WINDOW的root window

    参数FRAME-OR-WINDOW若为nil则表示返回当前选中frame的root window

-   (window-parent &amp;optional window)

    WINDOW的父window, 默认为选中窗口的父window

-   (window-top-child &amp;optional window)

    返回指定WINDOW的最上方的子window

    当然WINDOW必须是是internal window且其子窗口应是垂直组合的,否则该函数返回nil

-   (window-left-child \*optional window)

    返回指定WINDOW的最左方的子window

    当然WINDOW必须是是internal window且其子窗口应是水平组合的,否则该函数返回nil

-   (window-child window)

    该函数返回指定WINDOW的第一个子window.

    该函数自动判断WINDOW中子window的排列方式,并返回最上方会最左方的子window

    WINDOW必须是internal window,否则返回nil

-   (window-combined-p &amp;optional window horizontal)

    判断WINDOW是否与其他WINDOW垂直/水平排列.

    参数HORIZONTAL为nil表示判断是否垂直排列,否则判断是否水平排列

-   (window-next-sibling &amp;optional window)

    返回WINDOW的下一个兄弟window

-   (window-prev-sibling &amp;optional window)

    返回WINDOW的上一个兄弟window

-   (frame-first-window &amp;optional frame-or-window)

    返回指定FRAME中的最最上方的live window

-   (window-in-direction direction &amp;optional window ignore sign wrap mini)

    返回与WINDOW在DIRECATION方向上相邻的live window

    参数DIRECTION可能是above,below,left,right

    参数WINDOW必须是live window,且默认为选中的window

    一般情况下, **该函数会跳过那些参数\`no-other-window'为非nil的window**,但若参数IGNORE为非nil,则该函数不跳过

    If the optional argument sign is a negative number, it means to use the right or bottom edge ofwindowas reference position instead ofwindow-point. If sign is a positive number, it means to use the left or top edge ofwindowas reference position.

    参数WRAP若为非nil,则表示允许越过frame边界,例如最右边的右边调到了最左边.

    参数mini指定了什么情况下返回minibuffer window,且若WRAP为非nil,则该函数只有在minibuffer被激活状态才返回minibuffer window

-   (window-tree &amp;optional frame)

    返回指定frame的window-tree

-   得到当前光标所在的窗口对象

    selected-window
-   得到当前frame里的所有窗口

    window-list
-   window-lists里排在某个window之后/之前的窗口对象

    next-window / previous-window
-   查找符合某个条件的窗口

    get-window-with-predicate
-   根据buffer获得window(如果有多个窗口显示同一个缓冲区,那么函数由window-list决定返回哪个).

    get-buffer-window
-   根据buffer获得全部的相应window

    get-buffer-window-list
-   根据给定的文件名,返回缓冲区

    find-buffer-visiting


### 窗口操作 {#窗口操作}

-   分割window

    split-window
-   删除当前选中的窗口

    delete-window
-   删除其他窗口

    delete-other-windows
-   得到当前窗口配置信息,可以用setq保存起来

    current-window-configuration
-   设置当前窗口配置信息

    set-window-configuration
-   使某个窗口对象变成选中的窗口

    select-window
-   执行的语句结束后,选择的窗口仍留在执行语句之前的窗口

    save-selected-window / with-selected-window
-   遍历窗口操作

    walk-windows
-   让某个窗口显示某个缓冲区

    set-window-buffer
-   让选中的窗口显示某个缓冲区

    switch-to-buffer


### 窗口信息 {#窗口信息}

-   得到当前窗口的结构

    window-tree
-   判断窗口对象是否存在

    window-live-p
-   获得窗口的高度(包括了mode line和 header line)

    window-height
-   获得窗口的高度(排除了mode line和 header line)

    window-body-height
-   窗口的宽度,不包括滚动条和边缘

    window-width
-   返回各顶点的坐标信息(包括滚动条,边缘,mode line ,header line)

    window-edges
-   返回窗口的文本区域的坐标信息

    window-inside-edges
-   用像素表示的window位置

    window-pixel-edges / window-inside-pixel-edges


## 文件 {#文件}


### 文件读写 {#文件读写}

-   打开一个文件

    (find-file myPath)

-   改变缓冲区关联的文件

    set-visited-file-name

-   保存当前文件

    (save-buffer)

-   另存为文件

    (write-file myPath)

-   把文本块追加到文件后

(append-to-file startPos endPos filePath)

-   把缓冲区当中的一部分写入到指定文件中

    wirte-region
    ```elisp
    (write-region start end filename &optional append visit lockname mustbenew)
    ```
    如果指定append则是添加到文件末尾。
    visit参数也会把缓冲区和文件关联，
    lockname 则是文件锁定的名字
    mustbenew(保文件存在时会要求用户确认操作。


### 文件信息 {#文件信息}

-   判断文件是否存在,对于目录和一般文件都能用这个函数进行判断,但是符号链接只有当目标文件存在时才返回t

    file-exists-p
-   文件属性判断

    file-readable-p / file-writable-p / file-executable-p / file-modes
-   判断文件类型是普通文件/目录/符号链接,其中file-symlink-p返回目标文件名

    file-regular-p / file-directory-p / file-symlink-p
-   文件的详细信息

    file-attributes
-   设置文件修改时间

    set-file-times
-   设置文件位模式

    set-file-modes

-   除去链接后的真实文件名

    file-truename


### 文件名相关函数 {#文件名相关函数}

-   分解文件路径各部分

    file-name-directory / file-name-nodirectory / file-name-sans-extension / file-name-extension / file-name-sans-versions
    ```elisp
    (file-name-directory "~/temp/test.txt") ; => "~/temp/"
    (file-name-nondirectory "~/temp/test.txt") ; => "test.txt"
    (file-name-sans-extension "~/temp/test.txt") ; => "~/temp/test"
    (file-name-extension "~/temp/test.txt") ; => "txt"
    (file-name-sans-versions "~/temp/test.txt~") ; => "~/temp/test.txt"
    (file-name-sans-versions "~/temp/test.txt.~1~") ; => "~/temp/test.txt"
    ```

-   测试一个路径是否是绝对路径

    file-name-absolute-p

-   得到绝对路径

    (expand-file-name myFilePath)

-   把绝对路径转换成相对路径

    (file-relative-name myFilePath dirPath)

-   把路径转换为目录形式,也就是确保它是以路径分隔符结束的

    file-name-as-directory
    ```elisp
    (file-name-as-directory "~rms/lewis") ; => "~rms/lewis/"
    ```

-   获得目录名

    directory-file-name
    ```elisp
    (directory-file-name "~lewis/") ; => "~lewis"
    ```
-   得到所在系统使用的文件名

    convert-standard-filename
    ```elisp
    (convert-standard-filename "c:/windows") ;=> "c:\\windows"
    ```
-   得到某个目录的全部或者符合某个正则表达式的文件名,directory-files-attributes返回的列表包含了file-attributes得到的信息

    directory-files / directory-files-and-attributes
-   得到某个文件在目录中的所有版本

    file-name-all-versions
-   得到通配符扩展厚的文件列表

    file-expand-wildcards


### 文件操作 {#文件操作}

-   重命名 拷贝 删除文件

    (rename-file fileName newName)

    (copy-file sourcName desName)

    (delete-file fileName)

(copy-directory dirPath newDirPath)

-   删除目录

    (delete-directory dirPath 是否循环删除子目录的标记 是否放入Trash的标记)

-   设置文件MODE

(set-file-mode FILE MODE)

-   获取目录中的文件列表

(directory-files DIR &amp;optional FULL MATCH NOSORT)

-   创建目录

(make-dirctory DIR &amp;optional PARENTS)

-


### 临时文件 {#临时文件}

-   这个函数按给定前缀产生一个不和现有文件冲突的文件，并返回它的文件名。如果给定的名字是一个相对文件名，则产生的文件名会用temporary-file-directory 进行扩展。也可以用这个函数产生一个临时文件夹。

    make-temp-file
    ```elisp
    (make-temp-file "foo") ; => "/tmp/foo5611dxf"
    ```
-   产生一个不存在的文件名

    make-temp-name
    ```elisp
    (make-temp-name "foo") ; => "foo5611q7l"
    ```


### 神奇的handler {#神奇的handler}

-   在Emacs里，底层的文件操作函数都可以托管给elisp中的函数，这样只要用elisp实现了某种类型文件的基本操作，就能像编辑本地文件一样编辑其它类型文件了


## Processes {#processes}

Emacs可以同步或异步的方式创建子进程,并以process对象的形式表现出来.

除了管理Emacs的子进程之外,Emacs还可以管理操作系统中的其他非Emacs子进程.

-   (processp object)

判断object是否为Emacs的子进程


### 创建子进程 {#创建子进程}

可以使用\`start-process'创建异步进程,并获得process object. 也可以使用\`call-process'和\`call-process-region'创建同步进程.

Emacs创建的子进程继承了Emacs的运行环境,但可以使用\`process-environment'覆盖默认的运行环境.

参数program表示要执行的程序名称. 需要注意: **program应该只包含要执行的程序名称,而不能包括参数**

参数buffer-or-name指定了程序的输出显示在哪个buffer中,nil表示丢弃输出. unless a custom filter function handles it. 对于同步进程,可以指定输出到文件中而不是buffer

参数args表示传递到程序的参数,所有的args都是string类型的. 在这些string中, **扩展字符和其他shell特殊字符并不会经过shell展开**,因为参数是直接传递给指定程序的.

-   配置项exec-suffixes

一个包含后缀字符串的list. 当搜索可执行程序时,会添加该列表中的后缀到执行程序名称的后面.

-   exec-directory

该变量是一个表示目录的字符串,该目录存放的是Emacs自带的一些程序

-   配置项exec-path

当Emacs创建子进程时,若参数program为相对路径,会从该list中指定的目录中搜索要执行的程序.

该list中的每个元素都是一个表示目录的字符串,若为nil则表示default directory(变量\`default-directory'的值)

-   (shell-quote-argument argument)

转义argument,使得返回的字符串以shell语法来看,其真实的内容就是argument

该函数常用于将特殊参数按字面上的意义转递給shell去执行

```emacs-lisp
;; This example shows the behavior on GNU and Unix systems.
(shell-quote-argument "foo > bar")      ; => "foo\\ \\>\\ bar"

;; This example shows the behavior on MS-DOS and MS-Windows.
(shell-quote-argument "foo > bar")      ; => "\"foo > bar\""
```

-   (split-string-and-unquote string &amp;optional separators)

    该函数常用于将一个字符串分拆成由独立的command-line argument组成的list,可以直接作为创建子进程时的arg参数来使用

    该函数根据正则separators的要求将string进行分拆, 并对substring进行反引用

-   (combine-and-quote-string list-of-strings &amp;optional seprator)

    该函数可以认为是\`split-string-and-unquote'的反作用.

    该函数将list-of-string合并成一个单独的string, 若有必要的话,还会对list中的每个string先做一次引用转换.

-   (call-process program &amp;optional infile destination display &amp;rest args)

    同步调用program,这时Emacs会暂停一直等到子进程结束,并返回结束码

    若参数infile不为nil,表示program从哪个文件读取输入. nil表示从null device读取输入

    若参数display为非nil,则\`call-process'会在有内容输出时,刷新buffer的显示.

    参数args表示传递給program的参数

    参数destination表示program的output输出到哪:

    -   buffer-or-string

    插入到buffer中

    -   t

    插入到当前buffer

    -   nil

    丢弃输出

    -   0

    丢弃输出,并且不等待子进程结束就直接返回nil.

    -   (:file FILE-NAME)

    覆盖FILE-NAME指代的文件内容

    -   (REAL-DESTINATION ERROR-DESTINATION)

    区分输出stdout和stderr.

    若error-destination为nil表示丢弃stderr,若为t表示与stdout输出到一个地方,若为字符串表示输出stderr到文件中.

    注意: **你无法将error-destincation设置为某个buffer**,因为实现起来太难了.

-   (process-file program &amp;optional infile buffer display &amp;rest args)

    类似\`call-process',但根据变量\`default-directory'的值不同,可能会invoke a file handler

    参数的意思跟\`call-process'极其相似,其区别在于:

    -   Some file handlers may not support all combinations and forms of the arguments INFILE, BUFFER, and DISPLAY.

    -   当file handler被调用时,该file handler根据参数program来决定应该运行哪个程序.

    例如,suppose that a handler for remote files is invoked.  Then the path that is used for searching for the program might be different from \`exec-path'

    -   参数infile也可能调用file handler. 该file handler可能与\`process-file'自己选择的file-handler不一样

    -   若参数buffer使用'(REAL-DESTINATION ERROR-DESTINATION)这样的格式,并且ERROR-DESTINATION为表示文件的字符串,则跟infile一样,可能调用起其他的file-handler

-   process-file-side-effect

    该变量指定了当调用\`process-file'时,是否可以修改远程文件.

    默认为t,表示可以修改.

    **该参数只能用在let-binding中,不要用在setq中**

-   (call-process-region start end program &amp;optional delete destination display &amp;rest args)

    类似\`call-process',该函数同步调用子进程,并将从start到end处的文本作为进程的stdin.

    若参数delete为非nil,则会删掉从start到end处的文本内容,这在参数destination为t时,可以实现替代的功能.

-   (call-process-shell-command command &amp;optional infile destination display &amp;rest args)

    该函数同步执行shell命令command

    其参数说明与\`call-process'类似

-   (process-file-shell-command command &amp;optional infile destination display &amp;rest args)

    类似\`call-process-shell-command',但是内部使用\`process-file'代替\`call-process'

-   (shell-command-to-string command)

    该函数将command作为shell command来执行,并将执行结果作为string返回

-   (process-lines program &amp;rest args)

    该函数运行program,等待它执行完成,然后以字符串list的形式返回输出.

    若参数program退出时返回非0的退出码,该函数会抛出error

-   (start-process name buffer-or-name program &amp;rest args)

    该函数异步创建PROGRAM子进程,并返回一个process object.

    参数NAME指定了返回process object的名称. 若改名称的process已经存在,则NAME会被修改(通过在后面添加&lt;1&gt;,&lt;2&gt;...)成唯一的名称.

    参数BUFFER-OR-NAME是与process相关联的buffer

    若PROGRAM为nil,则Emacs创建一个新的伪终端(pty)并且将它的input和output与BUFFER-OR-NAME指定的buffer相关联,但是不会去创建子进程,而且参数ARGS被忽略
    ```emacs-lisp
    (start-process "my-process" "foo" "sleep" "100")
    ;;=> #<process my-process>

    (start-process "my-process" "foo" "ls" "-l" "/bin")
    ;;=> #<process my-process<1>>
    ```

-   (start-file-process name buffer-or-name program &amp;rest args)

    类似\`start-process',但是根据\`default-directory'的值不同,可能会调用file handler

    This function does not try to invoke file name handlers for PROGRAM or for the PROGRAM-ARGS.

    有些file handler不能支持\`start-file-process'(例如函数\`ange-ftp-hook-function'). 这种情况下,该函数什么也不做并返回nil

-   (start-process-shell-command name buffer-or-name command)

    类似\`start-process',只是它使用shell来执行COMMAND. 使用哪种shell由变量\`shell-file-name'决定

    与直接用\`start-process'来执行COMMAND相比,使用shell来执行COMMAND的好处在于可以使用shell特性来处理参数力的通配符.
    也正因为这样,当在命令中包含任何特殊字符时,都需要使用\`shell-quote-argument'来转义.
    同样的,执行的命令来自于用户的输入,则为了安全考虑,也需要做一下转义

-   (start-file-process-shell-command name buffer-or-name command)

    类似\`start-process-shell-command',但是在内部使用\`start-file-process'代替\`start-process'

-   process-connection-type

    Emacs通过"pty"或"pipe"来控制异步进程. 具体使用哪种方式由该变量的值决定.

    non-nil表示使用pty,nil表示使用pipe
    ```emacs-lisp
    (let ((process-connection-type nil))  ; use a pipe
      (start-process ...))
    ```
    对于那些用户可见的进程,使用"pty"更好点,因为它允许用户在进程与它的子进程之间进行job control(\`C-c',\`C-z'等操作)

    而对于程序内部使用的进程来说,偏向使用"pipe",因为它更有效率,and because they are immune to stray character injections that ptys introduce for large (around 500 byte) messages. 而且pty的总数是有限制的,最好不要浪费


### Deleting Processes {#deleting-processes}

delete process会立即断开Emacs与process的连接,并且Emacs会发送信号去终止process的执行,并调用process sentinel

要注意: **process被删除掉,不代表process object会被立刻回收** 有些函数可以接收表示被删除process的process object,但是尝试对它作IO操作,发发送信号給它时,会爆出错误.

-   配置项delete-exited-processes

该变量决定了当一个process终止运行了,是否自动删除它.

若为nil则不自动删除,直到运行\`list-processes'命令才作删除

非nil表示process退出时,自动删除它

-   (delete-process process)

该函数主动删除process,会发送\`SIGKILL'信号来关闭进程.

参数process可以是一个process,process的名称,process关联的buffer,process关联的buffer名称.

```emacs-lisp
(delete-process "*shell*")              ; => nil
```


### process的属性 {#process的属性}

-   命令(list-processes &amp;optional query-only buffer)

该命令显示所有living process,并且会删除那些状态为\`Exited'或\`Signaled'的process. 该函数返回nil

参数buffer指定了结果显示到哪个buffer中,默认为\`\*Process List\*',它的major mode需要是Process Menu mode

若参数query-only为非nil,则只列出那些query flag为非nil的process

-   (process-list)

列出未删除掉的process的列表

```emacs-lisp
(process-list)
=> (#<process display-time> #<process shell>)
```

-   (get-process name)

获取名称为name的process object,若没有,则返回nil

```emacs-lisp
(get-process "shell")
=> #<process shell>
```

-   (process-command process)

该函数返回PROCESS的执行命令. 该结果为string的列表,第一个元素为执行的program,剩下的元素为args

```emacs-lisp
(process-command (get-process "shell"))
          => ("bash" "-i")
```

-   (process-contact process &amp;optional key)

该函数返回serial process/network的set up的相关信息

参数process可能为一个process,也可能一个connection

若参数KEY为nil,则返回serial process的'(PORT SPEED)信息,或返回network process的'(HOSTNAME SERVICE)信息

若参数KEY为t,则返回的值包含了connection,server或serial port的完整信息(即使用\`make-network-process'或\`make-serial-process'创建process时指定的所有keyword的值)

若参数KEY为某个特定的keyword,则只返回对应的value

若process为普通的child process则该函数总是返回t

-   (process-id process)

获取PROCESS的pid

-   (process-name process)

以string的形式返回PROCESS的名字

-   (process-status process-or-name-or-buffer)

以symbol类型返回process的状态

参数process-or-name-or-buffer必须是process,buffer或process-name中的一个.

返回值说明如下:

-   'run

    进程正在运行

-   'stop

    进程被暂停执行,但等到时间片后会继续执行

-   'exit

    进程已经退出

-   'signal

    进程已经收到致命signal

-   'open

    process为network connection,该连接是open的

-   'closed

    process为network connection,该连接已关闭

-   'connect

    process为non-blocking connection,等待连接完成

-   'failed

    process为non-blocking connection,连接失败

-   'listen

    process为network server,正在监听

-   nil

    process不存在

-   (process-live-p process)

process是否alive.

所谓alive指的它的status为'run,'open,'listen,'connect或stop

-   (process-type process)

返回process的类型.

'network表示process为network connection或server

'serial表示process为serial port connection

'real表示process为real subprocess

-   (process-exit-status process)

返回PROCESS的exit status或被kill时的signal number. 若PROCESS没有终止运行,则返回0

-   (process-tty-name process)

该函数返回PROCESS用来与Emacs交流时使用的terminal name

若process使用pipe,则返回nil

若process表示一个在remote host上运行的程序,则terminal name为process的peroperty \`remote-tty'

-   (process-coding-system process)

以'(DECODE . ENCODE)的格式描述decode process的输出时使用到编码,和encode process的输入时使用的编码

-   (set-process-coding-system process &amp;optional decoding-system encoding-sytem)

设置ENCODE/DECODE PROCESS的输入/输出时使用的编码规则

-   (process-buffer process)

该函数返回PROCESS的关联buffer

process的关联buffer有两个用处:存储process的输出内容和决定何时kill掉该process.

关闭与process相关联的buffer,也会关闭对应的process.

-   (process-mark process)

返回PROCESS的process-mark,

默认的filter函数会将process的输出插入到关联的buffer. 插入的位置由函数\`process-mark'的返回值决定,通常情况下,为buffer的尾端

-   (set-process-buffer process buffer)

该函数设置PROCESS的关联buffer,若参数BUFFER为nil,则PROCESS无关联buffer

-   (get-buffer-process buffer-or-name)

返回关联到该buffer的未删除process

若有多个process关联到该buffer,则只会返回其中之一的process

-   (process-filter process)

返回PROCESS的filter function

subprocess的stdout会传递給"filter function"来处理.

-   (set-process-filter process filter)

设置PROCESS的filter function,若FILTER为nil表示使用默认的filter function

filter function需要接收2个参数:关联的process和输出的字符串.

一般情况下,会在filter function中屏蔽调quitting. 否则按下C-g可能会发生无法预料的结果.

在执行filter function时抛出的error默认情况下会自动被捕获,这样就不会影响正在执行的program. 然而若\`debug-on-error'为non-nil,则error不会被自动捕获,这使得使用Lisp debugger调试filter function称为i可能.

一个普通的filter function模板大概如下:

```emacs-lisp
(defun ordinary-insertion-filter (proc string)
  (when (buffer-live-p (process-buffer proc))
    (with-current-buffer (process-buffer proc)
      (let ((moving (= (point) (process-mark proc))))
        (save-excursion
          ;; Insert the text, advancing the process marker.
          (goto-char (process-mark proc))
          (insert string)
          (set-marker (process-mark proc) (point)))
        (if moving (goto-char (process-mark proc)))))))
```

Note that Emacs automatically saves and restores the match data while executing filter functions.

另外,需要注意: **传递給filter function的输出内容,可能是任意大小的一块输出,很可能一整个句子会被拆分成很多块传递过来.**

-   (process-sentinel process)

返回process的sentinel

-   (set-process-sentinel process sentinel)

设置process的sentinel. 若参数sentinel为nil,则使用默认的sentinel(它仅仅将message插入process buffer中)

```emacs-lisp
(defun msg-me (process event)
    (princ
    (format "Process: %s had the event `%s'" process event)))
   (set-process-sentinel (get-process "shell") 'msg-me)
    => msg-me
   (kill-process (get-process "shell"))
    -| Process: #<process shell> had the event `killed'
    => #<process shell>
```

"process sentinel"是一种函数,该函数在每次所关联的process的status发生改变时,都会被调用一次(包括程序退出时)

sentinel function接收两个参数:process和描述事件类型的字符串

其中字符串格式有下面几种:

-   "finished\n"

-   "exited abnormally with code EXITCODE\n"

-   "NAME-OF-SIGNAL\n"

-   "NAME-OF-SIGNAL (core dumped)\n"

    类似fileter function, 只有在Emacs处于waiting状态时,才会调用sentinel function.

    Emacs不会使用一个队列来保持调用sentinel的原因,它只记录当前状态和发生改变的原因. 因此若状态在很短时间内发生连续变化,只能触发一次sentinel的调用. 不过由于process终止运行后就不会再发生状态变更了,所以process终止操作总会触发一次sentinel

    **每次运行process sentinel前,Emacs都会明确地检查一次process是否有输出**

    **若sentinel要将输出写到process buffer中时,一定要记得检查buffer是否还在,往被kill掉的buffer中写入会引发错误.** 可以用\`(buffer-name (process-buffer PROCESS))'是否为nil来检查buffer是否存在

    sentinel在处理quiting和error时,跟filter function一样. 默认情况下都会屏蔽quitting和自动捕获error

    在sentinel的执行期间,process的sentinel会临时设置为nil,以防止sentinel的重复调用

    Note that Emacs automatically saves and restores the match data while executing sentinels.

-   (waiting-for-user-input-p)

当sentinel或fileter function正在运行时,该函数返回nil.

-   (process-query-on-exit-flag process)

返回PROCESS的query flag

每个程序都由个query flag,若参数为t,则表示kill该process之前询问用户是否确定要作这项操作,默认为t

-   (set-process-query-on-exit-flag process flag)

设置process的query flag

每个process跟symbol类型,也带有自己的property list.

-   (process-get process property)

获取process的property的value

-   (process-put process property value)

设置process的property的值为value

-   (process-plist process)

获取process的plist

-   (set-process-plist process plist)

设置process的plist


### 与Process的交互 {#与process的交互}


#### Sending Input to Processes {#sending-input-to-processes}

可以使用如下命令为异步process的stdio流发送数据,在这些函数中,参数 **PROCESS可以为process object或process的名称,或buffer或buffer的名称,或nil表示当前buffer的process**

-   (process-send-string process string)

    給process的stdio发送string,该函数返回nil

-   (process-send-region process start end)

    将region的内容作为process的stdio

    参数START,END必须是integer或marker,否则会报错

-   (process-send-eof &amp;optional process)

    传递eof到process的stdio

    process若为nil,则表示属于当前buffer的process

    该函数返回process

-   (process-running-child-p &amp;optional process)

    该函数告诉你,process是否将对terminal的控制权交给了它的child process.

    注意: **当Emacs无法分辨时,也返回t**


#### Sending Signals to Processes {#sending-signals-to-processes}

每个发送signal給process的函数都接收两个可选参数:PROCESS和CURRENT-GROUP

参数PROCESS必须为process object或process的名称或buffer或buffer name或nil(表示当前buffer的process).

参数CURRENT-GROUP是一个标志,当运行一个job-control shell作为Emacs的subprocess时,该标志的不同值才有不同的意思.

当参数CURRENT-GROUP的值为非nil,则信号是发送到Emacs与subprocess交互所使用的那个Terminal的process-group,而不是process的process group.  **若process本身就是个job-control shell,表示中断的是shell的current subjob而不是shell本身(注意,这时Emacs是在与shell的当前job交互而不是shell交互)**

若参数CURRENT-GROUOP的值为nil,则信号发送到Emacs的直接子进程的process group.  **若subproces为job-control shell,这就是shell本身**

当Emacs使用pipe与subprocess交互时,参数CURENT-GROUP是无效的.

-   (interrupt-process &amp;optioal process current-group)

    发送SIGINT給process

-   (kill-process &amp;optioal process current-group)

    发送SIGKILL

-   (quit-process &amp;optioal process current-group)

    发送SIGQUIT

-   (stop-process &amp;optioal process current-group)

    发送SIGSTP. 进程暂停后,可以使用\`continue-process'再次让它运行起来

-   (continue-process &amp;optioal process current-group)

    发送SIGCONT

-   命令(signal-process process signal)

    向process发信号

    参数signal必须为一个整数,或以signal为名称的symbol

    参数process除了可以使process object,process的名称,buffer,和buffer的名称外,可以为 **process的pid**,这允许你发送信号給非Emacs的子进程


#### Receiving Output from Process {#receiving-output-from-process}

subprocess的stdout会传递給"filter function"来处理.

默认的filter function只是简单的将内容插入到process相关联的buffer中去, 若process没有相关联的buffer,则丢弃该输出

当subprocess结束运行后,Emacs reads any pending output, 然后就不再从subprocess中读取任何输出了,即使这时候subprocess的child process还有输出也不管.

需要注意的是:subprocess的输出只有当Emacs处于waiting时才回被读取,即Emacs读取terminal input或调用\`accept-process-output'时才会被读取.

在某些操作系统中,当Emacs读取subprocess的输出时,输出的内容是以一小块一小块的方式被读取的,这就照成了读取的效率低下. 通过设置\`process-adaptive-read-buffering'可以适当提高一些效率,因为它对这些进程进行延迟读取,等他们产生更多的输出时再读取出来.

**要区分subprocess中的stdout和stderr是不可能的,因为Emacs通常在pty中调用子进程,而pty只有一个stdout,若想区分他们,只能将其中一个重定向到文件中**

-   (accept-process-output &amp;optional process seconds millisec just-this-one)

    该函数允许Emacs读取PROCESS的输出.

    若参数PROCESS为非nil,则该函数会一直等待,直到从PROCESS读取到输出为止.

    参数SECONDS和MILLISEC为超时时间. 由于参数SECONDS可以是浮点型,因此参数MILLISEC不推荐使用.

    若参数PROCESS为非nil,而参数JUST-THIS-ONE为非nil,则只有该process的输出被处理.
    若JUST-THIS-ONE为一个整数,则还会暂停定时器的执行

    若\`accept-process-output'等待超时而没有读到任何东西,则返回nil


### Accessing Other Processes {#accessing-other-processes}

Emacs除了可以与自己创建的subprocess交互外,也能与同机器上的其他进程交互,这些进程称为"System process"

-   (list-system-processes)

列出所有正在运行的进程pid的list

-   (process-attributes pid)

返回指定system process的属性组成的alist

这些属性有:

-   euid

    effective user id

-   user

-   egid

    The group id of the effective user id

-   group

-   comm

    运行该process的command

-   state

    该process的状态码

    | 状态码 | 说明                                                                  |
    |-----|---------------------------------------------------------------------|
    | "D" | uninterruptible sleep (usually I/O)                                   |
    | "R" | running                                                               |
    | "S" | interruptible sleep (waiting for some event)                          |
    | "T" | stopped, e.g., by a job control signal                                |
    | "Z" | "zombie": a process that terminated, but was not reaped by its parent |

-   ppid

-   pgrp

-   sess

    session ID of the process

-   ttname

    process的控制终端的名称

-   tpgid

    The numerical process group ID of the foreground process group that uses the process's terminal.

-   minflt

    The number of minor page faults caused by the process since its beginning.
      (Minor page faults are those that don't involve reading from disk.)

-   majflt

    The number of major page faults caused by the process since its beginning.
      (Major page faults require a disk to be read, and are thus more expensive than minor page faults.)

-   cminflt / cmajft

    Like \`minflt' and \`majflt', but include the number of page faults for all the child processes of the given process.

-   utime

    process在user context下的运行时间(消耗CPU时间片的时间)

-   stime

    process在system context下的运行时间

-   time

    utime+stime

-   cutime / cstime / ctime

    类似utime,stime,time,但是包括指定process下的子进程

-   pri

    优先级

-   nice

-   thcount

    process中的进程数

-   start

    process开始的时间

-   etime

    process开始后,经过了多少时间

-   vsize

    process占用虚拟内存的大小,kb为单位

-   rss

    process占用物理空间的大小,kb为单位

-   pcpu

    占cpu的百分比

-   pmen

    占总物理内存的百分比

-   args

    command的参数


### 交易队列 {#交易队列}

通过交易队列,可以使用交易与process进行通讯.

首先使用\`tq-create'创建交易队列,然后使用\`tq-enqueue'发送交易

-   (tq-create process)

创建与PROCESS通讯的交易队列

参数PROCESS必须可读写的,既可以是子进程, **也可以是网络连接.**

-   (tq-equeue queue question regexp closure fn &amp;optional delay-question)

发送交易到队列QUEUE

参数QUESTION为发送的消息

参数FN为当answer回来时的回调函数. 他接收两个参数:参数CLOSURE和接收到的answer

参数REGEXP为一个正则表达式,该正则表达式应该匹配整个完整答案结束时的文本. \`tq-enqueue'使用该正则来判断answer是否已经接收完全

若参数DELAY-QUESTION为非nil,则会暂缓question的发送,直到process对之前的question都回应完后再发送. 对某些process来说,这样会得到比较靠谱的answer

-   (tq-close queue)

等待所有的交易完成,然后关闭队列

Transaction queues的实现依靠filter function


### Network Process {#network-process}

Emacs Lisp程序可以创建TCP/UDP链接(通过内建/外部支持,甚至还能创建加密的网络连接)，既能创建客户端,也能创建服务端.

Elisp把网络链接看成时跟subprocess类似的东西，也用process object来表示。当然这种process object没有pid，也不能对它发送信号什么的。

通过调用\`make-network-process‘可以创建Network connection和Network server。 该函数接受keyword参数\`:server t’表示创建Network server process. \`:type datagram'表示创建UDP链接

可以通过\`stop-process'和\`continue-process'来stop/resume network process的操作.
对于server process来说,stop意味着不再接收新连接请求.
对于network connection来说,stop意味着不再接收输入流

通过带参数\`:server t'调用\`make-network-process'创建的network server. 它接受客户端的连接请求,并创建一个新的process object表示这个新的network connection.
这个新生成的表示network connection的process object有如下几个特征:

-   connection process的名称为server process的名称+客户端唯一标识
-   若server process没有默认的filter function,则connection process没有自己独立的process buffer. 否则Emacs为process buffer创建自己独立的buffer,buffer名称为server的buffer名称或process名称加上client的唯一标识
-   connection process的链接类型,filter connection和sentinel都继承至server process
-   connection process的process contact信息根据client端的地址信息来设置
-   connection process的本地地址根据用于该链接的端口号决定
-   client process的plist初始化为server process的plist一样


#### 创建表示网络连接/网络服务的process object {#创建表示网络连接-网络服务的process-object}

-   (make-network-process &amp;rest args)

    该函数创建一个network connection或network server,并返回一个process object.

    这里参数args可能是以下keyword:

    -   :name NAME

    使用NAME作为process的名称

    -   :type TYPE

    定义链接类型. nil表示TCP链接; 'datagram表示UDP链接;'seqpacket表示"sequenced packet stream"链接

    -   :server SERVER-FLAG

    若参数为非nil,表示创建network server,否则为创建Network connection

    若为stream type server(TCP类型的server),则该参数必须为一个整数,表示最大可以等待的连接数

    -   :host HOST

    表示要连接的host. 它可以是一个表示Internet地址的字符串,或symbol 'local.

    若为Network Server指定host,则Network Client必须连接到这个HOST的请求才会被接受

    -   :service SERVICE

    SERVICE指定了要连接的端口. 它可以是表示service名称的字符串,或者表示端口号的整数

    对于Network server.该参数还可以为t,表示让系统自己选择一个未用的port

    -   :family FAMILY

    FAIMILY指定了链接协议的种类.

    nil表示由系统自动选择.

    'local表示为Unix socket,这种情况下:host参数可以被忽略

    'ipv4或'ipv6表示使用IPv4和IPv6

    -   :Local LOCAL-ADDRESS

    对于Network Server. LOCAL-ADDRSS为监听的地址.

    该参数会覆盖FAMILY,HOST和SERVICE的值

    其中LOCAL-ADDRESS的格式根据FAMILY的不同而不同

    -   IPv4的地址使用一个5元素的vector表示[A B C D Port]

    -   IPv6的地址使用一个9元素的vector表示[A B C D E F G H Port]

    -   本地地址使用字符串表示

    -   :remote REMOTE-ADDRESS

    对于Network connection来说REMOTE-ADDRESS为要连接到的地址. 该参数会覆盖FAMILY,HOST和SERVICE

    对于UDP Server来说,REMOTE-ADDRESS指定了remote datagram address的初始设置

    REMOTE-ADDRESS的格式参见LOCAL-ADDRESS的格式

    -   :nowait BOOL

    若BOOL为非nil,则对于TCP connection来说,函数不等待连接完成就返回.

    当链接连接成功或失败后,会调用process的sentinel function

    -   :stop STOPPED

    若STOPPED为非nil,则创建的network connection或network server处于stopped状态

    -   :buffer BUFFER

    使用BUFFER作为process buffer

    -   coding CODING

    设置process的编码,格式为'(DECODING ENCODING)

    -   :noquery QUERY-FLAG

    初始化process的query flag,若为非nil,则在delete 该process时会提升用户确认

    -   :filter FILTER

    初始化process的filter function

    -   :filter-multibyte MULTIBYTE

    若MULTIBYTE为非nil,则传递给process filter function的字符串为multbyte格式的.否则为unibyte格式的.

    默认为参数\`enable-multibyte-characters'的值

    -   :sentinel SENTINEL

    初始化process的sentinel

    -   :log LOG

    初始化Network server的log function.

    每次server接受一次client发起的连接请求就会调用一次log function.

    传递给log function的参数有:SERVER,CONNECTION和MESSAGE

    -   :plist PLIST

    初始化process的plist

    下面的参数是专为network connection使用的

    -   :bindtodevice DEVICE-NAME

    指定绑定到哪张网卡,只有从该网卡接受到的报文才会被处理,若DEVICE-NAME为nil,表示任何网卡

    -   broadcast BROADCAST-FLAG

    对于datagram process来说,若BROADCAST-FLAG为非nil,则process会接受发送到广播域的UDP报文,也能将UDP报文发送到广播域

    对TCP 链接无效

    -   :dontroute DONTROUTE-FLAG

    若DONTROUTE-FLAG为非nil,则表示不由路,及只能发送报文给统一网段的地址

    -   :keepalive KEEPALIVE-FLAG

    若为TCP链接,且KEEPALIVE-FLAG为非nil,则开启low-level keep-alive messages交换功能

    -   linger LINGER-ARG

    若LINGER-ARG为非nil,则会在关闭链接前会等待链路上的报文都转发成功后才关闭.

    若LINGER-ARG为整数,它表示等待报文转发的最大时间.

    默认参数nil表示关闭链接时直接丢弃所有未转发的报文

    -   :oobinline OOBINLINE-FLAG

    若为TCP链接,且OOBINLINE-FLAG为非nil,则接受out-of-band数据包

    -   :priority PRIORITY

    设置数据包的优先级,为整数.

    该参数与系统,协议都相关

    -   :reuseaddr REUSEADDR-FLAG

    该参数对stream server process生效.

    默认REUSEADDR-FLAG为非nil,表示该服务可以立刻重用指定的端口.

    若REUSEADDR-FLAG为nil,表示在一个进程使用了指定端口后,一段时间内该端口不能被其他进程所使用.

-   (set-network-process-option process option value &amp;optional no-error)

    设置已存在network process的网络属性,可设置的属性参见\`make-network-process'中的属性(但不包括reuseaddr属性)

    若参数NO-ERROR为非nil,则当设置的参数不支持时不会抛出error,只返回nil

    若设置成功,则返回t

-   (open-network-stream name buffer host service &amp;rest parameters)

    该函数创建一个TCP连接(可选择加密),并返回一个process object

    参数NAME表示该process object的名称,若有重复名称则会自动添加编号

    参数BUFFER为与该connection相连的buffer. 默认情况下connection的输出会插入到该buffer中. 若BUFFER为nil表示没有连接的buffer

    参数HOST和SERVICE指明了要连接的服务端的地址和端口. 其中host为字符串类型,SERVICE可以为字符串也可以为整数

    剩下的参数PARAMETERS,是各种keyword参数:

    -   :nowait BOOLEAN

        若为飞nil,则表示创建异步连接

    -   :type TYPE

        连接的类型.

        | TYPE          | 说明                                                                             |
        |---------------|--------------------------------------------------------------------------------|
        | plain         | 普通的,未加密的链接                                                              |
        | tls / ssl     | TLS链接                                                                          |
        | nil / network | 自动决定类型. 若系统支持参数:success和:capability-command,则先尝试通过STARTTLS建立加密链接,若失败了,使用普通的未加密链接 |
        | starttls      | 类似nil,但是若通过STARTTLS创建链接失败了,则关闭该链接                            |
        | shell         | shell connection                                                                 |

    -   :always-query-capabilities BOOLEAN

        若为非nil,则总是询问server端能支持的特性,及时创建的只是普通的链接

    -   :capability-command CAPABILITY-COMMAND

        定义查询server端支持特性时的命令串

    -   :end-of-command REGEXP

        匹配command结束的正则表达式

    -   :end-of-capability REGEXP

        匹配CAPABILITY-COMMAND命令串结束的正则表达式,默认为:end-of-command的值

    -   :starttls-function FUNCTION

        该function应该能接收一个参数,该参数为服务端对CAPABILITY-COMMAND的回应.

        该function应该返回nil或激活STARTTLS的命令

    -   :success REGEXP

        使用该正则表达式来判断是否正常开启STARTTLS特性

    -   :use-starttls-if-possible BOOLEAN

        若值为非nil,计时Emacs没有内建TLS支持,也尝试开启STARTTLS特性

    -   :client-certificate LIST-OR-T

        可以以'(KEY-FILE CERT-FILE)的格式明确指明了certificate key file和certificate file的地址

        或者t表示通过查询\`auth-source'来获取信息

    -   :return-list CONS-OR-NIL

        指明\`make-network-stream'的返回值.

        若为nil则返回process object

        否则返回'(PROCESS-OBJECT . PLIST). 其中PLIST包含如下keyword:

        -   :greeting STRING-OR-NIL

            表示服务端返回的欢迎信息

        -   :capabilities STRING-OR-NIL

            表示服务端的支持的特性信息

        -   :type SYMBOL

            链接的类型,可能为'plain或'tls

-   (process-datagram-address process)

    若PROCESS为UDP connection或UDP server. 则该函数返回remote peer address

-   (set-process-datagram-address process address)

    若PROCESS为UDP connection或UDP server. 则该函数设置remote peer address为ADDRESS


#### 测试Network特性 {#测试network特性}

要测试本机上的make-network-process支持哪些特写特性,可以使用\`featurep'函数:

```emacs-lisp
(featurep 'make-network-process '(KEYWORD VALUE))
```

它表示\`make-network-process'时KEYWORD的值是否为VALUE

下面是一些例子

| KEYWORD VALUE PAIRS | 说明                                                              |
|---------------------|-----------------------------------------------------------------|
| (:nowait t)         | Non-\`nil' if non-blocking connect is supported.                  |
| (:type datagram)    | Non-\`nil' if datagrams are supported.                            |
| (:family local)     | Non-\`nil' if local (a.k.a. "UNIX domain") sockets are supported. |
| (:family ipv6)      | Non-\`nil' if IPv6 is supported.                                  |
| (:service t)        | Non-\`nil' if the system can select the port for a server.        |

也可以使用如下格式的form来测试指定的network选项是否能设置

```emacs-lisp
(featurep 'make-network-process 'KEYWORD)
```


#### 其他Network函数 {#其他network函数}

这里的函数,需要操作系统的支持.

-   (network-interface-list)

    该函数返回当前机器中各网卡的描述列表.

    该列表的格式为一个由'(NAME . ADDRESS)组成的alist
    ```emacs-lisp
    (network-interface-list)
    ;; ("wlan3" . [192 168 8 113 0]) ("lo" . [127 0 0 1 0])
    ```

-   (network-interface-info interface-name)

    该函数返回指定网卡的信息.

    该信息是一个格式格式为'(ADDR BROADECAST-ADDR NETMASK HARDWARE-ADDR FLAGS)的list
    ```emacs-lisp
    (network-interface-info "wlan3")
    ;; (
    ;; [192 168 8 113 0]
    ;; [192 168 8 255 0]
    ;; [255 255 255 0 0]
    ;; (1 . [8 16 120 53 33 177])
    ;; (multicast running broadcast up))
    ```

-   (format-network-address address &amp;optional omit-port)

    该函数将lisp格式的网络地址转换为字符串表示

    参数OMIT-PORT表示转换时是否不带端口信息


### Serial Port Process {#serial-port-process}

通过\`make-serial-process'创建serial port process可以与serial port通讯

serial port process可以通过\`serial-process-configure'实时的修改配置,而不用关闭后重新连接.

-   (make-serial-process &amp;rest args)

该函数创建serial port process及其相关连的buffer. 在内部会使用函数\`serial-process-configure'进行真正的配置工作

args可以是如下keyword参数:

-   :port PORT

    serial port的名称(Unix下为/dev/ttyS0,Win下为COM1或\\\\.\COM10)

-   :speed SPEED

    serial port的速率

-   :name NAME

    process的名称,可能会添加后缀以保证唯一

-   :buffer BUFFER-OR-NAME

    相关连的buffer,若忽略该参数则与:name的参数值一样

-   :coding CODING

    读取的编码格式,参数格式为'(DECODING . ENCODING)

-   :noquery QUERY-FLAG

    初始化process的query flag. 决定被关闭时是否提示用户确认

-   :stop BOOL

    创建的process是否一开始就处于stopped状态,这是process不能接受数据,但是可以发送数据

-   :filter FILTER

    设置rocess filter function

-   :sentinel SENTINEL

    设置process的sentinel

-   :plist PLIST

    设置process的初始plist

-   :bytesize BYTESIZE

    设置每个byte包含多少bit,可以为7或8. 默认为8

-   :parity PARITY

    可以为nil,'odd或'even

-   :stopbits STOPBITS

    每个byte中用于终止传输的stopbit的位数. 可以是1或2. 默认为1

-   :flowcontrol FLOWCONTROL

    flow的类型决定了如何使用该链接. 可以是nil(不使用流控制特性),'hw(使用RTS/CTS硬件流控制),'sw(使用XON/XOFF软件流控制)

可以通过函数\`process-conntact'查看那些参数可以被修改.

-   (serial-process-configure &amp;rest args)

重新设置serial port process的属性


## 操作系统相关 {#操作系统相关}


### Emacs启动说明 {#emacs启动说明}


#### Emacs启动流程 {#emacs启动流程}

1.  搜索\`load-path'中的各个目录,看是否存在\`subdirs.el'这个文件,有则执行该文件.

    \`subdirs.el'这个文件正常情况下是由Emacs在安装时自动生成的,它的作用是加载目录中的各个子目录到\`load-path'中,并且递归执行子目录中的\`subdirs.el'文件

2.  在\`load-path'中的各目录中寻找\`leim-list.el'并加载

    \`leim-list.el'这个文件被用来注册输入法. 并且Emacs在搜索\`leim-list.el'时会跳过那些存放Emacs自带库的目录.

3.  设置变量\`before-init-time'的值为当前时间. 设置变量\`after-init-time'为nil,表示Emacs还未初始化完成.

4.  根据\`LANG'环境变量设置language environment 和 the terminal coding system

5.  对命令行参数进行基本的处理,这一步处理掉的是emacs本身支持的那些参数.

6.  若不是运行在batch模式下,Emacs根据\`initial-window-system'的值来为不同的窗口系统进行不同的初始化.

    某个窗口系统分别加载哪个初始化函数,是根据变量\`window-system-initialization-alist'来决定的.

    存放初始化函数的文件路径应该为\`term/窗口系统类型-win.el'中

7.  触发\`before-init-hook'

8.  在可以的情况下,创建图形化的frame.

    若设置了参数\`--batch'或\`--daemon'或\`--script'则直接跳过这一步

9.  初始化最初的frame的显示界面,如果需要的话,还会设置菜单栏和工具栏.

    若系统支持图形化的frame,则不管当前frame是否为图形化的frame,都会去设置工具栏,因为在后面的操作中有可能再次创建图形化frame

10. 使用函数\`custom-reevaluate-setting'来重新初始化变量\`custom-delayed-init-variables'中的各成员的值.

11. 若\`site-start'库存在,则加载之.

    但若设置了参数\`-Q'或\`--no-site-file'则跳过这一步

12. 加载用户的初始化文件,用户初始化文件可能是\`~/.emacs',\`~/.emacs.el',\`~/.emacs.d/init.el'

    但若设置了参数\`-q',\`-Q'或\`--batch'则跳过这一步

13. 若\`default'库存在,则加载之.

    但若变量\`inhibit-default-init'为非nil,或设置了参数\`-q',\`--batch'则跳过这一步

14. 尝试从变量\`abbrev-file-name'变量指定的文件中加载缩写配置信息.

    若设置了参数\`--batch',则跳过这一步

15. 若\`package-enable-at-startup'为非nil,则Emacs调用函数\`package-initialize'来激活所有的已安装的第三方Emacs Lisp包. 具体参见[Packaging Basics](https://www.gnu.org/software/emacs/manual/html_node/elisp/Packaging_002520Basics.html "Emacs Lisp: (info \"(elisp) Packaging%20Basics\")")

16. 设置变量\`after-init-time'的时间为当前时间,表示初始化过程已经完成

17. 触发\`after-init-hook'

18. 若\`\*scratch\*' buffer存在,且出于Fundamental mode下,则根据变量\`initial-major-mode'的值却换Major Mode

19. 若在文本终端环境启动的Emacs,则还会加载与终端相关的lisp library,并触发\`tty-setup-hook'

20. 若\`inhibit-startup-echo-area-message'为nil,则在echo area显示初始信息

21. 处理尚未被处理的命令行参数,这一步处理的是用户自定义的参数

22. 若设置了参数\`--batch'则此时退出

23. 若\`initial-buffer-choice'为字符串,则访问该字符串所表示的文件/目录

    若为函数,则不带参数调用该函数,并根据返回结果选择显示哪个buffer

    若\`\*scratch\*' buffer存在且无内容,则插入\`initial-scratch-message'变量值到buffer中

24. 触发\`emacs-startup-hook'

25. 调用\`frame-notice-user-settings',该函数根据初始化文件修改选中的frame参数

26. 触发\`window-setup-hook'

27. 显示\`startup screen' buffer

    但若\`inhibit-startup-screen'或\`inital-buffer-choice'为非nil,或设置了\`--no-splash'/\`-Q'命令行参数,则跳过这一步

28. 若设置了选项\`--daemon',则调用\`server-start'函数,并从控制终端相分离(参见[Emacs Server](https://www.gnu.org/software/emacs/manual/html_node/emacs/Emacs_002520Server.html "Emacs Lisp: (info \"(emacs) Emacs%20Server\")"))

29. 若Emacs由X Session Manager调起,则会调用\`emacs-session-restore'函数,调用参数为之前X Session的ID


#### Emacs相关初始化文件 {#emacs相关初始化文件}

用户初始化文件可能是\`~/.emacs',\`~/.emacs.el',\`~/.emacs.d/init.el'

有些Emacs会有一个名为\`default.el'的默认初始化文件,若在\`load-path'中能找到该文件的话,则当启动Emacs,会加载该文件.

个人用户的初始化文件优先级比\`default.el'的要高, **若在加载个人初始化文件时将\`inhibit-default-init'设置非nil,则不再加载\`default.el'了**

当然\`-q'和\`-Q'参数,使得Emacs既不加载个人初始化文件,也不加载默认初始化文件

还有一个配置site相关的初始化文件叫做\`site-start.el',Emacs在加载用户初始化文件前会加载该文件,但你可以通过参数\`--no-site-file'来跳过加载该文件

-   配置项site-run-file

    该变量指定了与site相关配置的初始化文件的文件名,默认为\`site-start'

    The only way you can change it with real effect is to do so before

-   配置项\`inhibit-default-init'

    该变量指定是否不加载默认初始化文件\`default.el',默认为nil表示加载

-   before-init-hook

    在加载所有的初始化文件(\`site-start.el',个人初始化文件,\`default.el')前触发

-   after-init-hook

    加载完所有的初始化文件后,在加载与终端相关library前(若在文本终端下启动的Emacs)或处理命令行参数前触发

-   emacs-startup-hook

    在处理完命令行参数后触发

    若在batch模式下,Emacs不触发该hook

-   window-setup-hook

    类似\`emacs-startup-hook',但是它触发的时间要晚一点,在设置完frame参数之后触发

-   user-init-file

    该参数的值为用户初始化文件的绝对路径名. 若实际加载的初始化文件为.elc文件,则该值为相应的源代码路径

-   user-emacs-directory

    该参数的值为\`.emacs.d'目录的路径. 除了MS-DOS平台,其他平台上该值都是\`~/.emacs.d'


#### 终端相关的library {#终端相关的library}

Emacs在不同类型的终端下启动时,都会加载不同的终端相关的library. 该library的名字由\`term-file-prefix'变量的值(默认为"term/")与终端类型(通常由环境变量\`TERM'表示)组合而成.

该terminal-specific librar的作用常用来使得Emacs能够识别special keys. 若操作系统的Termcap或Terminfo项无法完全识别所有的终端功能键,则可以需要修改变量\`input-decode-map'的值

若终端类型名中包含\`-'或\`_',且使用改名字查找library时未找到,则会尝试去除终端名中最后那个\`_'或\`-'部分后,作为终端名称在此查询library.

可以在初始化文件中通过设置\`term-file-prefix'为nil,以阻止Emacs加载terminal specific library

在Emacs完成初始化文本终端后,会触发\`tty-setup-hook',You could use this hook to define initializations for terminals that do not have their own libraries.

-   term-file-prefix

    若变量为nil表示不加载终端初始化文件. 否则Emacs加载名为 `(load (concat term-file-prefix (getenv "TERM"))` 的文件作为初始化终端的脚本.

-   tty-setup-hook

    该hook在Emacs初始化万一个新文本终端后触发.(This applies when Emacs starts up in non-windowed mode, and when making a tty ‘emacsclient’ connection.)

-   (suspend-tty &amp;optional TTY)

    参数TTY为Emacs使用的终端. 该函数挂起指定的终端,此时使用该终端的Frame依然存在,但是Emacs并不再从该终端读取任何输入,也不再更新使用该终端的frame.

    参数TTY可以使一个终端对象,也可以是一个frame(表示该frame所在的终端),或nil(表示当前frame所在的终端)

    若TTY已经出于挂起状态,该函数不做任何事情.

    该函数还会触发\`suspend-tty-functions',以终端对象作为参数来调用其中的每个函数.

-   (resume-tty &amp;optional tty)

    参数TTY为之前挂起的终端设备,该函数恢复该终端,并触发\`resume-tty-functions',同样以终端对象作为参数来调用其中的每个函数.

    若TTY不处于挂起状态,则该函数不做任何事

    If the same device is already used by another Emacs terminal, this function signals an error.

-   (controlling-tty-p &amp;optional tty)

    判断TTY是否为控制终端.

    参数TTY可能是终端对象,frame(表示该frame所在的终端),或nil(表示当前frame所在的终端)


#### Emacs是如何处理命令行参数的 {#emacs是如何处理命令行参数的}

当使用emacs --script xxx.el args时,为了获取command-line参数,可以在xxx.el中使用变量\`argv\`获取参数列表

-   (command-line)

    该函数解析调用Emacs时的command line,处理该command line,加载用户初始化文件,然后显示启动信息

-   command-line-processed

    该变量标识了,comand line是否已经被处理过了, 若处理过了则该值为t

    当通过\`dump-emacs'函数来redump Emacs时,常常会先将该变量设为nil,这样可以让新dumped Emacs会去再一次处理它的command-line arguments

-   command-switch-alist
    该变量是素为\`(option . handler-function)'的alist. 这里

    -   option为command-line argument中的\`-option'参数(**带-**),为字符串格式

    -   handler-function为相应的处理函数名,它接收option为参数

    若command line option后还带了其他参数,则在handler-function中可以通过变量\`command-line-args-left'来获取剩余的命令行参数

-   command-line-args
    传递给Emacs的完整command-line argument列表

-   command-line-args-left
    尚未处理的command-line argument列表. **自定义函数有时需要修改该变量**

-   command-line-functions
    该变量是一系列函数的列表,这些函数用来处理无法识别的command-line参数.

    每次处理一个没有特殊意义的command line argument时,该变量中的函数都会被依次调用, **直到有一个函数返回非nil的值**

    **这些函数被调用时并不传递参数,但在这些函数内可以通过变量\`argi'获取当前待处理的command-line argument. 可以通过变量\`command-line-args-left'获取尚未被处理的command line arguments**.

    **若某函数除了当前待处理的函数,同时也把后面的参数給处理过了,则需要把后面那些被处理过的参数从\`command-line-args-left'中删除**

    **若某函数已经处理了当前代处理的参数,则一定记得返回非nil值**. **若所有的函数都返回nil,该参数会被认为是Emacs要打开的文件名称**


### 退出Emacs {#退出emacs}


#### 退出Emacs {#退出emacs}

-   命令(kill-emacs &amp;optional exit-data)

    该命令触发\`kill-emacs-hook',并退出Emacs进程

    参数\`EXIT-DATA'若为整数,则表示Emacs进程的退出码

    参数\`EXIT-DATA'若为字符串,则表示Emacs退出时输出的内容

-   kill-emacs-hook

    该hook在\`kill-emacs'真正退出Emacs进程前被触发

    **由于\`kill-emacs'被调用的时候可能已经与用户失去了交互,因此该hook的参数不能包含与用户交互的语句.**

    若需要在退出时与用户交互,使用下面的\`kill-emacs-query-functions'

-   kill-emacs-query-functions

    当\`save-buffers-kill-terminal'(C-x C-c)尝试退出Emacs时,它会触发该hook.

    在该hook的函数中可以继续询问用户确认是否退出. 若该hook中任何一个函数返回nil,则\`save-buffer-kill-emacs'并不会真正退出Emacs,并且也不执行hook之后的函数.

    **直接调用\`kill-emacs'并不会触发该hook**


#### 挂起Emacs {#挂起emacs}

在 **文本终端(图形终端下无效)** 中调用Emacs的情况下,可以对Emacs执行挂起操作(对于不支持挂起操作的shell来说,该功能只是临时再启动一个shell而已).

-   命令(suspend-emacs &amp;optional string)

    该函数阻塞并挂起Emacs并将控制权交回给它的父进程. 当重新激活Emacs后,该函数返回nil

    该函数仅当Emacs是在控制终端下启动时才有用.to relinquish control of other tty devices, use‘suspend-tty’

    在挂起Emacs之前,你必须删除该Emacs在其他终端上的frame,否则该函数会抛出异常. 参见[Multiple Terminals](https://www.gnu.org/software/emacs/manual/html_node/elisp/Multiple_002520Terminals.html "Emacs Lisp: (info \"(elisp) Multiple%20Terminals\")")

    若参数string为非nil,则字符串中的每个字符都会发送到上层shell,作为 **终端输入(注意:不是作为进程输出)  该输入会被shell读取并执行**

    在挂起Emacs前,\`suspend-emacs'会触发\`suspend-hook'.在恢复Emacs后,\`suspend-emacs'会触发\`suspend-resume-hook'

-   suspend-hook

    Emacs挂起前触发

-   suspend-resume-hook

    Emacs恢复后触发

-   命令(suspend-frame)

    挂起当前frame.

    若出于图形界面下,则它调用函数\`iconify-frame'最小化frame.

    若出于文本界面下,则根据当前frame是否出于控制终端下,而调用\`suspend-emacs'或\`suspend-tty'


### 操作系统环境相关 {#操作系统环境相关}

-   system-configureation

the standard GNU configuration name for the hardware/software configuration of your system,字符串类型

-   system-type

表示操作系统类型的symbol

-   'aix
-   'berkeley-unix
-   'cygwin
-   'darwin
-   'gnu
-   'gnu/linux
-   'gnu/kfreebsd
-   hpux
-   irix
-   ms-dos
-   usg-unix-v
-   windows-nt

-   配置项mail-host-address

email地址,若该参数为非nil,则会用来替代\`system-name'作为email地址.

-   命令(getenv var &amp;optional frame)

获取环境变量VAR的值.若找不到对应的环境变量,返回nil

参数VAR为字符串

在Emacs中环境变量存放在变量\`process-environment'中

-   命令(setenv variable &amp;optional value substitute)

该命令设置环境变量VARIABLE的值为VALUE,返回VARIABLE的新值或nil(表示从环境中删除该变量)

参数VARIABLE为字符串类型,VALUE可以为nil或字符串

若VALUE为nil或忽略(interactively with prefix argument)时,\`setenv'从环境变量中删除VARIABLE, **这时\`setenv'返回被删除的VARIABLE**

若参数SUBSTITUTE为非nil,Emacs调用函数\`substitute-env-vars'来扩展环境变量的值为VALUE(什么意思??)

-   process-environment

保持了系统变量的值,\`getenv'和\`setenv'都是通过设置改变量的值进行的.

在实际应用中,经常会用let form临时改变该参数的值

若参数中包括了重复的元素,则只有地一个元素生效

-   initial-environment

改变了存储的是Emacs从父进程中集成到的环境变量

-   path-separator

在搜索路径变量(PATH)中分隔各路径的分隔符. unix类操作系统为":",win下为";"

-   (parse-colon-path path)

接受搜索路径的字符串($PATH的值),并根据path-separator进行分割,返回各个目录组成的list

```emacs-lisp
(parse-colon-path ":/foo:/bar")
;; => (nil "/foo/" "/bar/")
```

-   invocation-name

调用的Emacs可执行文件的名称,不包括目录名称

-   invocation-directory

调用的Emacs,可执行文件所在的目录,若无法决定,则返回nil

-   installation-directory

若为非nil,则指定了当Emacs无法在标准安装路径下找\`lib-src'和\`etc'子目录时,Emacs应该到在哪个目录下查找\`lib-src'和\`etc'子目录.

-   (load-average &amp;optional use-float)

返回最近1分钟,5分钟和15分钟的系统负载

默认情况下该函数返回的值为百分比,但若参数use-float为非nil,则直接使用小数代替

```emacs-lisp
(load-average)
;; => (169 48 36)
(load-average t)
;; => (1.69 0.48 0.36)
```

-   (emacs-pid)

emacs的pid

-   tty-erase-char

该变量存储的是在Emacs启动前,系统d额终端驱动所选择的删除键


### 用户信息 {#用户信息}

-   (user-login-name &amp;optional uid) / user-login-name

获取username

若环境变量\`LOGNAME'或\`USER'有值,则返回该值. 否则根据 **effective UID(而不是real UID)** 计算该值

-   (user-real-login-name) / user-real-login-name

该函数根据Emacs的real UID计算用户名,而不管\`LOGNAME',\`USER'和effective UID

-   (user-full-name &amp;optional uid) / user-full-name

该函数获取用户的全名

若环境变量\`NAME'有值,则返回它.

若Emacs进程的uid不属于任何已知用户,则返回"unkown"

参数uid可以为nil,或一个表示uid的数字或一个表示登录名的字符串. 若根据指定的uid无法找到名称,则返回nil

-   (user-real-uid)

用户的real uid,若uid太大了超过整数的范围,则可能使用浮点数

-   (user-uid)

用户的effective uid

-   (group-gid)

Emacs进程的effective GID

-   (group-real-gid)

Emacs进程的real GID

-   (system-users)

列出该系统所有的用户名列表.

若Emacs不能获取到这些信息,则返回只包含\`user-real-login-name'的列表

```emacs-lisp
(system-users)
;;("cl-builder"
"dictd" "statd" "dnsmasq" "memcache" "sshd" "ftp" "mysql" "Debian-exim" "lujun9972" "saned" "hplip" "speech-dispatcher" "rtkit" "pulse" "kernoops" "usbmux" "avahi" "avahi-autoipd" "whoopsie" "lightdm" "colord" "messagebus" "syslog" "libuuid" "nobody" "gnats" "irc" "list" "backup" "www-data" "proxy" "uucp" "news" "mail" "lp" "man" "games" "sync" "sys" "bin" "daemon" "root")
```

-   (system-groups)

系统中所有的用户组列表.

若Emacs不能获取到这些信息,则返回nil

```emacs-lisp
(system-groups)
;;("cl-builder" "dictd" "memcache" "ftp" "mysql" "winbindd_priv" "Debian-exim" "sambashare" "lujun9972" "saned" "rtkit" "utempter" "pulse-access" "pulse" "avahi" "avahi-autoipd" "ssh" "mlocate" "whoopsie" "netdev" "nopasswdlogin" "lightdm" "ssl-cert" "lpadmin" "colord" "scanner" "bluetooth" "messagebus" "fuse" "syslog" "crontab" "libuuid" "nogroup" "users" "games" "staff" "plugdev" "sasl" "video" "utmp" "shadow" "gnats" "src" "irc" "list" "operator" "backup" "www-data" "dip" "audio" "sudo" "tape" "floppy" "cdrom" "voice" "fax" "dialout" "kmem" "proxy" "man" "uucp" "news" "mail" "lp" "disk" "tty" "adm" "sys" "bin" "daemon" "root")
```


### 播放声音 {#播放声音}

-   (play-sound sound)

播放指定SOUND.

参数SOUND的格式为'(sound PROPERTIES...),其中PROPERTIES可以是以下keyword参数

-   :file FILE

    声音文件的地址. sound-file必须是.wav或.au格式的. 可以是绝对路径或相对\`data-directory'的相对路径

-   :data DATA

    表示使用DATA作为声音的内容,而不用从:file中读取.

    参数DATA必须是包含声音比特流的字符串

-   :volume VOLUME

    指定了音量的大小. 取值范围从0到1

-   :device DEVICE

    指定通过哪台设备播放声音. DEVICE为字符串

-   (play-sound-file file &amp;optional volume device)

播放声音文件

-   play-sound-functions

在播放声音前,会触发该hook. 每个函数都接受一个参数:描述sound的plist

-   (set-message-beep SOUND)

设置beep蜂鸣时的声音.

参数SOUND可以是nil,'asterisk,'exclamation,'hand,'question,'ok或'silent


### Batch Mode {#batch-mode}

在启动Emacs时若带了参数\`batch'会使得Emacs进入batch mode,这种状态下的Emacs不能与用户交互,任何输出到echo area的信息都会输出到Emacs的stdout, 任何从minibufer读取的输入都被链接到Emacs的stdin中.

该模式常用于使用Emacs来运行某个elisp程序,程序运行完成后,Emacs就退出了.

-   noninteractive

该参数指明了Emacs是否运行在Batch Mode下


### Session管理 {#session管理}

Emacs支持X Session Management Protocol,该协议用于suspend/restart应用程序.
在X window系统中,一个名为"session manager"的程序负责保持正在运行的程序的状态.
当X server关闭时,session manager会要求应用程序保存它们的状态,并推迟X server关闭直到收到应用程序的回应. 此时应用程序也可能会取消这次关闭.

当session manager重启挂起的session时,它会引导应用程序分别加载自己保存的状态.
session manager是通过给应用程序调用是添加一个特殊的参数来引导应用程序加载状态的.
对于Emacs来说,该参数为"--smid SESSION"

-   emacs-save-session-functions

Emacs通过调用该hook来支持状态的保存.

当session manager通知Emacs说window系统要关闭时,Emacss会调用该hook中的函数.
  每个函数调用是都不带参数,并且会将当前buffer设置为临时buffer.
最终,Emacs保持buffer到文件中,该文件被成为"session file"

当session manager重启挂起的Emacs时,Emacs会自动通过执行函数\`emacs-session-restore'来加载session file

若\`emacs-save-session-functions'中有函数返回非nil的值,则Emacs会通知Session manager取消这次关闭.


### Desktop Notificatino {#desktop-notificatino}

若Emacs编译时开启了D-Bus支持,则通过加载\`notifications'库,Emacs可以给某些操作系统发送"通知"

-   (notifications-notify &amp;rest params)

通过D-Bus,使用Freedesktop notification protocol发送通知,该函数返回一个整数作为通知的id

参数params可以是如下keyword参数

-   :bus BUS

    D-Bus bus. 该参数只有在bus不是\`:session'时使用

-   :title TITLE

    通知的标题

-   :body TEXT

    通知的内容. 某些notification server甚至支持HTML标签

-   :app-name NAME

    发送通知的应用程序名称. 默认为\`notifications-application-name'

-   :replaces-id ID

    表示该通知要替代指定id的原先通知. ID必须是之前\`notifications-notify'调用的返回值

-   :app-icon ICON-FILE

    通知的图标文件. 若ICON-FILE为nil则不显示图标. 默认为\`notifications-application-icon'

-   :actions (KEY TITLE KEY TITLE...)

    一系列要应用的动作. KEY和TITLE都是字符串. 其中TITLE会在通知上以按钮的形式展现.

    若要设置默认动作(通常该动作在点击notification时触发)的key为"default".

-   :timeout TIMEOUT

    显示多少毫秒后自动关闭. 默认值-1表示超时时间遵照notification server的设置. 0表示无限时间

-   :urgency URGENCY

    紧急的级别. 可以是'low,'normal或'critical

-   :action-items

    若设置了该关键字,则TITLE string of the action也被解释为icon name

-   :category CATEGORY

    通知的类型,字符串格式

-   :desktop-entry FILENAME

    This specifies the name of the desktop filename representing the calling program, like \`"emacs"'.

-   :image-data (WIDTH HEIGHT ROWSTRIDE HAS-ALPHA BITS CHANNELS DATA)

    这是一个raw data 图像格式描述了宽,高,rowstride,是否有alpha通道,每个sample的比特数,通道和图像数据

-   :image-path PATH

    PATH为一个URI(目前只支持"file"类型)或"$XDG_DATA_DIRS/icon"目录下的某个icon theme的名称

-   :sound-file FILENAME

    弹出通知时,播放声音文件

-   :sound-name NAME

    "$XDG_DATA_DIRS/sound"目录下定义的sound theme

-   :suppress-sound

    若设置了,则不播放任何声音.

-   :resident

    若设置了该参数,则即使对该通知做了动作,该通知也不会自动关闭,除非明确的对该通知发出关闭操作

-   :transient

    若设置了该参数,则server会认为该通知是暂时的,而忽略server的持久化能力(?When set the server will treat the notification as transient and by-pass the server's persistence capability, if it should exist?)

-   :x POSITION / :y POSITION

    定义通知在屏幕上的显示位置

-   :on-action FUNCTION

    当按下了表示action的按钮时,会调用该函数. 该函数接受两个参数:notification id和action key

-   :on-close FUNCTION

    当通知因为超时或人为关闭时调用该函数. 该函数接受两个参数:notification id和关闭的REASON.

    函数中的REASON参数可以是:

    -   'expired

    由于超时而关闭

    -   'dismissed

    被人为关闭

    -   'close-notification

    通过调用\`notifications-close-notification'函数来关闭

    -   'undefined

    notification server未告知原因
    ```emacs-lisp
    (defun my-on-action-function (id key)
    (message "Message %d, key \"%s\" pressed" id key))
    ;; => my-on-action-function

    (defun my-on-close-function (id reason)
    (message "Message %d, closed due to \"%s\"" id reason))
    ;; => my-on-close-function

    (notifications-notify
    :title "Title"
    :body "This is <b>important</b>."
    :actions '("Confirm" "I agree" "Refuse" "I disagree")
    :on-action 'my-on-action-function
    :on-close 'my-on-close-function)
    ;; => 22

    这时会弹出一个message window. 按下 "I agree"
    ;; => Message 22, key "Confirm" pressed
    ;;    Message 22, closed due to "dismissed"
    ```

-   (notifications-close-notification notification-id &amp;optional bus)

    关闭指定id的通知. 参数BUS可以是一个表示D-Bus连接的字符串.默认为:session

-   (noifications-get-capabilities &amp;optional bus)

    返回notification server支持的特性的列表.

    它的返回值是一个由sybmol组成的list:

    -   :actions

    The server will provide the specified actions to the user

    -   :body

    支持定义Body的内容

    -   :body-hyperlinks

    body中支持超链接

    -   :body-images

    body中支持嵌入图片

    -   :body-markup

    支持在body中嵌入标记

    -   :icon-muti

    server会把图片数组中的每帧整合成一个动画

    -   :icon-static

    server只显示图片数组中的地一个帧图片,该参数与:icon-multi互斥

    -   persistence

    支持通知的持久化

    -   :sound

    支持播放声音

-   (notifications-get-server-information)

    以字符串列表的形式返回notification server的信息.

    返回值的格式为'(NAME VENDOR VERSION SPEC-VERSION). 其中:

    -   NAME 为server的产品名称

    -   VENDOR 为vendor名称. 常见的有"KDE"和"GNOME"

    -   VERSION 为server的版本号

    -   SPEC-VERSION 为The specification version the server is compliant with


### File Notification {#file-notification}

若编译Emacs时链接了\`gfilenotify',\`inotify',\`w32notify'或其他对应的库,通过加载\`filenotify'库,Emacs可以监视文件系统中文件的改变.

-   (file-notify-add-watch file flag callback)

为文件FILE添加监视器. 该函数返回添加的监视器的descriptor.

若FILE无法被监视,则该函数抛出错误\`file-notify-error'

注意: **有些文件系统不能监视文件是否发生改变. 因此该函数返回非nil的值并不一定表示文件的改变一定会通知到Emacs**

参数FLAG为一个列表,指明了要监视FILE的哪些改变:

-   'change

    文件内容改变

-   'attribute-change

    文件属性改变

参数FILE可以是某个文件,也可以是目录. 若FILE为目录,则会监视该目录下的所有文件,但不递归监视子目录.

当监视的事件发生时,Emacs会调用CALLBACK函数,并传递EVENT作为参数. 其中EVENT的格式为:
'(DESCRIPTOR ACTION FILE [FILE1])

其中DESCRIPTOR即为监视器的描述符

ACTION为事件描述,它的可能值有:

-   'created
-   'deleted
-   'changed
-   'renamed
-   'attribute-changed

FILE和FILE1为事件发生的文件名称.

下面是一个例子

```emacs-lisp
(require 'filenotify)
;; => filenotify

(defun my-notify-callback (event)
(message "Event %S" event))
;; => my-notify-callback

(file-notify-add-watch
"/tmp" '(change attribute-change) 'my-notify-callback)
;; => 35025468

(write-region "foo" nil "/tmp/foo")
;; => Event (35025468 created "/tmp/.#foo")
;;    Event (35025468 created "/tmp/foo")
;;    Event (35025468 changed "/tmp/foo")
;;    Event (35025468 deleted "/tmp/.#foo")

(write-region "bla" nil "/tmp/foo")
;; => Event (35025468 created "/tmp/.#foo")
;;    Event (35025468 changed "/tmp/foo") [2 times]
;;    Event (35025468 deleted "/tmp/.#foo")

(set-file-modes "/tmp/foo" (default-file-modes))
;; => Event (35025468 attribute-changed "/tmp/foo")
```

根据不同的底层库的实现不同,rename操作可能被认为是一个rename操作或一个delete+create操作

It can be expected, when a directory is watched, and both FILE and FILE1 belong to this directory.  Otherwise, the actions \`deleted' and \`created' could be returned in a random order.

```emacs-lisp
(rename-file "/tmp/foo" "/tmp/bla")
;; => Event (35025468 renamed "/tmp/foo" "/tmp/bla")

(file-notify-add-watch
"/var/tmp" '(change attribute-change) 'my-notify-callback)
;; => 35025504

(rename-file "/tmp/bla" "/var/tmp/bla")
;; => ;; gfilenotify
;;    Event (35025468 renamed "/tmp/bla" "/var/tmp/bla")

;; => ;; inotify
;;    Event (35025504 created "/var/tmp/bla")
;;    Event (35025468 deleted "/tmp/bla")
```

-   (file-notify-rm-watch descriptor)

删除指定的监视器


### 动态加载 {#动态加载}

Emacs supports such on-demand loading of support libraries for **some of its features.**

需要注意: **不是所有的特性都能使用这种方式动态加载**

-   dynamic-library-alist

该参数是一个由动态库和对应的动态库文件组成的alist.

该alist中每个元素的格式为'(LIBRARY FILES...)

例如:

```emacs-lisp
(setq dynamic-library-alist
      '((xpm "libxpm.dll" "xpm4.dll" "libXpm-nox4.dll")
        (png "libpng12d.dll" "libpng12.dll" "libpng.dll"
             "libpng13d.dll" "libpng13.dll")
        (jpeg "jpeg62.dll" "libjpeg.dll" "jpeg-62.dll"
              "jpeg.dll")
        (tiff "libtiff3.dll" "libtiff.dll")
        (gif "giflib4.dll" "libungif4.dll" "libungif.dll")
        (svg "librsvg-2-2.dll")
        (gdk-pixbuf "libgdk_pixbuf-2.0-0.dll")
        (glib "libglib-2.0-0.dll")
        (gobject "libgobject-2.0-0.dll")))
```


### 定时器 {#定时器}

Emacs只有在空闲时才会调用定时器,即在等待输入,\`site-for'函数和\`read-event'函数时才会触发定时器.

Emacs会在触发定时器前将\`inhibit-quit'设置为t,这是因为从定时器函数中推出很容易产生不一致状态.

通过定时器函数修改buffer会导致undo命令分不清哪些改动是人工改动的,哪些改动是触发器函数修改的. 因此一般不再定时器函数中修改buffer,若要修改,注意在修改buffer前和修改buffer后调用一次\`undo-boundary'函数

定时器函数也应该避免调用会导致Emacs等待的函数,因为这样的话又会导致其他定时器函数的处罚,可能产生无法预期的后果. 若真要等待一段时间后再执行,请再分配一个新的定时器来处罚

If a timer function calls functions that can change the match data, it should save and restore the match data

-   命令(run-at-time time repeat function &amp;rest args)

设置等时间为TIME时,用参数ARGS调用FUNCTON函数.

若参数repeat为某个数字,则会每隔repeat秒后执行一次. 若repeat为nil表示只执行一次

参数TIME可以是一个相对时间或一个绝对时间:

> Absolute times may be specified using a string with a limited
>   variety of formats, and are taken to be times <span class="underline">today</span>, even if
>   already in the past.  The recognized forms are \`XXXX', \`X:XX', or
>   \`XX:XX' (military time), and \`XXam', \`XXAM', \`XXpm', \`XXPM',
>   \`XX:XXam', \`XX:XXAM', \`XX:XXpm', or \`XX:XXPM'.  A period can be
>   used instead of a colon to separate the hour and minute parts.
>
> To specify a relative time as a string, use numbers followed by
> units.  For example:
>
> \`1 min'
>       denotes 1 minute from now.
>
> \`1 min 5 sec'
>       denotes 65 seconds from now.
>
> \`1 min 2 sec 3 hour 4 day 5 week 6 fortnight 7 month 8 year'
>       denotes exactly 103 months, 123 days, and 10862 seconds from
>       now.
>
> For relative time values, Emacs considers a month to be exactly
> thirty days, and a year to be exactly 365.25 days.
>
> Not all convenient formats are strings.  If TIME is a number
> (integer or floating point), that specifies a relative time
> measured in seconds.  The result of \`encode-time' can also be used
> to specify an absolute value for TIME.
>
> In most cases, REPEAT has no effect on when <span class="underline">first</span> call takes
> place--TIME alone specifies that.  There is one exception: if TIME
> is \`t', then the timer runs whenever the time is a multiple of
> REPEAT seconds after the epoch.  This is useful for functions like
> \`display-time'.
>
> The function \`run-at-time' returns a timer value that identifies
> the particular scheduled future action.  You can use this value to
> call \`cancel-timer' (see below).

-   配置项timer-max-repeats

由于Emacs只有在空闲时才会执行定时器,这时有可能会推迟定时器的执行. 但推迟定时器执行的动作不会对下一次定时器执行的时间产生影响.
即使说,若下一次定时器函数触发的时间并不是在上一次执行时间的基础上累加的. 因此推迟的时间太长,很可能会致使连续几次执行定时器函数.

该变量则限定了一次性定时器函数所能执行的最大的循环次数

-   命令(run-with-idle-time secs repeat function &amp;rest args)

设置一个定时器,该定时器在下次Emacs空闲了SECS秒后执行

参数SECS可以是数字,也可以是\`current-dile-time'返回的对象

若参数REPEAT为nil表示定时器只触发一次. 若为非nil表示 **每次Emacs空闲时间超过SECS秒后都触发一次**

>  Do not write an idle timer function containing a loop which does a
> certain amount of processing each time around, and exits when
> \`(input-pending-p)' is non-\`nil'.  This approach seems very natural but
> has two problems:
>
> -   It blocks out all process output (since Emacs accepts process
>
> output only while waiting).
>
> -   It blocks out any idle timers that ought to run during that time.
>
> Similarly, do not write an idle timer function that sets up another
> idle timer (including the same idle timer) with SECS argument less than
> or equal to the current idleness time.  Such a timer will run almost
> immediately, and continue running again and again, instead of waiting
> for the next time Emacs becomes idle.  The correct approach is to
> reschedule with an appropriate increment of the current value of the
> idleness time, as described below.

-   (current-idle-time)

若Emacs处于空闲状态,该函数返回Emacs空闲的时间. 格式与\`current0time'一致,为(SEC-HIGH SEC-LOW MICROSEC PICOSEC

若Emacs不处于idle状态,\`current-idle-time'返回nil

-   (cancel-timer timer)

取消定时器

-   宏(with-timeout (seconds timeout-forms...) bodys...)

执行bodys,但设置了超时时间为SECONDS秒.

若BODY在SECONDS秒内执行完成,则\`with-timeout'返回正常的bodys执行结果.

但若BODY在执行期间超时了,则会执行timeout-forms,并返回timeout-forms的执行结果

该宏的内部使用定时器来完成超时功能,因此若Emacs执行bodys的过程中,没有等待输入的情况,则超时的功能没什么作用.


## Dired-mode相关函数 {#dired-mode相关函数}

-   获取dired中marked file

使用函数\`dired-get-marked-files\`


## 执行命令 {#执行命令}

-   执行shell命令并等待shell命令结束

(shell-command "shell命令")

-   执行shell命令,等待shell命令结束,并获得命令的输出

(shell-command-to-string "shell命令")

-   使用外部命令对所选择Region进行处理

shell-command-on-region

-   执行shell命令,但是不等待shell命令结束

start-process

start-process-shell-command

call-process-region


## Register函数 {#register函数}

-   将内容复制到Register中

(copy-to-register ?1 startPos endPos)

-   从Register中取出内容

(insert-register ?1 t)


## Org相关函数 {#org相关函数}

取entry属性
: (org-entry-get nil "属性名" 是否继承属性)

取entry的tag list
: (org-get-tags)

取entry的TODO state
: org-get-tags

判断哪些state是完成状态
: org-done-keywords

跳转到headline
: (org-back-to-heading)

判断是否处于clockin状态
: org-clock-clocking-in

clock in
: (org-clock-in)

clock out
: (org-clock-out)

当前clock entry的headline
: org-clock-heading

clock out触发的hook
: org-clock-out-hook

更新mode-line的内容
: (org-clock-update-mode-line)


## 版本信息 {#版本信息}

-   变量emacs-version
-   变量emacs-major-version
-   变量emacs-minor-version


## wait function {#wait-function}

wait function用于等待一段时间或等待input.

-   (sit-for seconds &amp;optional nodisp)

sit-for等待一段时间或直到收到某个input event. 在这段时间内Emacs会实时更新屏幕显示.

若等待不是由收到input而中断的,则返回t,否则返回nil

参数seconds可以是浮点数. (sit-for 0)等价于(redisplay),它只是立即更新屏幕显示.

若参数nodisp为非nil,则sit-for在等价期间不会更新屏幕显示.

在Emacs处于batch mode时,\`sit-for'不能被interrupt event中断,这时等价于\`sleep-for'

-   (sleep-for seconds &amp;optional millisec)

该函数等价seconds秒,等待期间不会被input event打断,也不会更新屏幕显示. 它总是返回nil

参数seconds可以为浮点数.

参数millisec表示在等待seconds秒后,再等待millisec毫秒


## Package相关说明 {#package相关说明}


### Package的分类 {#package的分类}

package可以分为单文件package和多文件package. 其中单文件package在package archive中以单个elisp文件形式存在,而多文件package以tar文件形式存在.


#### 单文件Package {#单文件package}

单文件package中elisp文件必须遵循[elisp library header conventions](elisp:Library%20Headers.html "Emacs Lisp: (info \"(elisp:Library%20Headers) Top\")")
下面是一个例子

```emacs-lisp
;;; superfrobnicator.el --- Frobnicate and bifurcate flanges

;; Copyright (C) 2011 Free Software Foundation, Inc.

;; Author: J. R. Hacker <jrh@example.com>
;; Version: 1.3
;; Package-Requires: ((flange "1.0"))
;; Keywords: multimedia, frobnicate
;; URL: http://example.com/jrhacker/superfrobnicate


;;; Commentary:

;; This package provides a minor mode to frobnicate and/or
;; bifurcate any flanges you desire.  To activate it, just type

```

package的名称与elisp文件的base name必须一致. package名称被写到第一行. 在上例中为"superfrobnicator"

单行描述也放在第一行,在上例中为"Frobnicate and bifurcate flange"

package的版本号,通过\`Package-Version' header表示,若不存在\`Package-Version' header则通过\`Version' header标识. 该属性必须存在. 在上例中为1.3

若文件中有\`;;; Commentary:'部分,则该部分内容作为package的详细描述.

若文件中有\`Package-Requires' header,则该部分标识了package的依赖包有哪些. 若没有该header表示package不依赖其他包.

\`Keywords'与\`URL'也是可选的,但一般建议添加这两个header. \`describe-package'函数会展示这两个header中的值.
\`Keywords' header应该至少包含一个\`finder-known-keywrods'中的标准keyword


#### 多文件Package {#多文件package}

多文件package不像单文件package那样有那么多的规范.

多文件package在package archive中以一个tar文件的形式存在. 该tar文件的名称必须是\`package名-package版本.tar'.

多文件package的content directory中必须有一个名为\`package名-pkg.el'的文件.
该文件的内容为一个\`define-package'函数的调用form. \`define-package'函数定义了package的版本,单行描述和依赖包.

若content directory中包含名为\`README'的文件,则该文件的内容作为package的详细描述

若content directory中包含名为\`dir'的文件,则该文件被认为是通过命令\`install-info'生成的Info directory文件. 同时,相关的Info文件也必须在content directory中.
这中情况下,当激活该package后,Emacs会自动将content directory添加到\`Info-directory-list'中.

**不要在package中包含任何.elc文件,它们是由package安装时自动生成的.且无法控制各文件编译的顺序**

**不要包含名为\`package名-autoloads.el'的文件,它也是由package安装时自动生成的.**

若package中包含其他格式的文件(例如图片等),则在elisp源代码中,可以使用\`load-file-name'变量值来引用存放这些文件的目录. 例如

```emacs-lisp
(defconst superfrobnicator-base (file-name-directory load-file-name))

(defun superfrobnicator-fetch-image (file)
  (expand-file-name file superfrobnicator-base))
```

<!--list-separator-->

-  (define-package name version &amp;optional docstring requirements)

    该函数定义了一个名为\`NAME'的package.

    参数\`VERSION'为一个表示版本的字符串,该字符串的格式必须被\`version-to-list'所认识.

    参数\`DOCSTRING'为单行描述

    参数\`REQUIREMENTS'为一个由依赖包和最低版本组成的list. 该list的每个元素应该是(DEP-NAME-symbol DEP-VERSION-string)


### Package的属性 {#package的属性}

每个package都具有如下一些属性:

-   Name

    pacakge name,常在程序中作为变量/函数名的前缀.

-   Version

    版本号,其格式必须被函数\`version-to-list'所知. 每次release package都应该增加该版本号属性

-   单行简洁描述

    该描述应该只占一行,最好不超过36个字符. 该描述会在list package时展示

-   详细描述

    常常包含该package的用处,以及使用方法. 该描述会在\`describe-package'中展示

-   Dependencies

    依赖关系, 该属性值为其他package的list. 该list中的每个元素可以是一个表示package的symbol或一个(package min-version)的列表.


### package的安装说明 {#package的安装说明}

通过\`pacakge-install-file'可以安装一个package. 安装package包含以下几个步骤:

1.  在\`package-user-dir'目录下创建一个名为\`package名称-package版本'的目录.

    该目录用于存放package的内容的,因此也被称为"content direcotry"

2.  Emacs搜索content directory中的所有elisp文件,并查找里面的autoload magic comments

    这些autoload定义被保存在名为\`package名称-autoloads.el'的文件中.

    这些autoload定义通常用来自动加载package中定义的供用户使用的主要command,但也能用来作其他事情,例如王\`auto-mode-alist'中添加内容

3.  Emacs编译package中的所有elisp文件


### package的加载过程 {#package的加载过程}

Emacs启动过程中,当Emacs加载完初始化完init file及abbrev file后,在触发\`after-init-hook'前,Emacs会自动调用\`package-initialize'函数来加载安装的package.

但是Emacs加载package的过程,在变量\`package-enable-at-startup'为nil时会被禁止.

Emacs加载package的过程由两个步骤组成

1.  添加package的content directory到\`load-path'中

2.  执行\`package名-autoloads.el'中的autoload定义.


#### (package-initialize &amp;optional no-activate) {#package-initialize-and-optional-no-activate}

该函数初始化Emacs用于记录哪些package已经安装的内部变量并加载这些package.

用户配置项\`package-load-list'指明了哪些package会被加载,默认是所有已经安装的package.

参数\`NO-ACTIVATE'若为非nil,则表示只更新已安装package的记录,而不加载这些package


### Package Archives {#package-archives}

-   配置项package-archives

    该配置项为一个alist,指明了Emacs包管理系统从哪些archive中搜索package

    该alist的每个元素的格式为(ID . LOCATION),这里ID为表示archive名称的字符串,LOCATION为表示archive地址的字符串.
    目前LOCATION只能为http url或一个本地目录的路径.

    一个package archive只是一个包含了package源代码文件及其相关文件的目录而已.
    若希望可以通过http获取该archive,则需要把该目录放在web server上.

一种设置和更新package archive的比较方便的方式是使用库\`package-x',该库是Emacs自带的,但是默认情况下并不加载.

\`package-x'的相关命令与变量如下:

-   配置项pacakge-archive-upload-base

    该变量的值应为一个目录的名称,它将作为package archive的base location.

    该变量的值必须是一个 **绝对路径**.

    若package archive不在本地,可以使用类似\`/ssh:foo@example.com:/var/www/packages/' 的形式来设置

-   (package-upload-file filename)

    该命令上传FILENAME到\`package-archive-upload-base'中.

    FILENAME必须是单文件package(el文件),多多文件package(tar文件),否则会抛出错误.

    若\`package-archive-upload-base'中的路径非法,则会提示用户重新输入,若路径不存在,则会自动创建该目录

-   (package-upload-buffer)

    类似\`package-upload-file',只不过是将当前buffer的内容上传上去.

    该buffer必须访问一个单文件package(el文件)或多文件pacakge(tar文件),否则会抛出错误
