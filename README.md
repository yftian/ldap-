Openldap数据迁移  
1.使用docker搭建Openldap  
```
# docker pull  osixia/openldap
```
```
# docker run -p 389:389 -p 689:689 --name my-openldap \
 --env LDAP_ORGANISATION="my-company" \
--env LDAP_DOMAIN="my-company.com" \
--env LDAP_ADMIN_PASSWORD="123456" \
--detach osixia/openldap
```
2.准备备份文件  
对openldap进行备份时，直接使用slapcat命令进行备份（如代码一），然后使用ldapadd还原  
会出现以下报错信息：  
ldap_add: Constraint violation (10)  
additional info: structuralObjectClass: no user modification allowed  
```
#代码一：
slapcat -v -l ldapbackup.ldif
```
原因：slapcat备份出来的ldapback.ldif中有系统自动生成的系统信息不能导入需要清除  
a.新建过滤正则表达式slapcat.regex  
```
cat >slapcat.regex <<EOF
/^creatorsName: /d
/^createTimestamp: /d
/^modifiersName: /d
/^modifyTimestamp: /d
/^structuralObjectClass: /d
/^entryUUID: /d
/^entryCSN: /d
EOF
```
b.过滤掉系统信息  
```
cat ldapback.ldif | sed -f slapcat.regex > slapdata.ldif
```
3、使用ldapadd导入  
```
ldapadd -c  -x -D "cn=admin,dc=my-company,dc=com" -w 123456 -f slapdata.ldif
```
