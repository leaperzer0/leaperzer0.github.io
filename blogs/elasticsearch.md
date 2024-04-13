# ElasticSearch学习笔记

## 认识Elasticsearch

### Elasticsearch简介

Elasticsearch 是一个开源的、高度可扩展的、实时的搜索与数据分析引擎，允许用户快速存储、搜索和分析大量数据。它通常用于全文搜索、结构化搜索、分析以及这三者的组合，主要有以下几个关键特性：

1. **分布式本质**：Elasticsearch 自动分配数据并处理多台服务器之间的负载，这使它能够处理PB级别的数据。
2. **实时**：Elasticsearch 具有近实时的搜索能力，这意味着从文档索引到可搜索的时间非常短，通常只有一秒。
3. **多样的接口**：提供 RESTful API，这使得与 Elasticsearch 的交互变得简单，可以使用任何能够发送HTTP请求的编程语言与之交互。
4. **全文搜索能力**：内建的分析引擎提供强大的文本分析功能，支持多语言。
5. **高度可配置的索引**：可以非常灵活地配置索引以满足不同的性能和存储需求。
6. **集成与扩展**：可以通过插件机制扩展 Elasticsearch 的功能，同时也可以很好地与其他 Elastic Stack 组件（如Logstash、Kibana）集成，以提供日志分析、可视化等功能。

Elasticsearch 适用于各种用途，从简单的搜索引擎，到复杂的数据分析应用，以及支撑大型网站的搜索功能等。由于其灵活性和扩展性，它被广泛应用于日志和事件数据分析、全文搜索、安全情报分析等多个领域。

elastic 是 elastic stack 的核心，负责存储、搜索和分析数据。

![image-20240409221551616](C:\Users\Administrator\Desktop\assets\image-20240409221551616.png)

其中无论是 Kibana，还是 Logstash、Beats，都是官方提供的可替换的组件，只有 Elasticsearch 是不可替换的，所以说 Elasticsearch 是 elastic stack 的核心。

### Lucene技术

Elasticsearch 的底层是 Lucene 技术实现的，Elasticsearch 实际上建立在 Lucene 库之上，使用了Lucene的许多核心功能来实现其搜索和索引功能。

Lucene 是一个高性能、可扩展的信息检索(IR)库，可以用来构建搜索引擎。其提供了一个丰富的查询语言，支持全文检索、高亮显示搜索结果、以及对文档进行评分等功能。Lucene 自身是用 Java 编写的，因此它可以很容易地被集成到使用Java的应用程序中，比如 Elasticsearch。

Lucene有如下几大技术优势：

1. **全文搜索能力**：Lucene提供了强大的全文搜索功能，支持各种复杂的查询类型，包括模糊查询、通配符查询、短语查询等，这是 Elasticsearch 强大搜索能力的基础。
2. **性能和效率**：Lucene基于倒排索引的数据结构优化，可以快速地对大量数据进行索引和搜索，这对于Elasticsearch这样需要处理大规模数据集的系统来说非常重要。
3. **灵活性和可扩展性**：Lucene允许开发者深度定制其功能，比如通过自定义分析器来处理特定类型的数据。Elasticsearch 利用了这一点，提供了丰富的API供用户定制和扩展。
4. **成熟稳定**：Lucene 是一个成熟的开源项目，拥有活跃的社区和丰富的文档资源，这为Elasticsearch 的开发和维护提供了坚实的基础。

Elasticsearch 提供了 RESTful API 和分布式特性，使其更容易被应用于现代的web环境中，但其核心搜索和索引的能力实际上是直接来源于 Lucene 的。Elasticsearch 可以被视为一个将 Lucene 的能力包装并扩展成一个分布式、易于使用的搜索引擎服务。

### 倒排索引与正向索引

Elasticsearch 和传统的关系型数据库（如MySQL）在索引构建和查询数据的方法上有本质的区别。这些差异使它们各自适合于不同的使用场景。因此，在学习 Elasticsearch 采用的倒排索引前，我们有必要先了解 MySQL 等传统关系型数据库采用的正向索引。

#### 正向索引

传统的关系型数据库（如MySQL）使用的是正向索引。在这种索引结构中，数据库维护一个表，其中每行表示一个数据记录，每列保存记录的一个属性。索引会针对一个或多个列创建，以加速查询操作。当进行查询时，数据库利用这些索引来快速找到满足条件的行。

**优势：**

- **事务支持**：关系型数据库提供了对事务的支持，保证了数据的一致性和完整性。
- **复杂查询**：支持复杂的SQL查询，包括联结、分组和聚合等操作。
- **数据模型的严格性**：数据以行和列的形式严格组织，每列数据类型固定，保证了数据的结构性和完整性。

#### 倒排索引

Elasticsearch 使用的是倒排索引。倒排索引的工作原理是将文档的内容分解成一系列的单词或术语，并为每个单词建立一个索引，指向包含该单词的所有文档。这种索引结构非常适合全文搜索，因为它允许系统快速定位包含特定词汇的所有文档。

**优势：**

- **高效的全文搜索**：倒排索引非常适合于全文搜索操作，因为它可以快速找到包含某个特定词汇的所有文档。
- **实时搜索**：Elasticsearch 设计用于近乎实时的搜索，可以快速地对新索引的文档进行搜索。
- **可扩展性**：Elasticsearch 为分布式搜索和分析设计，易于水平扩展，能够处理大量数据。

#### 对比

例如在 MySQL 中有下面这张表 tb_goods ，可以看到该表是对表中的id字段创建索引的：

| id   | title          | price |
| ---- | -------------- | ----- |
| 1    | 小米手机       | 3499  |
| 2    | 华为手机       | 4999  |
| 3    | 华为小米充电器 | 49    |
| 4    | 小米手环       | 299   |
| ...  | ...            | ...   |

此时，如果我们想查看商品表中有哪些商品是包含“手机”这个词的，需要执行以下SQL语句来进行模糊匹配：

```sql
select * from tb_goods where title like '%手机%'
```

进行模糊匹配时，尽管该表有索引，但显然也是不生效的。在索引不生效的情况下，MySQL 数据库会采取逐条扫描的方法以判断每一行数据中是否包含有“手机”。如果不包含，则丢弃；如果包含，则存放入结果集当中去。

![image-20240409225017286](C:\Users\Administrator\Desktop\assets\image-20240409225017286.png)

这样逐条扫描的方式，固然能够得到正确的结果集，但付出的时间代价是巨大的，查询的性能较差。因此，我们可以感受到正向索引在进行数据表的局部内容检索时，性能不佳。下面我们来看看倒排索引在此场景下的表现。

Elasticsearch 采取倒排索引，结构如下：

- 文档(document)：每条数据就是一个文档
- 词条(term)：文档按照语义划分成的词语

