# Practice Exam for RHCSA-EX200

This is a sample of RHCSA-EX200 exam that I’ve created to prepare for my exam. As with the real exam, no perfect answers to the sample exam questions will be provided, but more or less correct and accurate. Remember you want to cover all topics first before start dealing with questions.

## Requirements

There are 18 questions in total could be more or less.
You will be given 2 RHEL 8 virtual machines which you need to configure properly to be able to successfully complete all questions and pass the exam.

One VM will be configured as an Server-A and another would be Server-B. there will be a repository VM which will enable you to install packages to your VMs. The following FQDNs will be used throughout the sample exam.

| FQDN                        | Description | IP Addresses   | Network mask  |
| --------------------------- | ----------- | -------------- | ------------- |
| node1.domain250.example.com | node1       | 172.25.250.100 | 255.255.255.0 |
| node2.domain250.example.com | node2       | 172.25.250.101 | 255.255.255.0 |

## Lab Setup

> you can create the lab setup manually, but instead i've `Vagrantfile` which you can use inorder to create this setup, please go to this website for more information regarding lab setup https://github.com/rdbreak/rhcsa8env

## Question:1

Configure your Host Name, IP Address, Gateway and DNS.

- Host name: `mars.domain250.example.com`
- IP Address: `172.25.250.100/24`
- Network Mask: `255.255.255.0`
- Gateway: `172.25.250.254`
- DNS: `172.25.250.254`

## Answer:1

> setting up hostname

```shell
[root@Server-A ~]# vim /etc/hostname
```

or

```shell
[root@Server-A ~]# hostnamectl set-hostname <hostname>
```

> setting ipv4/GW/DNS

```shell
[root@Server-A ~]# nmcli connection modify enp0s3 autoconnect yes ipv4.method manual ipv4.addresses 172.25.250.100 ipv4.gateway 172.25.250.254 ipv4.dns 172.25.250.254
```

or

```shell
[root@Server-A ~]# vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
IPADDR=172.25.250.100
GATEWAY=172.25.250.254
DNS1=172.25.250.254
```

## Question:2 Configure your system to use the default repository

YUM repositories are already available from `http://foundation0.ilt.example.com/dvd/BaseOS` and `http://foundation0.ilt.example.com/dvd/AppStream`

- Configure your system to use these locations as default repository

## Answer:2 Configure your system to use the default repository

1. Install the yum-config-manager installation package

```shell
[root@clear ~]# rpm -ivh http://foundation0.ilt.example.com/dvd/BaseOS/Packages/yum-utils-4.0.12-3.el8.noarch.rpm
```

2. repository file download and install

```shell
# yum-config-manager command could be helpful to set a local repository quickly
# yum-config-manager -h command to look for some help
```

```shell
[root@clear ~]# yum-config-manager --add-repo http://foundation0.ilt.example.com/dvd/BaseOS
Adding repo from: http://foundation0.ilt.example.com/dvd/BaseOS

[root@clear ~]# yum-config-manager --add-repo http://foundation0.ilt.example.com/dvd/AppStream
Adding repo from: http://foundation0.ilt.example.com/dvd/AppStream
```

3. Check if the installation is successful

```shell
[root@clear ~]# cd /etc/yum.repos.d/
[root@clear yum.repos.d]# ls
foundation0.ilt.example.com_dvd_AppStream.repo  foundation0.ilt.example.com_dvd_BaseOS.repo
```

4. you should also ensure some parameters are correct

```shell
[root@clear yum.repos.d]# vim foundation0.ilt.example.com_dvd_BaseOS.repo
# enabled=1
# gpgcheck=0
```

## Question:3 Debug SELinux (service)

A web server running on a non-standard port 82 is having trouble serving content. Debug and resolve the issue as necessary so that the following conditions are met:

- The web server on the system is able to serve all existing HTML files in `/var/www/html` (note: do not delete or otherwise alter the contents of existing files)

- The web server serves this content on port 82

- The web server starts automatically at system startup

## Answer:3 Debug SELinux (service)

- semanage command is used to query and modify the security context of the SELinux default directory

- apache belongs to `httpd` service

1. View httpd service status

```shell
[root@node1 ~]# systemctl status httpd
Active: failed (Result: exit-code)
```

2. View the security context of the HTML file

```shell
ls -Z  #prints the security context of the file
```

```shell
[root@node1 html]# ls -Z /var/www/html/*
system_u:object_r:default_t:s0 /var/www/html/file1
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/file3
```

3. Modify the security context of the original `/var/www/html/file1` file

```shell
man semange fcontext
# parameters to look for -a is replaced by -m (modify)
```

```shell
[root@node1 html]# semanage fcontext -m -t httpd_sys_content_t "/var/www/html/file1"
```

4. Refresh the security context

