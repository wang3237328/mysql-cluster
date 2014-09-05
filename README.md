mysql-cluster
=============

use the puppetlabs-mysql to achieve the HA for mysql . It can also be used by openstack 
根据puppetlabs-mysql修改的可以实现mysql-cluster并兼容puppetlabs-openstack的模块（为测试）

&&守护进程mysql_safe开启的条件是mgm管理进程开启并且所有节点的ndbd数据节点初始化成功，所以用puppet部署时 我是在所需节点上同时部署来解决这个问题
实际的测试环境是什么情况不得而知

&&在官网http://dev.mysql.com/downloads/cluster/#downloads 上下载合适的mysql-cluster版本
并更名为mysql-cluster.tar.gz 保存于/files

&&在manifests/init.pp manifests/init.pp中添加了is_mgm参数来判断节点是否为mgm管理节点并在/manifests/server.pp中调用

&&修改了manifests/init.pp manifests/init.pp中datadir basedir参数（datadir=/usr/local/mysql/data basedir=/usr/local/mysql） 
因为 ndbd和mysql_safe的默认初始化文件夹是/usr/local/mysql

&&/files/config.ini配置文件 用于配置管理需要cluster的节点 根据实际ip修改

[ndb_mgmd]

hostname=192.168.0.160   //管理节点

datadir=/var/lib/mysql-cluster

[ndbd]      

hostname=192.168.0.160

datadir=/var/lib/mysql

[ndbd]

hostname=192.168.0.159

datadir=/var/lib/mysql

[mysqld]

hostname=192.168.0.160

[mysqld]

hostname=192.168.0.159

[mysqld]   //有多少节点 就需要多少空的[mysqld]，否则初始化ndbd时会报错

[mysqld]
&&在openstack/manifests/profire/mysql.pp中

  class { '::mysql::server':
    root_password                => $::openstack::config::mysql_root_password,
    is_mgm                   => (true or false),     //添加
    restart                      => true,
    override_options             => {
      'mysqld'                   => {
        'bind_address'           => $::openstack::config::controller_address_management,
        'default-storage-engine' => 'innodb',
      }
    }
  }
&&mysql模块中删除修改的部分是 /manifests/server文件夹里的模块
