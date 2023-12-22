# Start
## New a InfluxDB
1. WebUI的左上角用户有`Create Orgnization`选项，选择并新建`bucket`。
2. 在`Load Data -> API Tokens`处生成token，存储token。
## Orgnization|Bucket
- Orgnization：InfluxDB的组织，包含多个Bucket。
- Bucket：InfluxDB的数据存储空间。

# Manual
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
- Field set，`Measurement`+`Field key-Field value`+`timestamp`。

### Tags
由两部分组成，`tag key`与`tag value`。并且`tag`是有索引(index)的，因此基于tag查询可以更加迅速，需要我们仔细考虑将什么字段设为`tag`。
- tag key，列名，`string`类型。
- tag value，对应列的值，`meta`类型，也就是说可以支持各种可变信息，比如`UUID`，`hash`等。这样的高可变性将使数据库中出现大量唯一序列，被称为`high series cardinality`，这种特性有利于数据库工作负载的内存使用率提升。
- tag set，`tag key1 = tag value1, tag key2 = tag value2, ...`

### Bucket schema
显式`schema-type`即描述`bucket`属性的`measurement`。多个`measurement`构成了`bucket schema`。
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
一组用户可以使用一个`organization`作为共同工作空间。一个`organization`中可以有多个`bucket`，并且能被`flux`语句一起操作管理。

## Line Protocol
`Line protocol`是将数据写入`InfluxDB`的基于`text`的底层协议，保证写入`data point`需要的基本信息。

### Line protocol elements
#### Required
- `measurement`，对写入数据的描述，能匹配`influxDB`中已经存在的`measurement`。
- `field set`，写入数据的内容。

#### Optional
- `tag set`，描述`field`用，可以辅助查找，提升查找速度。
- `timestamp`，`influxDB`本身支持`ns`级的`timestamp`，如果不想使用`ns`，那么自行指定。

#### Line protocol element parsing
![parsing example](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics/202307181154455.png)
- Note: `line protocal`对空格敏感，所以当`string`数据包含空格时，需要用`\`转义。

# InfluxDB with Golang
## influxdb-client-go/v2/readme
[official doc](https://pkg.go.dev/github.com/influxdata/influxdb-client-go/v2@v2.12.3#section-readme)

### Write point
#### 使用参数方法
```go
p := influxdb2.NewPoint(
"system",
map[string]string{
    "id":       fmt.Sprintf("rack_%v", i%10),
    "vendor":   "AWS",
    "hostname": fmt.Sprintf("host_%v", i%100),
},
map[string]interface{}{
    "temperature": rand.Float64() * 80.0,
    "disk_free":   rand.Float64() * 1000.0,
    "disk_total":  (i/10 + 1) * 1000000,
    "mem_total":   (i/100 + 1) * 10000000,
    "mem_free":    rand.Uint64(),
},
time.Now())
```

#### 直接使用`line protocol`
```go
client := influxdb2.NewClient(url, token)

// WriteAPIBlocking返回同步的、阻塞的写客户端。
writeAPI := client.WriteAPIBlocking("<organization>", "<bucket>")
line := fmt.Sprintf("<line protocol>")
err = writeAPI.WriteRecord(context.Background(), line)
```

#### Non-blocking writing and blocking writing
客户端提供两种写入数据的方式。

- Non-blocking，有隐式缓冲池。数据会被**异步**地写入到缓冲池里，当**缓冲超过缓冲池大小**时或到达**冲刷周期**（flush interval）时，缓冲池内的数据才被自动发送到服务器。  
   `writeAPI.WritePoint(p) // write asynchronously`。
- Blocking，使用`synchronous blocking`方法保证每次都会**同步**写完并发送到数据库。  
   `err := writeAPI.WritePoint(context.Background(), p) // write synchronously`

### Queries

#### QueryTableResult
将`query`返回的`CSV`流式响应解析成`FluxTableMetaData, FluxColumn and FluxRecord`对象以更好的查看结果。

