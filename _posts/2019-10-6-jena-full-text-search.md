---
layout: post
title: Jena 全文搜索配置
categories: 图数据库
tags: Jena 全文搜索
author: ZekeTian
---




* content
{:toc}
在本文中将介绍如何在 Jena 中使用全文搜索技术，涵盖配置过程、全文搜索的查询示例。另外需要注意的是，在本文中使用的 Jena、Jena fuseki 版本均为 3.6.0，不同的版本在配置过程中可能会有略微差异。



## 1. Jena 全文搜索简介

### 1.1 全文搜索介绍

> 将非结构化数据中的一部分信息提取出来，重新组织，使其变得有一定结构，然后对此有一定结构的数据进行搜索，从而达到搜索相对较快的目的。这部分**从非结构化数据中提取出的然后重新组织的信息，我们称之索引**。
>
> 这种**先建立索引，再对索引进行搜索的过程就叫全文检索(Full-text Search)**，使用了全文搜索技术后，查询速度将大提升。
>

### 1.2 Jena 全文搜索支持

通过 [Lucene](https://lucene.apache.org/) 或 [ElasticSearch](https://www.elastic.co/) ，ARQ 的扩展可以将 SPARQL 和全文搜索结合起来，从而使得应用程序能够在 SPARQL 查询中执行索引全文搜索，加快查询速度。 Jena 版本兼容性如下表所示：

| Jena          | Lucene     | Solr          | ElasticSearch  |
| ------------  | ---------  | ------------  | -------------  |
| upto 3.2.0    | 5.x or 6.x | 5.x or 6.x    | not supported  |
| 3.3.0 - 3.9.0 | 6.4.x      | not supported | 5.2.2 - 5.2.13 |
| 3.10.0        | 7.4.0      | not supported | 6.4.2          |
| TBD           | 7.5.x +    | not supported | 6.5.x +        |

## 2. 修改数据库配置文件

在通过浏览器可视化工具（输入 http://localhost:3030/ 访问的管理页面）创建 **Persistent** 类型的数据库后，会在 **“ jena-fuseki安装目录/run/configuration”** 自动创建该数据库对应的配置文件 **“database-name.ttl”**。首先，我们在修改该文件前，将文件复制一份作为备份，然后打开文件进行如下的一些修改。

### 2.1 添加 text 前缀

在配置文件中前面加上如下一行代码：

```
@prefix text:    <http://jena.apache.org/text#> .
```

### 2.2 设置 TextDataset

在配置文件中 **“:service_tdb_all”** 内容的下方添加如下代码：

```
# TextDataset 是一个含有文本索引的常规数据集
:text_dataset rdf:type     text:TextDataset ;
    text:dataset   :tdb_dataset_readwrite ;
    text:index     <#indexLucene> .
```

在配置文件中搜索**“fuseki:dataset ”**，将其默认的值**“:tdb_dataset_readwrite”** 更改为**“:text_dataset”**。

### 2.3 添加文本索引描述

配置文件中 **“:text_dataset”** 内容的下方添加如下代码：

```
# 文本索引描述
<#indexLucene> a text:TextIndexLucene ;
    text:directory  <file:databaseNameLucene> ;  # Lucene 索引存放的文件路径，前面使用 “file:” 表明生成的索引文件是存放在与配置文件同目录下的 databaseNameLucene 文件夹下
    # text:directory  "mem" ; # 如果为 "mem" 则是保存在内存中
    text:entityMap <#entMap> ;
    text:storeValues true ; 
    text:analyzer [ a text:StandardAnalyzer ] ;
    #text:queryAnalyzer [ a text:KeywordAnalyzer ] ; # 指定查询字符串的分析器。使用官方文档案例中的 text:KeywordAnalyzer ，查询的字符串会区分大小写。如果未设置，则使用索引给定字段时的使用的分析器
    text:queryParser text:ComplexPhraseQueryParser ;
    text:multilingualSupport true . # optional
```

**注意：**如果按照 Jena 的官方文档进行配置的话，加入**“text:queryAnalyzer [ a text:KeywordAnalyzer ] ;”** 指定了字符串分析器为 KeywordAnalyzer ，那么查询的字符串会区分大小写。如果未设置，则使用索引给定字段时的使用的分析器。

### 2.4 设置需要创建索引的谓语

配置文件中 **“<#indexLucene> a text:TextIndexLucene ;”** 内容的下方添加如下代码

```
# 设置需要创建索引的谓语
<#entMap> a text:EntityMap ;
    text:defaultField     "label" ; # text:query 时默认的字段名，需要从 text:map 中所有的 text:field 值里面选择一个
    text:entityField      "uri" ;
    text:uidField         "uid" ;
    text:langField        "lang" ;
    text:graphField       "graph" ;
    text:map (
        [ text:field "label" ; # lucene 索引的字段名
          text:predicate rdfs:label; # rdf 中的谓语名，索引是为谓语建立的
        ] 
        [ text:field "comment" ;  text:predicate rdfs:comment ;]
    ) .
```

上述配置内容中只为 rdfs:label、rdfs:comment 两个谓语创建了索引，如果需要为多个谓语创建索引，则在 text:map 中添加多行如下格式的内容即可。

```
[ text:field "comment" ;  text:predicate rdfs:comment ;]
```

### 2.5 修改前后配置文件对比

默认的配置文件内容如下：

```
@prefix :      <http://base/#> .
@prefix tdb:   <http://jena.hpl.hp.com/2008/tdb#> .
@prefix rdf:   <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix ja:    <http://jena.hpl.hp.com/2005/11/Assembler#> .
@prefix rdfs:  <http://www.w3.org/2000/01/rdf-schema#> .
@prefix fuseki: <http://jena.apache.org/fuseki#> .

:service_tdb_all  a                   fuseki:Service ;
        rdfs:label                    "TDB databaseName" ;
        fuseki:dataset                :tdb_dataset_readwrite ; # 需要修改
        fuseki:name                   "databaseName" ;
        fuseki:serviceQuery           "query" , "sparql" ;
        fuseki:serviceReadGraphStore  "get" ;
        fuseki:serviceReadWriteGraphStore
                "data" ;
        fuseki:serviceUpdate          "update" ;
        fuseki:serviceUpload          "upload" .

:tdb_dataset_readwrite
        a             tdb:DatasetTDB ;
        tdb:location  "jena-fuseki安装路径/run/databases/databaseName" .
```

全文搜索简单用法的完整配置文件内容如下：

```
@prefix :      <http://base/#> .
@prefix tdb:   <http://jena.hpl.hp.com/2008/tdb#> .
@prefix rdf:   <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix ja:    <http://jena.hpl.hp.com/2005/11/Assembler#> .
@prefix rdfs:  <http://www.w3.org/2000/01/rdf-schema#> .
@prefix fuseki: <http://jena.apache.org/fuseki#> .
@prefix text:    <http://jena.apache.org/text#> .

:service_tdb_all  a                   fuseki:Service ;
        rdfs:label                    "TDB databaseName" ;
        fuseki:dataset                :text_dataset ;
        fuseki:name                   "databaseName" ;
        fuseki:serviceQuery           "query" , "sparql" ;
        fuseki:serviceReadGraphStore  "get" ;
        fuseki:serviceReadWriteGraphStore
                "data" ;
        fuseki:serviceUpdate          "update" ;
        fuseki:serviceUpload          "upload" .

# TextDataset 是一个含有文本索引的常规数据集
:text_dataset rdf:type     text:TextDataset ;
    text:dataset   :tdb_dataset_readwrite ;
    text:index     <#indexLucene> .

:tdb_dataset_readwrite
        a             tdb:DatasetTDB ;
        tdb:location  "jena-fuseki安装路径/run/databases/databaseName" . # 创建的数据库存储路径

# 文本索引描述
<#indexLucene> a text:TextIndexLucene ;
    text:directory  <file:databaseNameLucene> ;  # Lucene 索引存放的文件路径，前面使用 “file:” 表明生成的索引文件是存放在与配置文件同目录下的 databaseNameLucene 文件夹下
    # text:directory  "mem" ;
    text:entityMap <#entMap> ;
    text:storeValues true ; 
    text:analyzer [ a text:StandardAnalyzer ] ;
    #text:queryAnalyzer [ a text:KeywordAnalyzer ] ; # 指定查询字符串的分析器。使用官方文档中的 text:KeywordAnalyzer ，查询的字符串会区分大小写。如果未设置，则使用索引给定字段时的使用的分析器
    text:queryParser text:ComplexPhraseQueryParser ;
    text:multilingualSupport true . # optional


# 设置需要创建索引的谓语
<#entMap> a text:EntityMap ;
    text:defaultField     "label" ; # text:query 时默认的字段名，需要从 text:map 中所有的 text:field 值里面选择一个
    text:entityField      "uri" ;
    text:uidField         "uid" ;
    text:langField        "lang" ;
    text:graphField       "graph" ;
    text:map (
        [ text:field "label" ; # lucene 索引的字段名
          text:predicate rdfs:label; # rdf 中的谓语名，索引是为谓语建立的
        ] 
        [ text:field "comment" ;  text:predicate rdfs:comment ;]
    ) .

```

**注意：**以上的配置只是使得 Jena 支持了全文搜索，是一些简单的用法，如果想要了解更高级的一些用法，可查看 [Jena 全文搜索文档](http://jena.apache.org/documentation/query/text-query.html) 。

## 3. 创建索引

在完成了上面的配置修改后，需要创建索引才能开始使用全文搜索功能。先进入 jena-fuseki 的安装目录，然后在终端或者命令行中，输入如下命令：

```
java -cp ./fuseki-server.jar jena.textindexer --desc=对应的数据库配置文件路径（即上面修改的 ttl 文件）
```

## 4. 索引查询

在这里将使用 Jena 文档中的一个查询案例来演示索引查询的用法，具体查询语句如下所示：

```
PREFIX   ex: <http://www.example.org/resources#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX text: <http://jena.apache.org/text#>

SELECT ?s ?lbl
WHERE { 
    ?s a ex:Product ;
    text:query (rdfs:label 'printer') ; # 共用主语 ?s
    rdfs:label ?lbl # 共用主语 ?s
}
```

上面的查询语句的功能是，在 rdfs:label 的属性（宾语为字面值时，宾语也称为属性）上查询含有 **“printer”** 的数据，然后为每个匹配的记录检索出完整的 label。

**注意：**如果使用如下的形式进行全文搜索，即没有指定要进行索引查询的谓语，那么默认使用的是  “label” 对应的谓语（原因是在配置文件中 text:defaultField 已经指定了）

```
SELECT ?s ?lbl
WHERE { 
    ?s a ex:Product ;
    text:query 'printer' ; # 默认在 rdfs:label 上进行索引查询
    rdfs:label ?lbl # 共用主语 ?s
}
```

如果想查看搜索的匹配得分，可以使用如下格式的查询。

```
SELECT ?s ?score
WHERE { 
    (?s ?score) text:query  'printer' ; # 默认在 rdfs:label 上进行索引查询
}
```



## 参考文章

[1] http://jena.apache.org/documentation/query/text-query.html

[2] https://blog.csdn.net/carolzhang8406/article/details/81660244 