根据先前的 tb_goods 表的内容， tb_goods 表是一个商品表，每一个商品就是一个文档(document)，而商品名则根据中文语义分割成相应的词条(term)。比如说，小米手机这个商品名可以分割为“小米”和“手机”两个词条，对应的文档id都是1；华为手机可以分割为“华为”和“手机”两个词条，其中“华为”是先前未出现过的词条，只需要和文档id = 2 添加进表中，而“手机”是之前出现过的词条，Elasticsearch 的倒排索引中不能出现重复的词条，因此需要将id = 2添加到文档id中。以此类推，生成的倒排索引表如下：

| 词条(term) | 文档id  |
| ---------- | ------- |
| 小米       | 1, 3, 4 |
| 手机       | 1, 2    |
| 华为       | 2, 3    |
| 充电器     | 3       |
| 手环       | 4       |

在 Elasticsearch 中，文档被分析并转化成一系列的词条（terms）。这些词条是从文档的内容中提取出的单词或短语，它们用于构建索引。无论原文档的大小如何，都可以通过这种方式被转换成一组词条。

为了高效地查找这些词条，Elasticsearch 需要为每个唯一的词条创建一个索引。这个索引会指向包含该词条的所有文档。为了管理这些索引，Elasticsearch 采用不同的数据结构来优化性能：

- **哈希法**：当数据量较小，词条数量有限时，使用哈希表来存储索引是非常高效的，因为哈希表提供了快速的查找时间，理想情况下是常数时间（O(1)）。
- **B+树**：随着数据量的增大，词条的数量也会急剧增加，此时维护一个高效的索引结构变得更加重要。B+树是一种自平衡的树，特别适合于处理大量数据的排序和范围搜索。B+树通过保持数据的有序性和优化磁盘读取操作，提高了大数据集上的搜索效率。

通过使用倒排索引，结合哈希法和B+树等数据结构的优势，Elasticsearch 能够极大地提升搜索的速度。无论是精确匹配还是复杂的全文搜索，Elasticsearch 都能快速定位到包含特定词条的文档，从而实现高效的搜索和数据分析，使得无论数据规模如何变化，都能保证快速的查询响应时间。

假如说我们在 Elasticsearch 中搜索“华为手机”，我们显然可以得到两组文档id：

- 华为：2, 3
- 手机：1, 2

再通过获取到的 id 去检索 id 为1，2，3的文档，并将所得文档放入结果集。

![image-20240410224540181](C:\Users\Administrator\Desktop\assets\image-20240410224540181.png)

### Elasticsearch 中的一些概念

#### 文档(Document)

- **定义**：在 Elasticsearch 中，文档是可搜索的信息单位。这些文档以JSON（JavaScript Object Notation）格式存储，它是一种广泛使用的轻量级数据交换格式。
- **性质**：每个文档都有唯一的ID，并存储在一个索引中。文档可以包含多种类型的数据，如字符串、数字、日期等。

一个电商网站的商品信息、社交媒体上的一条消息、或日志文件中的一个条目，都可以作为一个文档被存储在 Elasticsearch 中。如前文中提到的数据表，转化为json格式后储存在 Elasticsearch 中。

```json
{
    "id": 1,
    title: "小米手机"
    price: 3499
}
...
```

#### 索引(Index)

- **定义**：索引是文档的集合。在 Elasticsearch 中，索引被用作组织文档的方式。
- **用途**：通过索引，可以高效地执行搜索、更新和分析操作。一个索引包含具有相似结构（例如字段集）的文档。

![image-20240411104033553](C:\Users\Administrator\Desktop\assets\image-20240411104033553.png)

Elasticsearch 提供了强大的索引管理功能，包括创建、删除索引，以及定义索引映射和设置。

#### 映射(Mapping)

- **定义**：映射是定义文档如何存储和索引的过程。它类似于数据库中的表结构定义。
- **组件**：映射包括字段名和字段类型，还可以包括字段如何被 Elasticsearch 索引和存储的详细信息。
- **动态映射**：Elasticsearch可以自动检测字段类型并创建映射，但也支持显式定义映射，以更细致地控制索引过程。

#### 与 MySQL 的概念对比

| 功能对比 | MySQL  | Elasticsearch | 说明                                                |
| -------- | ------ | ------------- | --------------------------------------------------- |
| 容器     | Table  | Index         | MySQL的Table对应ES的Index，存储相关数据集。         |
| 数据项   | Row    | Document      | MySQL的Row对应ES的Document，表示单个数据实体。      |
| 数据属性 | Column | Field         | MySQL的Column对应ES的Field，表/文档中的一个数据点。 |
| 数据结构 | Schema | Mapping       | 定义数据模型和类型，MySQL的Schema对应ES的Mapping。  |
| 查询语言 | SQL    | DSL           | MySQL使用SQL，而ES使用基于JSON的DSL进行数据操作。   |

- Mysql: 擅长事务类型操作，可以确保数据的安全和一致性

- Elasticsearch: 擅长海量数据的搜索、分析、计算

MySQL 和 Elasticsearch 在不同的领域拥有各自的优势。

MySQL是一个关系型数据库管理系统，擅长处理事务类型操作。事务是一系列数据库操作的集合，它们要么全部成功执行，要么全部失败回滚，以确保数据的一致性和完整性。MySQL通过使用ACID（原子性、一致性、隔离性和持久性）特性来支持事务。这意味着在并发访问和多用户环境下，MySQL可以确保数据的安全性和一致性。如果业务场景（如下单、付款等场景）中需要频繁地进行数据修改、更新和删除，并且需要确保数据的完整性，那么MySQL是一个很好的选择。

另一方面，Elasticsearch 是一个分布式的搜索和分析引擎，专注于处理海量数据的搜索、分析和计算。它基于Apache Lucene构建，提供了强大的全文搜索功能和复杂的数据分析能力。Elasticsearch特别适用于处理大规模的文本数据、日志数据、时间序列数据等。它具有快速的搜索速度和灵活的查询语言，能够处理来自不同数据源的结构化和非结构化数据。如果你需要构建一个能够快速检索和分析海量数据的应用，比如搜索引擎、日志分析系统、实时监控系统等，那么选择 Elasticsearch 显然是更为合适的。

### Elasticsearch 搜索的一些底层原理

ES的搜索原理主要基于倒排索引和TF-IDF打分算法。

倒排索引（Inverted Index）是一种索引结构，它将词项（term）映射到包含该词项的文档列表。倒排索引的关键是将每个文档拆分成独立的词项，然后为每个词项建立索引。这样，当一个查询词出现时，可以快速地找到包含该词的文档列表。

TF-IDF（Term Frequency-Inverse Document Frequency）是一种用于评估一个词对于一个文档集或语料库的重要性的统计方法。TF指的是词项在文档中的出现次数，IDF指的是逆文档频率，即词项在整个文档集合中的逆频率。TF-IDF打分算法的目的是根据词项在文档中的频率和在整个文档集合中的分布情况，为每个文档计算一个得分。得分高的文档在搜索结果中排名靠前。

