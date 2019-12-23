[TOC]
一个问题的产生与解决:
```
#虚拟环境中的操作
(my_env) $python3 manage.py test_permission
这个用户没有访问者权限！
#添加代码:
##147讲
@manager.option('-e','--email',dest='email')
@manager.option('-n','--name',dest='name')
def add_user_to_role(email,name):
    user = CMSUser.query.filter_by(email=email).first()
    if user:
        role = CMSRole.query.filter_by(name=name).first()
        if role:
            role.users.append(user)
            db.session.commit()
            print('用户添加到角色成功！')
        else:
            print('没有这个角色：%s'%role)
    else:
        print('%s邮箱没有这个用户!'%email)
=============================================================
#回到虚拟环境中:
(my_env) $python3 manage.py add_user_to_role -e 596414331@qq.com -n 访问者
用户添加到角色成功！
(my_env) $python3 manage.py add_user_to_role -e 596414331@qq.com -n 开发者
用户添加到角色成功！
#回到mysql里面
mysql> show databases;
+-----------------------+
| Database |
+-----------------------+
| information_schema |
| alembic_demo |
| flask_alembic_demo |
| flask_migrate_demo |
| flask_restful_demo |
| flask_script_demo |
| flask_sqlalchemy_demo |
| icbc_demo |
| mysql |
| performance_schema |
| sys |
| zlbbs |
+-----------------------+
12 rows in set (0.00 sec)

mysql> use zlbbs;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-----------------+
| Tables_in_zlbbs |
+-----------------+
| alembic_version |
| cms_role |
| cms_role_user |
| cms_user |
+-----------------+
4 rows in set (0.00 sec)

mysql> select * from cms_role_user;
+-------------+-------------+
| cms_role_id | cms_user_id |
+-------------+-------------+
| 1 | 3 |
| 4 | 3 |
+-------------+-------------+
2 rows in set (0.00 sec)

mysql> select * from cms_user;
+----+----------+------------------+---------------------+------------------------------------------------------------------------------------------------+
| id | username | email | join_time | _password |
+----+----------+------------------+---------------------+------------------------------------------------------------------------------------------------+
| 1 | richard | 123456@qq.com | 2019-12-19 13:50:49 | |
| 2 | richard | xxx@qq.com | 2019-12-19 14:15:59 | pbkdf2:sha256:150000$59zHwBdv$878eeb2b1e8f94cb834f6ef67a023917692bc339fd2dcffafbc30883fc8ade89 |
| 3 | admin | 596414331@qq.com | 2019-12-19 23:05:32 | pbkdf2:sha256:150000$eX7lLRCl$fd9551de2851d0eea4a9987ee903166dd89b8c1b78df894fc9b5e658a768e264 |
| 4 | admin | richard@126.com | 2019-12-20 19:59:36 | pbkdf2:sha256:150000$oup9Dihl$d61a278c14693bced36a6314e21a5d79deebbc939666c68e24065904e9274268 |
+----+----------+------------------+---------------------+------------------------------------------------------------------------------------------------+
4 rows in set (0.00 sec)

#回到虚拟环境
(my_env) $python3 manage.py test_permission
这个用户没有访问者权限！
(my_env) $python3 manage.py test_permission
这个用户没有访问者权限！
发现不对啊,代码基本没有问题:
看manage.py:
@manager.command
def test_permission():
    user = CMSUser.query.first()//这里可能有问题！！！
    # if user.has_permission(CMSPermission.VISITOR):
    if user.is_developer:

        print('这个用户有访问者的权限！')
    else:
        print('这个用户没有访问者权限！')

#回到mysql里面
mysql> delete from cms_user where email='123456@qq.com';
Query OK, 1 row affected (0.01 sec)

mysql> select * from cms_user;
+----+----------+------------------+---------------------+------------------------------------------------------------------------------------------------+
| id | username | email | join_time | _password |
+----+----------+------------------+---------------------+------------------------------------------------------------------------------------------------+
| 2 | richard | xxx@qq.com | 2019-12-19 14:15:59 | pbkdf2:sha256:150000$59zHwBdv$878eeb2b1e8f94cb834f6ef67a023917692bc339fd2dcffafbc30883fc8ade89 |
| 3 | admin | 596414331@qq.com | 2019-12-19 23:05:32 | pbkdf2:sha256:150000$eX7lLRCl$fd9551de2851d0eea4a9987ee903166dd89b8c1b78df894fc9b5e658a768e264 |
| 4 | admin | richard@126.com | 2019-12-20 19:59:36 | pbkdf2:sha256:150000$oup9Dihl$d61a278c14693bced36a6314e21a5d79deebbc939666c68e24065904e9274268 |
+----+----------+------------------+---------------------+------------------------------------------------------------------------------------------------+
3 rows in set (0.00 sec)

mysql> delete from cms_user where email='xxx@qq.com';
Query OK, 1 row affected (0.00 sec)

mysql> select * from cms_user;
+----+----------+------------------+---------------------+------------------------------------------------------------------------------------------------+
| id | username | email | join_time | _password |
+----+----------+------------------+---------------------+------------------------------------------------------------------------------------------------+
| 3 | admin | 596414331@qq.com | 2019-12-19 23:05:32 | pbkdf2:sha256:150000$eX7lLRCl$fd9551de2851d0eea4a9987ee903166dd89b8c1b78df894fc9b5e658a768e264 |
| 4 | admin | richard@126.com | 2019-12-20 19:59:36 | pbkdf2:sha256:150000$oup9Dihl$d61a278c14693bced36a6314e21a5d79deebbc939666c68e24065904e9274268 |
+----+----------+------------------+---------------------+------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

#再回到虚拟环境里
(my_env) $python3 manage.py test_permission
这个用户有访问者的权限！
(my_env) $python3 manage.py test_permission
这个用户有访问者的权限！
(my_env) $
正常了！

```