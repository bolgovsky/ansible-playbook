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


Пример вывода:
```bash 
denis@DenisPC:~/ansible-playbook-cvl/playbook$ ansible-playbook -i inventory/prod.yml site.yml -v
Using /etc/ansible/ansible.cfg as config file


PLAY [Install Clickhouse] ************************************************************************************************************************************
TASK [Gathering Facts] ***************************************************************************************************************************************ok: [clickhouse-01]

TASK [Get clickhouse distrib] ********************************************************************************************************************************ok: [clickhouse-01] => (item=clickhouse-client) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-client-22.6.3.35.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-client", "mode": "0644", "msg": "HTTP Error 304: Not Modified", "owner": "root", "size": 67423, "state": "file", "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-client-22.6.3.35.noarch.rpm"}
ok: [clickhouse-01] => (item=clickhouse-server) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-server-22.6.3.35.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-server", "mode": "0644", "msg": "HTTP Error 304: Not Modified", "owner": "root", "size": 91129, "state": "file", "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-server-22.6.3.35.noarch.rpm"}
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.6.3.35.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-common-static", "mode": "0644", "msg": "Request failed", "owner": "root", "response": "HTTP Error 404: Not Found", "size": 259749094, "state": "file", "status_code": 404, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.6.3.35.noarch.rpm"}

TASK [Get clickhouse distrib] ********************************************************************************************************************************ok: [clickhouse-01] => {"changed": false, "dest": "./clickhouse-common-static-22.6.3.35.rpm", "elapsed": 0, "gid": 0, "group": "root", "mode": "0644", "msg":
TASK [Create database] ***************************************************************************************************************************************ok: [clickhouse-01] => {"changed": false, "cmd": ["clickhouse-client", "-q", "create database logs;"], "delta": "0:00:00.169411", "end": "2022-07-16 12:29:01.077041", "failed_when_result": false, "msg": "non-zero return code", "rc": 82, "start": "2022-07-16 12:29:00.907630", "stderr": "Received exception from server (version 22.6.3):\nCode: 82. DB::Exception: Received from localhost:9000. DB::Exception: Database logs already exists.. (DATABASE_ALREADY_EXISTS)\n(query: create database logs;)", "stderr_lines": ["Received exception from server (version 22.6.3):", "Code: 82. DB::Exception: Received from localhost:9000. DB::Exception: Database logs already exists.. (DATABASE_ALREADY_EXISTS)", "(query: create database logs;)"], "stdout": "", "stdout_lines": []}

PLAY [Install Vector] ****************************************************************************************************************************************
TASK [Gathering Facts] ***************************************************************************************************************************************ok: [vector-01]

TASK [Get vector distrib] ************************************************************************************************************************************ok: [vector-01] => (item=vector) => {"ansible_loop_var": "item", "changed": false, "dest": "./vector-0.22.1.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "vector", "mode": "0644", "msg": "HTTP Error 304: Not Modified", "owner": "root", "size": 52417559, "state": "file", "uid": 0, "url": "https://packages.timber.io/vector/0.22.1/vector-0.22.1-1.x86_64.rpm"}

TASK [Install vector packages] *******************************************************************************************************************************ok: [vector-01] => (item=vector) => {"ansible_loop_var": "item", "changed": false, "item": "vector", "msg": "", "rc": 0, "results": ["vector-0.22.1-1.x86_64 providing vector-0.22.1.rpm is already installed"]}

PLAY RECAP ***************************************************************************************************************************************************clickhouse-01              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
vector-01                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```