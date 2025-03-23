````puml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml
!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Component.puml

System_Boundary(PropDevelopment, "PropDevelopment") {

    Person(client, "Клиент", "")
    Person(tenant, "Собственник", "")
    
    Person(aBI, "Аналитик BI", "")
    Person(manager, "Менеджер", "")
    Person(aS, "Бизнес аналитик", "")
    
    
    Boundary(firewall, "Firewall") {
        
        Boundary(client-b, "") {
            Container(web, "Витрина продаж", "React, JavaScript", "Подбор и бронирование недвижимости")
            Container(clientMartApp, "client-mart-app", "Kotlin, SpringBoot", "Приложение для: - предоставления информации об объектах недвижимости, - проведения онлайн-сделок")
            Container(clientTourApp, "client-tour-app", "Kotlin, SpringBoot", "Приложение для проведения 3D-тура по объекту недвижимости (онлайн-тур)")
            Container(clientMartEstate, "client-mart-estate-app", "Kotlin, SpringBoot", "Приложение для управления информацией об объектах недвижимости")
            Container(clientCRM, "client-crm-app", "Dynamics CRM", "Управление клиентскими данными")
            ' Базы данных
            ContainerDb(clientMartDB, "client-mart-db", "PostgreSQL", "Данные недвижимости и сделок")
            ContainerDb(clientMartEstateDB, "client-mart-estate-db", "PostgreSQL", "Данные объектов недвижимости")
            ContainerDb(clientTourDB, "client-tour-db", "PostgreSQL", "База данных онлайн-туров")
            ContainerDb(clientCRMDB, "crm-db", "MSSQL", "Данные клиентов")
        }   
    }

    Boundary(firewall2, "Firewall") {
        Boundary(client-t, "") {
            ' Группа сервисов ЖКУ
            Container(tenantApp, "Витрина сервисов для собственников", "Mobile app(iOS, Android)", "Мобильные приложения для собственников")
            Container(tenantCore, "tenant-core-app", "Kotlin, SpringBoot", "Услуги для собственников")
            Container(tenantCRM, "CRM", "Kotlin, SpringBoot", "Управление данными собственников")
            
            ' Базы данных
            ContainerDb(tenantCoreDB, "tenant-core-db", "PostgreSQL", "Данные услуг ЖКХ")
            ContainerDb(tenantCRMDB, "crm-tenant-db", "PostgreSQL", "Данные CRM собственников")
        }
        
    }        
    
    
    ' Финансы
    Boundary(fin, "") {
        Person(finPerson, "Бухглтер", "")
        Container(accountantService, "accountant-service-1", "Keycloak", "Система финансового учёта")
        ContainerDb(authDB2, "auth-db-2", "PostgreSQL", "Данные аутентификации пользователей")

        Container(activeDirectoryP, "Active Directory", "MS Active directory", "Служба каталогов, которая обслуживает прикладные бизнес-системы и системы обработки данных")
        

        Rel(finPerson, accountantService, "", "")

        Rel(accountantService, activeDirectoryP, "", "Аутентификация")
        Rel(accountantService, authDB2, "", "")

    }
    
    ' Дата
    Boundary(data, "") {
        Container(dataWarehouse, "Хранилище данных", "Greenplum", "Централизованное хранилище данных")
        ContainerDb(dataWarehouseS3, "DWH Хранилище", "S3", "Данные ")

        Rel(dataWarehouse, dataWarehouseS3, "", "")
    }
    
    
    Container(activeDirectory, "Active Directory", "MS Active directory", "Служба каталогов, которая обслуживает финансовые сервисы")
    Container(authService, "auth-service-1", "Keycloak", "Сервис аутентификации покупателей")
    ContainerDb(authDB, "auth-db-1", "PostgreSQL", "Данные аутентификации пользователей")
    
    ' Внешние системы
    System_Ext(externalAPI, "Поставщик ресурсов ЖКХ", "Software System")
    System_Ext(paymentSystem, "Платёжная система", "Software System")
    System_Ext(govRegister, "Гос. регистрационные органы", "Регистрация недвижимости")
    
    ' Взаимодействие
    Rel(web, clientMartApp, "REST", "каталог объектов, бронирование")
    Rel(web, clientTourApp, "", "")
    Rel(web, clientMartApp, "REST", "")
    Rel(clientMartApp, clientMartDB, "JDBC", "")
    Rel(web, clientCRM, "REST", "Регистрация клиента, Карточка клиента, Профиль клиента")
    Rel(web, clientMartEstate, "REST", "Получить данные по объектам недвижимости")
    Rel(clientMartEstate, clientMartEstateDB, "JDBC", "")
    Rel(clientMartApp, govRegister, "SOAP", "гос. регистрация")
    Rel(clientCRM, clientMartApp, "REST", "")
    Rel(clientCRM, clientTourApp, "REST", "")
    Rel(clientCRM, activeDirectory, "REST", "Аутентификация менеджеров")
    Rel(tenantCRM, activeDirectory, "REST", "Аутентификация менеджеров")
    Rel(clientTourApp, clientTourDB, "Kafka", "Публикация каталогов объктов")
    Rel(web, authService, "REST", "Регистрация Аутентификация")
    Rel(authService, authDB, "", "")
    Rel(clientMartDB, dataWarehouse, "CDC", "Передача сырых данных")
    Rel(clientMartEstateDB, dataWarehouse, "CDC", "Передача сырых данных")
    Rel(clientCRMDB, dataWarehouse, "CDC", "Передача сырых данных")
    Rel(clientCRM, clientCRMDB, "JDBC", "")
    
    Rel(tenantApp, tenantCore, "REST, WS", "")
    Rel(tenantCore, tenantCoreDB, "JDBC", "")
    Rel(tenantCoreDB, dataWarehouse, "CDC", "Передача сырых данных")

    Rel(tenantApp, tenantCRM, "REST", "")
    Rel(tenantCore, tenantCRM, "REST", "")

    Rel(tenantCRM, tenantCRMDB, "JDBC", "данные собственников")
    Rel(tenantCRMDB, dataWarehouse, "CDC", "Передача сырых данных")
    Rel(tenantCore, externalAPI, "", "")
    Rel(tenantCore, paymentSystem, "", "")

    Rel(accountantService, authDB, "", "")
    
    
    Rel(client, web, "", "")
    Rel(tenant, tenantApp, "", "")
    
    Rel(aBI, dataWarehouse, "", "")    
    
}

Rel(manager, PropDevelopment, "", "")
Rel(aS, PropDevelopment, "", "")

@enduml


```