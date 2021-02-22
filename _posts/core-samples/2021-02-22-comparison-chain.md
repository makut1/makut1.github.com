---
title: Guava ComparableisonChain链式比较器
layout: post
guid: urn:uuid:2ef3550f-8cf3-400b-a55b-c512c9af8b2d
tags : [technology]


---

{% include JB/setup %}

有个业务需求需要获取该小时段内用户推送视频的数据，按照以下排序规则取数据：

排序：
1).优先3+作者内容；
2).优先原创内容；
3).优先视频频道与用户偏好频道一致内容；
4).按VV从高到低排序；

  最先想到的就是一堆if条件判断，压根没有想起来使用比较器来处理这种情况。但是想到这么多if条件判断挺麻烦的，就和同事沟通了下看看有没有更好的解决方法，同事说可以使用比较器来处理comparator或是comparable。实现Comparator，或者是实现Comparable接口都是我们经常使用的，而在面对实际业务情况的时候却没有想到，真是把知识点学死了。

  既然说到可以使用jdk自带比较器，但是有多条排序规则的时候也难免需要if判断语句，所以就在网上检索有没有类似责任链模式那种的比较器，可以多次比较，而不需要写多个判断语句。最后发现Guava提供了ComparableisonChain（A utility for performing a chained comparison statement.），记录一下它的用法。

  Guava官方给出的使用demo，可以很清晰的看出使用ComparableisonChain很方便。

```
class Person implements Comparable<Person> {
  private String lastName;
  private String firstName;
  private int zipCode;

  public int compareTo(Person other) {
    int cmp = lastName.compareTo(other.lastName);
    if (cmp != 0) {
      return cmp;
    }
    cmp = firstName.compareTo(other.firstName);
    if (cmp != 0) {
      return cmp;
    }
    return Integer.compare(zipCode, other.zipCode);
  }
}
```

  这段代码很容易混乱，很难查找 bug，而且冗长得让人不舒服。我们应该能做得更好。为此，guava提供ComparisonChain，执行“ lazy”比较: 它只执行比较，直到找到一个非零结果，然后忽略进一步的输入。

```
 public int compareTo(Foo that) {
     return ComparisonChain.start()
         .compare(this.aString, that.aString)
         .compare(this.anInt, that.anInt)
         .compare(this.anEnum, that.anEnum, Ordering.natural().nullsLast())
         .result();
   }
```

实际项目开发中使用如下：

![image-20210222163002567](D:\iqiyicode\makut1.github.com\media\files\image-20210222163002567.png)

![image-20210222163304076](D:\iqiyicode\makut1.github.com\media\files\image-20210222163304076.png)

![image-20210222163454513](D:\iqiyicode\makut1.github.com\media\files\image-20210222163454513.png)

  在日常工作开发过程中，减少代码量，在实现前多想想哪些知识点可以应用到实际项目中，别不懂得灵活变通，尽量做到抽象。