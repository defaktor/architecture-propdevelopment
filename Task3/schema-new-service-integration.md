````puml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml
!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Component.puml

System_Boundary(PropDevelopment, "PropDevelopment") {

    Person(tenant, "Собственник", "")

    Boundary(firewall2, "Firewall") {
        Boundary(client-t, "") {
            Container(tenantApp, "Витрина сервисов для собственников", "Mobile app(iOS, Android)", "Мобильные приложения для собственников")
            Container(tenantCore, "tenant-core-app", "Kotlin, SpringBoot", "Услуги для собственников")
            ContainerDb(tenantCoreDB, "tenant-core-db", "PostgreSQL", "Данные услуг ЖКХ")
        }
    }

    Container(authService, "auth-service-1", "Keycloak", "Сервис аутентификации покупателей")
    ContainerDb(authDB, "auth-db-1", "PostgreSQL", "Данные аутентификации пользователей")

    ' Внешние системы
    System_Ext(externalAPI, "Поставщик ресурсов ЖКХ", "Software System")
    System_Ext(paymentSystem, "Платёжная система", "Software System")
    System_Ext(SmartHomeAPI, "SmartHome API", "Партнёрский API для Умного дома")

    ' Новые сервисы Умный дом
    Container(smartDoor, "Интеллектуальный домофон", "API", "Распознавание лиц и управление доступом")
    Container(smartGate, "Интеллектуальный шлагбаум", "API", "Распознавание номеров автомобилей и управление доступом")

    ' Взаимодействие
    Rel(tenant, tenantApp, "", "")
    Rel(tenantApp, tenantCore, "REST, WS", "")
    Rel(tenantCore, tenantCoreDB, "JDBC", "")
    Rel(tenantCore, externalAPI, "REST", "")
    Rel(tenantCore, paymentSystem, "REST", "")
    Rel(tenantApp, smartDoor, "REST", "управление доступом по биометрии")
    Rel(tenantApp, smartGate, "REST", "управление доступом по номерам")
    Rel(smartDoor, SmartHomeAPI, "REST", "биометрические данные")
    Rel(smartGate, SmartHomeAPI, "REST", "данные автомобилей")
    Rel(tenantApp, authService, "REST", "Аутентификация")
    Rel(authService, authDB, "JDBC", "")

}

@enduml
```