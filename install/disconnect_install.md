# 离线安装OpenShift 3.10

## 目标
离线安装openshift 3.10 

## 安装环境

主机信息如下

主机 | 配置 | os | 备注
--- | --- | ---  | --- 
yum.ocp.com | 2C + 4G + 200G | RHEL7.5 | 安装节点 + 本地yum源
registry.ocp.com | 1C + 2G + 300G | RHEL7.5 | registry 
master1.ocp.com | 2C + 4G + 200G | RHEL7.5 | 
master2.ocp.com | 2C + 4G + 200G | RHEL7.5
master3.ocp.com | 2C + 4G + 200G | RHEL7.5
node1.ocp.com | 2C + 4G + 200G | RHEL7.5
node2.ocp.com | 2C + 4G + 200G | RHEL7.5
node3.ocp.com | 2C + 4G + 200G | RHEL7.5
infra1.ocp.com | 1C + 2G + 200G | RHEL7.5
infra2.ocp.com | 1C + 2G + 200G | RHEL7.5


## 安装准备

## All nodes 

### 配置hosts文件

```
cat > /etc/hosts << EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.3.50 single-master.ocp.com
192.168.3.51 single-node1.ocp.com
192.168.3.52 single-node2.ocp.com
192.168.3.53 single-infra1.ocp.com
192.168.3.54 single-infra2.ocp.com
192.168.3.53 nfs.ocp.com
192.168.3.61 registry.ocp.com
192.168.3.60 yum.ocp.com
EOF
```

echo "192.168.3.11 openshift-internal.ocp.com " >> /etc/hosts;

## yum.ocp.com

### 配置ssh key

### 下载yum文件

```
subscription-manager register
subscription-manager refresh
subscription-manager list --available --matches '*OpenShift*'
subscription-manager attach --pool=<pool_id>
subscription-manager repos --disable="*"

subscription-manager repos \
    --enable="rhel-7-server-rpms" \
    --enable="rhel-7-server-extras-rpms" \
    --enable="rhel-7-server-ansible-2.4-rpms" \
    --enable="rhel-7-server-ose-3.10-rpms"

# install required packages
sudo yum -y install yum-utils createrepo docker git

# 下载yum文件
for repo in \
rhel-7-server-rpms \
rhel-7-server-extras-rpms \
rhel-7-server-ansible-2.4-rpms \
rhel-7-server-ose-3.10-rpms
do
  reposync --gpgcheck -lm --repoid=${repo} --download_path=</path/to/repos> 
  createrepo -v </path/to/repos/>${repo} -o </path/to/repos/>${repo} 
done
```

### 创建本地yum源

```
sudo yum install httpd 

cp -a /path/to/repos /var/www/html/
chmod -R +r /var/www/html/repos
restorecon -vR /var/www/html

sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload

systemctl enable httpd
systemctl start httpd

```

### 安装openshift-ansible 
```
yum install openshift-ansible
```

## registry.ocp.com 
 
