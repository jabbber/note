##############################
CouchDB CMDB 使用手册
##############################

WEBUI
=======================
http://10.214.160.113:5984/_utils

REST API 常用操作
=======================

查看数据库状态::

    curl http://10.214.160.113:5984/

查看所有的库::

    curl -X GET http://10.214.160.113:5984/_all_dbs

查看"devtest"库的状态::

    curl -X GET http://10.214.160.113:5984/devtest

查看"devtest"库的所有数据(doc)::

    curl -X GET http://10.214.160.113:5984/devtest/_all_docs

查看一条id为cdvl-bkin-a01的数据::

    curl -X GET http://10.214.160.113:5984/devtest/cdvl-bkin-a01

创建用户::

    curl -X PUT 'http://admin:admin@10.214.160.113:5984/_users/org.couchdb.user:zhouwenjun' -d '{"name": "zhouwenjun", "password": "123456", "roles": ["admins"], "type": "user"}'

添加/修改一条数据(doc)::

    curl -X PUT http://zhouwenjun:123456@10.214.160.113:5984/devtest/cdvl-bkin-a01 -d \
    '{
        "CPU": "8C", 
        "主IP": "10.214.160.113", 
        "主机名": "cdvl-bkin-a01",
        "供应商": "", 
        "供应商联系电话": "",
        "保修开始": "",
        "保修级别": "",
        "保修结束": "",
        "内存": "8G", 
        "到货时间": "",
        "品牌": "", 
        "型号": "", 
        "增加内存": "",
        "增加配件": "",
        "备注": "", 
        "带外管理IP": "",
        "序列号": "", 
        "应用项目": "", 
        "操作系统": "CentOS7.2",
        "数量": "", 
        "机房模块": "",
        "机柜": "",
        "电源": "", 
        "硬盘": "", 
        "结束U位": "",
        "网卡": "", 
        "联系人": "zhouwenjun",
        "设备U高": "", 
        "设备名称": "",
        "设备类型": "",
        "起始U位": "",
        "_rev":"4-675596dfcb3bcf5f8ee3e40ed595fa9e"
    }'

**注**:"_rev"为couchdb内部字段，由数据库自动创建，添加新数据(doc)时不用写，修改现有数据时需要指定"_rev"，否则会报错：
    `{"error":"conflict","reason":"Document update conflict."}`

查看"devtest"库的所有server的数据::

    curl 'http://10.214.160.113:5984/devtest/_design/server/_view/by_ip?include_docs=true'

对接到 ansible
====================

ansible可以使用动态主机脚本从couchdb获取主机列表::

    ansible -i inventory_couchdb.py --list-hosts KVM

**附件**: :download:`inventory_couchdb.py <_static/couchdb/inventory_couchdb.py>`.

附件脚本使用前需要配置 ``view_url`` 和 ``gruop_key`` 参数::

    #couchdb view api url
    view_url = 'http://10.214.160.113:5984/devtest/_design/server/_view/by_ip?include_docs=true'
    #分组的参考属性
    group_key = '应用项目'

导入数据
===================

可以使用脚本批量导入excel中的数据:

    #. 将excel打开，选择另存为 **csv utf-8** 格式，例如 ``hostlist.csv``

    #. 运行脚本::

        ./csv2couchdb.py hostlist.csv http://zhouwenjun:123456@10.214.160.113:5984/devtest

**附件**: :download:`csv2couchdb.py <_static/couchdb/csv2couchdb.py>`


