## 1 Подготовка среды для тестового задания

### Установка ansible:
apt-get update  
apt install ansible  
ansible-community --version  

### Команды на клиентах:
### создаём ssh-ключи
ssh-keygen -t rsa -m PEM  
ssh-keygen -f /root/.ssh/id_rsa.pub -m 'PEM' -e > /root/.ssh/id_rsa.pem  
cp ~/.ssh/id_rsa.pem /root  
#далее копируем эти ssh ключи (~/.ssh/id_rsa.pem) на хостовую машину в папку ~/.ssh (я использую GUI MobaXTerm)

### Команды на хосте (Мастер нода):
mkdir ansible  
cd ansible  
#Генерирую ключи
ssh-keygen -t rsa -m PEM  
cat ~/.ssh/id_rsa.pub  

#на клиентских(подчинённых) машинах заносим этот публичный ключ:  
nano /root/.ssh/authorized_keys  
ssh-rsa AAAAB3Nza.....  

### Создаём файл inventory
```nano hosts.txt
[servers_group]
server-one ansible_host=IP1 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_rsa1.pem
server-two ansible_host=IP2 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_rsa2.pem
```
### Проверка 
ansible -i hosts.txt all -m ping

## 2 Выполнение тестового задания

### переносим файл hosts.txt в папку /root/ansible/inventories/prod/hosts.txt
mkdir inventories
cd inventories/
mkdir prod
cd prod
mv /root/ansible/hosts.txt /root/ansible/inventories/prod

### Создаём файл ansible.cfg
```[defaults]
host_key_checking            = false
inventory                    = /root/ansible/inventories/prod/hosts.txt
```

### Переносим переменные в group_vars/server_group
cd ansible  
mkdir group_vars  
nano group_vars/servers_group  
```
ansible_user: root
ansible_ssh_private_key_file: /root/.ssh/id_rsa
var_exmpl_xx: Hello World
var_exmpl_yy: 123321
var_exmpl_zz: 000
```
### Переопределяем переменные для конкретных хостов(клиентских) в group_vars/server_group
cd ansible  
mkdir host_vars  
nano host_vars/server-two  
```var_exmpl_zz: 111```

### Создать сценарий scenario_one.yml и запустить его на хост "server-one"
nano scenario_one.yml
```
---
- name: Create directory and install packages
  hosts: server-one
  become: true
  tasks:
    - name: Create directory
      file:
        path: /tmp/example_dir
        state: directory
        owner: root
        group: root

    - name: Install packages
      apt:
        name: "{{ item }}"
        state: present
      with_items:
        - htop
        - screen
```

#Запуск  
ansible-playbook -i inventories/prod/hosts.txt scenario_one.yml  
#или  
ansible-playbook scenario_one.yml  

### Создать роль 
Documentation: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html 

cd ansible    
mkdir roles  
cd roles  
ansible-galaxy init my_role  
### Далее разбиваем playbook по ролевой структуре (см. roles/my_role)

### Создать playbook scenario_two.yml
nano scenario_two.yml  
```
---
- name: Create directories, copy files, templates and do cron task
  hosts: servers_group
  become: yes

  roles:
    - my_role
```
ansible-playbook scenario_two.yml  

### Проверяем, что крон отрабатывает
tail -n 10 /var/log/syslog  
Должен получиться такой вывод:  
```
2024-05-06T13:31:01.790595+00:00 ubuntu-2gb-nbg1-2 CRON[12635]: (root) CMD (date)
```  

