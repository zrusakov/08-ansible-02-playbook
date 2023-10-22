# Структура проекта
./group_vars - переменные
./group_vars/clickhouse/vars.yml - переменные для clickhouse 
./group_vars/vector/vars.yml - переменные для vector 

./inventory/prod.yml - описание инстансов

./templates - шаблоны конфигураций приложений

./playbook.yml - playbook для развертывания

---
# Описание Playbook
## Install Clickhouse
Установка Install Clickhouse, task состоит из следующих plays:
* **Get clickhouse distrib** - загрузка на инстанс дистрибутива Clickhouse
* **Install clickhouse packages** - установка загруженных дистрибутивов
* **Start clickhouse service** - перезагрузка службы Clickhouse
* **Create database** - создание БД Clickhouse

## Install Vector
Установка Vecto, task состоит из следующих plays:
* **Vector | Download packages** - загрузка на инстанс дистрибутива Vector
* **Vector | Install packages** - установка загруженных дистрибутивов
* **Vector | Apply template, Vector | change systemd unit** - применние стандартной конфигурации 
