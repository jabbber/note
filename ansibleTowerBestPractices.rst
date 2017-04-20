==============================
Ansible Tower 最佳实践
==============================

本文以一些例子演示Ansible Tower的使用方法

从零开始配置一个作业，批量修改sudoer用户
=============================================

*前提: 已经有git server和若干可以ssh登录root用户的linux系统*

#. 建立一个git repository，如testyml，用来管理ansible play book文件，在git上提交两个文件:

    add_sudoers.yml::

        - name: Add user to sudoers
            hosts: all
            tasks:
                - name: Add user to sudoers
                    lineinfile: dest=/etc/sudoers state=present  line='{{ item }} ALL=(root) ALL' validate='visudo -cf %s'
                    with_items:  
                        - "{{ user_sudo_add }}"

    delete_sudoers.yml::

        - name: Remove user to sudoers
            hosts: all
            tasks:
                - name: Remove user to sudoers
                    lineinfile: dest=/etc/sudoers state=absent  regexp="^{{ item }}"  validate='visudo -cf %s'   
                    with_items:
                        - "{{ user_sudo_del }}"
   

#. 创建一个 **PROJECT** "Demo Project"
    #. ``SCM TYPE`` 选 ``Git``
    #. ``SCM URL`` 添加一个git repository地址，如: http://10.214.128.30:10080/root/testyml.git
    #. ``SCM UPDATE OPTIONS`` 勾选 ``Clean``, ``Delete on Update``, ``Update on Launch``
    #. 保存，然后点击"Demo Project"右侧 ``ACTIONS`` 中的 ``Start an SCM update``


#. 创建一个 **INVENTORY** "demo"
    #. 在INVENTORIES标签页Add创建NAME为demo的Inventory
    #. 点击demo，点击 ``ADD HOST`` 添加主机， 
    #. ``HOST NAME`` 写主机名
    #. 如果ansible tower所在的服务器不能解析这个hostname,那么在``VARIABLES`` 填写:: 
           
        ansible_host: 该机器的IP地址
    #. 重复上面2-4步，添加需要的主机

#. 点击右上角"settings"，创建一个 **CREDENTIALS**,填写上面 **INVENTORY** 中添加的主机的登录验证方式

#. 创建一个 **TEMPLATE** "Add Sudoers Template"
    #. ``INVENTORY`` 选 "demo",勾上"Prompt on launch"
    #. ``PROJECT`` 选 "Demo Project"
    #. ``PLAYBOOK`` 选 add_sudoers.yml
    #. ``MACHINE CREDENTIAL`` 选 "Demo Credential"，勾上"Prompt on launch"
    #. ``EXTRA VARIABLES`` 填写,勾上"Prompt on launch"::

        user_sudo_add: 
            - cuihengchun

    #. 保存
    #. delete_sudoers也按照上面的步骤

#. 执行JOB,点击 **TEMPLATES** 中 "Add Sudoers Template" 右侧 ``ACTIONS`` 中的 ``Start a job using this template``
    #. Inventory选demo
    #. Credential选Demo Credential
    #. Extra Variables按需要修改成自己需要添加的用户名
    #. 点Launch开始执行Job

#. 等待完成，查看执行结果

从CMDB同步主机列表,并过滤特定主机执行Job
================================================================================

*前提: 有一个配置好的couchdb作为cmdb，参见* :doc:`../couchdb`

#. 创建 **INVENTORY SCRIPTS**
    #. 点击右上角"settings:,添加一个 **inventory scripts**
    #. name写"couchdb-cmdb"， ``CUSTOM SCRIPT`` 使用下面的脚本::

        #!/usr/bin/env python2
		# -*- coding: utf-8 -*-
		#This is a inventory script for ansible
		#author: zhouwenjun
		#version: 1.0.2

		import requests
		import json

		#couchdb view api url
		view_url = 'http://10.214.160.113:5984/devtest/_design/server/_view/by_ip?include_docs=true'
		#分组的参考属性
		group_key = '应用项目'

		def jsondump(item):
			return json.dumps(item, sort_keys=True,indent=4).decode('unicode_escape').encode('utf-8')

		r = requests.get(view_url)

		if r.status_code != 200:
			print(r.content)
			exit(1)

		view = json.loads(r.content)
		rows = view['rows']

		result = {}
		result['_meta'] = {'hostvars':{}}
		result['null'] = []
		group_key = group_key.decode('utf-8')
		for row in rows:
			doc = row['doc']
			result['_meta']['hostvars'][row['id']] = {'ansible_host':row['key']}
			if group_key not in doc:
				result['null'].append(row['id'])
			elif doc[group_key] not in result:
				result[doc[group_key]] = [row['id']]
			else:
				result[doc[group_key]].append(row['id'])

		print(jsondump(result))

#. 同步主机列表到 **inventory**
    #. 回到 **INVENTORIES** 标签，选择一个inventory，如"demo"
    #. 添加一个 ``GROUP``,如"devtest"
    #. ``SOURCE`` 选择"custom script"
    #. ``CUSTOM INVENTORY SCRIPT`` 选择刚才添加的couchdb-cmdb
    #. ``UPDATE OPTIONS`` 勾上 ``Overwrite``, ``Overwrite Variables``, ``Update on Launch``
    #. 保存
    #. 点击"devtest"这个groups上的"start sync process"，等待同步完成
    #. 点击"devtest",可以看到同步到了一些HOSTS和GROUPS

#. 创建或者修改一个 **TEMPLATE**,如"hello world"，主要是要勾上 **LIMIT** 上的 ``Prompt on lannch``

#. 从这个template执行Job，inventory选"demo",LIMIT填写"CMDB测试"(假设前面同步主机列表后，"demo"下的"devtest"中同步到一个group叫做"CMDB测试"

#. LAUNCH,等待执行结果