#### Get query
```go
queryAPI := client.QueryAPI("<organization>")

// Query执行flux查询语句，并return将服务器返回的流式响应解析为结构体
// 的QueryTableResult，里面包含flux table parts。
result, err := queryAPI.Query(context.Background(), `<query command>`)

// 如果不出错，则使用Next()迭代query结果的每一行
for result.Next() {
    // 是否有新表
    if result.TableChanged() {
        fmt.Printf("table: %s\n", result.TableMetadata().String())
    }
    // 读取结果
    fmt.Printf("row: %s\n", result.Record().String())
}
```

#### QueryRaw()
返回未解析的原始`CSV formatted string`数据。

### Concurrency
支持并发，使用`channel`实现并发。

### Checking Server State
- Health()，详细。
- Ready()，服务器在线时间相关信息。
- Ping()，服务器是否在线

### type Client interface
`CLient`提供与`InfluxDB Server`交流的接口。

- `Setup()`初始化`InfluxDB`服务器，返回最新创建的实体以及其授权对象的信息。
- `SetupWithToken()`同上，还包括一个`token`参数。
- `Ready()`
- `Health()`
- `Ping()`
- `Close()`确保所有异步写客户端都结束；在`HTTP client`是程序内部创建的情况下，关闭所有闲散连接。
- `Options()`返回与客户端有关的选项。
- `ServerURL()`返回当前客户端联系的服务器的`URL`。
- `HTTPService()`返回被客户端使用的`HTTP`服务器对象。
- `WriteAPI()`返回异步的、非阻塞(`non-blocking`)的写客户端。
   - 注意每个`WriteAPI`实例对应一组`org/bucket`对。
- `WriteAPIBlocking()`返回同步的、阻塞的写客户端。注意事项同上。
- `QueryAPI()`返回查询客户端。
   - 注意每个`QueryAPI`实例对应一个`org`。
- `AuthorizationsAPI()`返回验证客户端。
- `OrganizationsAPI()`返回组织客户端。
- `UserAPI()`。
- `DeleteAPI()`
- `BucketsAPI()`
- `LabelsAPI()`
- `TasksAPI()`
- `APIClient()`

### type Options struct
`Options`配置与`InfluxDB Server`通讯时的属性。

- `DefaultOptions()`返回默认值的配置对象（`Options object`）。
- `WriteOptions()`返回写相关的`options`。  
- `AddDefaultTag()`设置一个默认`tag`，会应用到每个写入的`data point`上。
- `SetApplicationName()`将应用名称设置到`HTTP header`里。
- `ApplicationName()`返回在当前`HTTP`通信中使用的应用名称（应用名称存储在`HTTP header`里）。
- `SetBatchSize()`设置一次`request`能发送的`data points`的数量。
- `BatchSize()`返回批次的大小。
- `SetExponentialBase()`设置指数重试的基数。
- `ExponentialBase()`返回指数重试的基数。
   - 默认为`2`。
- `SetFlushInterval()`设置冲刷间隔。
- `FlusInterval()`返回以`ms`为单位的冲刷间隔。
- `SetHTTPCLient()`创建一个`HTTP`客户端。
   - 使用该方法会使其他`HTTP`选项被忽略，可以使用`HTTPClient.Jar`存储`session cookie`。
- `HTTPClient()`返回已经配置好的用于`HTTP requests`的`HTTP`客户端。
   - 首先返回`SetHTTPClient()`配置的客户端，如果没有，则使用其他配置好的`options`自动构建一个默认的客户端。
- `HTTPOptions()`返回`HTTP`相关选项。
- `SetHTTPRequestTimeout()`设置`HTTP`请求超时时间（s）。
- `HTTPRequestTimeout()`返回请求超时的时间。
- `SetLogLevel()`设置日志等级，过滤非设置等级的日志信息。
   - 默认`ErrorLevel`
   - 等级：
      - `ErrorLevel`
      - `WarningLevel`
      - `InfoLevel`
      - `DebugLevel`
