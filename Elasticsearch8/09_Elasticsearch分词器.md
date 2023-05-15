## 分词器

### 定义

文本分析是把全文本转换一系列单词的过程，也叫分词。Analysis是通过Analyzer来实现的。

分词器的作用就是把整篇文档，按一定的语义切分成一个一个的词条，目标是提升文档的召回率，并降低无效数据的干扰。

- recall召回率，也叫可搜索性，指搜索的时候，增加能够搜索到的结果的数量。
- 降噪：指降低文档中一些低相关性词条对整体搜索排序结果的干扰。

我们知道在全文检索中，有精确匹配和模糊匹配两种，精确匹配就是值的相等与否，而模糊匹配则要根据关键词在文本中出现的次数进行打分才能得出，在Elastic App Search中我们看到的score就是一个相关性评分，评分越高匹配度越高，那这里就要使用分词器对文本进行分词。



### 组成

> 分析器（Analyzer）都由三种构件块组成的：**Character Filters** ， **Tokenizers** ， **Token Filters**

- **Character Filters：** 字符过滤器，在一段文本进行分词之前，先进行预处理，比如说最常见的就是，过滤html标签（hello --> hello），& --> and（I&you --> I and you）
- **Tokenizers：** 根据指定规则进行切分。英文分词可以根据空格将单词分开，中文分词比较复杂，可以使用中文分词器甚至机器学习算法来分词。切分同时也负责标记出每个单词的顺序、位置以及单词在原文本中开始和结束的偏移量。
- **Token Filters：**叫Token过滤器，将切分的单词进行加工。大小写转换（例将“Quick”转为小写），去掉词（例如停用像“a”、“and”、“the”等等），或者增加词（例如同义词像 “jump” 和 “leap” ）

**Analyzer = Character Filters（0个或多个） + Tokenizer（一个） + TokenFilters（0个或多个）**



### 内置分词器

#### 1. Standard Analyzer

ES 默认分词器，按词切分，小写处理，它提供了基于语法的标记化（基于Unicode文本分割算法），适用于大多数语言

![image-20230408110932906](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230408110932906.png)

#### 2. Simple Analyzer

按照非字母切分(符号被过滤), 小写处理，simple 分析器当它遇到**只要不是字母的字符**，就将文本解析成term，而且所有的term都是小写的。

![image-20230408111108456](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230408111108456.png)

#### 3. Whitespace Analyzer

按照空格切分，不转小写，区分大小写，汉语不分词

![image-20230408111221473](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230408111221473.png)

#### 4. Stop Analyzer

与 Simple Analyzer 类似，但增加了停用词过滤（如 a、an、and、are、as、at、be、but 等）。

#### 5. Pattern Analyzer 

使用正则表达式切分，默认使用 \W+ (非字符分隔)。支持小写转换和停用词删除。

#### 6. Keyword Analyzer 

不进行分词。

#### 7. Language Analyzer 

提供了多种常见语言的分词器。如 Irish、Italian、Latvian 等。

#### 8. Customer Analyzer

自定义分词器。

```shell
PUT restaurants
{
  "settings": {
    "analysis": {
      "char_filter": { // 自定义Character Filter
        "my_charater_filter": {  // 自定义Character Filter,将 & 变为 and
          "type": "mapping",
          "mappings": ["& => and"]
        }
      },
      "filter": { // 自定义Token Filter
        "my_stop_filter": {	// 停用词
          "type": "stop",
          "stopwords": ["a", "an"]
        }
      },
      "analyzer": {
        "custom_analyzer": {	// 自定义分词器
          "type": "custom",
          "char_filter": ["html_strip", "my_charater_filter"],
          "tokenizer": "standard",
          "filter": ["lowercase", "my_stop_filter"]
        }
      }
    }
  }
}


GET restaurants/_analyze
{
  "analyzer": "custom_analyzer",
  "text": "<b>Mr & Mrs Band</b> is a very good restaurant!"
}
```

![image-20230408111308883](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230408111308883.png)

### 中文分词

中文分词指的是将一个汉字序列切分成一个一个单独的词。中文分词是文本挖掘的基础，对于输入的一段中文，成功的进行中文分词，可以达到电脑自动识别语句含义的效果。因为中文句子是由词语组成，没有空格方便切分，而且还有上下文语境因此切词相对来说比较困难。

#### 分词算法

> 现有的分词算法可分为三大类：基于字符串匹配的分词方法、基于理解的分词方法和基于统计的分词方法。

##### 1.  基于字符串匹配的分词方法

这种方法又叫做机械分词方法，它是按照一定的策略将待分析的汉字串与一个 “充分大的”机器词典中的词条进行配，若在词典中找到某个字符串，则匹配成功（识别出一个词）。

按照扫描方向的不同，串匹配分词方法可以分为正向匹配和逆向匹配；按照不同长度优先匹配的情况，可以分为最大（最长）匹配和最小（最短）匹配；按照是否与词性标注过程相结合，又可以分为单纯分词方法和分词与标注相结合的一体化方法。常用的几种机械分词方法如下：

- **正向最大匹配法（由左到右的方向）**

- **逆向最大匹配法（由右到左的方向）**

- **最少切分（使每一句中切出的词数最小）**

- **双向最大匹配法（进行由左到右、由右到左两次扫描）**

> 还可以将上述各种方法相互组合。

例如：可以将正向最大匹配方法和逆向最大匹配方法结合起来构成双向匹配法。

由于汉语单字成词的特点，正向最小匹配和逆向最小匹配一般很少使用。一般说来，逆向匹配的切分精度略高于正向匹配，遇到的歧义现象也较少。统计结果表明，单纯使用正向最大匹配的错误率为1/169，单纯使用逆向最大匹配的错误率为1/245。但这种精度还远远不能满足实际的需要。实际使用的分词系统，都是把机械分词作为一种初分手段，还需通过利用各种其它的语言信息来进一步提高切分的准确率。

