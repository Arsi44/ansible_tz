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

### Команды на хосте (Главная нода):
mkdir ansible  
cd ansible  
#Генерирую ключи
ssh-keygen -t rsa -m PEM  
cat ~/.ssh/id_rsa.pub  

#на клиентских(подчинённых) машинах заносим этот публичный ключ:  
nano /root/.ssh/authorized_keys  
ssh-rsa AAAAB3Nza.....  

### Создаём фал inventory
```nano hosts.txt
[staging]
linux1 ansible_host=IP1 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_rsa1.pem
[production]
linux2 ansible_host=IP2 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_rsa2.pem
```
### Проверка 
ansible -i hosts.txt all -m ping
