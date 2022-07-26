## **Tomcat 开启 host-manager和manager**

> **访问 host-manager**
>
> /usr/local/bttomcat/tomcat8/webapps/host-manager/META-INF/context.xml

```xml
  <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
```

> **访问 manager**
>
> /usr/local/bttomcat/tomcat8/webapps/manager/META-INF /context.xml

```xml
  <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
```

> **角色设定**
>
> /usr/local/bttomcat/tomcat8/conf/tomcat-users.xml

```xml
<!--
  <role rolename="tomcat"/>
  <role rolename="role1"/>
  <user username="tomcat" password="<must-be-changed>" roles="tomcat"/>
  <user username="both" password="<must-be-changed>" roles="tomcat,role1"/>
  <user username="role1" password="<must-be-changed>" roles="role1"/>
-->
<role rolename="admin-gui"/>
<role rolename="manager-gui"/>
<role rolename="manager-jmx"/>
<role rolename="manager-script"/>
<role rolename="manager-status"/>
<user username="admin" password="123456" roles="admin-gui,manager-gui,manager-jmx,manager-script,manager-status"/>
<!--访问的用户名和密码-->
</tomcat-users>
```

