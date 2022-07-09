# hw11
ansible 

**Домашнее задание**

*Подготовить стенд на Vagrant как минимум с одним сервером. На этом
сервере используя Ansible необходимо развернуть nginx со следующими
условиями:*
- необходимо использовать модуль yum/apt
- конфигурационные файлы должны быть взяты из шаблона jinja2 с
переменными
- после установки nginx должен быть в режиме enabled в systemd
- должен быть исполþзован notify для старта nginx после установки
- сайт должен слушать на нестандартном порту - 8080, для этого использовать
переменную в Ansible
- Сделать все это с использованием Ansible роли
Домашнее задание считается принятым, если:
- предоставлен Vagrantfile и готовый playbook/роль ( инструкция по запуску
стенда, если посчитаете необходимым )
- после запуска стенда nginx доступен на порту 8080
- при написании playbook/роли соблюдены перечисленные в задании условия

**Решение**

* Создал папку \Ansible\. В неё скачал предоставляемый шаблон vagrandfile (изменил IP адрес из-за конфликта с адресным пространстом моей машины).*
* Поднял виртуальную машину*
```
igels@LaptopAll:~/hw11/Ansible$ vagrant up
Bringing machine 'nginx' up with 'virtualbox' provider...
==> nginx: Importing base box 'centos/7'...
==> nginx: Matching MAC address for NAT networking...
==> nginx: Checking if box 'centos/7' version '2004.01' is up to date...
```
* Создаю инвентори файл с информацией о в. машинке*
```
igels@LaptopAll:~/hw11/Ansible$ vi staging/hosts
web]
nginx ansible_host=127.0.0.1 ansible_port=2222 ansible_user=vagrant
ansible_private_key_file=.vagrant/machines/nginx/virtualbox/private_key
```
* Проверяю доступ к машине*
```
igels@LaptopAll:~/hw11/Ansible$ ansible nginx -i staging/hosts -m ping
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```
* Чтобы не указывать каждый раз явно инвентори файл, создаю ansible.cfg файл и прописываю конфигурацию в нем *
```
igels@LaptopAll:~/hw11/Ansible$ vim ansible.cfg
[defaults]
inventory = staging/hosts
remote_user = vagrant
host_key_checking = False
retry_files_enabled = False
```
* Из предыдущего файла удаляю информацию о пользователе*
```
igels@LaptopAll:~/hw11/Ansible$ vi staging/hosts
[web]
nginx ansible_host=127.0.0.1 ansible_port=2222
ansible_private_key_file=.vagrant/machines/nginx/virtualbox/private_key
```
* Проверяю доступ к машине учитывая изменения*
```
igels@LaptopAll:~/hw11/Ansible$ ansible nginx -m ping
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```
* Проверяю какое ядро установлено на хосте *
```
igels@LaptopAll:~/hw11/Ansible$ ansible nginx -m command -a "uname -r"
nginx | CHANGED | rc=0 >>
3.10.0-1127.el7.x86_64
```
* Проверяю статус сервиса firewalld *
```
igels@LaptopAll:~/hw11/Ansible$ ansible nginx -m systemd -a name=firewalld
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "name": "firewalld",
    "status": {
        "ActiveEnterTimestampMonotonic": "0",
        "ActiveExitTimestampMonotonic": "0",
        "ActiveState": "inactive",
```
* Устанавливаю пакет epel-release на хост*
```
igels@LaptopAll:~/hw11/Ansible$ ansible nginx -m yum -a "name=epel-release state=present" -b
nginx | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "installed": [
            "epel-release"
        ]
    },
```
* Создаю файл epel.yml, который будет простым плейбуком описывающим установку "epel-release" *
* (Для работы с этим файлом использую Visio Studio Code, до неё очень долго мучался с отработкой строк и отспуплений в файле плейбука)*
```---
- name: Install EPEL Repo
  hosts: nginx
  become: true
  tasks:
    - name: Install EPEL Repo package from standard repo
      yum:
       name: epel-release
       state: present
```
* Запускаю выполнение плейбука *
```
igels@LaptopAll:~/hw11/Ansible$ ansible-playbook epel.yml

PLAY [Install EPEL Repo] *******************************************************

TASK [Gathering Facts] *********************************************************
ok: [nginx]

TASK [Install EPEL Repo package from standard repo] ****************************
ok: [nginx]

PLAY RECAP *********************************************************************
nginx                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
* Запускаю команду  ansible nginx -m yum -a "name=epel-release state=absent" -b, с последующей установкой плейбука, чтобы увидеть разницу *
```
igels@LaptopAll:~/hw11/Ansible$ ansible-playbook epel.yml

PLAY [Install EPEL Repo] *******************************************************

TASK [Gathering Facts] *********************************************************
ok: [nginx]

TASK [Install EPEL Repo package from standard repo] ****************************
changed: [nginx]

PLAY RECAP *********************************************************************
nginx                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
* Создаю файл nginx.yml, который будет устанавливать nginx *
* В плейбук введены tag, проверяю их *
```
igels@LaptopAll:~/hw11/Ansible$ ansible-playbook nginx.yml --list-tags

playbook: nginx.yml

  play #1 (nginx): NGINX | Install and configure NGINX	TAGS: []
      TASK TAGS: [epel-package, nginx-configuration, nginx-package, packages]
* Установка только nginx *
```
igels@LaptopAll:~/hw11/Ansible$ ansible-playbook nginx.yml -t nginx-package

PLAY [NGINX | Install and configure NGINX] *************************************

TASK [Gathering Facts] *********************************************************
ok: [nginx]

TASK [NGINX | Install nginx package from EPEL Repo] ****************************
changed: [nginx]

RUNNING HANDLER [restart nginx] ************************************************
changed: [nginx]

PLAY RECAP *********************************************************************
nginx                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
* Добавляю шаблон для конфига nginx и модуль, который будет копировать этот шаблон на хост *
```
- name: NGINX | Create NGINX config file from template
 template:
 src: templates/nginx.conf.j2
 dest: /tmp/nginx.conf
 tags:
 - nginx-configuration
 ```
 * Добавляю переменные, чтобы nginx слушал на 8080 порту, добавляю секцию vars*
 ```
 vars:
 nginx_listen_port: 8080
 ```
 * Теперь создаю handler и добавляю notify к копированию шаблона. Теперь каждýй раз когда *
 * конфиг будет изменяться - сервис перезагрузиться. Секция с handlers будет выглядеть следующим образом: *
 ```
 handlers:
 - name: restart nginx     # рестарт и включение сервиса при загрузке сервиса
 systemd:
 name: nginx
 state: restarted
 enabled: yes

 - name: reload nginx      # перечитать конфиг
 systemd:
 name: nginx
 state: reloaded
 ```
 * Проверяю доступ к nginx через порт 8080 *
 ```
 igels@LaptopAll:~/hw11/Ansible$ curl http://192.168.56.150:8080
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
  <title>Welcome to CentOS</title>
  <style rel="stylesheet" type="text/css"> 

	html {
	background-image:url(img/html-background.png);
	background-color: white;
	font-family: "DejaVu Sans", "Liberation Sans", sans-serif;
	font-size: 0.85em;
	line-height: 1.25em;
	margin: 0 4% 0 4%;
	}
```
 
