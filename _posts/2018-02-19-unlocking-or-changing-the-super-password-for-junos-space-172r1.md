---
layout: post
title: "Unlocking or changing the super password for Junos Space 17.2R1"
description: "In the 17.2R1 release for Junos Space, the process for unlocking or changing the password for the super account has changed."
#thumb_image: "documentation/sample-image.jpg"
tags: [juniper, guides]
---
## Introduction
For Junos Space and its associated applications, if you have forgotten the password to the __super__ account (or have locked yourself out of the account), you can reset the password and/or unlock the account by modifying the super user's entry in the `build_db` DB of MySQL; via the Unix CLI in Space's "debug" mode.

However in release 17.2R1, the default password of the __jboss__ user (which is required in order to access the MySQL CLI in the first place) has changed. As far as I can tell, Juniper has not documented this anywhere.

Previously, you would access the MySQL CLI using the user __jboss__ and the password __netscreenos__:

{% highlight shell %}
[root@space-005056a51a72 ~]# mysql -u jboss -pnetscreenos build_db
...
...
Welcome to the MySQL monitor.  Commands end with ; or \g.
Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.
mysql>
{% endhighlight %}

In Space 17.2R1, the same credentials do not work:

{% highlight shell %}
[root@space-005056a51a72 ~]# mysql -u jboss -pnetscreenos build_db
ERROR 1045 (28000): Access denied for user 'jboss'@'localhost' (using password: YES)
{% endhighlight %}

So if you've locked your __super__ user account or forgotten its password, what do you do now?

## New SQL password
It seems as of 17.2, during installation, Space will automatically generate a random password for the __jboss__ user of the MySQL database.

Lucky for us (?), Space also stores this generated password in plaintext (!! 😱) for us to reference.

The passwords are located at: `/etc/sysconfig/JunosSpace/pwd`

Dumping them to screen, we can copy the password for the __jboss__ user:

{% highlight shell %}
[root@space-005056a51a72 ~]# cat /etc/sysconfig/JunosSpace/pwd

mysql.repUser=-BkGJCdQSEktjxIVRlF4rBOx9g3LHgCM
mysql.repAdmin=p8AGI_-jAbsNlKq4cnnSVb0rh1aeOLO-
postgres.postgres=postgres
postgres.opennms=opennms
postgres.replication=replication
cassandra.jboss=netscreen
jboss.admin=LJrZaVdj++++50395922
mysql.root=BCiIAlkj5_NZj83VQpuX4E2oSRBHTlWA
mysql.jboss=AEYZIMccv0Uq8ct_WoSgbo0BB4Wwu8RH
{% endhighlight %}

You want the `mysql.jboss=` string, which in the example above is: `AEYZIMccv0Uq8ct_WoSgbo0BB4Wwu8RH`

__This is your password to authenticate the MySQL jboss user!__

You can now access the MySQL CLI:

{% highlight shell %}
[root@space-005056a51a72 ~]# mysql -u jboss -p<YOU-JBOSS-PASSWORD> build_db
...
...
Welcome to the MySQL monitor.  Commands end with ; or \g.
Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.
mysql>
{% endhighlight %}

Replace the `<YOUR-JBOSS-PASSWORD>` with the string retrieved above. You will now have access to the CLI.

## Fixing the super account
Now that we have the password to access our MySQL CLI, we can finally unlock or reset the __super__ account to regain access to our Space applications.


### Unlocking the super account
If the super account is __locked__, you can unlock it from the MySQL CLI. Enter the following two DB queries/commands at the `mysql>` shell prompt:

{% highlight shell %}
select * from USER_IP_ADDRESS where user_id in (select id from USER where name LIKE '%super%')\G
{% endhighlight %}

{% highlight shell %}
DELETE FROM USER_IP_ADDRESS where user_id in (select id from USER where name LIKE '%super%');
{% endhighlight %}

The super account should now be unlocked.

### Changing the super account password
If you have forgotten the password to the super account, you can change it via the MySQL CLI. Enter the following two DB queries/commands at the `mysql>` shell prompt:

{% highlight shell %}
update USER set password="<YOUR-NEW-SUPER-PASSWORD>" where name="super";
{% endhighlight %}

Replace the `<YOUR-NEW-SUPER-PASSWORD>` with the new password you want to assign to the __super__ user.
