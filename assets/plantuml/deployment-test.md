```plantuml
@startuml
actor client
node web
node app
database db

client -> web
web -> app
app -> db
@enduml
```
