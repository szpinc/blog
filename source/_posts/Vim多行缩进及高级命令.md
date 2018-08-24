---
title: Vim多行缩进及高级命令
date: 2018-08-12 15:14:18
tags:
    - vim
    - vi
categories:
    - Linux
---

### Vim多行缩进技巧

按v进入visual状态，选择多行，用>或<缩进或缩出 

通常根据语言特征使用自动缩进排版：在命令状态下对当前行用== （连按=两次）, 或对多行用n==（n是自然数）表示自动缩进从当前行起的下面n行。你可以试试把代码缩进任意打乱再用n==排版，相当于一般IDE里的code format。使用gg=G可对整篇代码进行排版。
gg   shift+G 首尾

全文字眼替换  %s/source/dist/g 或者 :1,$ s/source/dist/g
　　:s/vivian/sky/ 替换当前行第一个 vivian 为 sky
　　:s/vivian/sky/g 替换当前行所有 vivian 为 sky

　　:n,$s/vivian/sky/ 替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky

　　:n,$s/vivian/sky/g 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky

　　n 为数字，若 n 为 .，表示从当前行开始到最后一行

　　:%s/vivian/sky/(等同于 :g/vivian/s//sky/) 替换每一行的第一个 vivian 为 sky

　　:%s/vivian/sky/g(等同于 :g/vivian/s//sky/g) 替换每一行中所有 vivian 为 sky

　　可以使用 # 作为分隔符，此时中间出现的 / 不会作为分隔符

　　:s#vivian/#sky/# 替换当前行第一个 vivian/ 为 sky/

　　:%s+/oradata/apras/+/user01/apras1+ (使用+ 来 替换 / )： /oradata/apras/替换成/user01/apras1/

　　* ************************************

　　1.:s/vivian/sky/ 替换当前行第一个 vivian 为 sky

　　:s/vivian/sky/g 替换当前行所有 vivian 为 sky

　　2. :n,$s/vivian/sky/ 替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky

　　:n,$s/vivian/sky/g 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky

　　(n 为数字，若 n 为 .，表示从当前行开始到最后一行)

　　3. :%s/vivian/sky/(等同于 :g/vivian/s//sky/) 替换每一行的第一个 vivian 为 sky

　　:%s/vivian/sky/g(等同于 :g/vivian/s//sky/g) 替换每一行中所有 vivian 为 sky

　　4. 可以使用 # 作为分隔符，此时中间出现的 / 不会作为分隔符

　　:s#vivian/#sky/# 替换当前行第一个 vivian/ 为 sky/

　　5. 删除文本中的^M

　　问题描述：对于换行,window下用回车换行(0A0D)来表示，linux下是回车(0A)来表示。这样，将window上的文件拷到unix上用时，总会有个^M.请写个用在unix下的过滤windows文件的换行符(0D)的shell或c程序。

　　· 使用命令：cat filename1 | tr -d “^V^M” > newfile;

　　· 使用命令：sed -e “s/^V^M//” filename > outputfilename。需要注意的是在1、2两种方法中，^V和^M指的是Ctrl+V和Ctrl+M。你必须要手工进行输入，而不是粘贴。

　　· 在vi中处理：首先使用vi打开文件，然后按ESC键，接着输入命令：%s/^V^M//。

　　· :%s/^M$//g

　　如果上述方法无用，则正确的解决办法是：

　　· tr -d "\r" < src >dest

　　· tr -d "\015" dest

　　· strings A>B

　　6. 其它

　　利用 :s 命令可以实现字符串的替换。具体的用法包括：

　　:s/str1/str2/ 用字符串 str2 替换行中首次出现的字符串 str1

　　:s/str1/str2/g 用字符串 str2 替换行中所有出现的字符串 str1

　　:.,$ s/str1/str2/g 用字符串 str2 替换正文当前行到末尾所有出现的字符串 str1

　　:1,$ s/str1/str2/g 用字符串 str2 替换正文中所有出现的字符串 str1

　　:g/str1/s//str2/g 功能同上

　　从上述替换命令可以看到：g 放在命令末尾，表示对搜索字符串的每次出现进行替换;不加 g，表示只对搜索

　　字符串的首次出现进行替换;g 放在命令开头，表示对正文中所有包含搜索字符串的行进行替换操作。
