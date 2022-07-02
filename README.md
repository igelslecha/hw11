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
переменнýе в Ansible
- Сделать все это с исполþзованием Ansible роли
Домашнее задание считаетсā принятым, если:
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

