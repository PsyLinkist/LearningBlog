## Data Elements
![例](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics/202307181102122.png)
### Timestamp
硬盘里，数据都以纳秒(ns)的`timestamp`存储。InfluxDB的`timestamp`格式是基于RFC3339 UTC的。

### Measurement
本身的名字数据类型是`string`，但它包含`tags`, `fields`, `timestamps`，在本例中，Measurement名字`census`包含了`field value`记录`bees`和`ants`的意思。

### Fields
由两部分组成，一部分是`_field`，存储`field key`；另一部分是`_value`，存储`field value`。
- Field key，存`Field`名字。
- Field value，存对应`Field`的值`value`。
- Field set，`Measurement`+`Field key - Field value`+`timestamp`。

### Tags
由两部分组成，`tag key`与`tag value`。并且`tag`是有索引(index)的，因此基于tag查询可以更加迅速，需要我们仔细考虑将什么字段设为`tag`。
- tag key，列名，`string`类型。
- tag value，对应列的值，`meta`类型，也就是说可以支持各种可变信息，比如`UUID`，`hash`等。这样的高可变性将使数据库中出现大量唯一序列，被称为`high series cardinality`，这种特性有利于数据库工作负载的内存使用率提升。
- tag set，`"tag key1 = tag value1, tag key2 = tag value2, ...`

### Bucket schema
显式`schema-type`即描述`bucket`属性的`measurement`。
![census](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics/202307181101463.png)

### Series
两部分，第一部分`series key`；第二部分`series`。通过`series key`可以查询到`series`。
- series key，`measurement + tag set + field key`。
- series，`timestamp + field value`。

### Point
一个`Point`包括`timestamp + measurement + field key + field value + tag1 value + tag2 value + ...`

### Bucket
InfluxDB的数据存储在`bucket`中，`bucket`包含两个概念，一是数据库，二则是数据保留时间（`retention period`，每个数据`point`能保留的时长）。`bucket`属于`organization`。

### Organization
一组用户可以使用一个`organization`作为共同工作空间。