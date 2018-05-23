---
title: 【GitHub】Markdown从入门到高级入门
tags: GitHub
abbrlink: c6d4da27
date: 2017-10-10 09:11:00
---
【引言】Markdown是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。此篇文章用于介绍其基础语法用法。
<!-- more -->

# 基础语法篇

## 标题
&emsp;&emsp;一般在行首加井号表示不同级别的标题 (H1-H6)，例如：## H1, ### H2
```
# 这是H1
## 这是H2
### 这是H3
#### 这是H4
##### 这是H5
###### 这是H6
```

## 斜体和粗体
&emsp;&emsp;一般使用左右各1个\*号进行包围的部分，是斜体：
* *我就是斜体*

&emsp;&emsp;一般使用左右各2个\*号进行包围的部分，是粗体：
* **我就是粗体**

## 外部链接引用
&emsp;&emsp;一般使用如下语法：\[描述内容\](链接地址) 添加外部链接
* 去[我的博客](https://ttfisher.github.io/)逛逛吧。

## 无序列表
&emsp;&emsp;一般添加 \*，\+，\- 于列表描述前表示无序列表
* 无序列表1
 + 无序列表2
  - 无序列表3

## 有序列表
&emsp;&emsp;一般添加 数字加点 于列表描述前表示有序列表
1. 有序列表1
2. 有序列表2
3. 有序列表3

## 文字引用
* 一般使用 > 表示文字引用，本条说明本身使用的就是文字引用语法样式。
* 引用的区块内也可以使用其他的 Markdown 语法，包括标题、列表、代码区块等：
* 区块引用可以嵌套（例如：引用内的引用），只要根据层次加上不同数量的 \> ；比如
> 我是第二层引用

## 行内代码块
&emsp;&emsp;一般使用 \`代码内容\` 表示行内代码块。
* 怎么给方法设置默认返回值呢？`public int length() default 255;`

## 代码块
&emsp;&emsp;一般使用 四个缩进空格 表示大段的代码块
    
    public static String extractVideos(String sourceUrl, String filePath, CrawlRegular regular) {
        
        try{
            // 读取文件
            Document doc = Jsoup.parse(StreamIOUtil.read(filePath));

            // 取全文Elements
            Elements els = doc.children();
            if (null == els) {
                return null;
            }
                
            // 根据xPath取得匹配区域，只要存在，则证明该页面是视频页面
            Elements xPathEles = els.select(regular.getxPath());
            if (null != xPathEles && xPathEles.size() > 0) {
                return sourceUrl;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

## 代码块高亮和行数显示
&emsp;&emsp;一般使用 \`\`\`代码开发语言 开头，以 \`\`\` 结尾包围的代码块即可进行高亮和行数显示

```java
    public static String extractVideos(String sourceUrl, String filePath, CrawlRegular regular) {
        
        try{
            // 读取文件
            Document doc = Jsoup.parse(StreamIOUtil.read(filePath));

            // 取全文Elements
            Elements els = doc.children();
            if (null == els) {
                return null;
            }
                
            // 根据xPath取得匹配区域，只要存在，则证明该页面是视频页面
            Elements xPathEles = els.select(regular.getxPath());
            if (null != xPathEles && xPathEles.size() > 0) {
                return sourceUrl;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
```

## 插入图像
&emsp;&emsp;一般使用 使用 \!\[描述\](图片链接地址) 插入图像，当然也可以用HTML语言的div来实现（考虑到能想得起玩儿Markdown的同学应该都有些编程基础，所以这里就不赘述了），Please enjoy this moment.
<div align=center><img src="/img/2018-05-17-01.jpg" width = "75%"/></div>

## 分隔线
&emsp;&emsp;一般在一行中用三个以上的 \*、\-、\_ 来建立一个分隔线，行内不能有除空格以外的其他内容
+ 我是多\*分隔线
*********************************
+ 我是多\-分隔线
------------------- ------ ------
+ 我是多\_分隔线
_________________________________

## 反斜杠
&emsp;&emsp;一般使用 \\特殊字符 进行字符转义（类似编程语言）
+ 我就是要展示星号，就在星号前面加入反斜杠（\*）

# 进阶语法篇

## 表格基本语法
```
| 水果        | 价格    |  数量  |
| --------   | -----:   | :----: |
| 香蕉        | $1      |   5    |
| 苹果        | $1      |   6    |
| 草莓        | $1      |   7    |
```

| 水果        | 价格    |  数量  |
| --------   | -----:   | :----: |
| 香蕉        | $1      |   5    |
| 苹果        | $1      |   6    |
| 草莓        | $1      |   7    |

## 表格转换工具
&emsp;&emsp;使用大杀器转换工具: [exceltk](https://link.zhihu.com/?target=http://fanfeilong.github.io/exceltk0.0.4.7z)

```
整个表格：    exceltk.exe -t md -xls xxx.xls  
              exceltk.exe -t md -xls xxx.xlsx
指定sheet：  
              exceltk.exe -t md -xls xx.xls -sheet sheetname   
              exceltk.exe -t md -xls xx.xlsx -sheet sheetnameexceltk
特性：
     转换Excel表格到MarkDown表格
     支持Excel单元格带超链接
     如果Excel里有合并的跨行单元格，在转换后的MarkDown里是分开的单元格，这是因为MarkDown本身不支持跨行单元格
     如果Excel表格右侧有大量的空列，则会被自动裁剪，算法是根据前100行来检测并计算
```

## 流程图
&emsp;&emsp;使用 三个\`加flow 起始，使用 三个\` 结尾，语法参考：

```
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request
st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
```

```flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request
st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
```

## 时序图（Hexo暂不支持）
&emsp;&emsp;使用 三个\`加sequence 起始，使用 三个\` 结尾，语法参考：

```sequence
Alice->Bob: Hello
Bob->Alice: Hello too
```

## Todo List（Hexo暂不支持）
&emsp;&emsp;使用带有 [ ] 或 [x] （未完成或已完成）项的列表语法撰写一个待办事宜列表例如：
[ ] 未完成任务
[x] 已完成任务

## 标签和分类
&emsp;&emsp;一般在文首输入tags添加标签，categories添加分类：
- tags: 基石工具
- categories: Java技术
