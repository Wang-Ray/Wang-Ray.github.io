```plantuml
@startuml
node 业务线 as businessLine
node 业务模块 as businessModule
node 业务渠道 as businessChannel
component product
component item
component SPU
component SKU

businessLine --> businessModule
businessModule --> businessChannel
product --> businessChannel
SPU --> item
item --> SKU
item --> product

@enduml
```