当使用TF-IDF算法进行打分时，可以通过以下公式计算每个词项的TF-IDF分值：

TF-IDF = TF * IDF

其中：

- TF（Term Frequency）指的是词项在文档中的出现频率。可以使用不同的计算方法，常见的有原始频率（Raw Frequency）和词频归一化（Term Frequency Normalization），可以根据实际需求选择。

- IDF（Inverse Document Frequency）指的是逆文档频率，是根据词项在整个文档集合中的出现情况来评估其重要性。常见的计算方法为1除以词项的文档频率的对数。计算公式如下：

  IDF = log(N / (DF + 1))

  其中，N是文档集合的总文档数，DF是包含词项的文档数。为了避免出现分母为0的情况，DF常常加上一个平滑项，通常设置为1。

举个例子，假设有一个文档集合包含4个文档，关键词"apple"在文档中的出现频率如下：

文档1: apple apple apple
文档2: apple
文档3: apple orange
文档4: orange

计算TF：
文档1的TF = 3
文档2的TF = 1
文档3的TF = 2
文档4的TF = 1

计算DF：
词项"apple"的DF = 3 （出现在文档1、文档2、文档3中）
词项"orange"的DF = 2 （出现在文档3、文档4中）

计算IDF：
词项"apple"的IDF = log(4 / (3 + 1)) = log(1) = 0
词项"orange"的IDF = log(4 / (2 + 1)) = log(4/3) ≈ 0.2877

计算TF-IDF：
文档1的TF-IDF = TF * IDF = 3 * 0 = 0
文档2的TF-IDF = TF * IDF = 1 * 0 = 0
文档3的TF-IDF = TF * IDF = 2 * 0.2877 ≈ 0.5755
文档4的TF-IDF = TF * IDF = 1 * 0.2877 ≈ 0.2877

根据TF-IDF分值的大小，可以将文档3排在搜索结果中的前面。

在ES中，倒排索引被用于记录每个词项（term）在哪些文档中出现，以及在每个文档中的位置信息等。当用户发起查询时，ES会根据查询词项在倒排索引中的出现情况，找到包含这些词项的文档列表。然后，ES会使用TF-IDF打分算法为每个文档计算得分，并根据得分进行排序，以确定搜索结果的排名。

### Kibana

Kibana 是一个开源的分析和可视化平台,它是 Elastic Stack (包括ElasticSearch、Logstash和Beats)的一个重要组件。它主要用于搜索、查看和可视化存储在 Elasticsearch 中的数据。

安装完成后，我们可以通过访问指定端口（默认5601）来使用Kibana。

我们可以使用 Kibana 内置的 Dev Tools 直接与Elasticsearch进行交互，执行各种操作和测试，无需通过代码或外部工具，方便开发、测试和故障排查。

### 分词器

在创建倒排索引时，需要对文档进行分词；在用户进行搜索时，需要对输入的内容进行分词。然而，Elasticsearch 默认的分词规则对中文处理并不友好。

#### 标准分词器(Standard Tokenizer)

默认使用的标准分词器(Standard Tokenizer)主要针对英文等西方语言进行了优化,对于中文分词存在一些缺陷。标准分词器将中文文本视为字节流,按照一些基本规则(如空格或标点符号)进行分词,无法准确地将中文分割为有意义的词语,可能导致搜索结果不准确或匹配度较低。

我们可以在 Kibana 中的 DevTools 中进行测试：

![image-20240411115131521](C:\Users\Administrator\Desktop\assets\image-20240411115131521.png)

可以看到对于英文单词可以正确划分，但是对于中文却是逐字划分。显然，默认分词器对于中文的语义是无法理解的。想要对中文内容做出正确的分词，我们需要引入新的分词器。

#### IK 分词器简介

为了优化中文分词,Elasticsearch 提供了一些第三方分词插件，如 IK、jieba 等,专门用于处理中文分词。这些插件基于统计学或词库方式,能够更准确地将中文文本分割为有意义的词语。在创建 Elasticsearch 索引时，可以通过配置分词器(analyzer)的方式,指定使用专门的中文分词插件，从而优化中文文档的索引和搜索效果。

除了使用现成的分词插件外，也可以根据实际需求自定义分词规则，以获得更精准的分词结果。在使用 Elasticsearch 处理中文文本时，合理配置和使用专门的中文分词器有助于提高索引的准确性，并能够显著改善搜索查询的效果和用户体验。

IK分词器是一个开源的，基于Java语言的轻量级的中文和英文分词工具，常用于结构化数据和非结构化数据的分词处理。它主要包括两部分：

- **IK 词典分词器**：基于词典分词的分词器，比如对英文采用最大正向切分、对中文采用细粒度切分策略。它提供了内置的主词典和Stop Word词典，也支持自定义词典和停用词词典。主词典包含了常用的中文词以及英文词，Stop Word词典则记录了很多语气词和无意义的单词。用户可以很容易的扩展和更改这两个词典。

- **IK Smart 分词器**：IK Smart分词器是IK分词中智能切分的实现，它不仅支持词典分词，还采用了类似HMM（隐马尔科夫模型）算法对未登录词进行分词。IK智能分词器能切分出混合中英文词汇，并且对长词或者不太常见的词有更好的分词效果。

使用IK分词器，我们可以很方便的对中文和中英文混合进行分词。比如对于"我是程序猿，我喜欢编程"，IK分词可以很好的分成："我/是/程序/猿/，/我/喜欢/编程"。

在Elasticsearch中要使用IK分词器，需要先安装ik分词器插件，然后在索引映射中指定使用 ik_smart 或者 ik_max_word 这两种分词器即可。ik_smart 的分词粒度较粗，而ik_max_word 的分词粒度较细

启用 ik_smart 分词器后，得到的结果如下：

![image-20240411124227964](C:\Users\Administrator\Desktop\assets\image-20240411124227964.png)

#### IK 分词器分词原理

前文中已经提到，IK分词器能够有效地处理中文分词，提供了两种分词模式：最细粒度的分词模式(ik_max_word)和智能分词(ik_smart)模式。下面是IK分词器的基本分词原理：

##### 词典的使用

IK分词器使用了两种词典：主词典和扩展词典。主词典包含了大量的中文词汇基础数据，扩展词典用于添加用户自定义词汇。

- **主词典**：包含了常用的中文词汇，是分词过程中的基础数据。

  主词典是IK分词器中最基础也是最重要的部分，它包含了大量中文词汇的基础数据，是分词过程中的核心词库。这个词典的设计目的是覆盖尽可能多的通用中文词汇，包括但不限于常用的名词、动词、形容词等。主词典的特点包括：

  - **广泛覆盖**：涵盖了大量的中文常用词汇，确保在大多数标准文本中能够高效准确地进行分词。
  - **优化性能**：词汇的选择和排列都经过优化，以加快分词过程中的匹配速度。
  - **基础性**：作为分词的基础，确保了分词器能够在没有额外信息的情况下运行。

