# mysql OOM资源问题，被系统重启

## 表象

- MySQL log 显示 OOM被重启

## 解决问题

1. log显示了OOM，就以为是MySQL占用内存过多导致系统内存紧张，被重启；
2. 思路方向一直是MySQL 内存分析、优化
   -  check内存相关参数配置，没有发现问题
   -  分析MySQL内存的使用,  并没有发现明显的问题
   -  脑海中各种问号，明明内存消耗没那么大，但是怎么就OOM被kill了呢
3. 分析syslog
   -  分析MySQL被重启前后的系统log，发现系统log记录了MySQL被重启时的系统状况：有python程序占用了很多内存
   -  log显示，MySQL是因为内存紧张被系统kill； 但是当时为什么系统内存突然紧张了，是因为有其他的python程序占用了内存；为什么killMySQL，是因为系统会kill占用内存最多的
4. 现在就明了很多了：MySQL被kill，不是因为MySQL占用内存过多，而是有其他的程序也占用大量内存 ，导致系统内存吃紧，kill占用内存最多的程序
5. 解决方案：
   - 直接增加机器内存，因为python程序是合理的；现在就是系统内存配置过少，需要增加内存
   - 将数据库跟python程序分开，放到不同的机器上，相互不影响； 
   - 成本上增加一台机器跟扩充内存差不多，最后是增加了一台机器

## 思路总结

1. 最开始被MySQL的OOM误导了；OOM是没错，但是不一定是因为MySQL过度消耗内存导致；也有可能是多个程序一起导致内存紧张，只是MySQL占用内存最多而已
2. 下次再遇到被系统重启的问题，第一反应当然也是看MySQL的log，但是紧接着也应该看一下syslog，确认被kill的真实原因