```shell
[root@node1 html]# restorecon -R -v /var/www/html/file1
Relabeled /var/www/html/file1 from system_u:object_r:default_t:s0 to system_u:object_r:httpd_sys_content_t:s0
```

5. Use semanage to release port 82

```shell
# man semanage port
[root@node1 ~]# semanage port -a -t http_port_t -p tcp 82
```

6. Check whether port 82 is allowed

```shell
# man semanage port
[root@node1 ~]# semanage port -l | grep http
http_port_t tcp 82, 80, 81, 443, 488, 8008, 8009, 8443, 9000
```

7. Restart the httpd service, set the boot to start automatically, and check whether the service is enabled

```shell
[root@node1 ~]# systemctl restart httpd
[root@node1 ~]# systemctl enable httpd
[root@node1 ~]# systemctl status httpd
```

8. Access verification

```shell
[root@node1 html]# curl http://172.25.250.100:82/file{1..3}
```

## Question:4 create user account

Create the following users, groups, and group memberships:
A group named sysmgrs

- User natasha , who also belongs to `sysmgrs` as a secondary group
- User harry , who also belongs to `sysmgrs` as a secondary group
- User sarah , does not have access to an interactive shell on the system and is not a member of `sysmgrs`
- passwords for natasha , harry and sarah should all be `flectrag`

## Answer:4 create user account

```shell
[root@clear ~]# groupadd sysmgrs
[root@clear ~]# useradd natasha -G sysmgrs
[root@clear ~]# useradd harry  -G sysmgrs
[root@clear ~]# useradd sarah --shell /sbin/nologin
[root@clear ~]# echo "flectrag" | passwd --stdin natasha
Changing password for user natasha.
passwd: all authentication tokens updated successfully.
[root@clear ~]# echo "flectrag" | passwd --stdin harry
Changing password for user harry.
passwd: all authentication tokens updated successfully.
[root@clear ~]# echo "flectrag" | passwd --stdin sarah
Changing password for user sarah.
passwd: all authentication tokens updated successfully.

Check connectivity
[root@node1 ~]# ssh natasha@localhost
[harry@node1 ~]$ ssh sarah@localhost
[harry@node1 ~]$ ssh sarah@localhost	#no interactive shell allocated for sarah /sbin/nologin
```

## Question:5 Configure a cron job (service)

Configure a cron job that runs every 2 minutes and executes the following command:

- logger "EX200 in progress", run as user natasha

## Answer:5 Configure a cron job (service)

```shell
1.check if cron service is currently running
[root@node1 ~]# systemctl status crond.service
Active: active (running)

2. Edit scheduled tasks
[root@node1 ~]# crontab -u natasha -e
[root@node1 ~]# crontab -u natasha -l
*/2 * * * * logger "EX200 in progress"

3. enable and restart service
[root@node1 ~]# systemctl enable crond.service

4.Check if log message are being loggged
#verify
[root@node1 ~]# grep EX200 /var/log/messages
node1 natasha[28082]: EX200 in progress
node1 natasha[28086]: EX200 in progress
node1 natasha[28097]: EX200 in progress
```

## Question:6 Create a collaboration directory

- `/home/managers` with the following characteristics:

- The group permissions for `/home/managers` are `sysmgrs`.
- The directory should be `read`, `write`, and `accessible by members of sysmgrs`, but not by `any other user`. (Of course, the root user has access to all files and directories on the system).
- Files created in `/home/managers` automatically set group ownership to the `sysmgrs group`.

## Answer:6 Create a collaboration directory

```shell
Check if directory exists
[root@node1 ~]# ll -d /home/managers

1. Create the specified directory file
[root@node1 ~]# mkdir /home/managers

2.Modify the group permission of `/home/managers` to sysmgrs

[root@node1 ~]# ll -d /home/managers/
drwxr-xr-x. 2 root root 6 May 14 18:16 /home/managers/

[root@node1 ~]# chown root:sysmgrs /home/managers/

[root@node1 ~]# ll -d /home/managers/
drwxr-xr-x. 2 root sysmgrs 6 May 14 18:16 /home/managers/

3. Modify the directory file and the permissions of the group to which it belongs
[root@node1 ~]# chmod 070 /home/managers/
[root@node1 ~]# chmod g=rwx,o=- /home/managers

[root@node1 ~]# ll -d /home/managers/
d---rwx---. 2 root sysmgrs 6 May 14 18:16 /home/managers/

4. Set special permissions for the `/home/managers` directory so that its subdirectories inherit the group of the parent directory
[root@node1 ~]# chmod g+s /home/managers/

[root@node1 ~]# ll -d /home/managers/
d---rws---. 2 root sysmgrs 6 May 14 18:16 /home/managers/

5. Verify the effect of g+s permission
[root@node1 managers]# touch file
[root@node1 managers]# ll
total 0
-rw-r--r--. 1 root sysmgrs 0 May 14 18:27 file
```
