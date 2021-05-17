```plantuml
@startuml
node 车轮
node 发动机
node 电池
node 驱动电机
node 发电机

note left of 发动机:HEV串联模式

发动机 -> 发电机
发电机 -> 驱动电机
发电机 ..> 电池
电池 -> 驱动电机
驱动电机 -> 车轮
@enduml
```
