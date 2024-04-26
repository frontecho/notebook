# 云服务器开启Root权限命令

`sudo sed -i 's/PasswordAuthentication /#PasswordAuthentication /g' /etc/ssh/sshd_config`

`sudo sed -i 's/PermitRootLogin /#PermitRootLogin /g' /etc/ssh/sshd_config`

`sudo sed -i 's/PermitEmptyPasswords /#PermitEmptyPasswords /g' /etc/ssh/sshd_config`

`echo "PermitRootLogin yes" | sudo tee -a /etc/ssh/sshd_config`

`echo "PasswordAuthentication yes" | sudo tee -a /etc/ssh/sshd_config`

`echo "PermitEmptyPasswords no" | sudo tee -a /etc/ssh/sshd_config`

`sudo systemctl restart sshd`

`sudo passwd root`

```makefile
sudo sed -i 's/PasswordAuthentication /#PasswordAuthentication /g' /etc/ssh/sshd_config
sudo sed -i 's/PermitRootLogin /#PermitRootLogin /g' /etc/ssh/sshd_config
sudo sed -i 's/PermitEmptyPasswords /#PermitEmptyPasswords /g' /etc/ssh/sshd_config
echo "PermitRootLogin yes" | sudo tee -a /etc/ssh/sshd_config
echo "PasswordAuthentication yes" | sudo tee -a /etc/ssh/sshd_config
echo "PermitEmptyPasswords no" | sudo tee -a /etc/ssh/sshd_config
sudo systemctl restart sshd
sudo passwd root
```

生成随机密码：

`cat /dev/urandom | tr -dc '_A-Z#\-+=a-z(0-9%^>)]{<|' | head -c 20 ; echo ''`