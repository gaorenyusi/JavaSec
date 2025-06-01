# JavaSec
记录 java 的一些学习笔记
## java基础
* [java 反射机制](https://gaorenyusi.github.io/posts/java%E5%8F%8D%E5%B0%84%E6%9C%BA%E5%88%B6/)
* [java 反序列化](https://gaorenyusi.github.io/posts/java%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96/)
* [java 动态代理](https://gaorenyusi.github.io/posts/java%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86/)
* [java动态加载字节码](https://www.cnblogs.com/gaorenyusi/p/18269747)

还有java类加载机制可以参考nivia师傅文章：https://nivi4.notion.site/Java-cedccc0611654bd99f841de3ef578e24
，除此自外还有一些双亲委派机制，SPI机制也可提前学学
## java反序列化
java中的反序列化链子非常多，除去一些常用的，不同依赖有不同的利用链，相互组合又是新的利用链，感觉最重要的还是记住哪些是jdk自带的利用点，方便和第三方依赖进行组合利用。
* [CC1 分析利用](https://gaorenyusi.github.io/posts/cc1%E9%93%BE%E7%9A%84%E5%88%86%E6%9E%90%E4%B8%8E%E5%88%A9%E7%94%A8/)
* [CC6 分析利用](https://gaorenyusi.github.io/posts/cc6/)
* [CC3 分析利用](https://gaorenyusi.github.io/posts/cc3/)
* [CC2 分析利用](https://gaorenyusi.github.io/posts/cc2/)
* [CC4+CC5 分析利用](https://gaorenyusi.github.io/posts/cc4-cc5/)
* [CC7 分析利用](https://gaorenyusi.github.io/posts/cc7/)

其他的CC链系列还可以参考：[CommonsCollections11 分析](https://wjlshare.com/archives/1536)，[java反序列化漏洞commons-collections3.2.1TransformedList触发transform ](https://xz.aliyun.com/news/13748)
* [CB链分析与利用](https://gaorenyusi.github.io/posts/cb/)
* [ROME 反序列化](https://gaorenyusi.github.io/posts/rome/)
* [C3P0 链子分析学习](https://gaorenyusi.github.io/posts/c3p0/)

接着是两条JDK原生利用链
* [jdk7u21 链子分析](https://gaorenyusi.github.io/posts/jdk7u21/)
* [jdk8u20 链子分析](https://www.cnblogs.com/gaorenyusi/p/18489806)

RMI系列
* [java RMI反序列化-原理篇](https://gaorenyusi.github.io/posts/rmi1/)
* [java RMI反序列化-攻击篇](https://gaorenyusi.github.io/posts/rm2/)
* [Java JRMP 反序化](https://gaorenyusi.github.io/posts/jrmp/)
* [Java JEP290](https://gaorenyusi.github.io/posts/jep290/)

JNDI系列
* [Java JNDI 注入](https://gaorenyusi.github.io/posts/java-jndi/)
* [JNDI 之 LDAP 过程原理](https://gaorenyusi.github.io/posts/jndi-ldap/)
* [JDK 高版本下 JNDI 注入深度剖析](https://xz.aliyun.com/news/17638)

其他反序列化
* [shiro 反序列化漏洞](https://gaorenyusi.github.io/posts/shiro%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96/)
* [Hessian 反序列化](https://gaorenyusi.github.io/posts/hessian/)

当然这里没写的还有很多，比如 jdbc，snakeyaml，以及CTF中经常用的 jackson 利用链这些，这里就先简单记录这些了。
## java内存马
* [Tomcat内存马](https://www.cnblogs.com/gaorenyusi?page=2)
* spring内存马
* java agent内存马

内存马本质就是获得回显，所以有些时候可以不用注入内存马只获得回显也行，这里推荐个内存马工具：[https://github.com/pen4uin/java-memshell-generator](https://github.com/pen4uin/java-memshell-generator)
## java表达式注入&模板注入
* [SpEL 表达式注入](https://www.cnblogs.com/gaorenyusi/p/18411264)
* EL 表达式注入
* OGNL表达式注入
* [QL表达式注入](https://xz.aliyun.com/news/15134)
* thymeleaf 模板注入
* freemarker 模板注入

## java漏洞系列
### fastjson
* [fastjaon 反序列化](https://www.cnblogs.com/gaorenyusi/p/18435525)
* fastjson 1.2.68反序列化
* fastjson 1.2.80反序列化
* fastjosn实战利用
* ······
### jackson
* jackson 反序列化
* [jackson 原生反序列化触发 getter 方法](https://www.cnblogs.com/gaorenyusi/p/18411269)
* ······
### Apache
* [Apache OfBiz 反序列化命令（CVE-2020-9496）](https://gaorenyusi.github.io/posts/apacheofbiz/)
* ······
### log4j2
* [Apache_log4j2（CVE-2021-44228）漏洞复现](https://gaorenyusi.github.io/posts/log4j2/)
* ······
### Struts2
* [S2-001](https://gaorenyusi.github.io/posts/s1-001/)
* ······

## java代码审计
* CodeQL使用
* Tabby使用
* [华夏 ERP CMS v2.3代码审计](https://gaorenyusi.github.io/posts/%E5%8D%8E%E5%A4%8Fcms/)
* ······



