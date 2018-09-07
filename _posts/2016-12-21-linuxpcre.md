---
layout: post
title:  "Linux PCRE正则"
categories: Linux
tags: Linux PCRE 正则表达式
author: DJY
---

* content
{:toc}
# PCRE

------

PCRE 完全兼容 perl 格式的正则表达式，性能远高于 linux 自带的 regex.h 中的正则表达式。关于性能的比较参考：<http://sljit.sourceforge.net/regex_perf.html>

PCRE 包含在 glib 库中，使用的时候需要包含 pcre.h 的头文件，并将 libglib.a 和 libpcre.a 两个库链接进程序。

另外，PCRE 同样提供了 Posix 类型的 API，参考：<http://www.pcre.org/original/doc/html/pcreposix.html>

## 基本使用步骤

- 编译规则
- 进行匹配
- 释放内存

### Compile

```
#include <pcre.h>pcre *pcre_compile(const char *pattern, int options, const char **errptr, int *erroffset, const unsigned char *tableptr); 
```

#### 参数说明

| arg       | destription                                                  |
| --------- | ------------------------------------------------------------ |
| pattern   | 正则表达式字符串                                             |
| options   | 0 或其他选项，详情查看 <http://www.pcre.org/original/doc/html/pcre_compile.html> |
| errptr    | 用来保存错误输出的指针                                       |
| erroffset | 用来保存错误位置的偏移量                                     |
| tableptr  | 字符表的指针，指定为 NULL 则直接用内建的字符表               |

#### 返回值

成功返回一个 pcre 类型的指针，失败返回 NULL

### Compare

```
int pcre_exec(const pcre *code, const pcre_extra *extra, const char *subject, int length, int startoffset, int options, int *ovector, int ovecsize); 
```

#### 参数说明

| arg         | description                                                  |
| ----------- | ------------------------------------------------------------ |
| code        | pcre_compile 返回的编译好的规则                              |
| extra       | pcre_study 返回的结果，没有则为 NULL                         |
| subject     | 需要匹配的字符串                                             |
| length      | 需要匹配的字符串的长度                                       |
| startoffset | 开始匹配的 subject 的偏移量                                  |
| options     | 选项                                                         |
| ovector     | 一个 int 类型数组，用来保存匹配结果的偏移量，如果匹配，则会有多个匹配的 group，这个数组就保存了各个 group 在 subject 中的偏移量 |
| ovecsize    | ovector 的大小，3 的倍数，如果小于所需的长度会导致 core dump |

#### 返回值

如果匹配则返回其实匹配的偏移量，否则返回 -1。

### Free

```
pcre_free(pcre *ctx);
```

释放 pcre_compile 占用的内存。

## 优化

- study
- send study result to pcre_result
- free

### Study

```
pcre_extra *pcre_study(const pcre *code, int options, const char **errptr); 
```

#### 参数说明

| arg     | description                                    |
| ------- | ---------------------------------------------- |
| code    | 已编译的规则，也即 pcre_compile 的返回结果     |
| options | PCRE_STUDY_JIT_COMPILE 为 pcre_exec 的唯一选项 |
| errptr  | 保存错误输出的字符串                           |

#### 返回值

成功则返回 pcre_extr 类型的指针；如果返回值为 NULL，且 errptr 为 NULL，则说明找不到可以优化的地方，如果返回值为 NULL 且 errptr 不为 NULL，说明出错

### Free

```
pcre_free_study(pcre_extr *extra_ctx);
```

## 优化后的基本步骤

- 编译规则
- 学习
- 执行比较
- 释放学习使用的内存
- 释放编译规则使用的内存

## 示例

### Code

```
char *zpagent_config_get_zone_id(char *line, char *zone_id){    char *pattern = "^\\s*zone\\s*:\\s*(<(([^<>\\s])+)>)\\s*$";    static pcre *ctx = NULL;    static pcre_extra *extra_ctx = NULL;    int err_offset = -1;    int ovecount = 12;    int ovector[ovecount];    const char *err;    int ret, i;    char *substring_start;    if(line == NULL) {        return NULL;    }    if (ctx == NULL) {        ctx = pcre_compile(pattern, 0, &err, &err_offset, NULL);        if (ctx == NULL) {            printf("Error: %s: %s found in %d\n", __FUNCTION__, err, err_offset);            return NULL;        }    }    if (extra_ctx == NULL) {        extra_ctx = pcre_study(ctx, PCRE_STUDY_JIT_COMPILE, &err);        if (extra_ctx == NULL) {            if (err == NULL) {                printf("Warning: %s: pcre could not find any additional information\n", __FUNCTION__);            } else {                printf("Warning: %s: pcre error: %s\n", __FUNCTION__, err);            }        }    }    ret = pcre_exec(ctx, extra_ctx, line, strlen(line), 0, 0, ovector, ovecount);    if (ret < 0) {        // Not match        return NULL;    }    for (i = 0; i < ret; i++) {        char *substring_start = line + ovector[2 * i];          int substring_length = ovector[2 * i+1] - ovector[2 * i];          printf("%2d: %.*s\n", i, substring_length, substring_start);      }     memset(zone_id, 0, ZONAL_ZONE_ID_MAX_LEN);    strncpy(zone_id, line + ovector[2] + 1, ovector[3] - ovector[2] -2);    pcre_free(ctx);    pcre_free_study(extra_ctx);    return zone_id;}
```

