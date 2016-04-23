---
title: 配置MySQL主从复制
date: 2016-04-22 11:09:18
categories:
- Development
tags:
- mysql
- MySQL
- MySQL主从配置
---


1. MySQL 服务器:  
	
		10.224.144.135（主)
		10.224.144.138（从）
			
2. 配置主服务器：编辑/etc/my.cnf

		# vi /etc/my.cnf
		#add for replication
		log-bin=/var/lib/mysql/log-bin
		server-id=1
		binlog-do-db = test
		binlog-do-db = CLOPSDB
		binlog-ignore-db = mysql
		innodb_flush_log_at_trx_commit=1
		sync_binlog=1
		#add for replication
		relay-log=/var/lib/mysql/relay-bin
		relay-log-index=/var/lib/mysql/relay-bin.index
		log_slave_updates = 1
		auto_increment_increment = 2
		auto_increment_offset = 1
		
3. 保存退出 ，在master机上为slave机添加一同步帐号

		#mysql –u root –p 
		Paswod:
		
		>grant replication slave on *.* to 'clopsrepl'@'10.224.144.138' identified by 'cscocmse'; 
		>quit
<!-- more -->

4. 重启MySQL服务

		#service mysql restart
		
5. 为从服务器指定主服务器：

		# mysql -u root -p
		Enter password:

		>CHANGE MASTER TO MASTER_HOST='10.224.144.135', MASTER_USER='clopsrepl', MASTER_PASSWORD='cscocmse';
		>start slave;
		
6. 如果要设置master-master  

		#那么在10.224.144.138上执行
		>grant replication slave on *.* to 'clopsrepl'@'10.224.144.135' identified by 'cscocmse'; 
		>quit
		
		#再在10.224.144.135上执行：
		>CHANGE MASTER TO MASTER_HOST='10.224.144.138', MASTER_USER='clopsrepl', MASTER_PASSWORD='cscocmse';
		>start slave;
  					