- `LogLevel()`返回日志等级。
- `SetMaxRetries()`设置重试的最大次数。
- `MaxRetries()`返回最大重试次数，默认5次。
- `SetMaxRetryInterval()`设置每两次重试间的最大延时。
- `MaxRetryInterval()`按`ms`返回每次重试的最大延迟，默认125000。
- `SetMaxRetryTime()`设置最大总重试时间。
- `MaxRetryTime()`返回重试总共花的时间。
- `SetPrecision()`设置写入时间戳的精度。
   - 包括：
      - `time.Nanosecond`
      - `time.Microsecond`
      - `time.Millisecond`
      - `time.Second`
- `Precision()`返回写的时间精度。
- `SetRetryBufferLimit()`设置重试时能保存的最大`data points`数目。
   - 要设置为`BatchSize()`的倍数。
- `RetryBufferLimit()`如函数名。
- `SetRetryInterval()`设置重试间隔。
- `RetryInterval()`以`ms`精度返回重试间隔时间。
- `SetTLSConfig()`配置`TLS`。
- `TLSConfig()`返回`TLS`配置信息。
- `SetUseGZip()`设置在写入请求是否使用`Gzip compression`。
- `UseGZip()`返回是否使用了`GZIP`。

## /v2/api/query
使用`Go`进行`FLux`查询。
### type FluxTableMetadata struct
```go
// FluxTableMetadata存储查询出的表，以列的集合保存，每个新表由注释分开。 
type FluxTableMetadata struct {
	position int
	columns []*FluxColumn
}
```
- `NewFluxTableMetadata()`指定`position`值创建`FluxTableMetadata`。
- `NewFluxTableMetadataFull()`创建`FluxTableMetadata`。
- `AddColumn()`为`FluxTableMetadata`添加新的`FluxColumn`。
- `Column()`根据`index`返回`FluxColumn`。
- `Columns()`返回`columns`。
- `Position()`返回`position`。
- `String()`以`string`类型返回`FluxTableMetadata`实例的所有值。

### type FluxClolumn struct
```go
// FluxColumn保存每列的属性
type FluxColumn struct {
	index int
	name string
	dataType string
	group bool
	defaultValue string
}
```
- `NewFluxColumn()`创建指定`index`值的`FluxColumn`。
- `NewFluxColumnFull()`创建`FluxColumn`。
#### 查询`FluxColumn`的属性值
- `DataType()`
- `DefaultValue()`
- `Index()`
- `IsGroup()`
- `Name()`
- `String()`以`stirng`形式返回`FLuxColumn`的所有属性值。

#### 设置`FluxColum`的属性值
- `SetDataType()`
- `SetDefaultValue()`
- `SetGroup()`
- `SetName()`

### type FluxRecord struct
```go
// FluxRecord表示查询结果表中的行
type FluxRecord struct {
	table int 
	values map[string]interface{} 
}
```
- `NewFluxRecord()`创建新`FLuxRecord`。

#### 查询`FluxRecord`的对应值
- `Field()`返回`values`中对应`_field`的名称，如果不存在则返回空字符串。
- `Measurement()`
- `Result()`
- `Start()`
- `Stop()`
- `String()`以`string`类型返回所有值。
- `Table()`
- `Time()`
- `Value()`
- `ValueByKey()`按键名返回`values`中对应的值，例如`ValueByKey("_field")。
- `Values()`

## /v2/api/write
使用`Go`进行`FLux`写
- `PointToLineProtocol()`根据传入的`data point`生成`Line Protocol`语句，并将时间戳修改为指定的时间精度。
- `PointToLineProtocolBuffer()`除以上功能外，还会将生成的语句写入`sb *strings.Builder`。

### type Consistency stirng
`Cosistency`规定对写入数据数量的确认。

### type Options struct
见`.../readme`

### type Point struct
`Point`表示存入`InfluxDB`的一条数据。

- `NewPoint()`创建`point`。
- `NewPointWithMeasurement()`用`AddTag`、`AddField`创建`point`。
   - `AddField()`
   - `AddTag()`

#### 返回`Point`的值
- `FieldList()`以切片类型返回`fields`。
- `Name()`返回`measurement`。
- `Taglist()`以切片类型返回`tags`。
- `Time()`返回时间戳。

#### 设置`Point`的值
- `SetTime()`设置时间戳。
- `SortFields()`给`Fields`按`key`的字母顺序排序。
- `SortTags()`给`Tags`排序。

# Using Flux
## Flux query
每个`query`需要三部分：
- A data source
- A time range
- Data filters

### Define data source
确定数据源。
`from(bucket:"<bucket-name/retention-policy>")

