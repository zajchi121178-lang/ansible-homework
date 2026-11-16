# Домашнее задание: Ansible

## Задание

Playbook Ansible, который содержит:

- имя плейбука: `homework.yaml`;
- inventory из двух хостов;
- авторизацию через техническую учётную запись `ansible` (по SSH-ключу);
- наименование плейбука: `netology-ml`;
- проверку доступности хостов через модуль `ping`;
- установку пакетов через переменные (`vars`): net-tools, git, tree, htop, mc, vim;
- обновление кэша пакетов (`update_cache`);
- копирование текстового файла `test.txt`;
- создание в цикле групп и пользователей `devops_1`, `test_1` с домашними директориями.

## Структура проекта

```
ansible-project/
├── inventory.ini
├── homework.yaml
├── test.txt
└── readme.md
```

## inventory.ini

```ini
[vm]
netology-vm1 ansible_host=192.168.1.106 ansible_user=ansible ansible_ssh_private_key_file=~/.ssh/id_rsa_ansible

[local]
localhost ansible_connection=local
```

## Подготовка технической учётной записи

На управляемом хосте создан пользователь `ansible` с правом sudo без пароля:

```bash
sudo useradd -m -s /bin/bash ansible
sudo usermod -aG sudo ansible
sudo passwd ansible
echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
sudo chmod 440 /etc/sudoers.d/ansible
```

Публичный ключ передан на хост:

```bash
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub ansible@192.168.1.106
```

## homework.yaml

```yaml
---
- name: netology-ml
  hosts: all
  become: true

  vars:
    packages:
      - net-tools
      - git
      - tree
      - htop
      - mc
      - vim
    user_groups:
      - devops_1
      - test_1

  tasks:
    - name: Проверка доступности хостов
      ansible.builtin.ping:

    - name: Обновление кэша пакетов (update)
      ansible.builtin.apt:
        update_cache: true

    - name: Установка пакетов из переменной
      ansible.builtin.apt:
        name: "{{ packages }}"
        state: present

    - name: Копирование текстового файла test.txt
      ansible.builtin.copy:
        src: test.txt
        dest: /tmp/test.txt
        mode: "0644"

    - name: Создание групп в цикле
      ansible.builtin.group:
        name: "{{ item }}"
        state: present
      loop: "{{ user_groups }}"

    - name: Создание пользователей и их домашних директорий
      ansible.builtin.user:
        name: "{{ item }}"
        group: "{{ item }}"
        home: "/home/{{ item }}"
        create_home: true
        shell: /bin/bash
```

## Запуск

```bash
ansible-playbook -i inventory.ini homework.yaml
```

## Результат выполнения

```
PLAY RECAP *************************************************************************************************************
localhost                  : ok=7    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
netology-vm1               : ok=7    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Все задачи выполнились успешно на обоих хостах. Разница в `changed` объясняется идемпотентностью: на `localhost` часть пакетов и групп уже была установлена, поэтому Ansible не выполнял изменений.

## Проверка результата

```bash
ssh -i ~/.ssh/id_rsa_ansible ansible@192.168.1.106
cat /tmp/test.txt
id devops_1
id test_1
ls -ld /home/devops_1 /home/test_1
```