```
[root@mynuc1 reposync]# cat 03-dockerpull-310.sh 
ocptag=v3.10.45

 docker pull registry.access.redhat.com/openshift3/csi-attacher:$ocptag
 docker pull registry.access.redhat.com/openshift3/csi-driver-registrar:$ocptag
 docker pull registry.access.redhat.com/openshift3/csi-livenessprobe:$ocptag
 docker pull registry.access.redhat.com/openshift3/csi-provisioner:$ocptag
 docker pull registry.access.redhat.com/openshift3/efs-provisioner:$ocptag
 docker pull registry.access.redhat.com/openshift3/image-inspector:$ocptag
 docker pull registry.access.redhat.com/openshift3/local-storage-provisioner:$ocptag
 docker pull registry.access.redhat.com/openshift3/manila-provisioner:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-ansible:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-cli:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-cluster-capacity:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-deployer:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-descheduler:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-docker-builder:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-docker-registry:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-egress-dns-proxy:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-egress-http-proxy:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-egress-router:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-f5-router:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-haproxy-router:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-hyperkube:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-hypershift:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-keepalived-ipfailover:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-pod:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-docker-builder:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-node-problem-detector:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-recycler:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-web-console:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-node:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-control-plane:$ocptag
 docker pull registry.access.redhat.com/openshift3/registry-console:$ocptag
 docker pull registry.access.redhat.com/openshift3/snapshot-controller:$ocptag
 docker pull registry.access.redhat.com/openshift3/snapshot-provisioner:$ocptag
 docker pull registry.access.redhat.com/rhel7/etcd

 docker pull registry.access.redhat.com/openshift3/logging-auth-proxy:$ocptag
 docker pull registry.access.redhat.com/openshift3/logging-curator:$ocptag
 docker pull registry.access.redhat.com/openshift3/logging-elasticsearch:$ocptag
 docker pull registry.access.redhat.com/openshift3/logging-eventrouter:$ocptag
 docker pull registry.access.redhat.com/openshift3/logging-fluentd:$ocptag
 docker pull registry.access.redhat.com/openshift3/logging-kibana:$ocptag
 docker pull registry.access.redhat.com/openshift3/oauth-proxy:$ocptag
 docker pull registry.access.redhat.com/openshift3/metrics-cassandra:$ocptag
 docker pull registry.access.redhat.com/openshift3/metrics-hawkular-metrics:$ocptag
 docker pull registry.access.redhat.com/openshift3/metrics-hawkular-openshift-agent:$ocptag
 docker pull registry.access.redhat.com/openshift3/metrics-heapster:$ocptag
 docker pull registry.access.redhat.com/openshift3/metrics-schema-installer:$ocptag
 docker pull registry.access.redhat.com/openshift3/prometheus:$ocptag
 docker pull registry.access.redhat.com/openshift3/prometheus-alert-buffer:$ocptag
 docker pull registry.access.redhat.com/openshift3/prometheus-alertmanager:$ocptag
 docker pull registry.access.redhat.com/openshift3/prometheus-node-exporter:$ocptag
 docker pull registry.access.redhat.com/cloudforms46/cfme-openshift-postgresql
 docker pull registry.access.redhat.com/cloudforms46/cfme-openshift-memcached
 docker pull registry.access.redhat.com/cloudforms46/cfme-openshift-app-ui
 docker pull registry.access.redhat.com/cloudforms46/cfme-openshift-app
 docker pull registry.access.redhat.com/cloudforms46/cfme-openshift-embedded-ansible
 docker pull registry.access.redhat.com/cloudforms46/cfme-openshift-httpd
 docker pull registry.access.redhat.com/cloudforms46/cfme-httpd-configmap-generator
 docker pull registry.access.redhat.com/rhgs3/rhgs-server-rhel7
 docker pull registry.access.redhat.com/rhgs3/rhgs-volmanager-rhel7
 docker pull registry.access.redhat.com/rhgs3/rhgs-gluster-block-prov-rhel7
 docker pull registry.access.redhat.com/rhgs3/rhgs-s3-server-rhel7

 docker pull registry.access.redhat.com/openshift3/apb-base:$ocptag
 docker pull registry.access.redhat.com/openshift3/apb-tools:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-service-catalog:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-ansible-service-broker:$ocptag
 docker pull registry.access.redhat.com/openshift3/mariadb-apb:$ocptag
 docker pull registry.access.redhat.com/openshift3/mediawiki-apb:$ocptag
 docker pull registry.access.redhat.com/openshift3/mysql-apb:$ocptag
 docker pull registry.access.redhat.com/openshift3/ose-template-service-broker:$ocptag
 docker pull registry.access.redhat.com/openshift3/postgresql-apb:$ocptag

 docker pull registry.access.redhat.com/jboss-amq-6/amq63-openshift
 docker pull registry.access.redhat.com/jboss-datagrid-7/datagrid71-openshift
 docker pull registry.access.redhat.com/jboss-datagrid-7/datagrid71-client-openshift
 docker pull registry.access.redhat.com/jboss-datavirt-6/datavirt63-openshift
 docker pull registry.access.redhat.com/jboss-datavirt-6/datavirt63-driver-openshift
 docker pull registry.access.redhat.com/jboss-decisionserver-6/decisionserver64-openshift
 docker pull registry.access.redhat.com/jboss-processserver-6/processserver64-openshift
 docker pull registry.access.redhat.com/jboss-eap-6/eap64-openshift
 docker pull registry.access.redhat.com/jboss-eap-7/eap70-openshift
 docker pull registry.access.redhat.com/jboss-webserver-3/webserver31-tomcat7-openshift
 docker pull registry.access.redhat.com/jboss-webserver-3/webserver31-tomcat8-openshift
 docker pull registry.access.redhat.com/openshift3/jenkins-1-rhel7
 docker pull registry.access.redhat.com/openshift3/jenkins-2-rhel7
 docker pull registry.access.redhat.com/openshift3/jenkins-agent-maven-35-rhel7:$ocptag
 docker pull registry.access.redhat.com/openshift3/jenkins-agent-nodejs-8-rhel7:$ocptag
 docker pull registry.access.redhat.com/openshift3/jenkins-slave-base-rhel7
 docker pull registry.access.redhat.com/openshift3/jenkins-slave-maven-rhel7
 docker pull registry.access.redhat.com/openshift3/jenkins-slave-nodejs-rhel7
 docker pull registry.access.redhat.com/rhscl/mongodb-32-rhel7
 docker pull registry.access.redhat.com/rhscl/mysql-57-rhel7
 docker pull registry.access.redhat.com/rhscl/perl-524-rhel7
 docker pull registry.access.redhat.com/rhscl/php-56-rhel7
 docker pull registry.access.redhat.com/rhscl/postgresql-95-rhel7
 docker pull registry.access.redhat.com/rhscl/python-35-rhel7
 docker pull registry.access.redhat.com/redhat-sso-7/sso70-openshift
 docker pull registry.access.redhat.com/rhscl/ruby-24-rhel7
 docker pull registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift
 docker pull registry.access.redhat.com/redhat-sso-7/sso71-openshift
 docker pull registry.access.redhat.com/rhscl/nodejs-6-rhel7
 docker pull registry.access.redhat.com/rhscl/mariadb-101-rhel7

 docker pull registry.access.redhat.com/jboss-webserver-3/webserver30-tomcat7-openshift:latest
 docker pull registry.access.redhat.com/jboss-webserver-3/webserver30-tomcat7-openshift:1.1
```

