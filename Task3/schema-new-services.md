````puml
@startuml

!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml
!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Component.puml

' Определение участников системы
Person(Owner, "Собственник", "Управляет доступом к дому и парковке через мобильное приложение.")
System_Boundary(PropDevelopment, "PropDevelopment") {
    System(MobileApp, "Мобильное приложение", "Позволяет собственнику управлять устройствами.")
    System(API, "API PropDevelopment", "Предоставляет интерфейс для интеграции с внешними сервисами.")
}

System_Ext(PartnerAPI, "API партнёра", "Обрабатывает запросы на управление устройствами умного дома.")
System_Ext(SmartIntercom, "Интеллектуальный домофон", "Распознаёт пользователей и управляет доступом в дом.")
System_Ext(SmartBarrier, "Интеллектуальный шлагбаум", "Распознаёт номера автомобилей и управляет въездом.")

' Взаимодействие участников
Rel(Owner, MobileApp, "Использует для управления устройствами")
Rel(MobileApp, API, "Отправляет запросы на управление устройствами")
Rel(API, PartnerAPI, "Интеграция с сервисами умного дома")
Rel(PartnerAPI, SmartIntercom, "Управляет домофоном")
Rel(PartnerAPI, SmartBarrier, "Управляет шлагбаумом")

@enduml

```