# Elasticsearch基础操作及ElasticsearchTemplate API

![image-20220907134617731](image/image-20220907134617731.png)

```java
String[] include = {"name", "price"};

//两个参数分别是要显示的和不显示的
FetchSourceFilter fetchSourceFilter = new FetchSourceFilter(include, null);   

SearchQuery searchQuery = new NativeSearchQueryBuilder().withSourceFilter(fetchSourceFilter).build();

return elasticsearchTemplate.queryForList(searchQuery, Product.class);
```

![image-20220907134755896](image/image-20220907134755896.png)

上面的注释的地方是自己写代码的时候粗心造成的问题.如果不标注filed的话会报错.报错信息:

```log
org.elasticsearch.search.aggregations.AggregationExecutionException: could not find the appropriate value context to perform aggregation [range_by_price]
```