### 安装&配置docker-distribution

```
yum -y install docker-distribution;

mkdir /etc/crts/ && cd /etc/crts
openssl req \
   -newkey rsa:2048 -nodes -keyout ocp.com.key \
   -x509 -days 3650 -out ocp.com.crt -subj \
   "/C=CN/ST=GD/L=SZ/O=Global Security/OU=IT Department/CN=*.ocp.com"

vi /etc/docker-distribution/registry/config.yml
-----------------------------------------------
http:
   addr: :443
   tls:
       certificate: /etc/crts/ocp.com.crt
       key: /etc/crts/ocp.com.key
-----------------------------------------------

systemctl daemon-reload
systemctl enable docker-distribution;
systemctl start docker-distribution;
netstat -ntlp  | grep registry
```

### push 镜像到registry

```
docker images |egrep "redhat.com|docker.io"|awk '{print "docker tag "$3" "$1":"$2}'| \
sed -e s/access.redhat.com/ocp.com/| \
sed -e s/docker.io/registry.ocp.com/| \
xargs -i bash -c "{}"

docker images |grep "ocp.com"| \
awk '{print "docker push "$1":"$2}'| \
xargs -i bash -c "{}"

docker images |egrep "redhat.com|docker.io"| awk '{print "docker rmi "$1":"$2}'| \
xargs -i bash -c "{}"
```

### 安装配置防火墙

```

cp /etc/sysconfig/iptables /etc/sysconfig/iptables.bak.$(date "+%Y%m%d%H%M%S");
sed -i '/.*--dport 22 -j ACCEPT.*/a\-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT' /etc/sysconfig/iptables;

systemctl restart iptables;

```

## All nodes 

