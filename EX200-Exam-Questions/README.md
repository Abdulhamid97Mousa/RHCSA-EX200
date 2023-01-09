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
   [root@clear ~]# rpm -ivh http://foundation0.ilt.example.com/dvd/BaseOS/Packages/yum-utils-4.0.12-3.el8.noarch.rpm

2. repository file download and install

# yum-config-manager

# yum-config-manager -h

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

- The web server on the system is able to serve all existing HTML files in /var/www/html (note: do not delete or otherwise alter the contents of existing files)

- The web server serves this content on port 82

- The web server starts automatically at system startup

## Answer:3 Debug SELinux (service)

#semanage command is used to query and modify the security context of the SELinux default directory
#apache belongs to `httpd` service

1. View httpd service status

```shell
[root@node1 ~]# systemctl status httpd
Active: failed (Result: exit-code)
```

2. View the security context of the HTML file

# ls -Z prints the security context of the file

```shell
[root@node1 html]# ls -Z /var/www/html/*
system_u:object_r:default_t:s0 /var/www/html/file1
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/file3
```

3. Modify the security context of the original /var/www/html/file1 file
   man semange fcontext
   -a is replaced by -m (modify)

```shell
[root@node1 html]# semanage fcontext -m -t httpd_sys_content_t "/var/www/html/file1"
```

4. Refresh the security context
   man semange fcontext

```shell
[root@node1 html]# restorecon -R -v /var/www/html/file1
Relabeled /var/www/html/file1 from system_u:object_r:default_t:s0 to system_u:object_r:httpd_sys_content_t:s0
```

5. Use semanage to release port 82
   man semanage port

```shell
[root@node1 ~]# semanage port -a -t http_port_t -p tcp 82
```

6. Check whether port 82 is allowed
   man semanage port

```shell
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
