---
layout: post
title: Getting Docker CE on RHEL
# image: /img/DSC_2623.png
tags: [docker, rhel]
# published: false
---
Hopefully this helps someone (me next time too), but installing Docker CE on RHEL (no, it's not officially supported, but hey - we're IT professionals, who needs support :)

To install Docker CE on RHEL:
```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum makecache fast
sudo yum -y install docker-ce
sudo usermod -aG docker <username>
```

You may see this error pop up:

```
[root@ip ~]# yum -y install docker-ce
Loaded plugins: amazon-id, rhui-lb, search-disabled-repos
Resolving Dependencies
--> Running transaction check
---> Package docker-ce.x86_64 0:17.09.0.ce-1.el7.centos will be installed
--> Processing Dependency: container-selinux >= 2.9 for package: docker-ce-17.09.0.ce-1.el7.centos.x86_64
--> Processing Dependency: libltdl.so.7()(64bit) for package: docker-ce-17.09.0.ce-1.el7.centos.x86_64
--> Running transaction check
---> Package docker-ce.x86_64 0:17.09.0.ce-1.el7.centos will be installed
--> Processing Dependency: container-selinux >= 2.9 for package: docker-ce-17.09.0.ce-1.el7.centos.x86_64
---> Package libtool-ltdl.x86_64 0:2.4.2-22.el7_3 will be installed
--> Finished Dependency Resolution
Error: Package: docker-ce-17.09.0.ce-1.el7.centos.x86_64 (docker-ce-stable)
           Requires: container-selinux >= 2.9
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```

From [Github](https://github.com/docker/for-linux/issues/21#issuecomment-314568863):
```
For AWS: sudo yum-config-manager --enable rhui-REGION-rhel-server-extras
For Azure: sudo yum-config-manager --enable rhui-rhel-7-server-rhui-extras-rpms
For on-prem: sudo yum-config-manager --enable extras
```
YMMV, but you might need to figure out **WHAT** your extras repo is called for your organization.
