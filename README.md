workspace "Сервис доставки" "Вариант 6" {
    model {
        // Люди
        person "Пользователь" "Отправитель или получатель посылки" {
            tags "Person"
        }

        // Внешние системы
        softwareSystem "Платежная система" "Обрабатывает платежи за доставку" {
            tags "External System"
        }
        softwareSystem "Служба уведомлений" "Отправляет email и SMS уведомления" {
            tags "External System"
        }

        // Наша система
        softwareSystem "Сервис доставки" "Позволяет пользователям создавать посылки и оформлять доставку" {
            tags "Software System"

            // Контейнеры
            container "Web Application" "Предоставляет интерфейс пользователя" "React, SPA" {
                tags "Container"
            }

            container "User Service" "Управление пользователями" "Spring Boot, REST API, Java" {
                tags "Container"
            }
            container "Parcel Service" "Управление посылками" "Spring Boot, REST API, Java" {
                tags "Container"
            }
            container "Delivery Service" "Управление доставками" "Spring Boot, REST API, Java" {
                tags "Container"
            }
            container "Payment Service" "Интеграция с платежной системой" "Spring Boot, REST API, Java" {
                tags "Container"
            }
            container "Notification Service" "Отправка уведомлений" "Spring Boot, REST API, Java" {
                tags "Container"
            }

            // Базы данных
            container "User Database" "Хранит данные пользователей" "PostgreSQL" {
                tags "Database"
            }
            container "Parcel Database" "Хранит данные посылок" "PostgreSQL" {
                tags "Database"
            }
            container "Delivery Database" "Хранит данные доставок" "PostgreSQL" {
                tags "Database"
            }
        }

        // Отношения (контекст)
        Пользователь -> "Сервис доставки" "Использует для создания посылок и доставок"
        "Сервис доставки" -> "Платежная система" "Выполняет платежи через"
        "Сервис доставки" -> "Служба уведомлений" "Отправляет уведомления через"

        // Отношения между контейнерами
        "Web Application" -> "User Service" "Вызывает API" "HTTPS/REST"
        "Web Application" -> "Parcel Service" "Вызывает API" "HTTPS/REST"
        "Web Application" -> "Delivery Service" "Вызывает API" "HTTPS/REST"

        "User Service" -> "User Database" "Читает/записывает данные" "JDBC"
        "Parcel Service" -> "Parcel Database" "Читает/записывает данные" "JDBC"
        "Delivery Service" -> "Delivery Database" "Читает/записывает данные" "JDBC"

        "Delivery Service" -> "User Service" "Проверяет существование пользователей" "HTTPS/REST"
        "Delivery Service" -> "Parcel Service" "Получает данные посылки" "HTTPS/REST"
        "Delivery Service" -> "Payment Service" "Инициирует оплату" "HTTPS/REST"
        "Delivery Service" -> "Notification Service" "Запрашивает отправку уведомлений" "HTTPS/REST"

        "Payment Service" -> "Платежная система" "Отправляет запросы на оплату" "HTTPS/REST"
        "Notification Service" -> "Служба уведомлений" "Отправляет письма/SMS" "SMTP/HTTP"
    }

    views {
        // Диаграмма System Context
        systemContext "Сервис доставки" "Диаграмма контекста системы" {
            include *
            autoLayout
        }

        // Диаграмма Container
        container "Сервис доставки" "Диаграмма контейнеров" {
            include *
            autoLayout
        }

        // Динамическая диаграмма для сценария "Создание доставки"
        dynamic "Создание доставки" "Последовательность взаимодействия при создании доставки" {
            // Участники: Пользователь, Web Application, Delivery Service, User Service, Parcel Service, Payment Service, Notification Service, их БД, внешние системы.
            // Используем элементы, которые были описаны.
            // Важно: в dynamic view нужно ссылаться на элементы по их ID или имени. Используем имена, так как в DSL мы их задали.
            // Укажем последовательность шагов.
            
            // Шаг 1: Пользователь отправляет запрос через Web Application
            Пользователь -> "Web Application" "Запрашивает создание доставки"

            // Шаг 2: Web Application вызывает Delivery Service
            "Web Application" -> "Delivery Service" "POST /deliveries"

            // Шаг 3: Delivery Service проверяет отправителя через User Service
            "Delivery Service" -> "User Service" "GET /users/{senderId}"

            // Шаг 4: User Service обращается к своей БД
            "User Service" -> "User Database" "SELECT"

            // Шаг 5: User Service возвращает данные отправителя
            "User Service" -> "Delivery Service" "Данные отправителя"

            // Шаг 6: Delivery Service проверяет получателя
            "Delivery Service" -> "User Service" "GET /users/{receiverId}"
            "User Service" -> "User Database" "SELECT"
            "User Service" -> "Delivery Service" "Данные получателя"

            // Шаг 7: Delivery Service получает данные посылки от Parcel Service
            "Delivery Service" -> "Parcel Service" "GET /parcels/{parcelId}"
            "Parcel Service" -> "Parcel Database" "SELECT"
            "Parcel Service" -> "Delivery Service" "Данные посылки"

            // Шаг 8: Delivery Service создает запись о доставке в своей БД
            "Delivery Service" -> "Delivery Database" "INSERT into deliveries"

            // Шаг 9: Delivery Service вызывает Payment Service для оплаты
            "Delivery Service" -> "Payment Service" "POST /payments"

            // Шаг 10: Payment Service взаимодействует с внешней платежной системой
            "Payment Service" -> "Платежная система" "Проведение платежа"

            // Шаг 11: Платежная система возвращает результат
            "Платежная система" -> "Payment Service" "Подтверждение оплаты"

            // Шаг 12: Payment Service возвращает результат Delivery Service
            "Payment Service" -> "Delivery Service" "Результат оплаты"

            // Шаг 13: Delivery Service обновляет статус доставки
            "Delivery Service" -> "Delivery Database" "UPDATE status"

            // Шаг 14: Delivery Service вызывает Notification Service для уведомлений
            "Delivery Service" -> "Notification Service" "POST /notifications"

            // Шаг 15: Notification Service отправляет уведомления через внешнюю службу
            "Notification Service" -> "Служба уведомлений" "Отправка email/SMS"

            // Шаг 16: Notification Service возвращает подтверждение
            "Служба уведомлений" -> "Notification Service" "Уведомления отправлены"
            "Notification Service" -> "Delivery Service" "Уведомления отправлены"

            // Шаг 17: Delivery Service возвращает успешный ответ Web Application
            "Delivery Service" -> "Web Application" "201 Created"

            // Шаг 18: Web Application отображает результат пользователю
            "Web Application" -> Пользователь "Информация о созданной доставке"

            autoLayout
        }

        // Дефолтные стили
        styles {
            element "Person" {
                background #08427b
                color #ffffff
                shape person
            }
            element "Software System" {
                background #1168bd
                color #ffffff
            }
            element "External System" {
                background #999999
                color #ffffff
            }
            element "Container" {
                background #438dd5
                color #ffffff
                shape roundedbox
            }
            element "Database" {
                background #438dd5
                color #ffffff
                shape cylinder
            }
        }

        // Описание диаграмм
        description "Сервис доставки" "Система позволяет пользователям создавать посылки и оформлять доставку."
    }
}
