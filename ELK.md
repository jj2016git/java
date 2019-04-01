# ELK

## 适用场景

无需入侵应用代码，只要有应用日志，就能实现监控

## 特征

- 上手快
- 搜索快
- 扩容快

## ELK

ELK

- ElasticSearch：分布式搜索和分析
- Logstash：实时数据收集
- Kibana：js，分析和可视化web平台

Solr VS ElasticSearch

- Solr实时性差

ELK前身

Lucene

- Document：任意格式，提取text内容
- index：检索条件
- analyzer分词器：提高查询精准度

Solr和ES都是对Lucene的二次开发

ES

- 弱化document

- index：结构化存储，将结构相同的document作为一个整体，index保存整体的文本内容
- IndexType：
- document
- analyzer

## 集群搭建

ES jdk>=1.8

port

- http 9200 可视化数据端口
- tcp 9300 后台api操作访问的端口

ES实现分布式，不需要依赖第三方

