## 环境 ## 
### ansible 2.4.1.0 ###
- 远端主机 192.168.130.11
- ansible主机 192.168.130.1
---
  
  
  
#### 远端主机 ####
创建NOPASSWD sudo用户
```
useradd -m -s /bin/bash admin
echo 'admin:My@ansible2.4'|chpasswd
echo 'admin ALL=(ALL)      NOPASSWD: ALL' > /etc/sudoers.d/admin
```
  
  
  
#### ansible主机 ####
1. ##### 分发ssh pubkey到远程主机 #####
```
ssh-keygen  -t rsa -N '' -f ~/.ssh/id_rsa -q -b 2048
ssh-copy-id admin@192.168.130.11
```
2. ##### 测试 #####
http://docs.ansible.com/ansible/latest/become.html
- 命令行传参
```
cat > hosts.ini <<EOF
[kubernetes-master]
master1 ansible_hostname=master1 ansible_host=192.168.130.11
EOF

ansible -i hosts.ini all -m shell -a 'echo root:root|chpasswd' --user=admin --private-key=~/.ssh/id_rsa --become
或
ansible -i hosts.ini all -m shell -a 'echo root:root|chpasswd' \
--user=admin \
--private-key=~/.ssh/id_rsa \
--become \
--become-method=sudo \
--become-user=root
```
- 主机变量传参
```
cat > hosts.ini <<EOF
[kubernetes-master]
master1 ansible_hostname=master1 ansible_host=192.168.130.11 ansible_user=admin ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_become=true ansible_become_method=sudo ansiblie_become_user=root
EOF

ansible -i hosts.ini all -m shell -a 'echo root:root|chpasswd'
```
说明: 对于需要密码的sudo用户，需传入 **\-\-ask-become-pass** 来手动输入sudo密码。  
特别地，对NOPASSWD sudo用户传入 **\-\-ask-become-pass** 提示***SUDO password:*** 时可直接**回车**
```
ansible -i hosts.ini all -m shell -a 'echo root:root|chpasswd' \
--user=admin \
--private-key=~/.ssh/id_rsa \
--become \
--become-method=sudo \
--become-user=root \
--ask-become-pass
```
- 混合传参
```
cat > hosts.ini <<EOF
[kubernetes-master]
master1 ansible_hostname=master1 ansible_host=192.168.130.11 ansible_user=admin ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_become=true ansible_become_method=sudo ansiblie_become_user=root
EOF

ansible -i hosts.ini all -m shell -a 'echo root:root|chpasswd' -K
```
**提示:** 上述参数同样适用**ansible-playbook**