一种方法是改进扫描方式，称为特征扫描或标志切分，优先在待分析字符串中识别和切分出一些带有明显特征的词，以这些词作为断点，可将原字符串分为较小的串再来进机械分词，从而减少匹配的错误率。另一种方法是将分词和词类标注结合起来，利用丰富的词类信息对分词决策提供帮助，并且在标注过程中又反过来对分词结果进行检验、调整，从而极大地提高切分的准确率。

##### 2. 基于理解的分词方法

这种分词方法是通过让计算机模拟人对句子的理解，达到识别词的效果。其基本思想就是在分词的同时进行句法、语义分析，利用句法信息和语义信息来处理歧义现象。

它通常包括三个部分：**分词子系统**、**句法语义子系统**、**总控部分**。

在总控部分的协调下，分词子系统可以获得有关词、句子等的句法和语义信息来对分词歧义进行判断，即它模拟了人对句子的理解过程。

这种分词方法需要使用大量的语言知识和信息。由于汉语语言知识的笼统、复杂性，难以将各种语言信息组织成机器可直接读取的形式，因此目前基于理解的分词系统还处在试验阶段。

##### 3. 基于统计的分词方法

从形式上看，词是稳定的字的组合，因此在上下文中，相邻的字同时出现的次数越多，就越有可能构成一个词。

因此字与字相邻共现的频率或概率能够较好的反映成词的可信度。可以对语料中相邻共现的各个字的组合的频度进行统计，计算它们的互现信息。

定义两个字的互现信息，计算两个汉字X、Y的相邻共现概率。互现信息体现了汉字之间结合关系的紧密程度。当紧密程度高于某一个阈值时，便可认为此字组可能构成了一个词。

这种方法只需对语料中的字组频度进行统计，不需要切分词典，因而又叫做无词典分词法或统计取词方法。

但这种方法也有一定的局限性，会经常抽出一些共现频度高、但并不是词的常用字组，例如“这一”、“之一”、 “有的”、“我的”、“许多的”等，并且对常用词的识别精度差，时空开销大。



> 实际应用的统计分词系统都要使用一部**基本的分词词典（常用词词典）进行串匹配分词**，同时使用**统计方法识别一些新的词**，即将**串频统计和串匹配结合**起来，既发挥匹配分词切分速度快、效率高的特点，又利用了无词典分词结合上下文识别生词、自动消除歧义的优点。
>
> 到底哪种分词算法的准确度更高，目前并无定论。对于任何一个成熟的分词系统来说，不可能单独依靠某一种算法来实现，都需要综合不同的算法。
>
> 中文的分词器现在大家比较推荐的就是 IK分词器，当然也有些其它的比如 smartCN、HanLP、Ansj、Jcseg等。

### IK分词器

#### 介绍

IK分词方式属于词库分词，采用机械分词方法进行分词，地址：https://github.com/medcl/elasticsearch-analysis-ik。

主要包含：

1. 词典：词典的好坏直接影响分词结果的好坏
2. 词的匹配：有了词典之后，就可以对输入的字符串逐字句和词典进行匹配
3. 消除歧义：通过词典匹配出来的切分方式会有多种，消除歧义就是从中寻找最合理的一种方式

#### 安装

可以按照以前的安装方式进行本地安装，此处也可以采用在线安装：

```shell
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.4.3/elasticsearch-analysis-ik-8.4.3.zip
```

或

```shell
unzip -d ./elasticsearch-8.4.3/plugins/analysis-ik elasticsearch-analysis-ik-8.4.3.zip
```



#### 分词

IK分词包含两种ik_smart 和 ik_max_word：

- ik_smart: 会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,国歌”，适合 Phrase 查询。
- ik_max_word: 会将文本做最细粒度的拆分，比如会将 ”中华人民共和国国歌“ 拆分为 “中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合，适合 Term Query；

> 设置了ik_smart 模式，就是在进行词切分后，在进行一次分词的选择，即通常说的消除歧义

![image-20230408115907624](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230408115907624.png)

#### 词典

ik为我们提供了三种常见的词典分别是：

1. main.dic：主词典，一些常用词
2. quantifier.dic：常用的量词
3. stopword.dic：停用词

这些词典用户都可以自行扩展，只需要配置IKAnalyzer.cfg.xml文件即可，文件在`/analysis-ik/config/IKAnalyzer.cfg.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict"></entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords"></entry>
</properties>
```

**热更新**

目前该插件支持热更新 IK 分词，通过上文在 IK 配置文件中提到的如下配置

```xml
<!--用户可以在这里配置远程扩展字典 -->
<entry key="remote_ext_dict">words_location</entry>
<!--用户可以在这里配置远程扩展停止词字典-->
```

> 其中 words_location是指一个 url，比如 http://yoursite.com/custom_dic，该请求只需满足以下两点即可完成分词热更新。
>
> 1. 该 http 请求需要返回两个头部(header)，一个是 `Last-Modified`，一个是 `ETag`，这两者都是字符串类型，只要有一个发生变化，该插件就会去抓取新的分词进而更新词库。
> 2. 该 http 请求返回的内容格式是一行一个分词，换行符用 `\n` 即可。

满足上面两点要求就可以实现热更新分词了，不需要重启 ES 实例。

可以将需自动更新的热词放在一个 UTF-8 编码的 .txt 文件里，放在 nginx 或其他简易 http server 下，当 .txt 文件修改时，http server 会在客户端请求该文件时自动返回相应的 Last-Modified 和 ETag。可以另外做一个工具来从业务系统提取相关词汇，并更新这个 .txt 文件。