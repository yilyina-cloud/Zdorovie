@startuml
!include <C4/C4_Context>

title Здоровье+ — целевая интеграционная архитектура
LAYOUT_TOP_DOWN()

Person(patient, "Пациент")
Person(staff, "Врач / администратор")
Person(lab, "Лаборант")

System(app, "Мобильное приложение", "НОВОЕ. Запись, оплата, анализы, ДМС")
System(portal, "Веб-портал", "Существующий интерфейс пациентов и сотрудников")

System(auth, "Сервис аутентификации", "НОВОЕ. Вход пациентов и выдача токенов")
System(gateway, "API Gateway", "НОВОЕ. Проверка токенов, маршрутизация, rate limiting")
System(backend, "Backend мобильного приложения", "НОВОЕ. API, координация операций, адаптеры и обработка событий")

System(schedule, "Расписание", "Слоты врачей и записи пациентов")
System(crm, "CRM", "Карточки пациентов и копии результатов анализов")
System(lis, "LIS", "Лабораторные исследования и результаты")
System(policies, "1С: Полисы", "Проверка и прикрепление полисов ДМС")
System_Ext(payment, "Платёжный шлюз", "Оплата, проверка статуса и возврат")

System(broker, "RabbitMQ", "Существующий брокер. Расширяется для обмена событиями")
System(notifications, "Уведомления", "Отправка СМС, писем и пушей")
System_Ext(providers, "Провайдеры уведомлений", "Доставка СМС, писем и пушей")

Rel(patient, app, "Использует")
Rel(patient, portal, "Использует")
Rel(staff, portal, "Управляет записями и просматривает данные")
Rel(lab, lis, "Просматривает результаты")

Rel(app, auth, "Вход и получение токенов", "OIDC / HTTPS")
Rel(app, gateway, "Запросы с access token", "REST / HTTPS")
Rel(gateway, backend, "Передаёт запросы", "REST / HTTPS")

Rel(backend, schedule, "Слоты, резервирование, подтверждение и отмена", "REST / HTTPS, синхронно")
Rel(backend, crm, "Чтение карточек и анализов; обновление результатов", "REST / HTTPS, синхронно")
Rel(backend, policies, "Проверка и прикрепление полиса", "HTTP-шлюз через защищённый канал, синхронно")
Rel(backend, payment, "Оплата, статус и возврат", "REST / HTTPS, синхронно")
Rel(backend, lis, "Резервирование и отмена сдачи анализов", "SOAP / REST через адаптер, синхронно")

Rel(lis, broker, "Публикует готовые и исправленные результаты", "AMQP / TLS, асинхронно")
Rel(schedule, broker, "Публикует изменения записей", "AMQP / TLS, асинхронно")
Rel(broker, backend, "Передаёт события результатов и записей", "AMQP / TLS, асинхронно")
Rel(backend, broker, "Публикует статус оплаты и доступность анализов в CRM", "AMQP / TLS, асинхронно")
Rel(broker, notifications, "Передаёт события для уведомлений", "AMQP / TLS, асинхронно")

Rel(notifications, providers, "Передаёт уведомления на доставку", "HTTPS")
Rel(providers, app, "Доставляет пуши")
Rel(providers, patient, "Доставляет СМС и письма")

Rel(portal, schedule, "Слоты и записи", "REST / HTTPS, синхронно")
Rel(portal, crm, "Карточки пациентов и анализы", "REST / HTTPS, синхронно")
Rel(portal, policies, "Проверка полиса", "HTTP-шлюз через защищённый канал, синхронно")
Rel(portal, payment, "Оплата", "REST / HTTPS, синхронно")
Rel(portal, broker, "Запросы на уведомления", "AMQP / TLS, асинхронно")

SHOW_LEGEND()
@enduml
