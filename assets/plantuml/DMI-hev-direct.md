```plantuml
@startuml
node 车轮
node 发动机
node 电池
node 驱动电机
node 发电机

note left of 发动机:直驱模式

发动机 -> 车轮
发动机 ..> 发电机
发电机 ..> 电池
@enduml
```