# POSIX

------

Linux 系统标准库中自带了正则表达式实现，性能比较差，但不需要引用其它的第三方库。

使用的时候只需要在代码中包含 regex.h 的头文件，即可使用自带的正则表达式函数。

## 步骤

### 1. 编译正则表达式 regcomp()

```
int regcomp (regex_t *compiled, const char *pattern, int cflags)
```

    将正则表达式 pattern 编译为 compiled，函数 regexec 用 compiled 作为正则表达式进行匹配。

    执行成功返回 0。

#### 参数说明：

1. regex_t 是一个结构体数据类型，用来存放编译后的正则表达式，它的成员re_nsub 用来存储正则表达式中的子正则表达式的个数，子正则表达式就是用圆括号包起来的部分表达式。

2. pattern 表示正则表达式的字符串，注意需要在普通的正则表达式的基础上用转义字符 ‘\’ 将特殊字符转换成普通字符，如在外部为 zone\s*:\s*(<.+>) 的正则表达式，在 C 语言中作为字符串需要改为：”zone\s*:\s*(<.+>)”

3. cflags 可用如下四个值或运算 (|) 后的值：

    

   ​

   - REG_EXTENDED 以功能更加强大的扩展正则表达式的方式进行匹配
   - REG_ICASE 匹配字母时忽略大小写
   - REG_NOSUB 不用存储匹配后的结果
   - REG_NEWLINE 识别换行符，这样’$’就可以从行尾开始匹配，’^’就可以从行的开头开始匹配

#### regex_t

