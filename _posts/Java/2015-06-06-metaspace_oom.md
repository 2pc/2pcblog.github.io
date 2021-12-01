---
title: 记一次Aviator使用不当导致的线上OOM
tagline: ""
category : Java
layout: post
tags : [Java, File, tools，OOM]
---

### 问题描述
1. 是的,没错,正如题中所说是Aviator使用不当造成的OOM,为何使用不当后边说
2. 项目上线几个月了,发现云平台时不时会重启server,一般出现在某些高峰期，比如早上7-9点,晚上12点
3. 通过-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath,配置可以获取到dump文件
4. 发现每次dump都是自带监控产生的oom，自带监控的实现是基于Prometheus


###  oom堆栈信息
具体监控如下
```
"pool-3-thread-2" prio=5 tid=124 RUNNABLE
    at java.lang.OutOfMemoryError.<init>(OutOfMemoryError.java:48)
    at sun.util.resources.TimeZoneNames.getContents(TimeZoneNames.java:47)
    at sun.util.resources.OpenListResourceBundle.loadLookup(OpenListResourceBundle.java:137)
    at sun.util.resources.OpenListResourceBundle.loadLookupTablesIfNecessary(OpenListResourceBundle.java:128)
    at sun.util.resources.OpenListResourceBundle.handleKeySet(OpenListResourceBundle.java:96)
    at java.util.ResourceBundle.containsKey(ResourceBundle.java:1824)
       Local Variable: sun.util.resources.TimeZoneNames#1
    at sun.util.locale.provider.LocaleResources.getTimeZoneNames(LocaleResources.java:263)
       Local Variable: sun.util.locale.provider.LocaleResources#1
       Local Variable: sun.util.resources.en.TimeZoneNames_en#1
       Local Variable: java.lang.String#140268
    at sun.util.locale.provider.TimeZoneNameProviderImpl.getDisplayNameArray(TimeZoneNameProviderImpl.java:124)
    at sun.util.locale.provider.TimeZoneNameProviderImpl.getDisplayName(TimeZoneNameProviderImpl.java:99)
    at sun.util.locale.provider.TimeZoneNameUtility$TimeZoneNameGetter.getName(TimeZoneNameUtility.java:240)
    at sun.util.locale.provider.TimeZoneNameUtility$TimeZoneNameGetter.getObject(TimeZoneNameUtility.java:198)
    at sun.util.locale.provider.TimeZoneNameUtility$TimeZoneNameGetter.getObject(TimeZoneNameUtility.java:184)
    at sun.util.locale.provider.LocaleServiceProviderPool.getLocalizedObjectImpl(LocaleServiceProviderPool.java:294)
       Local Variable: java.lang.String#39894
       Local Variable: java.util.HashSet#79
       Local Variable: sun.util.locale.provider.TimeZoneNameProviderImpl#1
       Local Variable: sun.util.locale.provider.TimeZoneNameUtility$TimeZoneNameGetter#1
    at sun.util.locale.provider.LocaleServiceProviderPool.getLocalizedObject(LocaleServiceProviderPool.java:265)
    at sun.util.locale.provider.TimeZoneNameUtility.retrieveDisplayNamesImpl(TimeZoneNameUtility.java:166)
       Local Variable: java.lang.String[]#4784
       Local Variable: java.util.Locale#16
       Local Variable: sun.util.locale.provider.LocaleServiceProviderPool#3
    at sun.util.locale.provider.TimeZoneNameUtility.retrieveDisplayName(TimeZoneNameUtility.java:137)
    at java.util.TimeZone.getDisplayName(TimeZone.java:400)
       Local Variable: java.lang.String#696
       Local Variable: sun.util.calendar.ZoneInfo#985
    at java.text.SimpleDateFormat.subFormat(SimpleDateFormat.java:1271)
    at java.text.SimpleDateFormat.format(SimpleDateFormat.java:966)
       Local Variable: java.lang.StringBuffer#8
       Local Variable: java.text.DontCareFieldPosition$1#1
       Local Variable: java.text.SimpleDateFormat#190
    at java.text.SimpleDateFormat.format(SimpleDateFormat.java:936)
    at java.text.DateFormat.format(DateFormat.java:345)
    at sun.net.httpserver.ExchangeImpl.sendResponseHeaders(ExchangeImpl.java:212)
       Local Variable: java.io.BufferedOutputStream#43
       Local Variable: sun.net.httpserver.PlaceholderOutputStream#1
       Local Variable: sun.net.httpserver.ExchangeImpl#1
    at sun.net.httpserver.HttpExchangeImpl.sendResponseHeaders(HttpExchangeImpl.java:86)
    at metrics.exporter.PrometheusMetricsExporter.lambda$startServer$0(PrometheusMetricsExporter.java:88)
```
这里翻了下Prometheus看起来来也没啥特殊的，跟中间件的对了下，好像也看不出啥问题，但是发现dump日志只有50M左右,与实际-Xmx2g相差剩余
没招只好找自己程序的代码的BUG,CR了几次都没有发现问题，没办法oom了只好看gc日志了，发现fullgc一直在进行，占用的时间也不少


![gc](https://github.com/2pc/2pc.github.io/blob/master/_posts/images/1.png)

