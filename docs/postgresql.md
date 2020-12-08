# Установка в качестве рекордера бд Postgresql в докере

## Ставим постгрес в докер контейнере и в volume прокидываем следующие данные

например, если использовать docker-composse
```yaml
    postgresql:
        container_name: postgresql
        image: postgres:latest
        restart: always
        ports:
            - 5432:5432
        environment:
            - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}

        volumes:
            - postgres_data:/var/lib/postgresql/data
            - unix_s:/var/run/postgresql

volumes:
    postgres_data:
    unix_s:
```

## Меняем контейнер ХА
Я Использовал portainer. 
1. В нем выбираем контейнер `homeassistant`
2. Нажимаем кнопку редактировать
3. Добавляем в разделы volumes новый с параметрами ```unix_s -> /tmp```
4. Нажимаем пересоздать

## Настройка пользователя
1. Создаем пользователя в интерфейсе ha, например `dbuser`.
2. Добавляем аналогичного в postgres Я делал через pgadmin. (не забываем давать разрешение на вход - на вкладках создания пользователя)
3. Также создаем базу данных, например `homeassistant`

## Настройка HA

в файле `configuration.yaml` необходимо добавить или изменить запись:
```yaml
recorder:
  db_url: "postgresql://dbuser:password@localhost/homeassistant"
```
Так как у меня это все критится на одной машине, то адрес будет localhost, иначе надо указать или имя машины или ее ip адрес.

## Примечание
1. Что бы проверить доступность сервера можно выполнить следующие действия
```sh
sudo docker -it homeassistant /bin/bash
psql -U dbadmin -h localhost -d homeassitant
```
так можно подключиться из контейнера homeassistant к бд postgres и возможно выявить проблемы подключения

2. Возможно в контейнере homeassistant не будет нужной команды `psql`. Ее можно установить командой ```apk add postgresql-client```