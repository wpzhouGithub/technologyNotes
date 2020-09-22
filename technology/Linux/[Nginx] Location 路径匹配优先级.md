## location表达式类型

~ 表示执行一个正则匹配，区分大小写
~* 表示执行一个正则匹配，不区分大小写
^~ 表示普通字符匹配。优先使用前缀匹配。如果匹配成功，则不再匹配其他location。
= 进行普通字符精确匹配。也就是完全匹配。
@ "@" 定义一个命名的 location，使用在内部定向时，例如 error_page, try_files

## location优先级说明
在nginx的location和配置中location的顺序没有太大关系。跟location表达式的类型有关。相同类型的表达式，字符串长的会优先匹配。
以下是按优先级排列说明：
第一优先级：等号类型（=）的优先级最高。一旦匹配成功，则不再查找其他匹配项。
第二优先级：^~类型表达式。一旦匹配成功，则不再查找其他匹配项。
第三优先级：正则表达式类型（~ ~*）的优先级次之。如果有多个location的正则能匹配的话，使用最先匹配的那个，即最上面的；
第四优先级：常规字符串匹配类型。按前缀匹配。


对于请求： http://example.com/static/img/logo.jpg

1、如果命中精确匹配，例如：
```
location = /static/img/logo.jpg {

}
```
则优先精确匹配，并终止匹配。

2、如果命中多个前缀匹配，例如：
```
location /static/ {

}

location /static/img/ {

}
```
则记住最长的前缀匹配，即上例中的 /static/img/，并继续匹配

3、如果最长的前缀匹配是优先前缀匹配，即：
```
location /static/ {

}

location ^~ /static/img/ {

}
```
则命中此最长的优先前缀匹配，并终止匹配

4、否则，如果命中多个正则匹配，即：
```
location /static/ {

}

location /static/img/ {

}

location ~* /static/ {

}

location ~* /static/img/ {

}
```
则忘记上述 2 中的最长前缀匹配，使用第一个命中的正则匹配，即上例中的 location ~* /static/ ，并终止匹配（命中多个正则匹配，优先使用配置文件中出现次序的第一个）

5、否则，命中上述 2 中记住的最长前缀匹配