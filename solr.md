# solr

## 一、正向索引：从文档到单词。

假如有三个 txt 文档，

* Document_1: The cow says moo.
* Document_2: The cat and the hat.
* Document_3: The dish ran away with the spoon.

解析每个文档出现的单词，然后建立从文档 (document) 到词组 (words) 的映射关系，这就是正向索引。

在mysql中，为某个字段建立索引，索引只有在等值查询、范围查询、少部分 like xxx% 查询中才有效

| Document   | Words                            |
| ---------- | -------------------------------- |
| Document_1 | the,cow,says,moo                 |
| Document_2 | the,cat,and,the,hat              |
| Document_3 | the,dish,ran,away,with,the,spoon |

## 二、反向索引：从词到文档。

反向索引方向则是正向索引的逆向，建立从单词 (word) 到文档 (document lsit) 的映射关系。

| Word | Documents                                                  |
| ---- | ---------------------------------------------------------- |
| the  | Document_1, Document_3, Document_4, Document_5, Document_7 |
| cow  | Document_2, Document_3, Document_4                         |
| says | Document_5                                                 |
| moo  | Document_6                                                 |

| 分类         | 图书                           |
| ------------ | ------------------------------ |
| 哲学与心理学 | 《苏菲的世界》，《理想国》     |
| 宗教         | 《圣经》，《古兰经》           |
| 社会科学     | 《宏观经济学》，《微观经济学》 |
| ...          | ...                            |

通常书籍最后的章节会罗列所有涉及到的行业术语，你可以通过这个列表查看哪一页交代了这些术语，并快速跳转到定义的地方，这也是一种索引 `word -> page`。