### Specify a time range
无界查询非常消耗资源，所以需要设置时间范围，没有时间范围的话`flux`不会查询。  
使用`range()`设置时间范围，参数包括`start`与`stop`：
- 值为负数，以当前时间为基准。
- 或者设置绝对时间。
使用（`|>`）从数据源传递数据到下一个命令中：
```flux
// Relative time range with start only. Stop defaults to now. 
from(bucket:"telegraf/autogen")
	|> range(start: -1h) 
// Relative time range with start and stop from(bucket:"telegraf/autogen")
	|> range(start: -1h, stop: -10m) 
```

### Filter data
将时间范围内的数据传入`filter()`函数，该函数基于数据属性或列进行筛选。  
`filter()`包括参数`fn`，代表匿名函数，匿名函数执行基于数据属性或列筛选的逻辑。  
`filter()`中的`r`表示传入的数据`record`，对`r`进行筛选操作，多个筛选条件用`and`连接：
```flux
from(bucket: "telegraf/autogen")
	|> range(start: -15m)
	|> filter(fn: (r) => r._measurement == "cpu" and r._field == "usage_system" and r.cpu == "cpu-total") 
```

### Output
使用`yield()`函数输出筛选过后的表结果。

## Transform data with Flux
### Aggregate windowed data
-`window()`可以设定时间条件，将数据按条件集合成一个个`table`。
- 然后通过`|>`进入的下一步操作将对每一个表都执行。
   - 例如：
      ```flux
      |> window(every: 5m) //每5min的数据放入一个table
      |> mean()
      ```

### Add times to the aggregates
因为在合并的时候，每个`table`中的多条数据`_time`不一样，集合成表后不能确定使用哪一条作为集合的`_time`，因此该属性被丢弃。但在下一步操作数据，`_time`是有必要的。
- `duplicate()`可以复制某一属性，生成自定义的另一属性。例：
   ```flux
   |> duplicate(column:  "_stop", as:   "_time")
   ```

### Unwindow aggregate tables
- `window()`将`every`的值设为`inf`，就可以拆分表中每一条数据并放到同一张表下。

## Flux basics
### Flux data model
`Flux`查询返回的结果是流式数据，由多个`table`组成。 
1. Group key。这些`tables`是基于`group key`分离的，在`group key`上拥有相同值的数据被放到一个`table`。
2. 对数据的操作，是在`table`级别上进行的，也就是说每次操作都是对`table`进行的操作。
3. 因此，如何使用`group key`对数据进行`table`切分，是操作`influxdb`数据的关键。
例：
```flux
// group key
[_measurement, facility, _field]

// 基于group key分成的3个table
[_measurement: "production", facility: "us-midwest", _field: "apq"] 
[_measurement: "production", facility: "eu-central", _field: "apq"] 
[_measurement: "production", facility: "ap-east", _field: "apq"] 
```
- 可以用以下函数更改`group key`:
- `group()`
- `window()`

### Functions
自定义函数实现复杂功能。
```flux
square = (n) => n * n //函数逻辑
square(n:3) //传入参数并使用函数
```
稍复杂一点的：
```flux
topN = (tables=<-, n) => // 这个函数有两个参数，tables接收从管道（|>）传下来的数据
tables
|> sort(desc: true)
|> limit(n: n) 
```

## 错误
### Unable to write gathered points && database not found
Cannot fix this.  
Instead delete the data-store of `~/.influxdbv2` which enable me to restart.
### Unable to insert points
- Q: influxdb设置retention之后，无法插入过时的数据?
   A: Points in a bucket with timestamps beyond the defined retention period (relative to now) are flagged for deletion (also known as “tombstoned”).