# 什么是Elasticsearch

![image-20220908111513100](image/image-20220908111513100.png)



term 是精确匹配，match 会分词、打分。

filter 不打分，可缓存，适合做过滤条件。

ES 默认使用 BM25 算法替代 TF-IDF。