- **扩展词典**：用户可以根据自己的需求添加新的词汇，用于满足特定领域的分词需求。

  扩展词典允许用户根据自己的需求添加新的词汇或专业术语，是一种自定义功能，让IK分词器可以更好地适应特定领域或专业文本的处理。扩展词典的特点包括：

  - **可定制性**：用户可以根据自己的需要，添加或删除词汇，使分词器更适合特定的应用场景。
  - **灵活性**：扩展词典可以随时更新，不需要修改分词器本身的代码，便于维护和升级。
  - **专业性**：通过添加专业领域的术语，可以提高分词的准确率和适用性，特别是在处理专业文本或行业内部资料时。

  要扩展 ik 分词器的词库，修改 ik 分词器挂载目录下的 config/analysis-ik/IKAnalyzer.cfg.xml 文件，在对应的标签内添加词典即可。

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
  <properties>
          <comment>IK Analyzer 扩展配置</comment>
          <!--用户可以在这里配置自己的扩展字典 -->
          <entry key="ext_dict"></entry>
           <!--用户可以在这里配置自己的扩展停止词字典-->
          <entry key="ext_stopwords"></entry>
          <!--用户可以在这里配置远程扩展字典 -->
          <!-- <entry key="remote_ext_dict">words_location</entry> -->
          <!--用户可以在这里配置远程扩展停止词字典-->
          <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
  </properties>
  ```

当进行分词处理时，IK分词器首先会在主词典中查找匹配的词汇。如果在主词典中找不到，然后会在扩展词典中搜索。这种结合使用主词典和扩展词典的方法，使得IK分词器能够在保持高效处理通用文本的同时，也能灵活适应于特定领域或专业文本的需求。

- **停用词典**：用于存放那些在文本分析中通常被视为不携带有用信息的词汇，比如一些常见的助词、介词、连词等。在进行中文分词时，来自停用词典的词汇会被自动过滤掉，不会出现在最终的分词结果中。这样做的目的是为了减少文本处理的噪音，提高后续文本分析或搜索的准确性和效率。

  停用词典通常包括但不限于以下类型的词汇：

  - **常见的助词和介词**：例如“的”、“了”、“在”等。
  - **数字和符号**：很多时候数字和符号对于文本分析的价值不大。
  - **通用的名词、动词**：例如“事情”、“进行”等词汇，它们在特定情境下可能不携带有用信息。

  用户可以根据自己的需要，自定义停用词典中的内容，从而在分词过程中过滤掉不需要的词汇，以获得更加精准的分词结果。

##### 分词算法

IK分词器主要使用了正向最大匹配和逆向最大匹配的分词算法，以及基于DAG（有向无环图）和最短路算法的智能分词方法。

- **最细粒度分词**：该模式下，IK分词器会将文本切分成尽可能小的词汇。
- **智能分词**：在智能分词模式下，IK分词器会分析整个句子结构，尝试找出最合理的分词方式，以达到既不遗漏重要信息，也不产生太多无关词汇的效果。

## 实操Elasticsearch

### mapping属性

在 Elasticsearch 中，映射（mapping）是定义索引中字段（field）的数据类型和如何处理这些字段的过程。每个索引都有自己的映射，类似于传统数据库中表的 schema，用于定义字段名和字段数据类型等信息。映射可以显式定义，也可以由 Elasticsearch 在文档首次被索引时自动推断生成。

主要的 mapping 属性如下：

#### **字段数据类型（Field Data Types）**

定义了一个字段存储的数据类型，例如、`date`、`long`、`double`、`boolean`等。常见的简单类型如下：

- 字符串：`text`、`keyword`

  `text`类型用于全文搜索，而`keyword`类型用于结构化内容的过滤、排序和聚合。`text`类型的文本可以进行分词，而`keyword`类型的文本不可拆分，只有合在一起时才有意义。

- 数值：`long`、`integer`、`short`、`byte`、`double`、`float`

- 布尔：`boolean`

- 日期：`date`

- 对象：`object`

  在Elasticsearch中，`object`类型的属性用于存储JSON对象，这些对象内部可以包含多个字段。这种类型的字段允许你将文档中的结构化数据组织成层次化的方式，类似于传统数据库中的嵌套表结构。使用`object`类型的字段非常适合处理复杂数据结构，如包含多个属性的对象。

  `object`属性中的子字段完全可以参与搜索。Elasticsearch 将这些子字段看作是独立的字段，并允许你对这些子字段进行查询和聚合操作。

#### **字段分析器（Field Analyzers）**

analyzer 一般结合`text`类型的数据使用。对于文本字段，可以指定分析器（analyzer）来处理文本数据。分析器由分词器（tokenizer）和零个或多个过滤器（filters）组成，用于将文本分割成一系列的词条（tokens）。

该属性的词一般就是分词器的名称，如前文中我们使用过的 id_smart 和 id_max_word。

#### **索引选项（Index Options）**

定义字段是否被索引以及如何被索引。例如，一个字段可以设置为不可索引（即index设置为false），这意味着你不能基于该字段进行搜索。该选项默认为 true。

当在Elasticsearch中将某个字段的`index`选项设置为`false`时，意味着对于这个特定的字段不会创建倒排索引。这并不影响整个索引（Index）中其他字段的倒排索引创建；只有被设置为`index: false`的字段不参与索引，从而无法通过该字段进行搜索。

例如，有一个包含`user_description`字段的文档，我们不希望通过该字段进行搜索：

```http
PUT /my_index
{
  "mappings": {
    "properties": {
      "name": { 
        "type": "text"
      },
      "user_description": {
        "type": "text",
        "index": false
      }
    }
  }
}
```

在这个例子中，`name`字段会被索引，我们可以通过它进行搜索。而`user_description`字段虽然存储在索引中，但因为其`index`选项被设置为`false`，所以无法对其进行搜索查询。在实际开发中，可以将不需要参与搜索查询的字段的`index`选项设置为`false`。

#### 属性配置（Properties）

`properties`在Elasticsearch中使用的时刻主要是在定义或更新索引映射（Mapping）时。Mapping 是定义索引中文档如何存储和索引的规则集合，而`properties`则用于具体定义映射中每个字段（field）的类型和行为。对于包含嵌套对象或数组的文档，`properties`可以用来定义这些复杂结构内部每个元素的类型和配置。

#### **多字段（Multi-fields）**

在Elasticsearch中，多字段（multi-fields）功能允许你对同一个字段内容使用不同的索引类型或分析器进行索引。这样，一个字段就可以在不同的上下文中以不同的方式被搜索，增加了索引的灵活性和搜索的功能性。

多字段最常见的使用场景之一是对文本数据既进行全文搜索又进行关键词（精确值）搜索。例如，一个文本字段可能需要通过全文搜索来查找包含特定词汇的文档，同时也需要作为一个过滤条件来匹配确切的值。

在映射（mapping）定义时，可以为特定字段指定一个或多个子字段。每个子字段可以有自己的类型和分析器设置。以下是一个简单的例子，展示了如何为`message`字段定义一个多字段，这个字段既可以作为全文搜索的`text`类型，也可以作为精确搜索的`keyword`类型：

```http
PUT /my_index
{
  "mappings": {
    "properties": {
      "message": { 
        "type": "text",
        "fields": {
          "raw": { 
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

一旦定义了多字段，就可以在查询时指定使用主字段或任何子字段。根据上面的定义，若对`message`字段进行全文搜索，可以直接查询`message`。而如果想要进行精确匹配搜索，则可以查询`message.raw`。

以下是一个使用`match`查询进行全文搜索的例子：

```http
jsonCopy codeGET /my_index/_search
{
  "query": {
    "match": {
      "message": "this is a test"
    }
  }
}
```

如果进行精确搜索，可以这样查询：

```http
jsonCopy codeGET /my_index/_search
{
  "query": {
    "term": {
      "message.raw": "this is a test"
    }
  }
}
```

### 索引库操作

在 Elasticsearch 中，索引库（通常简称为索引）是其最核心的概念之一，相当于传统数据库中的“数据库”概念。一个索引存储的是某类数据的集合，比如一个索引可以存储所有的客户数据，另一个索引存储所有的订单数据。

#### 创建索引库

在 Elasticsearch 中，通过 RESTful 请求操作索引和文档。请求内容用 DSL 语句来表示。创建索引库和 mapping 的 DSL 语法如下：

以下是使用 Elasticsearch RESTful API 进行索引创建和映射定义的基本步骤和 DSL（领域特定语言）语法：

##### 基本的创建索引请求

创建一个新的索引，最简单的请求只需要指定索引名称：

```http
PUT /<index-name>
```

这里，`<index-name>`是你想要创建的索引的名称，也即索引库的名称。

##### 添加映射

在创建索引的同时，你也可以定义该索引的映射。映射定义了索引中文档的结构，包括字段名、字段类型以及其他设置（如是否索引该字段、是否存储原文等）。

```http
PUT /<index-name>
{
  "mappings": {
    "properties": {
      "info": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email": {
        "type": "integer",
        "index": "false"
      },
      "name": {
        "properties": {
            "firstname": {
                "type": "keyword"
            },
            "lastname": {
                "type": "keyword"
            }
        }
      },
      // 更多字段定义...
    }
  }
}
```

在这个例子中，`"mappings"`部分定义了索引的结构，包括各个字段的名称和类型。`"properties"`下面是字段名和字段类型的定义，例子中提到的字段名如下：

1. **info** 字段：
   - 类型被设置为 `text`，这意味着该字段用于全文搜索。
   - 使用 `ik_smart` 分析器对文本进行分词处理，`ik_smart` 是IK Analysis Plugin提供的一个分词器，适用于对中文文本做更智能的分词。
2. **email** 字段：
   - 类型被设置为 `keyword`，用于精确值搜索，比如过滤或聚合。
   - `index` 设置为 `false`，表示该字段不会被索引，即不能基于这个字段进行搜索操作。
3. **name** 字段：
   - 它是一个对象，包含一个名为 `firstName` 的子字段。
   - `firstName` 的类型被设置为 `keyword`，这表明它用于精确值搜索。

##### 指定索引设置

除了映射之外，创建索引时还可以指定一些索引级别的设置，如分片数（`number_of_shards`）和副本数（`number_of_replicas`）等：

```http
PUT /<index-name>
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
  },
  "mappings": {
    "properties": {
      // 字段定义...
    }
  }
}
```

在这个例子中，`"settings"`部分用于配置索引的分片和副本数量。分片数决定了索引的数据分布方式，而副本数则关系到数据的可用性和搜索操作的性能。

##### 注意事项

- 索引名称必须全部小写，不可包含空格、冒号等特殊字符。
- 一旦创建，索引的某些设置（如分片数）不能更改。不过，可以通过变更策略、重新索引等方式来调整。
- 映射一旦定义，字段的数据类型不能更改。如果需要修改，只能通过创建新的索引和重新索引数据来完成。

#### 删除索引库

在 Elasticsearch 中，删除索引是一个不可逆的操作，一旦执行，所有的索引数据和元数据都将被永久移除。因此，在进行删除操作之前，应确保已经正确备份了所有需要的数据，或者确定不再需要该索引中的数据。

删除一个索引库的基本 HTTP 请求如下：

```http
DELETE /<index-name>
```

这里的`<index-name>`是想要删除的索引的名称。

例如，如果想要删除名为`my_index`的索引，请求如下：

```http
DELETE /my_index
```

当执行删除操作时，Elasticsearch会返回一个响应，表明操作是否成功。如果索引成功删除，通常会返回以下响应：

```http
{
  "acknowledged": true
}
```

如果尝试删除不存在的索引，Elasticsearch 默认会返回一个错误。不过，我们可以添加`?ignore_unavailable=true`参数来避免因为索引不存在而导致的错误响应：

```http
DELETE /my_index?ignore_unavailable=true
```

此外，还有一些其他的删除操作参数，比如`?allow_no_indices=false`，这个参数的作用是在没有匹配的索引时返回错误。

##### 注意事项

- 删除操作不可逆，请确保真的不再需要该索引中的数据再执行删除。
- 如果是在生产环境中，建议在删除前执行适当的备份。
- 在删除操作时，可加入相应的错误处理机制，比如检查索引是否存在。

#### 修改索引库

在 Elasticsearch 中，一旦创建了索引，某些核心的索引设置就**不能修改**。例如，你不能更改一个索引的分片数(`number_of_shards`)。这是因为分片数直接影响了数据在集群中的分布方式和 Elasticsearch 路由机制的核心部分。改变它会要求重建索引和重新分配数据，这是 Elasticsearch 当前设计中不支持的。

##### 更改可变的索引设置

然而，某些索引的设置是可以修改的，例如：

- **副本数量(number_of_replicas)**：可以更改索引的副本数量以适应不同的可用性或性能需求。
- **索引设置**：某些特定的索引级别的设置也是可以动态调整的，例如，索引的刷新间隔(`refresh_interval`)。

假设想要更改索引的副本数量，可以发送一个请求到Elasticsearch的`_settings`端点：

```http
PUT /<index-name>/_settings
{
  "number_of_replicas": 2
}
```

在这个例子中，`<index-name>`是你的索引名称，`number_of_replicas`设置为2意味着每个分片有两个副本。

如果要更改索引的刷新间隔，可以使用如下请求：

```http
PUT /<index-name>/_settings
{
  "refresh_interval": "30s"
}
```

这将更改索引的刷新间隔为每30秒。

##### 添加新的字段到映射

关于映射（mapping），我们不能删除或更改原有字段的信息，但是一些特性也是可以更改的：

- **添加字段**：你可以向现有映射中添加新的字段。
- **更新字段**：虽然你不能更改现有字段的类型，但可以添加新的多字段（multi-fields）或者更新某些字段的设置（只要它们不会改变字段的数据类型或者分析器）。

如果你需要向现有索引的映射中添加一个新字段，你可以使用`_mapping`端点：

```http
PUT /<index-name>/_mapping
{
  "properties": {
    "new_field": {
      "type": "text"
    }
  }
}
```

`new_field`是要添加的新字段名，类型被设置为`text`。

##### 使用_reindex API重新索引数据

如果需要更改不能修改的索引设置或映射，我们需要创建一个新的索引并将数据从旧索引重新索引（reindex）到新索引。

如果需要更改不可变的索引设置，例如更改分片数量，你需要重新索引你的数据到一个新的索引。这可以通过`_reindex` API完成：

```http
POST /_reindex
{
  "source": {
    "index": "old_index_name"
  },
  "dest": {
    "index": "new_index_name"
  }
}
```

在这个例子中，`old_index_name`是旧的索引名，`new_index_name`是新创建的索引名，新索引应该已经被创建并带有所需的设置和映射。

##### 更改多字段设置

如果想在现有字段上添加一个多字段来支持不同的搜索需求，比如添加一个不分词的`keyword`字段：

```http
PUT /<index-name>/_mapping
{
  "properties": {
    "existing_field": {
      "type": "text",
      "fields": {
        "raw": { 
          "type":  "keyword"
        }
      }
    }
  }
}
```

在这个例子中，`existing_field`是已经存在的字段，我们添加了一个名为`raw`的多字段。

##### 注意事项

- 在 Elasticsearch 中，绝大多数对于索引的更改都是不被允许的。

- 添加新字段通常是安全的，但修改现有字段的类型或分析器则通常不被允许。

#### 索引库操作小结

- 创建索引库：PUT /<index-name>
- 查询索引库：GET /<index-name>
- 删除索引库：DELETE /<index-name>
- 添加字段：PUT /<index-name>/_mapping

### 文档操作

在 Elasticsearch 中，数据以 JSON 文档的形式存储在索引中，每个文档都具有唯一的 ID，并属于一种类型（从 7.x 版本开始，Elasticsearch 弃用了多类型的概念）。文档操作是 Elasticsearch 日常使用中的基本环节，主要包括以下几个方面：

#### 添加文档

添加新文档到索引中。如果文档已经存在，则索引操作将替换旧文档。这可以通过 POST 或 PUT 请求来完成，指定文档的 ID 或让 Elasticsearch 自动生成一个 ID。

注意：在 Elasticsearch 中，

- **使用 PUT 请求时，必须指定文档 ID**。PUT 请求通常用于创建新文档或替换现有文档，你需要提供文档 ID 作为 URL 的一部分。如果文档已存在，则该请求会更新该文档并增加其版本号。

- **使用 POST 请求时，可以不指定文档 ID**。当使用 POST 请求向一个索引添加文档而不在 URL 中指定文档 ID 时，Elasticsearch 会自动为这个文档生成一个唯一的 ID。这种方式适用于那些文档 ID 不重要或者不希望自己管理文档 ID 的场景。

其对应的 DSL 语法如下：

- **PUT 请求**：需要在 URL 中明确指定文档的 ID。例如：

  ```http
  PUT /<index>/_doc/<doc_id>
  {
      "field": "value",
      "field2": "value2"
  }
  ```

  这个请求会将数据存储到指定的 `<doc_id>` 中。如果该 ID 已存在，则原有文档将被新文档替换。

- **POST 请求**：如果不在 URL 中指定文档 ID，则 Elasticsearch 会自动生成一个 ID。例如：

  ```http
  POST /<index>/_doc/
  {
      "field": "value",
      "field2": "value2"
  }
  ```

  这个请求会让 Elasticsearch 自动生成一个文档 ID 并添加这个新文档到索引中。

#### 查询文档

查询，也叫检索(Retrieving)，即通过文档的 ID 获取文档。直接通过文档的 ID 进行访问，非常快速。

```http
GET /<index>/_doc/<doc_id>
```

#### 删除文档

删除(Deleting) ，即通过文档的 ID 删除文档。

```http
DELETE /<index>/_doc/<doc_id>
```

#### 修改文档

在 Elasticsearch 中，修改文档可以通过两种基本方式进行：全量修改和增量修改。每种方式有其特定的使用场景和实现方式。

##### 全量修改(Full Update)

全量修改通常涉及替换现有文档的整个内容。在 Elasticsearch 中，这通常是通过发送一个 PUT 请求实现的。如果指定的文档 ID 已经存在，此操作会替换该文档的全部内容。如果文档 ID 不存在，则相当于创建了一个新的文档。

```http
PUT /<index>/_doc/<doc_id>
{
    "newField1": "newValue1",
    "newField2": "newValue2",
    ...
}
```

在这个例子中，如果 `<doc_id>` 存在，Elasticsearch 会用提供的新内容替换整个文档。如果 `<doc_id>` 不存在，它将创建一个新文档。这意味着所有之前在文档中的数据将会被新数据完全覆盖。

##### 增量修改(Partial Update)

增量修改是指只更新文档的一部分字段，而不是替换整个文档。这种操作在 Elasticsearch 中通常是通过 _update API 实现的。

```http
POST /<index>/_doc/<doc_id>/_update
{
    "doc": {
        "fieldToUpdate": "newValue",
        "anotherFieldToUpdate": "anotherNewValue"
    }
}
```

在这个例子中，只有 `fieldToUpdate` 和 `anotherFieldToUpdate` 会被更新到新的值，文档中的其他字段保持不变。这种方式非常适合只需要修改部分信息的情况，可以减少数据传输并且提高效率。

##### 冲突处理

在进行文档更新时，尤其是在高并发环境下，可能会出现冲突，例如两个并发进程试图同时更新同一个文档。在 Elasticsearch 中处理并发和冲突时，可以使用版本控制和重试机制帮助来数据的一致性。

- **版本控制**：每个文档在 Elasticsearch 中都有一个版本号。我们可以在更新请求中指定一个版本号，如果该版本号与服务器上的版本号不匹配，更新操作将失败。这可以避免丢失并发更新带来的更改。

  假设现在有一个文档当前的版本号是 10，我们希望它基于这个版本进行更新：

  ```http
  POST /<index>/_doc/<doc_id>/_update?version=10
  {
      "doc": {
          "fieldToUpdate": "updatedValue"
      }
  }
  ```

  在这个请求中，通过查询字符串 `version=10` 指定了期望的版本号。如果文档的当前版本号不是 10（例如，如果其他用户已经更新了该文档），Elasticsearch 将拒绝这次更新，并返回一个版本冲突错误。

- **重试机制**：在 _update API 中，你可以指定 `retry_on_conflict` 参数，告诉 Elasticsearch 在发生版本冲突时自动重试更新操作若干次。

  尤其是在高并发环境中，你可能希望 Elasticsearch 在遇到版本冲突时自动重试更新操作。这可以通过设置 `retry_on_conflict` 参数来实现。

  假设 Elasticsearch 在更新文档时，如果发生版本冲突，自动重试最多 3 次：

  ```http
  POST /<index>/_doc/<doc_id>/_update?retry_on_conflict=3
  {
      "doc": {
          "fieldToUpdate": "newValue"
      }
  }
  ```

  在这个请求中，`retry_on_conflict=3` 告诉 Elasticsearch 如果初次尝试失败了（由于版本冲突），可以再尝试最多三次更新操作。这种机制可以有效处理在高并发环境下出现的临时冲突。

#### 文档操作小结

- 创建文档

  ```http
   POST /索引名/_doc/文档id { [json文档] }
  ```

- 查询文档

  ```http
  GET /索引名/_doc/文档id
  ```

- 删除文档

  ```http
  DELETE /索引名/_doc/文档id
  ```

- 修改文档

  - 全量修改：使用 PUT 请求替换索引中的指定文档

    ```http
    PUT /索引名/_doc/文档id
    {
        [json文档]
    }
    ```

  - 增量修改：使用 POST 请求更新索引中的指定文档的部分信息

    ```http
    POST /索引名/_update/文档id
    {
        "doc": {
            [字段更新]
        }
    }
    ```

### RestClient

`RestClient` 通常指的是一个用于在应用程序中发送 HTTP 请求并处理 HTTP 响应的客户端库。这种客户端库可以在多种编程环境中实现，包括但不限于 .NET、Java、Ruby 或 Python 等。它们通常用于与 RESTful API 交互，这些 API 是按照 Representational State Transfer（表现层状态转换）原则设计的网络应用程序接口。

在不同的编程语言和框架中，`RestClient` 的具体实现和功能可能会有所不同，但它们大多数提供以下基本功能：

1. **发送 HTTP 请求**：允许你构造和发送包括 GET、POST、PUT、DELETE 等在内的 HTTP 请求。
2. **处理响应**：接收并解析 HTTP 响应，包括状态码、响应体和头部信息。
3. **错误处理**：提供错误处理机制，例如捕获网络错误或解析错误响应代码。
4. **配置请求**：支持配置请求头、查询参数、请求体等。
5. **认证与授权**：支持常见的认证机制，例如基本认证、Token 认证等。
   在 Elasticsearch 中，Java REST Client 通常包括两种客户端：低级客户端（Low-Level Client）和高级客户端（High-Level Client）。这两种客户端各有特点和适用场景，以下是它们的主要区别：

#### 低级客户端（Low-Level Client）

1. **基本功能**：低级客户端提供了与 Elasticsearch 集群通信的基本功能。它主要负责发送 HTTP 请求和接收响应，不对请求或响应做任何高级处理。
2. **灵活性**：低级客户端允许开发者直接控制 HTTP 请求的各个方面，如URL、请求头、请求体等。这种灵活性对于需要精细控制其 Elasticsearch 交互的高级用户来说非常有用。
3. **返回类型**：低级客户端通常返回 HTTP 响应的原始数据，如状态码和响应体（通常是 JSON 格式的字符串），而不是将其转换为 Java 对象。
4. **错误处理**：低级客户端不会对错误做特殊处理。如果 Elasticsearch 返回一个错误状态码，客户端只是简单地将这些信息返回给用户，不会抛出异常。

#### 高级客户端（High-Level Client）

1. **易用性**：高级客户端在低级客户端的基础上构建，提供了更多易用的特性。它提供了丰富的 API 来直接操作 Elasticsearch，如搜索、索引管理、聚合等。
2. **对象映射**：高级客户端自动将 HTTP 响应映射为 Java 对象，方便开发者使用。这包括对返回的 JSON 数据进行解析，并将其转换成相应的 Java 类型。
3. **错误处理**：高级客户端会处理来自 Elasticsearch 的错误响应，并根据错误类型抛出 Java 异常。这样，开发者可以通过异常处理来管理错误情况。
4. **依赖**：高级客户端依赖于低级客户端来执行实际的 HTTP 通信。这意味着高级客户端实际上是一个在低级客户端之上的抽象层，提供更方便的 API 和更丰富的功能。

### Elasticsearch Java API Client

事实上，在 Elasticsearch 7.17版本后，官方已经将 RestClient 的高级客户端 RestHighLevelClient 标记为弃用状态(decreased)

Elasticsearch Java API Client 是 Elasticsearch 官方推出的新一代 Java 客户端，用于与 Elasticsearch 集群进行交互。这个客户端旨在提供一种更现代、类型安全、易于使用的方式来访问 Elasticsearch 的功能。它从 Elasticsearch 7.x 版本开始提供，对应于 Elasticsearch 8.x 版本提供了全面支持。

`WebClient`提供了一个更现代的、功能强大的异步HTTP客户端。它支持同步和异步操作，可以与其他反应式组件一起使用。官方文档的地址如下：

https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/index.html

#### 索引操作

以下是几个基本的操作示例，包括创建索引、获取索引信息、修改索引以及删除索引。

##### 创建索引

在创建索引时，可以指定索引的各种设置，如分片数量、副本数量以及映射类型等。这里是一个基本的创建索引的示例：

```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.indices.CreateIndexRequest;
import co.elastic.clients.elasticsearch.indices.CreateIndexResponse;

public class ElasticsearchService {

    private final ElasticsearchClient elasticsearchClient;

    public ElasticsearchService(ElasticsearchClient elasticsearchClient) {
        this.elasticsearchClient = elasticsearchClient;
    }

    public void createIndex(String indexName) throws Exception {
        CreateIndexResponse response = elasticsearchClient.indices().create(create -> create
            .index(indexName)
            .settings(settings -> settings
                .numberOfShards(1)
                .numberOfReplicas(1))
        );
        if (response.acknowledged()) {
            System.out.println("索引创建成功");
        } else {
            System.out.println("索引创建失败");
        }
    }
}
```

##### 获取索引信息

获取索引的信息可以帮助我们了解索引的配置和状态。以下是获取索引信息的代码示例：

```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.indices.GetIndexResponse;

public void getIndex(String indexName) throws Exception {
    GetIndexResponse response = elasticsearchClient.indices().get(get -> get
        .index(indexName)
    );
    System.out.println("索引信息：" + response.toString());
}
```

##### 修改索引

修改索引通常涉及到改变索引的配置，如调整副本数量。这里是一个调整索引副本数量的示例：

```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.indices.PutSettingsResponse;

public void updateIndexSettings(String indexName) throws Exception {
    PutSettingsResponse response = elasticsearchClient.indices().putSettings(put -> put
        .index(indexName)
        .settings(settings -> settings
            .numberOfReplicas(2))
    );
    if (response.acknowledged()) {
        System.out.println("索引设置更新成功");
    } else {
        System.out.println("索引设置更新失败");
    }
}
```

##### 4. 删除索引

删除索引是一个危险操作，一旦执行，所有索引内的数据都将丢失。以下是删除索引的代码示例：

```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.indices.DeleteIndexResponse;

public void deleteIndex(String indexName) throws Exception {
    DeleteIndexResponse response = elasticsearchClient.indices().delete(del -> del
        .index(indexName)
    );
    if (response.acknowledged()) {
        System.out.println("索引删除成功");
    } else {
        System.out.println("索引删除失败");
    }
}
```

#### 文档操作

在 Elasticsearch 中，对文档的操作包括创建（索引）、读取（获取）、更新和删除文档。在 Spring Boot 应用中使用 Elasticsearch Java API Client 进行这些操作时，可以按照以下方式实现。

##### 索引（创建）文档

索引文档实际上是在 Elasticsearch 中创建或更新文档。如果指定的文档 ID 不存在，它将创建一个新文档；如果已存在，则会更新该文档。

```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.IndexResponse;

public class DocumentService {

    private final ElasticsearchClient elasticsearchClient;

    public DocumentService(ElasticsearchClient elasticsearchClient) {
        this.elasticsearchClient = elasticsearchClient;
    }

    public void indexDocument(String indexName, String docId, Object document) throws Exception {
        IndexResponse response = elasticsearchClient.index(i -> i
            .index(indexName)
            .id(docId)
            .document(document)
        );
        System.out.println("文档操作状态：" + response.result().toString());
    }
}
```

##### 获取（读取）文档

获取操作是读取 Elasticsearch 中某个特定文档的内容。这里需要提供索引名和文档的 ID。

```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.GetResponse;

public void getDocument(String indexName, String docId, Class<?> clazz) throws Exception {
    GetResponse<?> response = elasticsearchClient.get(g -> g
        .index(indexName)
        .id(docId), clazz
    );
    if (response.found()) {
        System.out.println("找到文档：" + response.source());
    } else {
        System.out.println("文档未找到");
    }
}
```

##### 更新文档

更新操作通常用于修改现有文档的内容。你可以使用部分更新来改变文档中的特定字段。

```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.UpdateResponse;

public void updateDocument(String indexName, String docId, Object partialDocument) throws Exception {
    UpdateResponse<?> response = elasticsearchClient.update(u -> u
        .index(indexName)
        .id(docId)
        .doc(partialDocument),
        Object.class // 适当的文档类
    );
    System.out.println("更新状态：" + response.result().toString());
}
```

##### 删除文档

删除操作用于从指定索引中删除一个文档。

```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.core.DeleteResponse;

public void deleteDocument(String indexName, String docId) throws Exception {
    DeleteResponse response = elasticsearchClient.delete(d -> d
        .index(indexName)
        .id(docId)
    );
    System.out.println("删除文档状态：" + response.result().toString());
}
```

##### 批量操作

在 Elasticsearch 中进行批量操作是一种有效管理文档的方式，特别是需要执行大量的创建、更新或删除操作时。使用 Elasticsearch Java API Client，在 Spring Boot 应用中执行批量操作可以大幅提高性能，减少网络往返次数。

批量操作（Bulk API）允许在单个 API 调用中执行多个索引、更新或删除操作。这是通过构建一个包含多个操作的请求完成的。

以下是如何在 Spring Boot 应用中使用 Elasticsearch Java API Client 进行批量操作的示例。这个例子将展示如何批量创建、更新和删除文档。

```java
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.Refresh;
import co.elastic.clients.elasticsearch.core.BulkOperation;
import co.elastic.clients.elasticsearch.core.BulkRequest;
import co.elastic.clients.elasticsearch.core.BulkResponse;
import co.elastic.clients.elasticsearch.core.bulk.*;

import java.util.ArrayList;
import java.util.List;

public class BulkDocumentService {

    private final ElasticsearchClient elasticsearchClient;

    public BulkDocumentService(ElasticsearchClient elasticsearchClient) {
        this.elasticsearchClient = elasticsearchClient;
    }

    public void performBulkOperations(String indexName) throws Exception {
        List<BulkOperation> operations = new ArrayList<>();

        // 准备创建文档的操作
        BlogPost blogPost1 = new BlogPost("Title 1", "Content 1");
        operations.add(BulkOperation.of(b -> b
            .index(idx -> idx
                .index(indexName)
                .document(blogPost1)
            )
        ));

        // 准备更新文档的操作
        operations.add(BulkOperation.of(b -> b
            .update(upd -> upd
                .index(indexName)
                .id("1")
                .doc(new BlogPost("Updated Title", "Updated Content"))
            )
        ));

        // 准备删除文档的操作
        operations.add(BulkOperation.of(b -> b
            .delete(del -> del
                .index(indexName)
                .id("2")
            )
        ));

        // 执行批量操作
        BulkResponse bulkResponse = elasticsearchClient.bulk(b -> b
            .index(indexName)
            .refresh(Refresh.True)
            .operations(operations)
        );

        // 检查操作结果
        if (bulkResponse.errors()) {
            System.out.println("批量操作中有错误发生");
        } else {
            System.out.println("批量操作成功完成");
        }
    }

    static class BlogPost {
        private String title;
        private String content;

        public BlogPost(String title, String content) {
            this.title = title;
            this.content = content;
        }

        // getters and setters
    }
}
```

在实际应用中，结合 MyBatis-Plus 和 Elasticsearch 进行批量操作确实是一个常见的做法，尤其是在需要从数据库批量提取数据并导入到 Elasticsearch 时。MyBatis-Plus 可以方便地处理批量数据的读取和写入，而 Elasticsearch 的 Bulk API 则可以高效地处理这些数据的索引化。

##### 使用 MyBatis-Plus 读取数据库数据

假设已经有一个数据库表对应的实体类和 Mapper，可以使用 MyBatis-Plus 的 `selectList` 或 `selectBatchIds` 方法来查询需要的数据。例如，从数据库批量读取数据：

```java
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.extension.service.IService;
import org.springframework.beans.factory.annotation.Autowired;

public class DatabaseService {

    @Autowired
    private IService<BlogPost> blogPostService;

    public List<BlogPost> fetchBlogPosts() {
        return blogPostService.list(new QueryWrapper<BlogPost>().lambda()
                .eq(BlogPost::getStatus, "active"));
    }
}
```