---
title: IEDA 自动生成类注释和方法注释
date: 2019-01-08 15:22:50
categories:
    - 学习笔记
---

- 新建类，自动生成类注释的模板配置

`File->Settings->Editor->File and Code Templates->Class`

``` java
/** 
* @Description: TODO
* @author: scott
* @date: ${YEAR}年${MONTH}月${DAY}日 ${TIME}
*/
```

![undefined](https://ws1.sinaimg.cn/large/005LP3H3ly1gg5yp1zlqvj311z0l9tbi.jpg)

- 通过快捷键，添加类注释和方法注释的模板设置

1. 类注释      【快捷键：cls + TAB】

`File->Settings->Editor->Live Templates->Class`

![undefined](https://ws1.sinaimg.cn/large/005LP3H3ly1gg5yq1r5xfj30xc0hrjvi.jpg)

![undefined](https://ws1.sinaimg.cn/large/005LP3H3ly1gg5yqf99i2j30xc0jagqb.jpg)

![undefined](https://ws1.sinaimg.cn/large/005LP3H3ly1gg5yqmo1zuj30xc0hitcx.jpg)

``` java
/**
* @Description: TODO
* @author: scott
* @date: $DATE$ $TIME$
*/
```

![undefined](https://ws1.sinaimg.cn/large/005LP3H3ly1gg5yr0qipvj30xc0jfdm0.jpg)

![undefined](https://ws1.sinaimg.cn/large/005LP3H3ly1gg5yr762vqj312g0l7gny.jpg)

![undefined](https://ws1.sinaimg.cn/large/005LP3H3ly1gg5yre42nkj30ix077q32.jpg)

2. 方法注释       【快捷键：/**  回车】

![undefined](https://ws1.sinaimg.cn/large/005LP3H3ly1gg5yrt7hn9j312f0ll40i.jpg)

``` java
*
* @Description: TODO
* @author: scott
* @date: $DATE$ $TIME$
$PARAMS$
* @Return: $RETURN$
*/
```

注：$PARAMS$是故意无星号开头，template的Abbreviation是*，template的第一行也是*，这样输入/**回车就能输出注释

![undefined](https://ws1.sinaimg.cn/large/005LP3H3ly1gg5ysbd2s7j30ix077weq.jpg)

$PARAMS$设置

``` javascript
groovyScript("if(\"${_1}\".length() == 2) {return '';} else {def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList();for(i = 0; i < params.size(); i++) {if(i==0){result+='* @param ' + params[i] + ': '}else{result+='\\n' + '* @param ' + params[i] + ': '}}; return result;}", methodParameters());
```