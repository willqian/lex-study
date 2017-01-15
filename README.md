# lex yacc 学习 #

## lex yacc参考文档 ##
- http://dinosaur.compilertools.net
- http://www.ibm.com/developerworks/cn/linux/l-lexyac.html

## code 01 ##

	%%
	[ \t]+$ ;
	[ \t]+  printf(" ");
	%%
	main() {
	    return yylex();
	}

	int yywrap() {
	    return 1;
	}

### 代码说明 ###
- `%%` 是分隔符，第一个 `%%` 下面是正则表达式规则，第二个 `%%` 下面是主程序部分
- `[ \t]` 是匹配空格或 `tab`
- `+` 是匹配 `1` 至多个
- `$` 是 `end of line` 行尾
- 动作 `;` 分号，表示该匹配不做任何操作，对应的字符也不会回显
- `printf(" ")` 则是自定义回显方式，这一行代码把非行尾的空格或 `tab` 显示为 `1` 个空格
- `main()` 默认取 `stdin` 标准输入流，通过调用 `yylex` 开始词法解析
- `yywrap()` 则是告诉程序，当流结束时，如何处理
	- 返回 `1` 说明结束程序扫描
	- 返回 `0` 则继续扫描

### 运行方法 ###
`./test` 然后从终端输入字符串即可

## code 02 ##

	%{
	int num_lines;
	int num_chars;
	%}
	%%
	\n      ++num_lines; ++num_chars;
	.       ++num_chars;
	%%
	main() {
	    yylex();
	    printf("# of lines = %d, # of chars = %d\n", num_lines, num_chars);
	}
	
	int yywrap() {
	    return 1;
	}

### 代码说明 ###
- `%%` 之上，是定义区，需要用 `%{` 和 `%}` 包住
- `\n` 是匹配换行，匹配成功时执行 `num_lines++` 以及 `num_chars++`
- `\.` 点号是匹配除 `\n` 之外的所有字符，匹配成功时执行`num_chars++`
- `main()` 在 `yylex()` 解析完成后，打印 `num_lines` 和 `num_chars`

### 运行方法 ###
`./test` 然后从终端输入多行字符串，回车， `ctrl + D` 即可看到输出

## 正则表达式 ##

### 关键字 ###
`"` `\` `[` `]` `^` `-` `?` `.` `*` `+` `|` `(` `)` `$` `/` `{` `}` `%` `<` `>`

关键字可以通过 `"` 双引号或 `\` 反斜杠转义，例如：  
`abc"++"` 及 `abc\+\+ ` 表示匹配 `abc++` 字符串

### 空白字符 ###
`\t` `\n` `\b` 分别是`tab` `newline` `backspace`

反斜杠`\`本身则是`\\`

### `[]`方括号 ###
- `[abc]` 表示匹配 `a` 或 `b` 或 `c`
- `\` `-` `^` 是 `[]` 中可解析的符号
	- `-` 表示范围，例如 `[a-z]` 匹配 `a` 到 `z` 的字符
	- `-` 若要转义，则需要放在 `[]` 的最前和最后，例如 `[-+0-9]`，第一个 `-` 就解析为 `-` 符号本身
	- `^` 表示取反，必须在 `[]` 的开头，例如 `[^abc]` 表示所有的不是`a` `b` `c` 的字符， `[^a-zA-Z]` 表示所有的非字母
	- `\` 表示数字串， `[\40-\176]` 表示匹配 `40` 到 `176` 的数字

### 其他正则表达式 ###
- `?` 匹配 `0` 或 `1` 个对应字符，`ab?c` 匹配 `abc` 或 `ac`
- `a*` 匹配 `0` 到多个 `a`
- `a+` 匹配 `1` 到多个 `a`
- `|` 匹配左右之一，`ab|cd` 匹配 `ab` 或 `cd`
- `(ab|cd+)?(ef)*` 匹配 `abefef` `efefef` `cdef` `cddd` 而不匹配 `abc` `abcd` `abcdef`
- `^` 匹配行的开头
- `$` 匹配行的结尾
- `/` 条件匹配， `ab/cd` 匹配 `ab`，但必须在 `cd` 之前，`ab$` 也就是 `ab/\n`
- `{digit}` 匹配指定的多个，例如 `a{1,5}` 匹配 `1` 到 `5` 个 `a`，`a{2}` 匹配 `2` 个 `a`
- `<>` 是用来处理上下文前导状态，参考 `Left Context Sensitivity`

## lex Action ##

### 要点 ###
- 有多个代码需要执行时，需要用大括号包住
- 当匹配了正则表达式后，会执行对应的 `action`
- 默认 `action` 是把标准输入直接传到标准输出，因此，当没有匹配时，也会有默认输出
- 直接 `;` 表示无操作
- 当用户想知道正则表达式到底匹配的是什么值，可以使用 `yytext`， 如下：  
  `[a-z]+   printf("%s", yytext);`
- 为了方便，也有宏可以表示同样的含义：  
  `[a-z]+   ECHO;`
- `yyleng` 可以得到匹配的长度，例如：  
  `[a-zA-Z]+ {words++; chars += yyleng;}`
- `yytext[yyleng-1]` 则可以得到最后匹配到的 `1` 个字符
- `|` 表示当前 `action` 使用下一个正则匹配的 `action`
- 如果当前词法解析不符合要求时，可以通过 `yymore()` 追加到当前的 `input` 中
- `yyless(n)` 则表示当前词法不需要这么多 `input` ，剩下 `n` 个字符保留在 `yytext` 中   

### 示例 ###

	\"[^"]*     {
		        if (yytext[yyleng-1] == '\\')
    	            yymore();
	            else
    	            ... normal user processing
	            }

这里说明遇到 `\` 时，保留 `yytext` 中的数据，需要继续执行解析  
例如， `"abc\"def"` 先匹配 `"abc\`，执行 `yymore()`，然后匹配 `"def`，并连接为 `"abc\"def`，最后匹配 `"`


	=-[a-zA-Z]  {
	            printf("Op (=-) ambiguous\n");
	            yyless(yyleng-1);
	            ... action for =- ...
	            }

这里说明遇到 `=-` 符号时，语法是多义的，需要减少一个字符去解析，也就是不解析 `-` 符号  
`=-a` 会解析为 `= -a` 这个意思，实际上，这里的正则表达式等价于 `=/-[A-Za-z]`  
如果要解析为 `=- a`，可以表示为 `=-/[^ \t\n]`

### IO处理 ###
- `input()` 返回下一个输入字符
- `output(c)` 输出 `c` 到输出
- `unput(c)` 返回 `c` 到 `input` 以供下次读取
- 输入流必须以 `NULL` 结尾
- `input` 和 `uninput` 必须保留原有做法，因为 `lookahead` 要用到，比如 `?` `+` `*` `$` `/` 都会用到 `lookahead`
- `yywrap()` 返回 `1` 时，是直接退出，返回 `0` 时，则允许程序继续读取输入流
