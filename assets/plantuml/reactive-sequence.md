```plantuml
@startuml
autonumber
Publisher -> Subscriber: subscribe(Subscriber)

Subscriber ->  Subscription: onSubcribe(Subscription)

Subscription -> Publisher: request(Long)

Publisher -> Subscriber: onNext(T)
Publisher -> Subscriber: onError(Throwable)
Publisher -> Subscriber: onComplete()
@enduml
```