#!/bin/bash

# Set the MySQL root password
mysql_root_password="abc123!"

# Update system packages
sudo apt update

# Download and install Zabbix repository configuration
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update

# Install Zabbix server, frontend, and agent
sudo apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

# Install required packages
sudo apt install -y apache2 mysql-server 

# Create a new MySQL database for Zabbix
sudo mysql -u root -p"${mysql_root_password}" -e "create database zabbix character set utf8mb4 collate utf8mb4_bin;"
sudo mysql -u root -p"${mysql_root_password}" -e "create user zabbix@localhost identified by 'abc123!'"
sudo mysql -u root -p"${mysql_root_password}" -e "grant all privileges on zabbix.* to zabbix@localhost;"
sudo mysql -u root -p"${mysql_root_password}" -e "set global log_bin_trust_function_creators = 1;"

# Import Zabbix database schema
sudo zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p"${mysql_root_password}" zabbix

sudo mysql -u root -p"${mysql_root_password}" -e "set global log_bin_trust_function_creators = 0;"

# Configure Zabbix server
sudo sed -i 's/# DBPassword=/DBPassword=abc123!/' /etc/zabbix/zabbix_server.conf
sudo sed -i 's/# startVMwarecollectors=/startVMwarecollectors=0/' /etc/zabbix/zabbix_server.conf

# Restart Apache without password prompt
echo "%sudo ALL=(ALL) NOPASSWD: /usr/sbin/service apache2 restart" | sudo tee /etc/sudoers.d/zabbix_apache_restart

# Restart Zabbix server and agent without password prompt
echo "%sudo ALL=(ALL) NOPASSWD: /usr/sbin/service zabbix-server restart" | sudo tee /etc/sudoers.d/zabbix_server_restart
echo "%sudo ALL=(ALL) NOPASSWD: /usr/sbin/service zabbix-agent restart" | sudo tee /etc/sudoers.d/zabbix_agent_restart

# Enable Zabbix server and agent on boot
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2

echo "------------------------------"
echo "Zabbix installation completed!"
