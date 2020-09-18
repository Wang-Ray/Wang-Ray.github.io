```plantuml
@startuml
frame frontendArea {
    node ios和android
    node web
}
frame intranet {
    frame webApplicationArea {
        node webapplication1
        node webapplication2
    }
    frame gatewayArea {
        node gateway1
        node gateway2
    }
    frame applicationArea {
        node application1
        node application2
    }
    frame databaseArea {
        database db1
        database db2
    }
}

frontendArea --> webApplicationArea
frontendArea --> gatewayArea
webApplicationArea --> applicationArea
gatewayArea --> applicationArea
applicationArea --> databaseArea
@enduml
```
