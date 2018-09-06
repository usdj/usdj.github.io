---
layout: post
title:  "Google搜索技巧"
categories: 信息安全
tags: Google Search
author: DJY
---

* content
{:toc}

##Google Hack Search
==
1.1 默认模糊搜索、自动拆分短语  
`kali linux渗透测试`  
1.2 短语精确搜索  
给关键词添加半角双引号精确搜索，不进行分词  
`kali linux web渗透测试`  
1.3 通配符  
谷歌的通配符是星号“*”，必须在精确搜索符双引号内部使用。用通配符代替关键词或短语中无法确定的字词。  
`"kali * web渗透测试"`  
1.4 点号匹配任意字符  
与通配符星号“*”不一样的是，点号“.”匹配的是字符，不是字、短语等内容。保留的字符有[、(、-等。  
`"黑客.渗透测试"`  
1.5 布尔逻辑  
布尔逻辑是许多检索系统的基本检索技术，在搜索引擎中也一样适用，在谷歌网页搜索中需要注意的是：谷歌和许多搜索引擎一样，多个词间的逻辑关系默认的是逻辑与（空格）。当用逻辑算符的时候，词与逻辑算符之间用需要空格分隔，包括后面讲的各种语法，均要有空格。逻辑非是特例，即减号必须与对应的词连在一起。对于复杂的逻辑关系，可用括号分组。  

- 逻辑与  
默认  
    `渗透测试玄魂的博客`  
	`渗透测试 玄魂的博客`  
逻辑运算符AND    
	`渗透测试 AND 玄魂的博客`    
- 逻辑或  
    `"玄魂的博客"(kali|node)`  
- 逻辑非  
	`"玄魂的博客"-kali`  
1.6 约束条件  
加号“+”用于强制搜索，即必须包含加号后的内容。一般与精确搜索符一起应用。  
    `"玄魂的博客" +"愚蠢的人类"`  
1.7 数字范围  
用两个点号“..”表示一个数字范围。一般应用于日期、货币、尺寸、重量、高度等范围的搜索。用作范围时最好给一定的含义。  
    `kali linux 2010年..2014年`  
1.8 括号分组  
逻辑组配时分组，避免逻辑混乱。括号“()”是分组符号。  
	`"玄魂的博客"(kali|node)`   
2.1 标题中搜索  
通常标题是内容的高度概括，在标题中搜索的结果准确率会更高。谷歌搜索中限定搜索网页或文档标题的语法是：intitle或allintitle。allintitle是intitle的变体，相当于在各个搜索词前加上intitle。二者差别不明显，我们一般直接用intitle。
    `intitle:"WSO 2.4" [ Sec. Info ], [ Files ], [ Console ], [ Sql ], [ Php ], [ Safe mode ], [ String tools ], [ Bruteforce ], [ Network ], [ Self remove ]`  
2.2 正文中搜索  
仅仅在网页或文档的正文部分搜索。谷歌搜索中限定搜索网页或文档正文的语法是：intext或allintext。  
    `intitle:"index" intext:"Login to the Administrative Interface"`
此示例结合intitle和intext寻找Web2Py的管理后台。  
2.3 网址中搜索  
谷歌搜索中限定搜索网址的语法是：inurl。是In-系指令中最强大的一个，换句话说，这个高级指令能够直接从网站的URL入手挖掘信息，只要略微了解普通网站的URL格式，就可以极具针对性地找到你所需要的资源－－甚至隐藏内容。网站构建者通常将某一类信息集中在一个网站的目录中，所以搜索URL中的词本身就是对某一方面内容的一个限定。如果在加上一定的词进行组配，搜索结果将更贴近需求。
    `inurl:phpmyadmin/index.php & (intext:username & password & "Welcome to")`
该示例通过url中的特征路径和页面的中的特殊文本，寻找安装有phpmyadmin的站点的对应管理入口。  
2.4 锚链链接搜索  
在做网站中有时候用锚点来链接一个页面中的其它部分内容，这样方便浏览和定位。也就是说锚点链接的内容通常是网页内容中重要的章节或内容的开始部分，因而对它们的搜索也更能反映网页的主题内容，提高搜索结果的准确度。对于熟悉网页制作的人来说，可以从网页源代码中查看有锚点的HTML代码。     
谷歌网页搜索在锚链链接中语法是：inanchor或allinanchor。搜索范围限制在页面的链接锚点描述文本进行搜索。
    `inanchor:修改密码`
2.5 文档类型限定  
谷歌网页搜索不仅仅能搜索网页，还能搜索各种文档，通过文档类型限定只对文档进行搜索，从而不显示页面的内容。语法是：filetype。这个语法非常有用，我们在网上常常要找一些范文或参考资料的时候常用这个语法。filetype是根据文件后缀搜索特定文件类型，比如支持的文档有：pdf、ppt、doc、xls等；网页文件：htm、asp、php等。
    `filetype:sql site:com and "insert into" admin "2014"`
本示例通过filetype 查找sql文件，并且希望得到的结果为插入admin用户的脚本。  
3.1 搜索谷歌缓存的页面（快照信息）  
用cache语法的一般情况是：当一个链接无法访问时（或信息被屏蔽时）；当信息已经被修改，想看以前的信息时。
    `cache:www.xuanhun521.com`  
3.2 相关网址  
related语法对于发现某一类信息非常有用，比如当你用related搜索一个图书馆网址的时候会出来大量图书馆的网站，如【related:lib.nit.net.cn】；当搜索某期刊网址的时候，能搜索出大量给学科领域的相关期刊，如【related:www.lis.ac.cn】。  
    `related:www.kali.org`  
3.3 LINK
搜索所有链接到某个特定URL上的页面.  
例如，想搜索所有链接www.xuanhun521.com的页面，但要排除本站网页。
    `link:www.xuanhun521.com -site:www.xuanhun521.com`  
3.4 SITE  
搜索范围限制在某网站或顶级域名中。  
    `site:www.xuanhun521.com`  
4.1 混合搜索范例  
第一个示例，我们通过下面的查询语句，查找可以未经授权就可以访问的phpMyAdmin的后台页面。  
    `inurl:.php? intext:CHARACTER_SETS,COLLATIONS, ?intitle:phpmyadmin`   
 第二个示例，我们搜索可能存在openssl心脏出血漏洞的站点。  
    `"OPENSSL" AND "1.0.1 SERVER AT" OR "1.0.1A SERVER AT" OR "1.0.1B SERVER AT" OR "1.0.1C SERVER AT" OR "1.0.1D SERVER AT" OR "1.0.1E SERVER AT" OR "1.0.1F SERVER AT"`  



