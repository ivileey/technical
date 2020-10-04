# 常见问题

## 一、String，StringBuffer，StringBuilder的区别

String类长度是不可变的，StringBuffer和StringBuilder类长度是可以改变的；

StringBuffer类是线程安全的，StringBuilder不是线程安全的；

需要经常改变字符串不要用String，String对象的拼接实际是StringBuffer对象的拼接。

StringBuilder在方法内部完成“+”的操作，StringBuffer在全局变量。