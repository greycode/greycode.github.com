---
layout: post
title: 使用Guava编写优雅代码
categories:
- Java开发
tags:
- [java,guava]
---
<link rel="stylesheet" href="/media/highlight/styles/zenburn.css">
<script src="/media/highlight/highlight.pack.js"></script>
<script>
$(document).ready(function() {
  $('pre code').each(function(i, block) {
    hljs.highlightBlock(block);
  });
});
</script>
###What is Guava
这个Guava当然不是指水果，Guava 是来自Google的工具类库集合，包含了collections, caching, primitives support, concurrency libraries, common annotations, string processing, I/O, Math等等。
###Why Guava
Guava有点类似于Apache Commons库，两者之间的区别在[Stackoverflow](http://stackoverflow.com/questions/4542550/what-are-the-big-improvements-between-guava-and-apache-equivalent-libraries)上已经很好的回答了这个问题，总结来说就是Guava相对来说设计更优秀、文档齐全、代码质量高、社区更活跃，Guava更加“Morden”。如果你做Java开发，就该把Guava加入到你的项目中。
![Guava vs Apache Commons](/media/pic2014/compare2apache.png)

###Using Guava
Maven
<pre><code class="xml">
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>18.0</version>
</dependency>
</code></pre>
或直接到 [Maven中央库](http://mvnrepository.com/artifact/com.google.guava/guava)下载jar包。
####Object common methods
<pre><code class="java">
public class Employee implements Comparable<Employee> {
	private String name;
	private Integer age;
	private Job job;

	// methods ...

	public int compareTo(Employee other) {
		return ComparisonChain.start()
		// 以英文字母(从a到z)的自然顺序，NULL值放在最后
			.compare(this.name, other.name, Ordering.natural().nullsLast())
		// 以数字的反序(从大到小)，NULL值处于最后
			.compare(this.age, other.age, Ordering.natural().reverse().nullsLast())
			.compare(this.job, other.job, Ordering.natural().nullsLast())
			.result();
	}

	@Override
	public int hashCode() {
		return Objects.hashCode(name, age, job);
	}

	@Override
	public String toString () {
		return MoreObjects.toStringHelper(this)
			.omitNullValues()
			.add("name", name)
			.add("age", age)
			.add("job", job)
			.toString();
	}
</code></pre>
####Lists and MutiMap
<pre><code class="java">
	List < Map < String, Object> > maps = Lists.newArrayList();
	List < String > langs = Lists.newArrayList("中文","English","日本語",null);
	String lang = Joiner.on("|").useForNull("Unkown").join(langs);
	// 中文|English|日本語|Unkown
	System.out.println(lang);                                               
	
	// Like Map< Job, Collection< Employee > >
	Multimap < Job, Employee > multimap = ArrayListMultimap.create();
	multimap.put(Job.CEO, new Employee("Tom",45));
	multimap.put(Job.DESIGNER, new Employee("Jack",24));
	multimap.put(Job.DEVELOPER, new Employee("Alice", 31));
	multimap.put(Job.DEVELOPER, new Employee("Jhone", 25));
	multimap.put(Job.DEVELOPER, new Employee("Jim", 27));
</code></pre>
更多代码示例可以关注我的[Glist](https://gist.github.com/greycode/2969fe130d345f87a208)。
###More Resource
1. 首要推荐官方的[WIKI](https://code.google.com/p/guava-libraries/wiki/GuavaExplained?tm=6)，这个最新最全，并发编程网上有[中文的翻译](http://ifeve.com/google-guava/)，但是中文翻译排版很不好，英文不好又想参考官网文档的可以看看，入门不推荐。
2. OSchina上有几篇翻译不错的教程：  
　　[Guava 教程1](http://www.oschina.net/translate/beautiful-code-with-google-collections-guava-and-static-imports-part-1)-使用 Google Collections,Guava,static imports 编写漂亮代码  
　　[Guava 教程2](http://www.oschina.net/translate/diving-into-the-google-guava-library-part-2)-深入探索Google Guava 库  
　　[Guava 教程3](http://www.oschina.net/translate/functional-java-filtering-and-ordering-with-google-collections-part-3)-Java 的函数式编程，通过 Google Collections 过滤和调用  
　　[Guava 教程4](http://www.oschina.net/translate/preconditions-multimaps-and-partitioning-with-google-collections-part-4)-条件，多重映射和分片  
3. “使用Google Guava来编写优雅的代码”系列，对Guava的集合做了简单介绍，适合入门。  
　[集合1](http://www.letonlife.com/writing-clean-code-with-google-guava-part-4-915)  
　[集合2](http://www.letonlife.com/writing-clean-code-with-google-guava-part-5-918)  
　[集合3(Multimap)](http://www.letonlife.com/writing-clean-code-with-google-guava-part-6-multimap-923)  
　[集合4(BiMap)](http://www.letonlife.com/writing-clean-code-with-google-guava-part-7-bimap-930)  
4. 博客园里的童鞋写的一系列[学习笔记](http://www.cnblogs.com/peida/p/Guava.html)。  
5. 疯狂的菠菜写的一篇[Google Guava 库用法整理](http://macrochen.iteye.com/blog/737058)，使用Guava前后的代码对比很直观，没有接触过的童鞋推荐首先看看这篇。
6. Speaker Deck上的这个[Guava By Example](https://speakerdeck.com/eneveu/guava-by-example)也是很赞，同样也是对使用Guava前后的代码做了直观的对比。
7. Oschina上转载的另一篇不错的文章：[使用 Google Guava 美化你的 Java 代码](http://my.oschina.net/leejun2005/blog/172328#OSC_h3_1)。
8. 如果以上都满足不了你的求知欲，可以看看[PACKT]出版的这本[Getting Started with Google Guava](https://www.packtpub.com/application-development/getting-started-google-guava)（电子版网上有没有免费下载什么的我才不知道呢~哼！）。
　
