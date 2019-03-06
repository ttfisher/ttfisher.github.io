---
title: 全文检索技术应用系列（二）：Lucene Demo
comments: true
categories:
  - 技术结构升级之全文检索
tags:
  - 全文检索
abbrlink: 603f4bd6
date: 2018-08-15 08:07:01
---
【引言】既然是Demo，基本上也就是代码和测试方法了，这里只是一个很简单很简单的演示效果，跟实际应用还差着十万八千里，不过通过这段简短的Demo，我们可以一窥Lucene的究竟。
<div align=center><img src="https://github.com/ttfisher/images/raw/master/public/000017.jpg" width="500"/></div>
<!-- more -->

# Maven依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.cc</groupId>
    <artifactId>lucene</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.apache.lucene/lucene-core -->
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-core</artifactId>
            <version>7.4.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.lucene/lucene-queryparser -->
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-queryparser</artifactId>
            <version>7.4.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.lucene/lucene-analyzers-common -->
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-analyzers-common</artifactId>
            <version>7.4.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.lucene/lucene-highlighter -->
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-highlighter</artifactId>
            <version>7.4.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/commons-io/commons-io -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>

    </dependencies>
</project>
```

# Demo源码
```java
package com.cc.lucene;

import org.apache.commons.io.FileUtils;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field.Store;
import org.apache.lucene.document.StoredField;
import org.apache.lucene.document.StringField;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.index.Term;
import org.apache.lucene.search.*;
import org.apache.lucene.store.FSDirectory;

import java.io.File;
import java.io.FilenameFilter;
import java.io.IOException;
import java.nio.file.FileSystems;
import java.util.ArrayList;
import java.util.List;

/**
 * Created by chenglin on 2018/8/16.
 */
public class LuceneDemo {

    /**
     * 根据srcPath下的文件创建索引库
     * @param srcPath
     * @param indexPath
     */
    public static void createIndex(String srcPath, String suffix, String indexPath) {
        System.out.println("Create index start... ");
        IndexWriter indexWriter = null;
        try {
            // 创建Document集合
            List<Document> docs = createDocuments(srcPath, suffix);
            if (docs.isEmpty()) {
                System.out.println("No document found, exit.");
                return;
            }

            // 定义索引操作对象indexWriter
            indexWriter = new IndexWriter(FSDirectory.open(FileSystems.getDefault().getPath(indexPath)),
                    new IndexWriterConfig(new StandardAnalyzer()));

            // 创建索引
            for (Document document : docs) {
                indexWriter.addDocument(document);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (null != indexWriter) {
                try {
                    // 关闭流
                    indexWriter.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        System.out.println("Create index finish... ");
    }

    /**
     * 取目录下固定后缀的文件生成Document
     * @param srcPath
     * @param suffix
     * @return
     * @throws IOException
     */
    public static List<Document> createDocuments(String srcPath, String suffix) throws IOException {

        List<Document> docList = new ArrayList<Document>();
        File folder = new File(srcPath);
        if (!folder.isDirectory()) {
            System.out.println("The srcPath must be a folder... ");
            return null;
        }

        // 以固定后缀获取文件
        File[] files = folder.listFiles(new FilenameFilter() {
            @Override
            public boolean accept(File dir, String name) {
                return name.endsWith(suffix);
            }
        });

        for (File file : files) {
            if (file.isFile()) {
                // 创建文档
                Document doc = new Document();

                // 将Field添加到文档中（TextField会进行语汇化也就是会剔除无意义的词，StringField则不会进行语汇化）
                doc.add(new StringField("fileName", file.getName(), Store.YES));
                doc.add(new TextField("fileContent", FileUtils.readFileToString(file, "UTF-8"), Store.YES));
                doc.add(new TextField("fileSize", String.valueOf(FileUtils.sizeOf(file)), Store.YES));
                doc.add(new StoredField("filePath", file.getAbsolutePath()));

                // 加入集合
                docList.add(doc);
            }
        }
        return docList;
    }

    /**
     * 根据索引目录查询
     * @param indexPath
     * @throws IOException
     */
    public static void indexQuery(String indexPath, String termName, String queryContent) throws IOException {
        // 创建查询对象
        Query query = new TermQuery(new Term(termName, queryContent));

        // 根据索引目录创建indexSearcher
        IndexSearcher indexSearcher =
                new IndexSearcher(DirectoryReader.open(FSDirectory.open(FileSystems.getDefault().getPath(indexPath))));

        // 搜索TopN
        TopDocs topDocs = indexSearcher.search(query, 100);
        System.out.println("Total hit records ：" + topDocs.totalHits);

        for (ScoreDoc scoreDoc : topDocs.scoreDocs) {
            Document doc = indexSearcher.doc(scoreDoc.doc);

            System.out.println("Matched file name = " + doc.get("fileName"));
            System.out.println("Matched file size = " + doc.get("fileSize"));
            System.out.println("Matched file content = " + doc.get("fileContent"));
        }
    }

    /**
     * Have a demo
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        // Params
        String srcPath = "D:/lucene/src";
        String suffix = "txt";
        String indexPath = "D:/lucene/index";

        // Index
        System.out.println("***********************Start building index*************************");
        createIndex(srcPath, suffix, indexPath);

        // Query
        System.out.println("***********************Start term query*************************");
        indexQuery(indexPath, "fileContent", "lucene");
        System.out.println("-----------------------------------------------");
        indexQuery(indexPath, "fileContent", "engine");
    }
}
```

# 测试文件

## 文件一：SiteSecurityServiceState.txt
> 内容：Lucene: Apache LuceneTM is a high-performance, full-featured text search engine library written entirely in Java. 

## 文件二：luceneIntro.txt
> 内容：2017年10月12日 -  Lucene是apache软件基金会4 jakarta项目组的一个子项目,是一个开放源代码的全文检索引擎工具包,但它不是一个完整的全文检索引擎,而是一个全文检索引...

# 运行结果
```
***********************Start building index*************************
Create index start... 
Create index finish... 
***********************Start term query*************************
Total hit records ：2
Matched file name = SiteSecurityServiceState.txt
Matched file size = 114
Matched file content = Lucene: Apache LuceneTM is a high-performance, full-featured text search engine library written entirely in Java. 
Matched file name = luceneIntro.txt
Matched file size = 219
Matched file content = 2017年10月12日 -  Lucene是apache软件基金会4 jakarta项目组的一个子项目,是一个开放源代码的全文检索引擎工具包,但它不是一个完整的全文检索引擎,而是一个全文检索引...
-----------------------------------------------
Total hit records ：1
Matched file name = SiteSecurityServiceState.txt
Matched file size = 114
Matched file content = Lucene: Apache LuceneTM is a high-performance, full-featured text search engine library written entirely in Java. 
```