```
typedef struct re_pattern_buffer regex_t;struct re_pattern_buffer{  /* Space that holds the compiled pattern.  It is declared as     `unsigned char *' because its elements are sometimes used as     array indexes.  */  unsigned char *__REPB_PREFIX(buffer);  /* Number of bytes to which `buffer' points.  */  unsigned long int __REPB_PREFIX(allocated);  /* Number of bytes actually used in `buffer'.  */  unsigned long int __REPB_PREFIX(used);  /* Syntax setting with which the pattern was compiled.  */  reg_syntax_t __REPB_PREFIX(syntax);  /* Pointer to a fastmap, if any, otherwise zero.  re_search uses the     fastmap, if there is one, to skip over impossible starting points     for matches.  */  char *__REPB_PREFIX(fastmap);  /* Either a translate table to apply to all characters before     comparing them, or zero for no translation.  The translation is     applied to a pattern when it is compiled and to a string when it     is matched.  */  __RE_TRANSLATE_TYPE __REPB_PREFIX(translate);  /* Number of subexpressions found by the compiler.  */  size_t re_nsub;  /* Zero if this pattern cannot match the empty string, one else.     Well, in truth it's used only in `re_search_2', to see whether or     not we should use the fastmap, so we don't set this absolutely     perfectly; see `re_compile_fastmap' (the `duplicate' case).  */  unsigned __REPB_PREFIX(can_be_null) : 1;  /* If REGS_UNALLOCATED, allocate space in the `regs' structure     for `max (RE_NREGS, re_nsub + 1)' groups.     If REGS_REALLOCATE, reallocate space if necessary.     If REGS_FIXED, use what's there.  */#ifdef __USE_GNU# define REGS_UNALLOCATED 0# define REGS_REALLOCATE 1# define REGS_FIXED 2#endif  unsigned __REPB_PREFIX(regs_allocated) : 2;  /* Set to zero when `regex_compile' compiles a pattern; set to one     by `re_compile_fastmap' if it updates the fastmap.  */  unsigned __REPB_PREFIX(fastmap_accurate) : 1;  /* If set, `re_match_2' does not return information about     subexpressions.  */  unsigned __REPB_PREFIX(no_sub) : 1;  /* If set, a beginning-of-line anchor doesn't match at the beginning     of the string.  */  unsigned __REPB_PREFIX(not_bol) : 1;  /* Similarly for an end-of-line anchor.  */  unsigned __REPB_PREFIX(not_eol) : 1;  /* If true, an anchor at a newline matches.  */  unsigned __REPB_PREFIX(newline_anchor) : 1;};
```

### 2. 匹配正则表达式 regexec()

```
int regexec (regex_t *compiled, char *string, size_t nmatch, regmatch_t matchptr [], int eflags)
```

    使用上一步得到的 compiled 作为正则表达式，对字符串 string 进行匹配，nmatch 是 matchptr 数组的长度。

    执行成功返回 0，失败返回 REG_NOMATCH。

regmatch_t 是一个结构体数据类型，在regex.h中定义：

```
typedef struct{    regoff_t rm_so;    regoff_t rm_eo;} regmatch_t;
```

成员 rm_so 存放匹配文本串在目标串中的开始位置，rm_eo 存放结束位置。通常我们以数组的形式定义一组这样的结构。因为往往我们的正则表达式中还包含子正则表达式。数组0单元存放主正则表达式位置，后边的单元依次存放子正则表达式位置。

#### 参数说明

1. compiled 是已经用 regcomp 函数编译好的正则表达式

2. string 是目标文本串

3. nmatch 是 regmatch_t 结构体数组的长度

4. matchptr regmatch_t 类型的结构体数组，存放匹配文本串的位置信息

5. eflags 有两个值

    

   ​

   - REG_NOTBOL 匹配行起始符总是匹配失败（取决于上面的编译标识 REG_NEW_LINE），当传递给 regexec 的是字符串的不同部分并且这部分的开头不应作为行的开头进行处理时，可以使用这个标识。
   - REG_NOTEOL 匹配行结束符总是匹配失败（取决于上面的编译标识 REG_NEW_LINE）

### 3. 释放正则表达式 regfree()

```
void regfree (regex_t *compiled)
```

    用完编译的正则表达式或需要重用一个 regex_t 结构体的时候，用 regfree 将 compiled 指向的结构体内容清空。**重新编译前必须先清空**。

### 4. 错误处理 regerror()

```
size_t regerror (int errcode, regex_t *compiled, char *buffer, size_t length)
```

    当执行regcomp 或者regexec 产生错误的时候，就可以调用这个函数而返回一个包含错误信息的字符串。

## 示例

------

### Code

```
#include <stdio.h>#include <regex.h>int main() {    char *str = "   zone:<zone1>";    int cflags = REG_EXTENDED;    const size_t nmatch = 1;    regmatch_t pmatch[nmatch];    regex_t reg;    const char *pattern = "zone\\s*:\\s*(<.+>)";    int status;    regcomp(®, pattern, cflags);    status = regexec(®, str, nmatch, pmatch, 0);     if(status == REG_NOMATCH){        printf("no match\n");    } else {        printf("status: %d\n", status);        printf("pmatch[0].rm_so: %d, pmatch[0].rm_eo: %d\n", pmatch[0].rm_so, pmatch[0].rm_eo);    }    regfree(®);    return 0;}
```

### Output

```
status: 0pmatch[0].rm_so: 3, pmatch[0].rm_eo: 15
```

# 表达式

------

## C 语言的表达式格式

    由于正则表达式的表达式在 C 语言中是以字符串的形式存在，某些字符可能会和 C 语言的转义字符相冲突，故而必须在这里特殊对待，进行转义。

### 目前已知必须转移的字符有：

| 特殊字符 | 转义结果 |
| -------- | -------- |
| \        | \\       |
| {        | (\{)     |

### 示例

| 正则表达式  | 作为 C 语言字符串的正则表达式 |
| ----------- | ----------------------------- |
| ^\s*$       | ^\\s*$                        |
| ^\s*(})\s*$ | ^\\s*(\})\\s*$                |

## 常用表达式

| 表达式          | C 字符串            | 说明                                                         | 实例                                        |
| --------------- | ------------------- | ------------------------------------------------------------ | ------------------------------------------- |
| <([^<>\s])+>    | <([^<>\s])+>        | 匹配一个 <> 括号内除了 < > 和不可见字符以外的字符，字符长度为一个或以上 | “AP: name = <\zone_1023-ap_0>; type = <3G>” |
| ^\s*AP          | ^\\s*AP             | 匹配任意不可见字符开头，以 AP 结尾的字符串                   | ” AP”                                       |
| AP\s*$          | AP\\s*$             | 匹配以任意数量不可见字符作为结尾                             | “AP “                                       |
| [+-]?\d+(.?\d)* | [+-]?\\d+(\\.?\\d)* | 匹配小数                                                     |                                             |

# 参考

- [在线测试工具](https://regex101.com/#javascript)
- [正则表达式 - 语法 ](http://www.runoob.com/regexp/regexp-syntax.html)
- [正则表达式 - 元字符](http://www.runoob.com/regexp/regexp-metachar.html)
- [PCRE 发布的几个正则表达式实现的性能比较](http://sljit.sourceforge.net/regex_perf.html)
- [Perl-compatible Regular Expressions (PCRE) doc](http://www.pcre.org/original/doc/html/index.html)
- [posix 类型的 pcre 接口文档](http://www.pcre.org/original/doc/html/pcreposix.html)
- [PCRE函数简介和使用示例](http://blog.csdn.net/sulliy/article/details/6247155)
- [深入浅出C/C++中的正则表达式库(三)——PCRE, PCRE++ ](http://www.wuzesheng.com/?p=994)