- source
  - taildir:实时监控文件，支持断点续传和多目录
  - avro：Flume之间
  - nc：接受网络端口
  - exec：不支持断点续传
  - spooling：监控文件夹
  - kafka source

- channel
  - file
  - memory
  - kafka channel

- sink
  - avro
  - hdfs sink
  - kafka sink
  
## Flume 配置

1. 定义组件
2. 配置sources
3. 配置channels
4. 配置sinks
5. 组装