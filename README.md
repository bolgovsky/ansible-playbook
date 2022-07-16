**ветка MNT-13**

Запуск производился в docker среде, поэтому рестарт сервисов:
```bash
systemctl ...
Failed to get D-Bus connection: Operation not permitted
```

Это происходит из-за особенности логики самих контейнеров, поэтому:
* Решение 1: вручную заходим на контейнер и перезапускаем
* Решение 2: [готовим docker container](https://serverfault.com/questions/824975/failed-to-get-d-bus-connection-operation-not-permitted)
* Решение 3: (перезапускаем контейнер) [пользуемся инструкцией самих разработчиков сервисов](https://vector.dev/docs/administration/management/)

В самом [playbook](https://github.com/bolgovsky/ansible-playbook/blob/main/site.yml): 

* Производится установка (с восстановлением возможности установки по другому имени дистрибутива) [Clickhouse](https://github.com/bolgovsky/ansible-playbook/blob/main/group_vars/clickhouse/vars.yml) 
версии 22.6.3.35

* Производится установка [Vector](https://github.com/bolgovsky/ansible-playbook/blob/main/group_vars/vector/vars.yml) 
версии 0.22.1 (более новые тащат кучу зависимостей и запуск выпадает в ошибку)

