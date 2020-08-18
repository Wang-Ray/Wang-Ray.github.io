```plantuml
@startuml
app -> gateway
gateway -> service
service -> db
db --> service
service --> gateway
gateway --> app
@enduml
```