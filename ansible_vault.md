#### 一.创建加密文件 ####
http://docs.ansible.com/ansible/latest/vault.html#encrypt-string-for-use-in-yaml
```
VAULT_ID='fooodev'
echo $VAULT_ID > ~/.vault_pass.txt
```
提示: ~/.vault_pass.txt是用来加密字符的
  
  
  
#### 二.生成密文变量 ####
```
REAL_PASSWORD='My@ansible2.4'

ansible-vault encrypt_string --vault-id dev@~/.vault_pass.txt $REAL_PASSWORD --name 'vault_the_dev_secret'
或
ansible-vault encrypt_string --vault-id dev@~/.vault_pass.txt $REAL_PASSWORD --name 'vault_the_dev_secret'
或
echo $REAL_PASSWORD | ansible-vault encrypt_string --vault-id dev@~/.vault_pass.txt --stdin-name 'vault_the_dev_secret'
```


#### 三.定义变量 ####
  
http://docs.ansible.com/ansible/latest/intro_inventory.html  
http://docs.ansible.com/ansible/latest/playbooks_best_practices.html#best-practices-for-variables-and-vaults
```
mkdir group_vars
ansible-vault encrypt_string --vault-id dev@~/.vault_pass.txt $REAL_PASSWORD --name 'vault_the_dev_secret' > group_vars/all
```
```
cat group_vars/all

vault_the_dev_secret: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          37666437633465346364643834613532363439393366396231393764396634376362306534343136
          6639343661346131373036386365366362656130626263350a323533396136366132313436646661
          30323666653431393131386434393634356330356366373738303739623562313436323134613134
          3932303662326332630a656163653632306538313163373837356434346662666337626262383537
          3236
```
  
  
  
#### 四.引用变量 ####
http://docs.ansible.com/ansible/latest/playbooks_vault.html#single-encrypted-variable
http://docs.ansible.com/ansible/latest/playbooks_vault.html#encrypt-string
```
cat > hosts.ini <<'EOF'
[all:vars]
ansible_user=admin   
ansible_ssh_private_key_file=~/.ssh/id_rsa   
ansible_become=true   
ansible_become_method=sudo   
ansiblie_become_user=root   
ansible_become_pass={{ vault_the_dev_secret }}

[kubernetes-master]
master1 ansible_hostname=master1 ansible_host=192.168.130.11
EOF

ansible -i hosts.ini all -m shell -a 'echo root:root|chpasswd' --vault-password-file ~/.vault_pass.txt
```


  
  
  
---
#### ansible-vault ####
- 创建密文文件
```
ansible-vault create foo.yml
```
在创建文件的同时给文件加密(设置密码),默认使用$EDITOR,文件内容是密文，如
```
$ANSIBLE_VAULT;1.1;AES256
37306164376138313163636134643662306161313732613932326134666564636438373531353138
3338666136623536373363326264626332626461636463300a383764373364646566636330383665
30366663636634396637363537323064373430313830306236633363393362366333383962396137
6531353232646635650a303861346262396661366137313136323333383439396534303138303566
61386166376438396562636637316336653233633333333038333566323965333031363565393462
3663303239393236663164343266636339393437323166396335
```

- 编辑密文文件
```
ansible-vault edit foo.yml
```
需要输入正确的密码后才能修改

- 明文文件加密
```
ansible-vault encrypt foo.yml
```

- 密文文件解密
```
ansible-vault decrypt foo.yml
```

- 密文文件查看
```
ansible-vault view foo.yml
```

- 修改密文文件的密码
```
ansible-vault rekey foo.yml                   
Vault password:
New Vault password:
Confirm New Vault password:
Rekey successful
```
需要先输入正确的密码解密后才能设置新密码

- **运行密文playbook**
```
ansible-playbook site.yml --ask-vault-pass
或
ansible-playbook site.yml --vault-password-file ~/.vault_pass.txt
```