### 使用本地yum
```
cat << EOF > /etc/yum.repos.d/ocp.repo
[rhel-7-server-rpms]
name=rhel-7-server-rpms
baseurl=http://yum.ocp.com/repos/rhel-7-server-rpms
enabled=1
gpgcheck=0
[rhel-7-server-extras-rpms]
name=rhel-7-server-extras-rpms
baseurl=http://yum.ocp.com/repos/rhel-7-server-extras-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.4-rpms]
name=rhel-7-server-ansible-2.4-rpms
baseurl=http://yum.ocp.com/repos/rhel-7-server-ansible-2.4-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ose-3.10-rpms]
name=rhel-7-server-ose-3.10-rpms
baseurl=http://yum.ocp.com/repos/rhel-7-server-ose-3.10-rpms
enabled=1
gpgcheck=0
EOF
```

### 更新软件 & 配置registry
```
yum -y install wget git net-tools bind-utils iptables-services bridge-utils bash-completion vim atomic-openshift-excluder atomic-openshift-docker-excluder lrzsz unzip atomic-openshift-utils;

scp registry.ocp.com:/etc/crts/ocp.com.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust extract

```

### 安装docker & 配置docker storage


```
yum -y install docker 

cp /etc/sysconfig/docker /etc/sysconfig/docker.bak.$(date "+%Y%m%d%H%M%S");
sed -i s/".*OPTIONS=.*"/"OPTIONS='--insecure-registry=172.30.0.0\/16 --selinux-enabled --log-opt max-size=1M --log-opt max-file=3'"/g /etc/sysconfig/docker;
echo "ADD_REGISTRY='--add-registry registry.ocp.com'" >> /etc/sysconfig/docker;

systemctl restart docker

reboot

```

## 安装

### single-master.ocp.com
```
ssh-keygen;

for i in  single-master.ocp.com  single-node1.ocp.com single-node2.ocp.com  single-infra1.ocp.com single-infra2.ocp.com nfs.ocp.com;
do
  ssh-copy-id $i;
done;
``` 

### 配置ansible inventory 文件
```
cat > /etc/ansible/hosts <<EOF
[OSEv3:children]
masters
nodes
etcd

# Set variables common for all OSEv3 hosts
[OSEv3:vars]
ansible_ssh_user=root
openshift_deployment_type=openshift-enterprise
oreg_url=registry.access.redhat.com/openshift3/ose-${component}:${version}
openshift_examples_modify_imagestreams=true

# uncomment the following to enable htpasswd authentication; defaults to DenyAllPasswordIdentityProvider
#openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]

# Native high availability cluster method with optional load balancer.
# If no lb group is defined installer assumes that a load balancer has
# been preconfigured. For installation the value of
# openshift_master_cluster_hostname must resolve to the load balancer
# or to one or all of the masters defined in the inventory if no load
# balancer is present.
openshift_master_cluster_method=native
openshift_master_cluster_hostname=openshift-internal.ocp.com
openshift_master_cluster_public_hostname=openshift-cluster.ocp.com

# host group for masters
[masters]
master1.ocp.com
master2.ocp.com
master3.ocp.com

# host group for etcd
[etcd]
master1.ocp.com
master2.ocp.com
master3.ocp.com

# Specify load balancer host
#[lb]
#lb.example.com

# host group for nodes, includes region info
[nodes]
master[1:3].ocp.com openshift_node_group_name='node-config-master'
node1.ocp.com openshift_node_group_name='node-config-compute'
node2.ocp.com openshift_node_group_name='node-config-compute'
node3.ocp.com openshift_node_group_name='node-config-compute'
infra1.ocp.com openshift_node_group_name='node-config-infra'
infra2.ocp.com openshift_node_group_name='node-config-infra'
EOF
```


### 安装
```
# 检查
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/prerequisites.yml

# 安装
ansible-playbook -vv /usr/share/ansible/openshift-ansible/playbooks/deploy_cluster.yml | tee /tmp/ansible.log;

```
# issues

## etcd镜像找不到
etcd:3.2.22 镜像找不到  

```
image rhel7/etcd:3.2.22 not found
```

原因 etcd 在安装包里hardcode了tag，所以会出错, bugzilla 地址：
https://bugzilla.redhat.com/show_bug.cgi?id=1619279

修改办法：
将etcd改一下tag，重新推到registry中即可


