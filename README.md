# Домашнее задание к занятию 2 «Работа с Playbook»

**1. Подготовьте свой inventory-файл prod.yml.**

```
cat inventory/prod.yml
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_connection: docker

vector:
  hosts:
    vector-01:
      ansible_connection: docker
```

**2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает vector. Конфигурация vector должна деплоиться через template файл jinja2. От вас не требуется использовать все возможности шаблонизатора, просто вставьте стандартный конфиг в template файл. Информация по шаблонам по ссылке.** </br>

**3. При создании tasks рекомендую использовать модули: get_url, template, unarchive, file.**</br>

**4. Tasks должны: скачать дистрибутив нужной версии, выполнить распаковку в выбранную директорию, установить vector.**</br>


cat vector.service.j2
```
[Unit]
Description=Vector
Documentation=https://vector.dev
After=network-online.target
Requires=network-online.target

[Service]
User={{ ansible_user_id }}
Group={{ ansible_user_id }}
ExecStart=/usr/bin/vector --config {{ vector_config_dir }}/vector.yml
ExecReload=/bin/kill -HUP $MAINPID
Restart=always

[Install]
WantedBy=multi-user.target
```

cat vector.yml.j2
```
{{ vector_config | to_nice_yaml }}
```

site.yml
```
- name: Install Vector
  hosts: vector
  handlers:
    - name: Start Vector service
      become: true
      become_method: su
      become_user: root
      ansible.builtin.service:
        name: vector
        state: restarted
  tasks:
    - name: Vector | Download packages
      ansible.builtin.get_url:
        url: "{{ vector_url }}"
        dest: "./vector-{{ vector_version }}-1.x86_64.rpm"
    - name: Vector | Install packages
      become: true
      become_method: su
      become_user: root
      ansible.builtin.yum:
        name: "./vector-{{ vector_version }}-1.x86_64.rpm"
    - name: Vector | Apply template
      ansible.builtin.template:
        src: vector.yml.j2
        dest: "{{ vector_config_dir }}/vector.yml"
        mode: "0644"
        owner: "{{ ansible_user_id }}"
        group: "{{ ansible_user_gid }}"
        validate: vector validate --no-environment --config-yaml %s
    - name: Vector | change systemd unit
      ansible.builtin.template:
        src: vector.service.j2
        dest: /usr/lib/systemd/system/vector.service
        mode: "0644"
        owner: "{{ ansible_user_id }}"
        group: "{{ ansible_user_gid }}"
        backup: true
      notify: Start Vector service
```

**5. Запустите ansible-lint site.yml и исправьте ошибки, если они есть.**
```
 ansible-lint site.yml
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml
WARNING  Listing 2 violation(s) that are fatal
yaml: truthy value should be one of [false, true] (truthy)
site.yml:41

yaml: truthy value should be one of [false, true] (truthy)
site.yml:51

You can skip specific rules or tags by adding them to your configuration file:
# .ansible-lint
warn_list:  # or 'skip_list' to silence them completely
  - yaml  # Violations reported by yamllint

Finished with 2 failure(s), 0 warning(s) on 1 files.
```
fixed + site.yml переименовал в playbook.yml

**6. Попробуйте запустить playbook на этом окружении с флагом --check.**
```
 ansible-playbook -i inventory/prod.yml playbook.yml --check

PLAY [Install Clickhouse] *******************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
changed: [clickhouse-01] => (item=clickhouse-client)
changed: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Install clickhouse packages] **********************************************************************************************************************************************************************************************************
fatal: [clickhouse-01]: FAILED! => {"changed": false, "msg": "No RPM file matching 'clickhouse-common-static-22.3.3.44.rpm' found on system", "rc": 127, "results": ["No RPM file matching 'clickhouse-common-static-22.3.3.44.rpm' found on system"]}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=1    ignored=0
```

**7. Запустите playbook на prod.yml окружении с флагом --diff. Убедитесь, что изменения на системе произведены.**

```
PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
vector-01                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**8. Повторно запустите playbook с флагом --diff и убедитесь, что playbook идемпотентен.**

```
PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
vector-01                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
**9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook,</br> какие у него есть параметры и теги. Пример качественной документации ansible playbook по ссылке.**



**10. Готовый playbook выложите в свой репозиторий, поставьте тег 08-ansible-02-playbook на фиксирующий коммит, в ответ предоставьте ссылку на него.**

https://github.com/zrusakov/08-ansible-02-playbook/commit/71c8e03637c61ef10213578072547d0eb25fec76
