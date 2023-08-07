## Q&As
Q：MQTT包是怎么接收订阅主题的消息？存在哪里？什么形式？
A：订阅，以及设置messageHandler，处理每次接收到的消息，存到缓存里？`message`结构体。形式如下：
```go
type message struct {
	duplicate bool
	qos       byte
	retained  bool
	topic     string
	messageID uint16
	payload   []byte
	once      sync.Once
	ack       func()
}
```