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
