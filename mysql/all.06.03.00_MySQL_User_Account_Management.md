


## 6.3.7 可插入身份式验证 ##

当一个客户端连接到MySQL服务器,服务器使用由客户提供用户名和客户端主机从`mysql.user`表中选择适当的账户行。服务器查询客户端使用的验证插件行,如下：

* 服务器决定通过查看从客户端身份验证插件的账户行:

	* 如果帐户行没有指定插件名称,服务器使用本地身份认证;那就是,身份验证的密码储存在账户行的`Password`栏。在MySQL服务5.5.7之前提供同样的身份验证方法,后来有了可插入式身份验证。
	
	* 如果帐户行指定了一个插件,服务器调用它来验证用户。如果服务器无法找到插件,则报错。

* 插件返回一个状态给服务器表示用户是否允许连接。

可插入身份验证允许的两个重要功能:

* 外部认证:可插入身份验证可以让客户连接MySQL服务器使用证书身份验证方法比基于本地身份验证把密码存储在`mysql.user`表更适全。例如,插件可以创建使用外部身份验证方法如PAM,Windows登录id、LDAP或Kerberos。

* 代理用户:如果一个用户被允许连接,一个身份验证插件可以给服务器返回不同的名字的用户名作为连接用户,表明连接用户是一个被代理用户。而连接过程中,作为代理用户对待，通过访问控制,有一个不同的用户的特权。实际上,一个用户扮演另一个用户。关于更多相关信息,请参见[6.3.8,“代理用户”][06.03.08]。

在MySQL上可用的身份验证插件:

* 插件执行本地身份验证密码匹配账户行的`Password`栏。`mysql_native_password`插件实现了身份验证的基于本地密码哈希加密算法。`mysql_old_password`插件实现了本地身份验证基于旧(pre-4.1)密码哈希加密算法(现在已不使用)。参考[6.3.7.2,“本地身份验证插件”][06.03.07.02],参考[6.3.7.3,The “Old” Native Authentication Plugin”][06.03.07.03]。如果帐户行没有明确使用的插件，本地身份验证默认给账号使用`mysql_native_password`,除非服务器启用`--default-authentication-plugin`选项来改变默认的插件。

* 一个插件使用sha-256的密码哈希算法执行身份验证。这个插件使用密码匹配账户行的`authentication_string`列。这个强大的加密比本地身份验证更实用。参考[6.3.7.4,“sha - 256身份验证插件”][06.03.07.04]。

* 一个插件执行外部身份验证对PAM(可插入身份验证模块), 支持MySQL服务器使用PAM身份验证MySQL用户。这个插件也支持代理用户。参考[6.3.7.5,”PAM身份验证插件”][06.03.07.05]。

* 一个插件在Windows上执行外部身份验证,让MySQL服务器启用本机Windows服务给客户端连接进行身份验证。用户登录到Windows，然后可以基于他们的环境连接到服务器MySQL客户程序,且没有另外指定一个密码。这个插件支持代理用户。参考[6.3.7.6,“Windows本地身份验证插件”][06.03.07.06]。

* 一个客户端插件发送密码给服务器没有哈希算法或加密。这个插件可以使用服务器端插件,需要客户端用户访问提供完全一样的密码。参考[6.3.7.7”,明文发送客户端身份验证插件”][06.03.07.07]。

* 一个插件对客户端进行身份验证,从本地主机通过Unix socket文件连接。参考[6.3.7.8,“套接字对等凭据的身份验证插件”][06.03.07.08]。

* 一个测试插件使用MySQL本地身份验证进行身份验证。这个插件以测试和开发的目的,有一个例子,如何写身份验证插件。参考[6.3.7.9,“测试验证插件”][06.03.07.09]。

关于如何使用身份验证插件的相关信息,参考[6.3.7.1,“身份验证插件的使用说明”][06.03.07.01]。

> **注意**
>
>关于限制使用可插入身份验证的信息,包括这连接器支持哪个插件,参考[E.9,“限制可插入身份验证”][E.09]。
>
>第三方连接器的开发人员应该读这部分的连接器来确定连接器可以利用可插入身份验证功能和将采取什么措施变得更加兼容。

如果你有兴趣编写自己的身份验证插件,参考[23.2.4.9,“写身份验证插件”][23.02.04.09]。

#### 6.3.7.1 身份验证插件的使用说明 ####

本节提供了安装和使用身份验证插件的用法说明。

一般来说,在服务器和客户端可插入身份验证会使用相应的插件,所以你使用一个给定的身份验证方法参考如下:

* 在服务器主机,安装库包含适当的服务器插件,如果有必要,服务器可以使用它来验证客户端连接。同样,在每个客户端主机,安装库包含适当的客户端插件给客户端程序使用。

* 创建MySQL账户,指定使用的身份验证插件。

* 当一个客户端连接,服务器插件告诉客户端程序,客户端插件使用身份验证。

有一个在MySQL上使用验证插件的例子(参考[6.3.7.9,“测试验证插件”][06.03.07.09])。过程与其他身份验证插件是类似的;替代相应的插件和文件名称即可。

例举验证插件的特点:

* 服务器端插件名为`test_plugin_server`。

* 客户端插件名为`auth_test_plugin`。

* 这两个插件都位于共享库的插件目录，对象文件名为`auth_test_plugin.so`(目录指定的`plugin_dir`系统变量)。文件名称在不同的系统,可能会有不同的后缀。

安装和使用身份验证插件的示例如下:

1.确保在服务器和客户端主机已安装插件库。

2.安装服务器端测试插件在服务器启动或运行时:

*  在启动时安装插件,使用`--plugin-load`选项。用这个插件加载法,每次启动服务器必须给出选项。例如,使用这些行在`my.cnf`文件中:

```sql
    [mysqld]
	plugin-load=test_plugin_server=auth_test_plugin.so
```

* 在运行时候安装该插件,使用`INSTALL PLUGIN`语句:

```sql
    mysql> INSTALL PLUGIN test_plugin_server SONAME 'auth_test_plugin.so';
```

安装插件永久性和一次性都需要做以上步骤。

3.验证插件是否安装。例如,使用`SHOW PLUGINS`:

```sql
    mysql> SHOW PLUGINS\G
	...
	*************************** 21. row ***************************
	Name: test_plugin_server
	Status: ACTIVE
	Type: AUTHENTICATION
	Library: auth_test_plugin.so
	License: GPL
```

对于其他方法来检查插件,参考[5.1.8.2”,获取服务器插件信息”][05.01.08.02]。

4.指定一个MySQL用户必须使用指定服务器插件来身份验证,`CREATE USER`创建用户在`IDENTIFIED WITH`语句后面带上插件的名称:

```sql
    CREATE USER 'testuser'@'localhost' IDENTIFIED WITH test_plugin_server;
```

5.连接到服务器使用一个客户端程序。测试插件验证同样的方式作为本地MySQL身份验证,所以提供常用的`--user`和`--password`选项,用来连接到服务器。例如:

```sql
    shell> mysql --user=your_name --password=your_pass
```

使用`testuser`用户连接,服务器看到帐户必须使用`test_plugin_server`身份验证，服务器端插件和客户端通信程序，在这种情况下必须使用`auth_test_plugin`。

如果帐户使用的身份验证方法是服务器和客户端程序默认的,服务器不需要跟客户端通信插件的使用,往返在客户机/服务器谈判是可以避免的。真实账户使用本地MySQL身份验证(`mysql_native_password`)。

`--default-auth=plugin_name`选项可以指定在mysql命令行明确指出客户端插件程序是否启用,尽管服务器将覆盖这个默认指定，如果用户帐户需要一个不同的插件。

如果客户端程序没有找到插件,指定一个`--plugin-dir=dir_name`选项指定插件的位置。

>**注意**
>
>如果您启动服务器使用`--skip-grant-tables`选项,身份验证插件没有使用，即使加载了,因为服务器没有执行客户端身份验证和允许任何客户机连接。因为这是不安全的,你可能需要使用`--skip-grant-tables`和`--skip-networking`连接使用防止远程客户端连接。

#### 6.3.7.2. 本地身份验证插件 ####

MySQL包括两个插件,实现本地认证;也就是说,认证不通过存储密码在`mysql.user`表的`Password`栏。本节描述`mysql_native_password`,它实现身份验证不通过`mysql.user`表使用本地密码哈希算法。关于`mysql_old_password`的更多信息,实现身份验证使用老的(pre-4.1)密码哈希算法,参考[6.3.7.3,”The “Old” Native Authentication Plugin”][06.03.07.03]。关于密码哈希算法的信息,请参考[6.1.2.4,“MySQL密码哈希算法”][06.01.02.04]。

`mysql_native_password`身份认证插件是向后兼容的。客户端的版本比MySQL 5.5.7老的话不支持身份验证插件，但可以使用本地身份验证协议, 所以他们可以连接到MySQL服务器使用5.5.7和5.5.7以上的版本。

下面的表显示了在服务器和客户端插件的名称。

**表6.8. MySQL本机密码身份验证插件**

<table><colgroup><col><col></colgroup><tbody><tr><td scope="row">服务器端插件名字</td><td><code class="literal">mysql_native_password</code></td></tr><tr><td scope="row">客户端插件名字</td><td><code class="literal">mysql_native_password</code></td></tr><tr><td scope="row">库对象文件的名字</td><td>None (plugins are built in)</td></tr></tbody></table>
</div>

插件在客户端和服务器端都存在: 　　

* 服务器端插件安装在服务器端,不需要显式地加载,无法被禁用除非通过卸载它。　
　
* 客户端插件在MySQL 5.5.7版本是内置在`libmysqlclient`客户端库和可用于任何程序对`libmysqlclient`版本或更新的版本。　　

* MySQL客户端程序使用`mysql_native_password`默认情况下。`--default-auth`选项可以用来明确指定插件:

```sql
    shell> mysql --default-auth=mysql_native_password ...
```

对于一般的信息可插入身份验证MySQL,参考[6.3.7,“可插入身份验证”][06.03.07]。

#### 6.3.7.3. The “Old” Native Authentication Plugin ####

MySQL包括两个插件,实现本地认证;也就是说,认证对密码存储在密码栏的`mysql.user`表。本节描述`mysql_old_password`,它实现身份验证对`mysql.user`表使用老(pre - 4.1)密码哈希算法。对于信息的`mysql_native_password`,实现身份验证使用本机密码哈希算法,看到部分[6.3.7.3,” “The Old” Native Authentication Plugin”][06.03.07.03]。对于这些密码哈希算法的信息,请参考[6.1.2.4,“MySQL密码哈希算法”][06.01.02.04]。

>注意
>
>密码,使用pre -4.1哈希算法的安全性也低于使用本机密码哈希算法的密码。这是可以避免的，pre-4.1密码在以后的MySQL版本中被放弃和不支持。

本地认证的`mysql_old_password`是向后兼容的。客户端版本低于MySQL 5.5.7不支持身份验证插件，但可以使用本地身份验证协议,所以们可以连接到MySQL服务器5.5.7和5.5.7以上版本。

下面的表显示了在服务器和客户端的插件名称。

**Table 6.9. MySQL “Old” Native Authentication Plugin**

<table><colgroup><col><col></colgroup><tbody><tr><td scope="row">服务器端插件名字</td><td><code class="literal">mysql_old_password</code></td></tr><tr><td scope="row">客户端插件名字</td><td><code class="literal">mysql_old_password</code></td></tr><tr><td scope="row">库对象文件的名字</td><td>None (plugins are built in)</td></tr></tbody></table>
</div>

插件在客户机和服务器中都存在:

* 服务器端插件是安装在服务器上,不需要显式地加载,无法被禁用只能卸载它。

* 库自MySQL的5.5.7起客户端插件是内置在`libmysqlclient`客户端库和任何程序都连接不通过`libmysqlclient`的版本或更新的版本。

* MySQL客户端程序可以使用`—--default-auth`选项来明确指定`mysql_old_password`插件:

```sql
    shell> mysql --default-auth=mysql_old_password ...
```

关于MySQL可插入身份验证的相关信息,参考[6.3.7,“可插入身份验证”][06.03.07]。

#### 6.3.7.4. sha-256认证的插件 ####

在MySQL 5.6.6中,MySQL提供身份验证插件,实现了用户帐户的密码使用sha-256哈希算法。

>**重要提示**
>
>连接到服务器的帐户使用的身份验证是`sha256_password`插件,您必须使用一个SSL连接或简单连接,使用RSA加密密码,在后面会讲到。无论哪种方式,使用`sha256_password`插件要求MySQL必须安装SSL功能.参考[6.3.9”,使用SSL的安全连接“][06.03.09]。

下面的表显示了在服务器和客户端插件的名称。

<div class="table">
<a name="idm47087463252368"></a><p class="title"><b>Table 6.10. MySQL SHA-256 Authentication Plugin</b></p>
<div class="table-contents">
<table><colgroup><col><col></colgroup><tbody><tr><td scope="row">服务端插件名称</td><td><code class="literal">sha256_password</code></td></tr><tr><td scope="row">客户端插件名称</td><td><code class="literal">sha256_password</code></td></tr><tr><td scope="row">库对象文件名称</td><td>None (plugins are built in)</td></tr></tbody></table>
</div>

服务器端安装`sha256_password`插件,不需要显式地加载,不能被禁用的只能卸载它。同样的,客户端不需要指定客户端插件的位置。

创建一个帐号,指定他认证方式使用`sha-256`密码哈希算法,参考以下步骤。　　

1.创建帐户和指定验证使用`sha256_password`插件:

```sql
    CREATE USER 'sha256user'@'localhost' IDENTIFIED WITH sha256_password;
```

2.设置`old_passwords`系统变量值为2，让`PASSWORD()`函数功能对密码字符串使用sha-256哈希算法:

```sql
    SET old_passwords = 2;
```

3.设置账户密码:

```sql
    SET PASSWORD FOR 'sha256user'@'localhost' = PASSWORD('sha256P@ss');
```

或者,启动服务器身份验证插件的默认设置为`sha256_password`。举一个例子,把这几行配置写在服务器配置文件中:

```sql
    [mysqld]
	default-authentication-plugin=sha256_password
```

这会让新账户默认使用`sha256_password`插件和设置`old_passwords`为2。因此,在创建帐户并设置密码的时候使用`CREATE USER`语句及`IDENTIFIED BY`项:

```sql
    mysql> CREATE USER 'sha256user2'@'localhost' IDENTIFIED BY 'sha256P@ss2';
	Query OK, 0 rows affected (0.06 sec)
```

在这种情况下,服务器分配`sha256_password`插件给指定帐户和密码加密使用sha-256。(另外一种结果是,创建一个帐户,使用一个不同的身份验证插件,您必须指定插件使用`CREATE USER`语句及`IDENTIFIED BY`子句,然后插件正确地设置`old_passwords`，在使用`SET PASSWORD`设置账户密码前。)

如果`old_passwords`的值而不是2,会有一个试图用sha-256设置帐户的密码的报错:

```sql
    mysql> SET old_passwords = 0;
	mysql> SET PASSWORD FOR 'sha256user'@'localhost' = PASSWORD('sha256P@ss');
	ERROR 1827 (HY000): The password hash doesn't have the expected format.
	Check if the correct password algorithm is being used with the
	PASSWORD() function.
```

了解`old_passwords` 和`PASSWORD`更多相关信息，请参考[5.1.4章,“服务器系统变量”][05.01.04],还有[12.13章，“加密和压缩功能”][12.13]。

账户在`mysql.user`表中,使用`sha-256`的密码可以被鉴别在`plugin`列的`sha256_password`栏和`sha-256`哈希算法密码在`authentication_string`栏。

MySQL可以安装使用yaSSL或OpenSSL和`sha256_password`插件可以使用分布安装也可以使用安装包。默认使用yaSSL。如果MySQL创建使用OpenSSL安装,可以使用RSA加密,并且`sha256_password`实现以下列表中附加功能。(使这些功能,你也必须遵循RSA配置过程可参考本节后面讲述。)

* 它是可能的客户端发送密码给服务器，密码在客户端连接过程中使用RSA加密,后面会描述到。

* 服务器公开了两个附加的系统变量,`sha256_password_private_key_path` 和 `sha256_password_public_key_path` 其目的是,在服务器启动的时候，数据库管理员会设置RSA的私钥和公钥的文件名。

* 服务器公开了一个状态变量,`Rsa_public_key`,显示RSA公钥值。

* `mysql`和`mysqltest`客户程序支持`--server-public-key-path`语句用于明确的指定一个RSA公钥文件。(这个选项在MySQL 5.6.6被添加`--server-public-key`和在5.6.7更名为`--server-public-key-path`。)

对于客户使用`sha256_password`插件,当连接到服务器绝不会暴露明文密码。密码怎么传输取决于SSL连接是否被使用和RSA加密是否可用:

* 如果SSL连接被使用,密码是明文发送但不能监听,因为连接是使用SSL加密。

* 如果不能使用SSL连接，但是RSA加密的可用的,密码发送使用的是未加密的连接,但是密码是rsa加密的防止窃听。当服务器收到密码,然后进行解密。使用加密方法登录防止重复攻击。

* 如果SSL连接不能被使用,并且RSA加密不可用,会导致`sha256_password`插件连接失败,因为密码不能以明文发送发送否则会暴露密码。

如前所述,只有MySQL建成使用OpenSSL，RSA密码加密才可用。这意味着,在MySQL建立使用yaSSL，只有当客户使用SSL连接访问服务器，sha-256密码才能被使用。关于使用SSL连接到服务器的更多信息,请参考[6.3.9章,“使用SSL进行安全连接”][06.03.09]。

假定MySQL构建已使用OpenSSL,下列步骤描述了在客户端连接过程中如何启用RSA密码加密:

1. 创建RSA私钥和公钥文件。登录到系统后运行这些命令，同时帐户用于运行MySQL服务器,文件归该帐户拥有:

```sql
	openssl genrsa -out mykey.pem 1024
	openssl rsa -in mykey.pem -pubout > mykey.pub
```

2. 设置key文件的访问模式。私钥应该是只有通过服务器是可读的:

```sql
    chmod 400 mykey.pem
```

公钥可以自由分发到客户端用户:

```sql
    chmod 444 mykey.pub
```

3. 在服务器配置文件中,给key文件配置适当的系统变量的名称。如果您将文件放入服务器数据目录,您不需要指定完整路径名称:

```sql
    [mysqld]
	sha256_password_private_key_path=mykey.pem
	sha256_password_public_key_path=mykey.pub
```

如果文件不是在数据目录,或使用具体位置选项值,使用完整路径名称:

```sql
    [mysqld]
	sha256_password_private_key_path=/usr/local/mysql/mykey.pem
	sha256_password_public_key_path=/usr/local/mysql/mykey.pub	
```

4.重启服务器,然后连接到它并检查`Rsa_public_key`状态变量值。与这里显示的该值会所不同,,但是应该非空的:

```sql
    mysql> SHOW STATUS LIKE 'Rsa_public_key'\G
	*************************** 1. row ***************************
	Variable_name: Rsa_public_key
		Value: -----BEGIN PUBLIC KEY-----
	MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDO9nRUDd+KvSZgY7cNBZMNpwX6
	MvE1PbJFXO7u18nJ9lwc99Du/E7lw6CVXw7VKrXPeHbVQUzGyUNkf45Nz/ckaaJa
	aLgJOBCIDmNVnyU54OT/1lcs2xiyfaDMe8fCJ64ZwTnKbY2gkt1IMjUAB5Ogd5kJ
	g8aV7EtKwyhHb0c30QIDAQAB
		-----END PUBLIC KEY-----
```

如果该值为空,服务器的key文件有问题。检查错误日志的诊断信息。

在服务器已经配置了RSA密钥文件,客户端能够使用账户连接到它，这验证了`sha256_password`插件。正如前面提到的,这种账户可以使用SSL连接(在这种情况下不使用RSA)或使用RSA加密密码的普通连接,。假设下面的讨论,没有使用SSL，连接到服务器，在客户端不需要特殊准备。例如:

```sql
    shell> mysql -u sha256user -p
	Enter password: sha256P@ss
```

使用`sha256user`尝试连接,服务器确定`sha256_password`是可用的身份验证插件和调用它。插件发现连接没有使用SSL，因而需要密码是使用RSA加密传输。它传递了RSA公钥客户端，使用它来加密的密码,并将结果返回给服务器。插件使用了RSA密钥在服务器端解密密码，并接受或拒绝连接请求，取决于密码是否正确的。

服务器发送到客户机的公钥是这是有必要的,但如果有可用的RSA公钥副本在客户端主机上,客户端保存并使用它，节省了在客户机/服务器协议往返的时间:

```sql
    shell> mysql -u sha256user -p --server-public-key-path=file_name
```

公钥值的文件命名为`--server-public-key-path`选项，在服务器端应该有一样的key值文件名为`sha256_password_public_key_path`系统变量。如果密钥文件包含一个有效公钥，但值是不正确的,发生拒绝访问的报错。如果密钥文件不包含一个有效的公钥,客户端程序不能使用它。在这种情况下,服务器发送公钥给客户端，如果没有`--server-public-key-path`选项被指定的话。

客户端用户可以得到RSA公钥的两种方式:

* 数据库管理员可以提供一份公钥文件。

* 客户端用户通过其他的方式连接到服务器，使用`SHOW STATUS LIKE 'Rsa_public_key'`语句，保存返回的key值到一个文件中。

#### 6.3.7.5. PAM身份验证插件 ####

>**注意**
>
>PAM是一个商业扩展的身份验证插件。学习更多关于商业产品(MySQL企业版),请参考http://www.mysql.com/products/.

在MySQL 5.6.10,商业发行版包括一个身份验证的MySQL插件,使MySQL服务器使用PAM(可插入身份验证模块)进行身份验证MySQL用户。Pam使一个系统使用一个标准的接口来访问各种各样的身份验证方法,比如Unix密码或LDAP目录。

PAM插件使用通过MySQL服务器传递给它的信息(如用户名、主机名、密码和身份验证字符串),加上任何方法可供PAM查找。插件检查用户凭证并返回`‘Authentication succeeded, Username is user_name’`或者`‘Authentication failed’`。

PAM身份验证插件提供了这些功能:

* 外部认证:插件支持MySQL服务器接受来自除MySQL授权表以外的用户定义的连接。

* 代理用户支持:插件可以返回到MySQL用户名不同于登录用户,基于组织外部用户在和身份验证字符串提供。这意味着插件可以返回MySQL用户定义特权外部pam身份验证用户应该有。例如,一个用户名为joe可以连接PAM和有特权的MySQL用户名developer。

下面的表显示了插件和库文件的名称。在不同的系统上文件名后缀不同。文件位置必须目录指定的plugin_dir系统变量。安装信息,请参考[6.3.7.5.1章,“安装PAM身份验证插件”][06.03.07.05.01]。

**表 6.11. MySQL PAM身份验证插件**
<div class="table-contents">
<table><colgroup><col><col></colgroup><tbody><tr><td scope="row">服务端插件名称</td><td><code class="literal">authentication_pam</code></td></tr><tr><td scope="row">客户端插件名称</td><td><code class="literal">mysql_clear_password</code></td></tr><tr><td scope="row">库对象文件名</td><td><code class="filename">authentication_pam.so</code></td></tr></tbody></table>
</div>

库文件只包含服务器端插件。客户端插件是内置的`libmysqlclient`客户端库。参考[06.03.07.07,“明文发送客户端身份验证插件”][06.03.07.07]。

服务器端PAM身份验证插件是只包括在商业发行版。这不是包括在MySQL社区分布。客户端插件,与明文服务器端插件是内置在MySQL客户端库,是包含在所有的发行版,包括社区分布。这允许客户端从任何5.6.10或新分布连接服务器,服务器端插件加载。

PAM身份验证插件已经使用Linux和Mac OS x。它需要MySQL服务器5.6.10或更新。

对于一般的信息可插入身份验证MySQL,参考[6.3.7,“可插入身份验证”][06.03.07]。代理用户的信息,请参考[6.3.8,“代理用户”][06.03.08]。

#### 6.3.7.5.1. 安装PAM身份验证插件 ####

PAM身份验证插件必须位于MySQL插件目录(目录命名这个`plugin_dir`系统变量)。如果有必要,将值设置的`plugin_dir`在服务器启动告诉服务器插件目录的位置。

要启用插件,启动服务器使用`--plugin-load`选项。例如,把在`my.cnf`中所做的以下几行你文件。如果目标文件有一个后缀不同。所以在你的系统,替代正确的后缀。

```sql
    [mysqld]
	plugin-load=authentication_pam.so
```

使用插件名称验证pam在认同条款的创建用户或GRANT语句为MySQL账户,应该经过用这个插件。

验证插件安装，检查`INFORMATION_SCHEMA.PLUGINS`表或使用`SHOW PLUGINS`语句。参考[5.1.8.2章，“获取服务器插件信息”][05.01.08.02]。

#### 6.3.7.5.2. 使用PAM身份验证插件 ####

本节描述如何使用PAM身份验证插件连接从MySQL客户端程序到服务器。它假定了服务器端插件是启用的,客户程序包括客户端插件时间上足够临近的。

注意

客户端插件与PAM插件仅发送通信服务器的密码以明文,因此它可以传递给PAM。这可能是一个安全问题在一些配置,但有必要使用服务器端PAM库。为了避免问题如果有任何可能性密码将被拦截,客户应该连接到MySQL服务器使用ssl。参考[6.3.7.7”,明文发送客户端身份验证插件”][06.03.07.07]。

指PAM身份验证插件在认同的法律条款创建用户或`GRANT`语句,使用名字验证pam。例如:

```sql
    CREATE USER user
	IDENTIFIED WITH authentication_pam
	AS 'authentication_string';
```

验证字符串指定以下类型的信息:

* PAM支持概念的“服务名称”,这是一个名称,系统管理员可以使用配置的身份验证方法为特定的应用程序。可以有几个这样的“应用程序”相关联的单个数据库服务器实例,所以选择服务的名字是离开SQL应用程序开发人员。当你定义一个帐户,应该使用PAM身份验证,指定服务的名字在身份验证字符串。

* PAM提供一种方式对于一个PAM模块返回服务器一个MySQL用户名以外的名在登录时提供。使用身份验证字符串控制之间的映射登录名和MySQL用户名。如果你想利用代理用户功能,验证字符串必须包括这种映射。

例如,如果服务的名字是mysql和用户用户组的`root PAM`组应该映射到`developer`和`data_entry`的用户,分别使用一个语句如下:

```sql
    CREATE USER user
		IDENTIFIED WITH authentication_pam
		AS 'mysql, root=developer, users=data_entry';
```

验证字符串的语法PAM身份验证插件遵循这些规则:

* 字符串包含一个PAM服务名称,选择之后是一组映射组成的列表一个或多个关键字/值对每个指定组名和一个SQL用户名:

```sql 
	pam_service_name[,group_name=sql_user_name]...
```

* 每个`group_name=sql_user_name`前面必须加上一对的逗号。

* 首尾空格内双引号被忽略。

* 结束引语`pam_service_name`, `group_name`, 和 `sql_user_name`的值不能有等号,逗号,或空格，其他值都可以。

* 如果`pam_service_name`, `group_name`, 或者 `sql_user_name`的值是引用得用双引号,值放在引号之间。这是必要的,例如,如果值包含空格字符。所有的字符都是有效的除了双引号和反斜杠(\)。字符之间,避免加上一个反斜杠。

插件解析验证字符串在每个登录检查。减少开销,保持字符串尽可能地短。

如果插件成功验证登录名称,它查找一组映射列表验证字符串,并使用它来返回一个不同的用户名到MySQL服务器基于组织外部的用户成员:

* 如果身份验证字符串不包含组映射列表,插件返回登录名。

* 如果身份验证字符串包含一组映射列表,检查每个插件`group_name=sql_user_name`列表中的一对从左到右,并试图找到一个匹配的`group_name`值在一个非mysql目录的组分配到的经过身份验证的用户和返回`sql_user_name`为它找到的第一个匹配项。如果插件没有发现匹配任何组,它将返回登录名。如果插件没有能够查找一组在一个目录,它忽将略了组映射列表,返回登录名。

以下部分描述如何设置几个验证场景使用PAM身份验证插件:

* 没有代理的用户。它使用PAM只有检查登录名和密码。每一个外部用户允许连接到MySQL服务器应该有一个匹配的MySQL账户定义使用外部PAM身份验证。身份验证可以由各种pam支持方法。讨论显示如何使用传统的Unix密码和LDAP。

PAM身份验证,当不通过代理用户或组,需要MySQL账户有相同的用户名作为Unix帐户。因为MySQL用户名被限制到16个字符(参考[6.2.2，“特权系统授权表”][06.02.02]),这限制PAM无代理验证账户的Unix名字最多16个字符。

* 仅代理登录和组映射。对于这个场景,创建一些MySQL账户定义不同的特权。(理想情况下,没有人应该通过这些直接登录。)然后定义一个默认用户验证通过PAM,使用一些用映射方案(通常通过外部组的用户),将所有外部登录到MySQL账户持有的少数权限集。任何用户,登录映射到一个MySQL账户的权限,并使用它。讨论显示了如何设置这个使用Unix密码,但其他PAM等方法LDAP可以代替现在使用的。

这些场景的变化是可能的。例如,您可以允许一些用户直接登录但需要其他人通过连接代理用户。

作出以下假设的例子。可以根据你的系统设置不同然后做一些调整。

* PAM配置目录是`/etc/pam.d`。

* PAM服务的名字是mysql,这意味着你必须设置一个PAM文件命名mysql为PAM配置目录(创建文件不存在时)。如果你使用一个不同的服务名称,文件的名字将是不同的,你必须使用AS项在`CREATE USER`语句和GRANT语句来使用一个不同的名称。

* 这个例子使用一个`antonio`的登录名和密码为`verysecr`。替换那些来验证你想验证用户。

PAM身份验证插件在初始化的时候检查是否`AUTHENTICATION_PAM_LOG`环境值设置。如果是这样,插件支持日志记录的诊断消息的标准输出。这些信息可能有利于调试pam相关问题时出现的插件执行身份验证。有关更多信息,请参考[6.3.7.5.3,”PAM身份验证插件的调试”][06.03.07.05.03]。

#### 6.3.7.5.2.1.Unix没有代理用户的密码身份验证 ####

这种身份验证场景使用PAM只有检查Unix用户登录名和密码。每一个外部用户允许连接到MySQL服务器应该有一个匹配的MySQL账户是定义使用外部PAM身份验证。

1. 验证在PAM身份验证允许您的Unix登录如`antonio`与密码`verysecret`。
 
2. 建立PAM身份验证mysql服务。把以下在`/etc/pam.d/mysql`:

```sql
	#%PAM-1.0
	auth 		include 	password-auth
	account 	include 	password-auth  
```

3.创建一个MySQL账户相同的用户名作为登录名的Unix和确定它使用PAM身份验证插件:

```sql
	CREATE USER 'antonio'@'localhost'
	IDENTIFIED WITH authentication_pam AS 'mysql';
	GRANT ALL PRIVILEGES ON mydb.* TO 'antonio'@'localhost';
```

4.尝试连接到MySQL服务器使用MySQL从客户端命令行。例如:

```sql
    mysql --user=antonio --password=verysecret mydb
```

服务器应该允许连接和下面的查询及返回输出如图所示:

```sql
    mysql> SELECT USER(), CURRENT_USER(), @@proxy_user;
	+-------------------+-------------------+--------------+
	| USER() | CURRENT_USER() | @@proxy_user			   |
	+-------------------+-------------------+--------------+
	| antonio@localhost | antonio@localhost | NULL 		   |
	+-------------------+-------------------+--------------+
```

由此可见，在没有代理的情况下，`antonio`使用了`antonio MySQL`的账户授与的特权。

#### 6.3.7.5.2.2. 不使用代理用户LDAP身份验证 ####

这种身份验证场景使用PAM只有检查LDAP用户登录名和密码。每一个外部用户允许连接到MySQL服务器应该有一个匹配的MySQL账户是定义使用外部PAM身份验证。

1. 验证在PAM身份验证允许您的Unix登录如`antonio`与密码为`verysecret`,`antonio`是一个成员的`root`或`users`组。

2. 建立PAM身份验证mysql服务。把以下在`/etc/pam.d/mysql`:

```sql
    \#%PAM-1.0
	auth include password-auth
	account include password-auth
```

3. 创建默认代理用户映射外部PAM用户代理的账户。这映射外部用户从`root PAM`组到`developer MySQL`账户和外部用户从`users PAM`组到`data_entry`MySQL账户:

```sql
    CREATE USER ''@''
	IDENTIFIED WITH authentication_pam
	AS 'mysql, root=developer, users=data_entry';
```

映射列表以下服务名称时需要您设置代理用户。否则,插件不能正确映射PAM代理用户名到组的名称。

4. 用于访问数据库代理账户创建:

```sql
    CREATE USER 'developer'@'localhost' IDENTIFIED BY 'very secret password';
	GRANT ALL PRIVILEGES ON mydevdb.* TO 'developer'@'localhost';
	CREATE USER 'data_entry'@'localhost' IDENTIFIED BY 'very secret password';
	GRANT ALL PRIVILEGES ON mydb.* TO 'data_entry'@'localhost';
```

如果你不想让任何人知道这些账户的密码,让其他用户无法使用它们直接连接到MySQL服务器。相反,希望用户将使用PAM验证,他们将使用`developer`或`data_entry`帐户由基于他们的PAM组的代理。

5. 授于PROXY[775]特权给代理帐户去代理其他账户们: 　

```sql
    GRANT PROXY ON 'developer'@'localhost' TO ''@'';
	GRANT PROXY ON 'data_entry'@'localhost' TO ''@'';
```

6. 客户端使用MySQL命令行尝试去连接到MySQL服务器。例如:

```sql
    mysql --user=antonio --password=verysecret mydb
```

服务器验证连接使用`“@”`账户。用户`antonio`的有哪些特权取决于他是哪个PAM组的成员。如果`antonio`是`root PAM`组的成员,PAM插件将映射`root`给`developer` MySQL用户名并返回这个名字到服务器。服务器验证,`“@”PROXY`特权为`developer`和特权允许连接。输出如图所示查询的返回:

```sql
    mysql> SELECT USER(), CURRENT_USER(), @@proxy_user;
	+-------------------+---------------------+--------------+
	| USER() 			| CURRENT_USER() 	  | @@proxy_user |
	+-------------------+---------------------+--------------+
	| antonio@localhost | developer@localhost | ''@'' 		 |
	+-------------------+---------------------+--------------+	
```

这表明,`antonio`使用特权授予`developer`MySQL账户,通过默认代理用户帐户连接到数据。

如果`antonio`不是`root PAM`组的成员，但是是`users`组的成员,一个类似的过程会发生,但插件映射`user`组成员的`data_entry` MySQL用户名并返回这个名字服务器。在这种情况下,`antonio`使用`data_entry` MySQL账户的特权:

```sql
    mysql> SELECT USER(), CURRENT_USER(), @@proxy_user;
	+-------------------+----------------------+--------------+
	| USER() 			| CURRENT_USER() 	   | @@proxy_user |
	+-------------------+----------------------+--------------+
	| antonio@localhost | data_entry@localhost | ''@'' 		  |
	+-------------------+----------------------+--------------+
```

#### 6.3.7.5.3. PAM身份验证插件的调试 ####

PAM身份验证插件在初始化的时候检查是否`AUTHENTICATION_PAM_LOG`环境设置值(这个值并不重要)。如果是这样,插件支持日志记录的诊断消息发送到标准输出。当插件执行身份验证的时候这些信息有利于调试`PAM-related`发生的问题。

一些消息包括参考PAM插件源文件和行号,使插件操作更加紧密地联系在一起的位置代码它们会出现在哪里。

通过启用日志记录产生以下类型的信息。这是一个从尝试连接到成功代理认证的过程。

```sql
    entering auth_pam_server
	entering auth_pam_next_token
	auth_pam_next_token:reading at [cups,admin=writer,everyone=reader], sep=[,]
	auth_pam_next_token:state=PRESPACE, ptr=[cups,admin=writer,everyone=reader],
	out=[]
	auth_pam_next_token:state=IDENT, ptr=[cups,admin=writer,everyone=reader],
	out=[]
	auth_pam_next_token:state=AFTERSPACE, ptr=[,admin=writer,everyone=reader],
	out=[cups]
	auth_pam_next_token:state=DELIMITER, ptr=[,admin=writer,everyone=reader],
	out=[cups]
	auth_pam_next_token:state=DONE, ptr=[,admin=writer,everyone=reader],
	out=[cups]
	leaving auth_pam_next_token on
	/Users/gkodinov/mysql/work/x-5.5.16-release-basket/release/plugin/pam-authentication-plugin/src/parser.auth_pam_server:password 12345qq received
	auth_pam_server:pam_start rc=0
	auth_pam_server:pam_set_item(PAM_RUSER,gkodinov) rc=0
	auth_pam_server:pam_set_item(PAM_RHOST,localhost) rc=0
	entering auth_pam_server_conv
	auth_pam_server_conv:PAM_PROMPT_ECHO_OFF [Password:] received
	leaving auth_pam_server_conv on
	/Users/gkodinov/mysql/work/x-5.5.16-release-basket/release/plugin/pam-authentication-plugin/src/authentication_auth_pam_server:pam_authenticate rc=0
	auth_pam_server:pam_acct_mgmt rc=0
	auth_pam_server:pam_setcred(PAM_ESTABLISH_CRED) rc=0
	auth_pam_server:pam_get_item rc=0
	auth_pam_server:pam_setcred(PAM_DELETE_CRED) rc=0
	entering auth_pam_map_groups
	entering auth_pam_walk_namevalue_list
	auth_pam_walk_namevalue_list:reading at: [admin=writer,everyone=reader]
	entering auth_pam_next_token
	auth_pam_next_token:reading at [admin=writer,everyone=reader], sep=[=]
	auth_pam_next_token:state=PRESPACE, ptr=[admin=writer,everyone=reader], out=[]
	auth_pam_next_token:state=IDENT, ptr=[admin=writer,everyone=reader], out=[]
	auth_pam_next_token:state=AFTERSPACE, ptr=[=writer,everyone=reader],
	out=[admin]
	auth_pam_next_token:state=DELIMITER, ptr=[=writer,everyone=reader],
	out=[admin]
	state=DONE, ptr=[=writer,everyone=reader], out=[admin]
	leaving auth_pam_next_token on
	/Users/gkodinov/mysql/work/x-5.5.16-release-basket/release/plugin/pam-authentication-plugin/src/parser.auth_pam_walk_namevalue_list:name=[admin]
	entering auth_pam_next_token
	auth_pam_next_token:reading at [writer,everyone=reader], sep=[,]
	auth_pam_next_token:state=PRESPACE, ptr=[writer,everyone=reader], out=[]
	auth_pam_next_token:state=IDENT, ptr=[writer,everyone=reader], out=[]
	auth_pam_next_token:state=AFTERSPACE, ptr=[,everyone=reader], out=[writer]
	auth_pam_next_token:state=DELIMITER, ptr=[,everyone=reader], out=[writer]
	auth_pam_next_token:state=DONE, ptr=[,everyone=reader], out=[writer]
	leaving auth_pam_next_token on
	/Users/gkodinov/mysql/work/x-5.5.16-release-basket/release/plugin/pam-authentication-plugin/src/parser.walk, &error_namevalue_list:value=[writer]
	entering auth_pam_map_group_to_user
	auth_pam_map_group_to_user:pam_user=gkodinov, name=admin, value=writer
	examining member root
	examining member gkodinov
	substitution was made to mysql user writer
	leaving auth_pam_map_group_to_user on
    /Users/gkodinov/mysql/work/x-5.5.16-release-basket/release/plugin/pam-authentication-plugin/src/authentication_auth_pam_walk_namevalue_list:found mapping
	leaving auth_pam_walk_namevalue_list on
	/Users/gkodinov/mysql/work/x-5.5.16-release-basket/release/plugin/pam-authentication-plugin/src/parser.c:270
	auth_pam_walk_namevalue_list returned 0
	leaving auth_pam_map_groups on
	/Users/gkodinov/mysql/work/x-5.5.16-release-basket/release/plugin/pam-authentication-plugin/src/authentication_auth_pam_server:authenticated_as=writer
	auth_pam_server: rc=0
	leaving auth_pam_server on
	/Users/gkodinov/mysql/work/x-5.5.16-release-basket/release/plugin/pam-authentication-plugin/src/authentication_
```

#### 6.3.7.6. Windows本地身份验证插件 ####

>**注意**
>
>Windows身份验证插件是一个商业扩展。学习更多关于商业产品(MySQL企业版),请参考http://www.mysql.com/products/。

在MySQL 5.6.10,Windows上的MySQL商业分布包括在Windows上执行外部身份验证的身份验证插件,使MySQL服务器能在本机Windows服务客户端连接进行身份验证。用户能登录到Windows可以连接到MySQL客户端程序到服务器，是基于他们的本地密码不再另外指定一个密码。

客户端和服务器在认证握手的过程中交换数据包。作为一个交换的结果,服务器创建一个安全的环境对象,在Windows操作系统代表客户端的身份。这个身份包括客户账户的名称。Windows身份验证插件使用客户端的身份检查是否是一个给定的帐户或是某个组的成员。默认情况下,协商使用`Kerberos`身份验证,如果`Kerberos`是不可用的就使用NTLM。

Windows身份验证插件提供了这些功能:

* 外部认证:插件支持MySQL服务器接受来自mysql授权表以外的用户以外连接。

* 支持用户代理:插件可以返回到MySQL用户名不同于客户端用户。这意味着插件可以返回MySQL用户定义的外部`Windowsauthenticated`的特权用户应该有。例如,Windows用户名叫乔可以连接和有特权的MySQL用户名为开发者。

代理用户支持:插件可以返回到MySQL不同的客户端用户的用户名。这意味着插件应该可以返回MySQL用户定义的外部`Windows-authenticated`的特权用户。例如,Windows用户名叫joe可以连接和拥有MySQL用户名为`developer`的特权。

<p>
<div class="table">
<a name="idm47311711087328"></a><p class="title"><b>表 6.12. MySQL Windows身份验证插件</b></p>
<div class="table-contents">
<table><colgroup><col><col></colgroup><tbody><tr><td scope="row">服务器端的插件名称</td><td><code class="literal">authentication_windows</code></td></tr><tr><td scope="row">客户端的插件名称</td><td><code class="literal">authentication_windows_client</code></td></tr><tr><td scope="row">库对象文件名称</td><td><code class="filename">authentication_windows.dll</code></td></tr></tbody></table>
</div>

库文件只包含服务器端插件。客户端插件是内置在`libmysqlclient`客户端库。

服务器端身份验证插件是只包括Windows在商业发行版。这不包括在MySQL社区分布。所有发行版都包含客户端插件, 包括社区分布。这允许客户端从任何分布连接到服务器由服务器端插件加载。

Windows身份验证插件必需工作在Windows 2000 Professional和以上的版本。它要求MySQL服务器5.6.10或更新的。

关于MySQL可插入式身份验证对于综合信息,参考[6.3.7章,“可插入式身份验证”][06.03.07]。代理用户的信息,参考[6.3.8章,“代理用户”][06.03.08]。

#### 6.3.7.6.1. 安装Windows身份验证插件 ####

Windows身份验证插件必须安装在MySQL插件目录(目录指定的`plugin_dir`系统变量)。如果有必要,将`plugin_dir`的值设置为服务器启动时加载来告诉服务器插件目录的位置。

要启用插件,启动服务器使用`--plugin-load`选项。例如,把在你的`my.ini`文件加上这几行:

```sql
    [mysqld]
	plugin-load=authentication_windows.dll
```

使用`authentication_windows`插件对MySQL账户使用`CREATE USER`或`GRANT`语句及`IDENTIFIED WITH`子句,应该由这个插件认证。

验证插件安装,检查`INFORMATION_SCHEMA.PLUGINS`表或使用`SHOW PLUGINS`语句。参考[5.1.8.2”,获取服务器插件信息”][05.01.08.02]。

#### 6.3.7.6.2. 使用Windows身份验证插件 ####

Windows身份验证插件支持使用从Windows可以连接到MySQL服务器不需要另外指定一个密码的MySQL账户。它假定了服务器端插件是启用的,客户端程序使用了最新的内置客户端插件`libmysqlclient`。一旦DBA已经启动了服务器端插件和设置账户来使用它,客户可以在不需要做其他设置的情况下使用这些账号连接。

指定Windows身份验证插件在`CREATE USER`或`GRANT`语句的子句`IDENTIFIED WITH`项使用名字`authentication_windows`。假设Windows用户`Rafal`和`Tasha`应该允许连接到MySQL,以及`Administrators`或`Power Users`组任何用户都一样可以。进行这个设置,创建一个MySQL账户命名sql_admin,起用Windows身份验证插件:

```sql
    CREATE USER sql_admin
	 IDENTIFIED WITH authentication_windows
	 AS 'Rafal, Tasha, Administrators, "Power Users"';
```

插件名称是`authentication_windows`。在AS后面跟着字符串作为关键字的验证字符串。它指定Windows用户名为Rafal或Tasha是允许通过身份验证为MySQL用户sql_admin连接到服务器,Windows用户管理员或超级用户组也可以通过MySQL用户sql_admin连接到服务器。后面那个组名称包含空格,所以它必须用双引号字符引用。

当您创建`ssql_admin`账户,用户已登录到Windows可以使用这个帐户尝试连接服务器:

```sql
    C:\> mysql --user=sql_admin
```

这里不需要密码。这个`authentication_windows`插件使用Windows安全API检查Windows用户的连接。如果该用户名为Rafal或Tasha,或者是在管理员或超级用户组,服务器授予访问权限和作为sql_admin对客户端进行身份验证和被授予sql_admin帐户所有权限。如果用户名不在那个范围之内,服务器将拒绝访问。

Windows身份验证插件验证字符串的语法遵循这些规则:

* 字符串包含一个或多个用户映射之间用逗号分隔。

* 把Windows用户或组名的每一个用户合起来都映射为一个MySQL用户名:

```sql
    win_user_or_group_name=sql_user_name
	win_user_or_group_name
```

对于后者的语法,不给定值`sql_user_name`，MySQL用户`CREATE USER`语句会给一个默认值。因此,这些语句是等价的:

```sql
    CREATE USER sql_admin
	  IDENTIFIED WITH authentication_windows
	  AS 'Rafal, Tasha, Administrators, "Power Users"';
		
	CREATE USER sql_admin
	  IDENTIFIED WITH authentication_windows
	  AS 'Rafal=sql_admin, Tasha=sql_admin, Administrators=sql_admin,
		"Power Users"=sql_admin';
```

* 每一个反斜杠(“\”)在一个值必须加倍,因为反斜杠是转义字符在MySQL的字符串。

* 没有在双引号里面的首尾空格会被忽略。

* `win_user_or_group_nam`e和`sql_user_name`的值不能包含等号,逗号,或空格，其他字符都可以。

* `win_user_or_group_name`或`sql_user_name`的值用双引用引起来,双引号中间是值。这是必要的,例如, 如果名称包含空格字符。所有的字符包含在双引号里面是有效的，除了双引号和反斜杠。任何值都应该逃避使用反斜杠。

* `win_user_or_group_name`值使用Windows主体的传统语法,无论是本地或在定义域。例子(注意反斜杠应该是翻倍):

```sql
    domain\\user
	.\\user
	domain\\group
	.\\group
	BUILTIN\\WellKnownGroup
```

当服务器调用验证客户端,插件从左到右扫描认证字符串匹配Windows用户为用户或组。如果有匹配,插件返回相应的`sql_user_name`到MySQL服务器。如果没有匹配,验证失败。

用户名匹配需要优先组名匹配。假设Windows用户`win_user`是`win_group`组的成员，验证字符串应该是这样的:

```sql
    'win_group = sql_user1, win_user = sql_user2'
```

当win_user连接到MySQL服务器,都`win_group`和`win_user`都要进行匹配。插件验证用户作为`sql user2`因为存在的用户匹配优先于组的匹配,尽管组是首先列出来的身份验证字符串。

indows身份验证总是连接到那台一直在运行的计算机上。对于交叉计算机连接,两个电脑必须注册与Windows活跃目录。如果他们是在同一个Windows域,它是不需要指定一个域名。这是也可以允许从一个不同的域进行连接,比如在这个例子:

```sql
    CREATE USER sql_accounting
		IDENTIFIED WITH authentication_windows
		AS 'SomeDomain\\Accounting';
```

这里`SomeDomain`是其他域的名称。反斜线字符是翻倍,因为它是作为MySQL转义字符。

MySQL支持代理用户的概念,一个客户端可以连接和MySQL服务器使用一个帐户验证但同时连接有特权的另一个帐户(参考[6.3.8,“代理用户”][06.03.08])。假设你想要的Windows用户连接使用同一个用户名字,但被映射基于他们的Windows用户和组名到指定的MySQL账户如下:

* `local_user`和`MyDomain\domain_user`本地和域Windows用户映射到MySQL账户`local_wlad`。

* 用户在`MyDomain\Developers`域组应该映射到MySQL帐户`local_dev`。

* 本地机器管理员应该映射到MySQL账户`local_admin`.

进行这个设置,给Windows用户连接创建一个代理帐户,并配置该帐户的用户和组映射到指定的MySQL账户(`local_wlad`, `local_dev`,`local_admin`)。此外,授权MySQL账户相应的权限来执行他们需要的操作。下面的说明使用`win_proxy`作为代理帐户,`local_wlad`, `local_dev`,和`local_admin`作为代理帐户。

1. 创建用来作代理的MySQL账户:

```sql
	CREATE USER win_proxy
	  IDENTIFIED WITH authentication_windows
	  AS 'local_user = local_wlad,
		MyDomain\\domain_user = local_wlad,
		MyDomain\\Developers = local_dev,
		BUILTIN\\Administrators = local_admin';	
```

2.  对代理工作,代理账户必须存在,所以创建它们:

```sql
	CREATE USER local_wlad IDENTIFIED BY 'wlad_pass';
	CREATE USER local_dev IDENTIFIED BY 'dev_pass';
	CREATE USER local_admin IDENTIFIED BY 'admin_pass';
```

如果你不让任何人知道这些账户的密码,其他用户也就无法使用它们直接连接到MySQL服务器。

用GRANT语句(输出没有显示)给每个代理账户授予他们需要的权限。

3.  代理帐户必须有每个代理的账户的代理特权:

```sql
	GRANT PROXY ON local_wlad TO win_proxy;
	GRANT PROXY ON local_dev TO win_proxy;
	GRANT PROXY ON local_admin TO win_proxy;
```

现在的Windows用户`local_user`和`MyDomain\domain_user`可以作为`win_proxy`用户进行身份验证连接到MySQL服务器和在这种情况下验证字符串是有特权的帐户,`local_wlad`。在`MyDomain\Developers`组的一个用户作为`win_proxy`的特权连接到`local_dev`帐户。在`BUILTIN\Administrators`组的一个用户有`local_admin`帐户的权限。

配置身份验证,防止那些没有MySQL账户的Windows用户去通过一个代理帐户进行连接,替代默认代理用户(“@”)为`win_proxy`在前面说明中。关于默认代理用户的信息,请参考[6.3.8,“代理用户”][06.03.08]。

Windows身份验证插件基于`Connector/Net`连接字符串，在`Connector/Net 6.4.4`和更高的版本,参考[22.2.5.5,"使用Windows本地身份验证插件”][22.02.05.05]。

Windows身份验证插件其他的控制是由`authentication_windows_use_principal_name`和`authentication_windows_log_level`系统变量。参考[5.1.4,“服务器系统变量”][05.01.04]。

#### 6.3.7.7. 客户端明文身份验证插件 ####

在MySQL 5.6.2版本的时候,客户端身份验证插件是可用的,发送的密码服务器没有使用哈希算法或加密。这个插件是内置在MySQL客户端库。

下面的表显示了插件名称。

**表 6.13. MySQL明文验证插件**

<div class="table-contents">
<table><colgroup><col><col></colgroup><tbody><tr><td scope="row">服务端插件名称</td><td>None, see discussion</td></tr><tr><td scope="row">客户端插件名称</td><td><code class="literal">mysql_clear_password</code></td></tr><tr><td scope="row">库对象文件名</td><td>None (plugin is built in)</td></tr></tbody></table>
</div>

与本地MySQL认证,客户端单向执行用哈希算法对密码加密发送给服务器。这使客户避免发送密码以明文。参考[6.1.2.4,“密码散列在MySQL”][06.01.02.04]。然而,由于哈希算法是一种方法,在服务器端不能恢复原始密码。

单向哈希算法不能做身份验证方案,需在客户端的输入密码发送给服务器来接收。在这种情况下,客户端插件的mysql以明文密码可以用来清晰地发送密码到服务器。没有对应的服务器端插件，相反,客户端插件需要发送明文密码给服务器端插件。

关于mysql可插入身份验证的信息,参考[6.3.7,“可插入身份验证”][06.03.07]。

注意

用明文的方式发送密码在一些配置这是一个安全问题。为了避免密码将被拦截的问题出现,客户应该使用密码保护的方式连接到MySQL服务器。可能包括SSL(参考[6.3.9,“使用SSL安全连接”][06.03.09]),IPsec,或专用网络。

在MySQL 5.6.7,无意中使用这个插件不太可能,这需要客户明确来启用它。可以用几种方法来启用:

* 设置`LIBMYSQL_ENABLE_CLEARTEXT_PLUGIN`环境变量值的初始值为1,Y,或Y。这使插件针对所有的客户端连接。

* mysql,mysqladmin和mysqlslap客户程序支持`——enable-cleartextplugin`选项,允许在每次调用插件的基础上使用。

* 在`mysql_options()` C API函数支持`MYSQL_ENABLE_CLEARTEXT_PLUGIN`选项使之能每个连接都使用这个插件。同样,任何程序使用`libmysqlclient`和读选项文件可以通过`enable-cleartext-plugin`选项在选项组读取客户端库启用插件。

#### 6.3.7.8. 套接字对等凭据的身份验证插件 ####

在MySQL 5.6.2,服务器端身份验证插件是可通过`Unix socket`文件验证从本地主机连接的客户端。

查看这个插件的源代码可以作为如何去写一个可加载的身份验证插件的一个简单演示示例。

下面的表显示了插件和库文件的名称。可能不同的系统显示的文件名后缀会不同。`plugin_dir`系统变量指定这个文件位置目录。关于的安装信息,参考[6.3.7,“可插入式身份验证”][06.03.07]。


<div class="table">
<a name="idm47568719158288"></a><p class="title"><b>表 6.14. MySQL套接字对等凭据的身份验证插件</b></p>
<div class="table-contents">
<table><colgroup><col><col></colgroup><tbody><tr><td scope="row">服务端插件名称</td><td><code class="literal">auth_socket</code></td></tr><tr><td scope="row">客户端插件名称</td><td>None, see discussion</td></tr><tr><td scope="row">库对象文件名</td><td><code class="filename">auth_socket.so</code></td></tr></tbody></table>
</div>

`auth_socket`身份验证插件验证通过本地主机的`Unix socket`文件对所连接客户端进程身份验证。插件使用`SO_PEERCRED`套接字选项来获得用户运行客户端程序信息。插件检查用户名是否匹配的MySQL用户名从指定的客户机程序到服务器,只要用户名称匹配就允许连接。支持`SO_PEERCRED`选项的系统就可以安装这个插件,如Linux。

假设一个MySQL账户创建一个用户名叫`valerie`，要连接得通过`auth_socket`插件对本地主机套接字文件进行身份认证：

```sql
    CREATE USER 'valerie'@'localhost' IDENTIFIED WITH auth_socket;
```

如果一个用户在本地主机上使用登录名`stefanie`请求连接mysql，使用`——user=valerie`项通过连接套接字文件连接,服务器使用`auth_socket`来进行对客户端验证。插件确定`--user`选项值(`valerie`)不同于客户端用户的名称(`stephanie`)和拒绝连接。如果一个用户名为`valerie`尝试同样的事情, 插件发现用户名和MySQL用户名称都是`valerie`和允许连接。但是,如果`valeri`使用不同的协议连接的话插件同样也会拒绝连接,比如TCP/IP。

关于MySQ可插入身份验证的其他信息,参考[6.3.7,“可插入式身份验证”][06.03.07]。

#### 6.3.7.9. 测试验证插件 ####

MySQL包含一个测试插件,验证使用MySQL本地身份验证,但是它是一个可加载插件(不是内置的),必须是使用前安装。它可以对正常或旧的(短的)密码哈希值进行验证。

这个插件是用于测试和开发的目的,而不是用于生产环境。测试插件的源代码是独立于服务器源码,与内置的本地插件不同,所以它可以检查作为一个相对简单的示例演示如何编写一个可加载的身份验证插件。

下面的表显示了插件和库文件的名称。可能根据不同的系统文件名后缀不同。这个文件位置目录指定的`plugin_dir`系统变量。关于安装信息,请参考[6.3.7,“可插入式身份验证”][06.03.07]。

**表 6.15. MySQL测试验证插件**

<div class="table-contents">
<table ><colgroup><col><col></colgroup><tbody><tr><td scope="row">服务器端插件名称</td><td><code class="literal">test_plugin_server</code></td></tr><tr><td scope="row">客户端插件名称</td><td><code class="literal">auth_test_plugin</code></td></tr><tr><td scope="row">库对象文件的名称</td><td><code class="filename">auth_test_plugin.so</code></td></tr></tbody></table>
</div>

因为测试插件验证跟本地MySQL认证一样,提供选项`--user`和`--password`,通常可以使用账户本机验证当你连接到服务器的时候。例如:

```sql
    shell> mysql --user=your_name --password=your_pass
```

在MySQL关于可插入身份验证的一般信息，参考[6.3.7,“可插入式身份验证”][06.03.07]。

### 6.3.8. 代理用户 ###

当验证MySQL服务器发生通过身份验证插件来认证,插件可能把企图请求连接(外部)的用户当作不同的用户进行权限检查。这使得外部用户是一个代理为第二个用户,即有第二个用户的特权。换句话说,外部用户是一个“代理用户”(一个用户谁可以模仿或成为被称为另一个用户)和第二个用户是一个“用户代理”(一个用户的身份他可以采取被代理用户)。

本节描述代理用户怎么工作。关于身份验证插件的相关信息,参考[6.3.7,“可插入身份验证”][06.03.07]。如果你有兴趣写你自己的支持代理用户身份验证插件,参考[23.2.4.9.4,"在身份认证插件中支持用户代理“][23.02.04.09.04]。

要使用代理,必须满足这些条件:

* 当一个连接客户端应该被视为一个代理用户,插件必须返回一个不同的名字,指定代理用户名。

* 代理用户帐户必须设置要认证的插件。使用`CREATE USER`或`GRANT`语句来关联一个帐户到一个插件。

* 代理用户帐户必须有PROXY特权的代理帐户。使用GRANT语句来授权。

参考下面的定义:

```sql
    CREATE USER 'empl_external'@'localhost'
	  IDENTIFIED WITH auth_plugin AS 'auth_string';
	CREATE USER 'employee'@'localhost'
	 IDENTIFIED BY 'employee_pass';
	GRANT PROXY
	 ON 'employee'@'localhost'
     TO 'empl_external'@'localhost';
```

当一个客户端作为`empl_external`从本地主机连接,MySQL使用`auth_plugin`来执行身份验证。如果`auth_plugin`返回`employee`用户名到服务器(内容是`“auth_string”`,也许通过咨询了一些外部认证系统),作为一个客户对服务器进行请求为目的,进行特权检查,作为`employee`的本地用户。

在这种情况下,`empl_external`是代理用户和`employee`是被代理用户。

服务器为`employee`验证代理认证`empl_external`用户，通过检查`empl_external`是否有为`employee`代理的特权。(如果这个特权没有被批准,将会报一个错。)

当发生代理的时候,`USER()`和`CURRENT_USER()`函数可以用来看到连接用户和账户的权限之间的区别在当前会话。对于刚才描述的示例,这些函数返回这些值:

```sql
    mysql> SELECT USER(), CURRENT_USER();
	+-------------------------+--------------------+
	| USER() 				  | CURRENT_USER()     |
	+-------------------------+--------------------+
	| empl_external@localhost | employee@localhost |
	+-------------------------+--------------------+
```

命名的身份验证插件`IDENTIFIED WITH`子句后面跟着的AS子句指定一个字符串,当用户连接服务器报字符串传递给插件。这取决于每个插件是否需要`AS`子句。如果有使用`AS`子句,验证字符串的格式依赖于插件怎么使用它。插件会给出一个参考文档关于插件它能接受的验证字符串值的信息。

**授予代理权限 **

一个特殊的代理特权是需要启用外部用户连接,有另一个用户的特权。授予这特权,使用grant语句。例如:

```sql
    GRANT PROXY ON 'proxied_user' TO 'proxy_user';
```

代理用户必须代表一个有效的外部验证MySQL用户在连接时间或连接尝试失败。被代理用户必须是一个有效的本地身份验证的用户在连接时间或连接尝试失败。

相应的REVOKE语法是:

```sql
    REVOKE PROXY ON 'proxied_user' FROM 'proxy_user';
```

MySQL `GRANT`和`REVOKE`语法扩展工作也是一样的。比如：

```sql
    GRANT PROXY ON 'a' TO 'b', 'c', 'd';
	GRANT PROXY ON 'a' TO 'd' IDENTIFIED BY ...;
	GRANT PROXY ON 'a' TO 'd' WITH GRANT OPTION;
	GRANT PROXY ON 'a' TO ''@'';
	REVOKE PROXY ON 'a' FROM 'b', 'c', 'd';
```

在前面的示例中,“@”是默认代理用户,意味着“任何用户”。默认代理用户稍后讨论在本节中。

代理权限可以授予在这些情况下:

* 由被代理用户对于本身的价值`USER()`必须完全匹配`CURRENT_USER()`和代理用户,对于用户名和主机名部分帐户名称也一样要匹配。

* 由一个用户`GRANT PROXY ... WITH GRANT OPTION` 给被代理用户授权。

root帐户创建默认情况下在MySQL安装`PROXY ... WITH GRANT OPTION`特权为`“@”`,即为所有用户。这使root设置为代理用户,作为也代表其他账户的权限来设置代理用户。例如,root可以做这个:

```sql
    CREATE USER 'admin'@'localhost' IDENTIFIED BY 'test';
	GRANT PROXY ON ''@'' TO 'admin'@'localhost' WITH GRANT OPTION;
```

现在`admin`用户可以管理所有具体`GRANT PROXY`映射。例如,admin可以这样做:

```sql
    GRANT PROXY ON sally TO joe;
```

**默认代理用户**

指定一些或所有用户连接的时候应该使用一个给定的外部插件,创建一个“空白”MySQL用户,设置它使用插件身份验证,让插件返回真实身份验证的用户名称(如果不是空白用户)。例如,假设存在一个假设的插件命名为ldap_auth实现LDAP身份验证:

```sql
    CREATE USER ''@'' IDENTIFIED WITH ldap_auth AS 'O=Oracle, OU=MySQL';
	CREATE USER 'developer'@'localhost' IDENTIFIED BY 'developer_pass';
	CREATE USER 'manager'@'localhost' IDENTIFIED BY 'manager_pass';
	GRANT PROXY ON 'manager'@'localhost' TO ''@'';
	GRANT PROXY ON 'developer'@'localhost' TO ''@'';
```

现在,假设客户机试图连接如下:

```sql
    mysql --user=myuser --password='myuser_pass' ...
```

服务器将不会找到`myuser`定义为一个MySQL用户。但因为有一个空白的用户帐户(“@”),匹配的客户端用户名和主机名,服务器验证客户端针对这个帐户:服务器调用`ldap_auth`,传递它`myuser`和`myuser_pass`作为用户名和密码。

如果`ldap_auth`插件发现在LDAP目录中`myuser_pass`不是`myuser`的密码，身份验证失败,服务器拒绝连接。

如果密码是正确的,`ldap_auth`插件发现了`myuser`是对应`developer`,它返回用户名`developer`给MySQL服务器,而不是`myuser`。服务器验证,“@”可以作为`developer`进行身份验证(因为它有代理特权)和接受连接。`myuser`用户连接会话接入拥有特权的`developer`。(这些特权应该设立的DBA使用GRANT语句,没有显示)。`USER()`和`CURRENT_USER()`函数返回这些值:

```sql
    mysql> SELECT USER(), CURRENT_USER();
	+------------------+---------------------+
	| USER() 		   | CURRENT_USER() 	 |
	+------------------+---------------------+
	| myuser@localhost | developer@localhost |
	+------------------+---------------------+
```

如果插件而不是发现在LDAP目录,`myuser`是对应`manager`,它返回`manager`作为用户名和会话连接`myuser`拥有特权的`manager`。

```sql
    mysql> SELECT USER(), CURRENT_USER();
	+------------------+-------------------+
	| USER() 		   | CURRENT_USER()    |
	+------------------+-------------------+
	| myuser@localhost | manager@localhost |
	+------------------+-------------------+
```

为简单起见,外部认证不能多层次:无论是`developer`还是那些`manager`凭证都一样的在前面的例子。然而,`developer`或`manager`他们仍然不能直接用帐户从客户端试图验证,这就是为什么那些账户应该设置密码。

默认代理帐户使用''在主机部分,它表示匹配所有主机。如果你设置一个默认的代理用户,小心还检查账户‘%’在主机部分,因为这也匹配所有主机,但规则优先匹配‘’服务器使用分类帐户行的内部(参考[6.2.4，“访问控制,阶段1:连接验证”][06.02.04])。

假设一个MySQL安装包括这两个账户:

```sql
    CREATE USER ''@'' IDENTIFIED WITH some_plugin;
	CREATE USER ''@'%' IDENTIFIED BY 'some_password';
```

第一个账户的目的是作为默认代理用户,是用于验证连接中未能匹配一个更具体的账户的用户。第二个账户可能已经被创建,例如,要让在没有自己的账户的用户作为匿名用户。

然而,在这个配置中,第一个帐户将不能使用,因为匹配规则排序”@“%”在“@”之前,账户不匹配任何更具体的帐户,服务器将试图验证他们，将拒绝“@”%”而不是“@”。

如果您想要创建一个默认的代理用户,检查其他现有的“匹配任何用户”账户,将优先于默认代理用户,从而妨碍了用户预期的工作。应该删除所有这样的帐户。

**代理用户系统变量**

两个系统变量帮助跟踪代理登录的过程:

* 代理用户:如果不使用代理这个值是NULL。否则,它表明代理用户帐户。例如,如果一个客户端验证通过默认代理帐户,变量将这么设置:

```sql
		mysql> SELECT @@proxy_user;
		+--------------+
		| @@proxy_user |
		+--------------+
		| ''@'' 	   |
		+--------------+
```

* 外部用户:有时候,身份验证插件可以使用一个外部的用户来给MySQL服务器认证。例如,当使用Windows本地认证,一个插件使用windows API认证,不需要传递登录ID。然而,它仍然使用一个Windows用户ID进行身份验证。插件可以返回这个外部用户ID(或第一个512 utf-字节的)服务器使用外部用户只读会话变量。如果插件不设置这个变量,它的值是NULL。

### 6.3.9. 使用SSL进行安全连接 ###

MySQL支持安全(加密)连接在MySQL客户端和服务器之间使用安全套接字层(SSL)协议。本节讨论如何使用SSL连接。对于信息如何要求用户使用SSL连接,参考讨论关于GRANT语句的子句REQUIRE在[13.7.1.4,“授权语法”][13.07.01.04]。

标准配置MySQL的目的是要尽可能快,所以默认情况下不使用加密连接。对于应用程序来说,它需要所提供的安全性加密的连接,花费一定的时间来计算加密数据是值得的。

MySQL允许对每个连接加密。你可以根据单独的应用程序的要求选择一个未加密的连接或一个安全的加密的SSL连接。

安全连接是基于OpenSSL API并有效地通过MySQL C API。可以复制使用C API,所以可以在主从复制之间使用安全连接。参考[16.3.7,"设置主从间复制使用SSL”][16.03.07]。

另一种方法从内部使用SSH安全地连接到MySQL服务器主机。有一个相关的例子,请参考[6.3.10,“从Windows系统中使用SSH远程连接到MySQL”][06.03.10]。

#### 6.3.9.1. SSL基本概念 ####

了解MySQL使用SSL,需要解释SSL和X509一些基本的概念。那些熟悉这些概念的人可以跳过这个讨论部分。

默认情况下,MySQL在客户机和服务器之间使用未加密的连接。这意味着有人可以通过访问网络可以看到你所有的通信和查看被发送或收到了的数据。他们甚至可以在传输时改变客户端和服务器之间的数据。提高一点安全性,你可以压缩客户机和服务器流量当调用客户端程序的时候通过使用--compress选项。然而,这并不能防止有心攻击者。

当你需要移动信息网中使用一种安全的方式,一个加密连接是不可能就够的。加密是使任何类型的数据不可读的一种方式。加密算法必须包括安全元素抵抗多种已知的攻击如改变加密消息的命令或重播两次数据。

SSL是一种协议,它使用不同的加密算法,以确保接收到的数据在一个公共的网络可以被信任。它有机制来检测发生变化的数据,丢失的,或被重新发送的。SSL还包含有算法,提供使用X509标准的身份验证。

X509它可以识别有人在互联网上。它是最常用的电子商务应用程序。在基本条款,应该有一些实体称为“证书权威”(CA)分配电子证书给所有需要他们的人。依赖不对称加密证书算法有两个加密密钥(公钥和密钥)。一个证书所有者可以显示证书给另一方作为身份证明。证书由主人的公钥。任何数据的加密与解密这个公钥可以只使用相应的密钥,这是的所有者所持有的证书。

关于SSL,X509，加密算法，或公钥密码学的更多信息,可以在网上搜索你感兴趣的关键词。

#### 6.3.9.2. 给MySQL配置SSL ####

使用SSL连接在MySQL服务器和客户端程序之间,系统必须支持其中一种OpenSSL或yaSSL,和你的MySQL的版本必须支持安装SSL。使它更容易使用安全连接,MySQL是附带有yaSSL,它和MySQL使用相同的许可证模型。(OpenSSL使用`Apache-style`许可证)甲骨文公司的yaSSL可以支持所有MySQL平台。

让MySQL安全连接和SSL有效,你必须做到以下几点:

1. 如果你不使用二进制(预编译的)版本来安装MySQL是建立与但已支持SSL,你打算使用OpenSSL而不是捆绑的yaSSL库,如果它默认没有安装可以手动安装一下OpenSSL。我们已经测试了MySQL和OpenSSL 1.0.0去获得OpenSSL,参考访问`http://www.openssl.org`。

安装mysql使用时OpenSSL需要的一个共享的MySQL OpenSSL库,否则连接程序会报错。另外,给MySQL安装使用yaSSL。

2. 如果你没有使用二进制(预编译的)版本的MySQL但已安装支持SSL,源代码安装一个MySQL及SSL。当您配置MySQL,像这样调用一下CMake:

```sql
    shell> cmake . -DWITH_SSL=bundled
```

该命令配置分配使用捆绑的yaSSL库。使用系统SSL库来代替,指定安装代替项:

	shell> cmake . -DWITH_SSL=system

参考[2.9.4,“MySQL源配置的设置”][02.09.04]。

然后编译和安装代码。

在Unix平台上,yaSSL从`/dev/urandom`或者`/dev/random`获取真正的随机数。错误#13164列出一些很老的平台不支持这些设备的解决方案。	

3. 检查mysqld服务器是否支持SSL,检查have_ssl系统变量的值:

```sql
		mysql> SHOW VARIABLES LIKE 'have_ssl';
		+---------------+-------+
		| Variable_name | Value |
		+---------------+-------+
		| have_ssl 		| YES 	|
		+---------------+-------+
```

如果该值是`YES`,服务器支持SSL连接。如果该值是`DISABLED`,服务器能够支持SSL连接但没有开启这个选项`——SSL -XXXX`来启用它们;请参考[6.3.9.3,“使用SSL连接”][06.03.09.03]。

#### 6.3.9.3. 使用SSL连接 ####

启用SSL连接,你的MySQL分布需要建立支持SSL,如前所示[6.3.9.2,“给MySQL配置SSL”][06.03.09.02]。另外,适当的ssl相关选项必须指定正确的证书和密钥文件。对于SSL的设置完整列表,请参考[6.3.9.4,“SSL命令设置”][06.03.09.04]。

启动MySQL服务器,允许客户使用SSL进行连接,在服务器上使用选项来确定证书和密钥文件来建立一个安全的连接:

* `——ssl CA`标识证书颁发机构(CA)证书。

* `--ssl-cert`标识服务器公钥证书。这可以被发送到客户端和针对它有CA证书认证。

* `--ssl-key`标识服务器私钥。

例如,启动服务器如下:

```sql
    shell> mysqld --ssl-ca=ca-cert.pem \
		 --ssl-cert=server-cert.pem \
	     --ssl-key=server-key.pem
```

在PEM格式文件每个选项名称。按照说明生成所需的SSL证书和关键文件，参考[6.3.9.5,“给MySQL设置SSL证书和密钥”][06.03.09.05]。如果你有一个MySQL源代码,还可以测试你的设置使用测试证书和密钥文件，在分布的测试目录`mysql-test/std_data`。

在客户端使用类似的选项,在这种情况下,`--ssl-cer`t和`--ssl-key`识别客户端公钥和私钥。注意,这个证书是权威证书,如果指定,服务器端也必须使用相同证书。

建立一个安全的连接到一个支持SSL的MySQL服务器,设置客户端必须产指定依赖SSL且客户端使用MySQL帐号的要求。(参考REQUIRE语句在[13.7.1.4,”授权语法”。][13.07.01.04])

假设您想连接使用一个账户,没有特殊需求或SSL创建使用GRANT语句,包括`REQUIRE SSL`选项。作为可以被推荐设置SSL语句,启动服务器跟上`--ssl-cert`和`--ssl-key`,SSL密钥和调用客户端`--ssl ca`。客户端可以安全地连接如下:

```sql
    shell> mysql --ssl-ca=ca-cert.pem
```

要求客户也应该指定端证书,创建帐户使用REQUIRE X509选项。然后客户机还必须指定正确的客户端密钥和证书文件否则服务器将拒绝连接:

```sql
    shell> mysql --ssl-ca=ca-cert.pem \
		--ssl-cert=client-cert.pem \
		--ssl-key=client-key.pem
```

客户端可以通过检查`Ssl_cipher`状态值来确定当前的服务器连接是否使用SSL。`Ssl_cipher`的值如果是非空的表示启用了，否则就是没有启用。例如:

```sql
    mysql> SHOW STATUS LIKE 'Ssl_cipher';
	+---------------+--------------------+
	| Variable_name | Value 			 |
	+---------------+--------------------+
	| Ssl_cipher 	| DHE-RSA-AES256-SHA |
	+---------------+--------------------+
```

对于mysql客户端,另一种方法是使用`STATUS`或者`\s`命令和检查SSL连接队列:

```sql
	mysql> \s
	...
	SSL: Not in use	
	...
```

或者：

```sql
	mysql> \s
	...
	SSL: Cipher in use is DHE-RSA-AES256-SHA
	...
```

C API允许应用程序使用SSL:

* 建立一个安全连接,使用`mysql_ssl_set()` C API函数来设置适当的证书在`mysql_real_connect()`选项之前。参考[22.8.7.68节, “mysql ssl设置()”][22.08.07.68]。

* 用来确定在连接建立后是否使用了SSL,使用`mysql_get_ssl_cipher()`。一个非空返回值表示一个安全连接和SSL密码的名称用于加密。返回空值表明SSL没有被使用。参考[22.8.7.33,“mysql_ssl_set()”][22.08.07.33]。

复制可以使用C API,所以可以使用安全连接主、从服务器之间。参考[16.3.7,“设置主从复制间使用SSL”][16.03.07]。

#### 6.3.9.4. SSL命令选项 ####

本节描述选项,这些选项用于指定是否使用SSL和SSL的名字证书和密钥文件。这些选项可以在命令行上指定的或在一个选项文件。他们本身是无效的,除非安装支持SSL的MySQL。参考[6.3.9.2,“配置MySQL对于SSL”][06.03.09.02]。

<div class="table">
<a name="idm47846486940976"></a><p class="title"><b>表6.16. SSL选项/变量总结</b></p>
<div class="table-contents">
<table><colgroup><col class="name"><col class="cmd-line"><col class="option_file"><col class="system_var"><col class="status_var"><col class="var_scope"><col class="dynamic"></colgroup><thead><tr><th scope="col">Name</th><th scope="col">Cmd-Line</th><th scope="col">Option file</th><th scope="col">System Var</th><th scope="col">Status Var</th><th scope="col">Var Scope</th><th scope="col">Dynamic</th></tr></thead><tbody><tr><td scope="row"><a class="link" href="server-system-variables.html#sysvar_have_openssl">have_openssl</a></td><td> </td><td> </td><td>Yes</td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"><a class="link" href="server-system-variables.html#sysvar_have_ssl">have_ssl</a></td><td> </td><td> </td><td>Yes</td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"><a class="link" href="ssl-options.html#option_general_ssl">skip-ssl</a></td><td>Yes</td><td>Yes</td><td> </td><td> </td><td> </td><td> </td></tr><tr><td scope="row"><a class="link" href="ssl-options.html#option_general_ssl">ssl</a></td><td>Yes</td><td>Yes</td><td> </td><td> </td><td> </td><td> </td></tr><tr><td scope="row"><a class="link" href="ssl-options.html#option_general_ssl-ca">ssl-ca</a></td><td>Yes</td><td>Yes</td><td> </td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"> - <span class="emphasis"><em>Variable</em></span>: <a class="link" href="server-system-variables.html#sysvar_ssl_ca">ssl_ca</a></td><td> </td><td> </td><td>Yes</td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"><a class="link" href="ssl-options.html#option_general_ssl-capath">ssl-capath</a></td><td>Yes</td><td>Yes</td><td> </td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"> - <span class="emphasis"><em>Variable</em></span>: <a class="link" href="server-system-variables.html#sysvar_ssl_capath">ssl_capath</a></td><td> </td><td> </td><td>Yes</td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"><a class="link" href="ssl-options.html#option_general_ssl-cert">ssl-cert</a></td><td>Yes</td><td>Yes</td><td> </td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"> - <span class="emphasis"><em>Variable</em></span>: <a class="link" href="server-system-variables.html#sysvar_ssl_cert">ssl_cert</a></td><td> </td><td> </td><td>Yes</td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"><a class="link" href="ssl-options.html#option_general_ssl-cipher">ssl-cipher</a></td><td>Yes</td><td>Yes</td><td> </td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"> - <span class="emphasis"><em>Variable</em></span>: <a class="link" href="server-system-variables.html#sysvar_ssl_cipher">ssl_cipher</a></td><td> </td><td> </td><td>Yes</td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"><a class="link" href="ssl-options.html#option_general_ssl-crl">ssl-crl</a></td><td>Yes</td><td>Yes</td><td> </td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"> - <span class="emphasis"><em>Variable</em></span>: <a class="link" href="server-system-variables.html#sysvar_ssl_crl">ssl_crl</a></td><td> </td><td> </td><td>Yes</td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"><a class="link" href="ssl-options.html#option_general_ssl-crlpath">ssl-crlpath</a></td><td>Yes</td><td>Yes</td><td> </td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"> - <span class="emphasis"><em>Variable</em></span>: <a class="link" href="server-system-variables.html#sysvar_ssl_crlpath">ssl_crlpath</a></td><td> </td><td> </td><td>Yes</td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"><a class="link" href="ssl-options.html#option_general_ssl-key">ssl-key</a></td><td>Yes</td><td>Yes</td><td> </td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"> - <span class="emphasis"><em>Variable</em></span>: <a class="link" href="server-system-variables.html#sysvar_ssl_key">ssl_key</a></td><td> </td><td> </td><td>Yes</td><td> </td><td>Global</td><td>No</td></tr><tr><td scope="row"><a class="link" href="ssl-options.html#option_general_ssl-verify-server-cert">ssl-verify-server-cert</a></td><td>Yes</td><td>Yes</td><td> </td><td> </td><td> </td><td> </td></tr></tbody></table>
</div>

* `--ssl`

对于服务器,该选项允许指定服务器允许的SSL连接。对于一个客户端程序,它允许客户端连接到服务器使用SSL,但这个选项本身是不够造成一个SSL连接。作为一个推荐组选项来启用SSL连接,至少还得使用：`--ssl-cert`和`--ssl-key`,在服务器端，`--ssl-ca`在客户端。

`——ssl`隐含了其他`——ssl xxx`选项如上一段所示描述了这些选项。出于这个原因,`——ssl`通常不特别地指定。它是最常用的方法是明确在其现在相反的形式覆盖其他SSL选项或者表明SSL不启用的情况，指定项分别为`--skip-ssl`或者`--ssl=0`。例如,您可能有SSL选项中指定的[client]组在你的选项文件中默认使用SSL连接的当您调用MySQL客户端程序的时候。但是你现在想使用一个未加密的连接,在命令行调用客户端程序时带上`--skip-ssl`选项来覆盖指定文件中的默认选项。

使用`--ssl`不需要使用ssl连接,它只允许ssl连接。例如, 如果你指定这个选项,服务器默认没有被配置允许SSL连接，客户端程序使用一个加密连接可以使用这个选项。

安全的方法需要使用SSL连接是在创建一个MySQL账户时,至少在`GRANT`语句有一个`REQUIRE SSL`子句。在这种情况下,连接的帐户将被拒绝,除非MySQL是支持SSL连接,服务器和客户端已经使用合适的SSL设置。

这个REQUIRE语句允许其他ssl相关限制。这些可以用于比`REQUIRE SSL`更严格需求。在[13.07.01.04. “授权语法”][13.07.01.04]有描述`REQUIRE`, 提供补充的细节部分,SSL命令选项可能或必须由客户指定用于连接使用的账户创建使用各种REQUIRE的选项。

* `--ssl-ca=file_name`

PEM格式的、可信的SSL官方证书及文件路径。这个选项的含义和`——ssl`一样。

如果你使用SSL当建立一个客户端连接,你可以告诉客户端进行身份验证的服务器证书通过指定可以用`——ssl ca`也也可以用`ssl-capath`。服务器还是验证客户端根据在客户端建立使用GRANT语句的需求,它仍然是使用`——ssl ca/——ssl-capath`的值, 在启动时传递给服务器。

* `--ssl-capath=dir_name`

这个路径的目录,其中包含用PEM格式的SSL证书权威证书。这选项含义跟`——ssl`一样。

如果你使用SSL当建立一个客户端连接,通过指定可以用`——ssl ca`也也可以用`ssl-capath`选项你可以告诉客户端进行身份验证的服务器证书位置。服务器还是验证客户端根据在客户端建立使用GRANT语句的需求,它仍然是使用`——ssl ca/——ssl-capath`的值, 在启动时传递给服务器。

MySQL主从安装OpenSSL支持`——ssl-capath`选项。主从建立与yaSSL不要因为yaSSL看上去并不在任何目录,不遵循一个链接证书的树。yaSSL要求CA证书树的所有组件都被包含在一个CA证书树里面,每一个证书的文件有一个独特的`SubjectName`值。为了解决这yaSSL的限制,连接各个证书文件组合到一个新的文件证书树。然后作为`——ssl-capath`选项的值指定新文件。

* `--ssl-cert=file_name`

SSL证书的名字用PEM文件格式保存的，用于建立一个安全的连接。这选项含义跟`——ssl`一样。

* `--ssl-cipher=cipher_list`

允许使用SSL加密密码的一个列表。如果没有在密码列表中,SSL连接将不起作用。这选项含义跟——ssl一样。

对于最大限度的可移植性,cipher_list是一个列表的一个或多个密码的名字,使用冒号隔开。这个格式OpenSSL和yaSSL都使用。例子:

```sql
    --ssl-cipher=AES128-SHA
	--ssl-cipher=DHE-RSA-AES256-SHA:AES128-SHA
```

OpenSSL支持更灵活的语法来指定密码,如前面描述的OpenSSL文档在`http://www.openssl.org/docs/apps/ciphers.html`。然而,yaSSL不支持,如果MySQL安装使用yaSSL失败了的话可以尝试使用其扩展语法。

* `--ssl-crl=file_name`

一个文件的路径包含PEM格式的证书撤销列表。这选项含义跟`——ssl`一样。可以使用`--ssl-crl` 也可以使用 `--ssl-crlpath`来赋值,没有CRL检查执行,即使CA路径包含证书撤销列表。

基于OpenSSL构建MySQL支持` ——ssl -crl`选项。安装yaSSL不要因为yaSSL撤销列表而使yaSSL不起作用。

在MySQL 5.6.3版本的时候这个选项被添加。

* `--ssl-crlpath=dir_name`

这个路径的目录,其中包含一些文件包含PEM格式的证书撤销列表。这选项含义跟`——ssl`一样。既可以使用`--ssl-crl` 也可以使用`--ssl-crlpath`选项,没有CRL执行检查,即使CA路径包含了证书撤销列表。

基于OpenSSL构建MySQL支持`——ssl-crlpath`选项。安装yaSSL不要因为yaSSL撤销列表与yaSSL不工作。

在MySQL 5.6.3这个版本的时候选项被添加。

* `--ssl-key=file_name`

保存为PEM文件格式的SSL密钥的文件名用于建立一个安全的连接。　　

如果MySQL分布在构建时使用OpenSSL或(MySQL的5.6.3)yaSSL和密钥文件是保护密码,程序会提示用户输入密码。密码必须有交互,它不能被存储在一个文件。如果密码不正确,程序它不能继续阅读的使用ssl。在MySQL 5.6.3,如果MySQL构建是使用yaSSL和密钥文件被加密保护的,则会报一个错。

*` --ssl-verify-server-cert`

这个选项是只能用于客户端程序,不能在服务器使用。它使客户检查服务器发送到客户端的证书上普通名称值。客户端确认客户端使用连接到服务器的主机名,如果有一个不匹配则连接失败。这个特性可以用来防止中间人攻击。验证默认情况下是禁用的。

#### 6.3.9.5. 给MySQL设置SSL证书和密钥 ####

本节讲述是在MySQL服务器端和客户端如何设置SSL证书和密钥文件。第一个示例显示了一个简单的步骤比如您可以从命令行设置。第二个是一个包含更多的细节的脚本。前两个例子属于一个部分，讲的是在Unix系统中使用openssl命令。第三个示例描述了在Windows系统中设置SSL文件。

>**注意**
>
>无论你用的是什么方法生成证书和密钥文件,共同的名称值用于服务器和客户端证书/密钥必须各自不同于CA证书的普通名称值。>否则, 证书和密钥文件服务器使用OpenSSL编译将不起作用。在这种情况下有一个典型的报错：
>
>    ```sql
>    	ERROR 2026 (HY000): SSL connection error:
>	 	error:00000001:lib(0):func(0):reason(1)
>	 ```

**示例1:在Unix系统命令行中创建SSL文件**

下面的示例演示了一组命令来创建MySQL服务器和客户端的证书和密钥文件。你需要影响openssl提示的命令。生成测试文件, 您可以按Enter键打印出提示。生成文件用于线上使用,你需要提供一个非空的响应。

```sql
    # Create clean environment
	shell> rm -rf newcerts
	shell> mkdir newcerts && cd newcerts
	# Create CA certificate
	shell> openssl genrsa 2048 > ca-key.pem
	shell> openssl req -new -x509 -nodes -days 3600 \
			-key ca-key.pem -out ca-cert.pem
	# Create server certificate, remove passphrase, and sign it
	# server-cert.pem = public key, server-key.pem = private key
	shell> openssl req -newkey rsa:2048 -days 3600 \
		  -nodes -keyout server-key.pem -out server-req.pem
	shell> openssl rsa -in server-key.pem -out server-key.pem
	shell> openssl x509 -req -in server-req.pem -days 3600 \
		  -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem
	# Create client certificate, remove passphrase, and sign it
	# client-cert.pem = public key, client-key.pem = private key
	shell> openssl req -newkey rsa:2048 -days 3600 \
		-nodes -keyout client-key.pem -out client-req.pem
	shell> openssl rsa -in client-key.pem -out client-key.pem
	shell> openssl x509 -req -in client-req.pem -days 3600 \
		-CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem
```

在生成证书之后,验证它们:

```sql
	shell> openssl verify -CAfile ca-cert.pem server-cert.pem client-cert.pem
	server-cert.pem: OK
	client-cert.pem: OK
```

现在你有一组可以使用的文件,如下:

* `ca-cert.pem`:使用`——ssl ca`作为参数在服务器和客户端。(CA证书,如果使用,必须两边是相同的)。

* `server-cert.pem`, `server-key.pem`:使用`--ssl-cert` 和`--ssl-key`作为参数在服务器端。

* `client-cert.pem`, `client-key.pem`:使用`--ssl-cert` 和`--ssl-key`作为参数在客户端。

使用文件来测试SSL连接,参考[6.3.9.3,“使用SSL进行连接”][06.03.09.03]。

**示例2:在Unix使用脚本创建SSL文件**

下面是一个示例脚本,该脚本显示了如何给MySQ设置SSL证书和密钥文件L。执行脚本完之后,使用文件来测试SSL连接中可参考[6.3.9.3,“使用SSL进行连接”][06.03.09.03]。

```sql
    DIR=`pwd`/openssl
	PRIV=$DIR/private

	mkdir $DIR $PRIV $DIR/newcerts
	cp /usr/share/ssl/openssl.cnf $DIR
	replace ./demoCA $DIR -- $DIR/openssl.cnf

	# Create necessary files: $database, $serial and $new_certs_dir
	# directory (optional)

	touch $DIR/index.txt
	echo "01" > $DIR/serial

	#
	# Generation of Certificate Authority(CA)
	#

	openssl req -new -x509 -keyout $PRIV/cakey.pem -out $DIR/ca-cert.pem \
		-days 3600 -config $DIR/openssl.cnf

	# Sample output:
	# Using configuration from /home/monty/openssl/openssl.cnf
	# Generating a 1024 bit RSA private key
	# ................++++++
	# .........++++++
	# writing new private key to '/home/monty/openssl/private/cakey.pem'
	# Enter PEM pass phrase:
	# Verifying password - Enter PEM pass phrase:
	# -----
	# You are about to be asked to enter information that will be
	# incorporated into your certificate request.
	# What you are about to enter is what is called a Distinguished Name
	# or a DN.
	# There are quite a few fields but you can leave some blank
	# For some fields there will be a default value,
	# If you enter '.', the field will be left blank.
	# -----
	# Country Name (2 letter code) [AU]:FI
	# State or Province Name (full name) [Some-State]:.
	# Locality Name (eg, city) []:
	# Organization Name (eg, company) [Internet Widgits Pty Ltd]:MySQL AB
	# Organizational Unit Name (eg, section) []:
	# Common Name (eg, YOUR name) []:MySQL admin
	# Email Address []:

	#
	# Create server request and key
	#
	openssl req -new -keyout $DIR/server-key.pem -out \
		$DIR/server-req.pem -days 3600 -config $DIR/openssl.cnf
	
	# Sample output:
	# Using configuration from /home/monty/openssl/openssl.cnf
	# Generating a 1024 bit RSA private key
	# ..++++++
	# ..........++++++
	# writing new private key to '/home/monty/openssl/server-key.pem'
	# Enter PEM pass phrase:
	# Verifying password - Enter PEM pass phrase:
	# -----
	# You are about to be asked to enter information that will be
	# incorporated into your certificate request.
	# What you are about to enter is what is called a Distinguished Name
	# or a DN.
	# There are quite a few fields but you can leave some blank
	# For some fields there will be a default value,
	# If you enter '.', the field will be left blank.
	# -----
	# Country Name (2 letter code) [AU]:FI
	# State or Province Name (full name) [Some-State]:.
	# Locality Name (eg, city) []:
	# Organization Name (eg, company) [Internet Widgits Pty Ltd]:MySQL AB
	# Organizational Unit Name (eg, section) []:
	# Common Name (eg, YOUR name) []:MySQL server
	# Email Address []:
	#
	# Please enter the following 'extra' attributes
	# to be sent with your certificate request
	# A challenge password []:
	# An optional company name []:

	#
	# Remove the passphrase from the key
	#
	openssl rsa -in $DIR/server-key.pem -out $DIR/server-key.pem
	#
	# Sign server cert
	#
	openssl ca -cert $DIR/ca-cert.pem -policy policy_anything \
		-out $DIR/server-cert.pem -config $DIR/openssl.cnf \
		-infiles $DIR/server-req.pem
	# Sample output:
	# Using configuration from /home/monty/openssl/openssl.cnf
	# Enter PEM pass phrase:
	# Check that the request matches the signature
	# Signature ok
	# The Subjects Distinguished Name is as follows
	# countryName :PRINTABLE:'FI'
	# organizationName :PRINTABLE:'MySQL AB'
	# commonName :PRINTABLE:'MySQL admin'
	# Certificate is to be certified until Sep 13 14:22:46 2003 GMT
	# (365 days)
	# Sign the certificate? [y/n]:y
	#
	#	
	# 1 out of 1 certificate requests certified, commit? [y/n]y
	# Write out database with 1 new entries
	# Data Base Updated
	#
	# Create client request and key
	#
	openssl req -new -keyout $DIR/client-key.pem -out \
		$DIR/client-req.pem -days 3600 -config $DIR/openssl.cnf
	# Sample output:
	# Using configuration from /home/monty/openssl/openssl.cnf
	# Generating a 1024 bit RSA private key
	# .....................................++++++
	# .............................................++++++
	# writing new private key to '/home/monty/openssl/client-key.pem'
	# Enter PEM pass phrase:
	# Verifying password - Enter PEM pass phrase:
	# -----
	# You are about to be asked to enter information that will be
	# incorporated into your certificate request.
	# What you are about to enter is what is called a Distinguished Name
	# or a DN.
	# There are quite a few fields but you can leave some blank
	# For some fields there will be a default value,
    # If you enter '.', the field will be left blank.
	# ---	--
	# Country Name (2 letter code) [AU]:FI
	# State or Province Name (full name) [Some-State]:.
	# Locality Name (eg, city) []:
	# Organization Name (eg, company) [Internet Widgits Pty Ltd]:MySQL AB
	# Organizational Unit Name (eg, section) []:
	# Common Name (eg, YOUR name) []:MySQL user
	# Email Address []:
	#
	# Please enter the following 'extra' attributes
	# to be sent with your certificate request
	# A challenge password []:
	# An optional company name []:
	#
	# Remove the passphrase from the key
	#

	openssl rsa -in $DIR/client-key.pem -out $DIR/client-key.pem
	#
	# Sign client cert
	#

	openssl ca -cert $DIR/ca-cert.pem -policy policy_anything \
		-out $DIR/client-cert.pem -config $DIR/openssl.cnf \
		-infiles $DIR/client-req.pem

    # Sample output:
	# Using configuration from /home/monty/openssl/openssl.cnf
	# Enter PEM pass phrase:
	# Check that the request matches the signature
	# Signature ok
	# The Subjects Distinguished Name is as follows
	# countryName :PRINTABLE:'FI'
	# organizationName :PRINTABLE:'MySQL AB'
	# commonName :PRINTABLE:'MySQL user'
	# Certificate is to be certified until Sep 13 16:45:17 2003 GMT
	# (365 days)
	# Sign the certificate? [y/n]:y
	#
	#
	# 1 out of 1 certificate requests certified, commit? [y/n]y
	# Write out database with 1 new entries
	# Data Base Updated

	#
	# Create a my.cnf file that you can use to test the certificates
	#

	cat <<EOF > $DIR/my.cnf
	[client]
	ssl-ca=$DIR/ca-cert.pem
	ssl-cert=$DIR/client-cert.pem
	ssl-key=$DIR/client-key.pem
	[mysqld]
	ssl-ca=$DIR/ca-cert.pem
	ssl-cert=$DIR/server-cert.pem
	ssl-key=$DIR/server-key.pem
	EOF
```

**示例3:在Windows系统中创建SSL文件**

下载Windows OpenSSL如果不安装在您的系统。可用包可以看这里找到:

```sql
	http://www.slproweb.com/products/Win32OpenSSL.html`
```

选择`Win32-OpenSSL`或`Win64-OpenSSL`的轻量级openSSL包,这取决于您的系统(32位或64位)。默认的安装位置是`C:\ openssl-win32`或`C:\ OpenSSL-Win64`,取决于你下载什么包。下面的说明是假设一个默认的位置`C:\openssl-win32`。这个可以根据你的系统来修改,如果你使用的是64位的包。

如果消息发生在设置指示”……失踪的关键组件:微软Visual C++ 2008的发布包”,取消安装和下载以下的 一个软件包,同样取决于你的系统(32位或64位):

* Visual c++ 2008的发布包(x86),可下载在:

```sql
    http://www.microsoft.com/downloads/details.aspx?familyid=9B2DA534-3E03-4391-8A4D-074B9F2BC1BF
```

* Visual c++ 2008的发布包(x64),可下载在:

```sql
    http://www.microsoft.com/downloads/details.aspx?familyid=bd2a6171-e2d6-4230-b809-9a8d7548c1b6
```

在安装完额外的包,重启一下OpenSSL程序。

在安装的过程中，放弃这默认`C:\OpenSSL-Win32`作为安装路径，也放弃在默认被选择'复制`OpenSSL DLL`文件到windows 系统目录'的选项

当安装结束后,在服务器上添加`C:\openssl-win32\bin`对Windows系统的路径变量:

1. 在Windows桌面,右键单击我的电脑图标,并选择`Properties`。

2. 选择Advanced选项卡的`System Properties`显示菜单,单击`Environment Variables`按钮。

3. 在`System Variables`,选择路径,然后单击`Edit`按钮。`Edit System Variable`应该弹出对话框。

4. 添加`';C:\openssl-win32\bin'`到最后(注意分号)。

5. 按3次OK。

6. 检查是否正确地集成到`OpenSSL Path`变量，通过打开一个新的命令控制台(`Start>Run>cmd.exe`)来验证OpenSSL是可用的:

```sql
    Microsoft Windows [Version ...]
	Copyright (c) 2006 Microsoft Corporation. All rights reserved.

	C:\Windows\system32>cd \

	C:\>openssl
	OpenSSL> exit <<< If you see the OpenSSL prompt, installation was successful.
	
	C:\>
```

根据不同版本的Windows,前面的路径设置指令可能略有不同。

安装完OpenSSL后,使用说明类似示例1(在这一节前面讲到的),使用以下更改:

* 替换以下的Unix命令:

```sql
    # Create clean environment
	shell> rm -rf newcerts
	shell> mkdir newcerts && cd newcerts
```

在Windows上,使用这些命令:

```sql
    # Create clean environment
	shell> md c:\newcerts
	shell> cd c:\newcerts
```

* 当一个'\'字符显示的命令行,这'\'字符必须被删除还有命令行输入所在的行。

生成证书和密钥文件后,使用它们来测试SSL连接,请参考[6.3.9.3，“使用SSL进行连接”][06.03.09.03]。


[13.07.01]:../Chapter_13/13.07.01_Account_Management_Statements.md
[06.02.02]:06.02.02_Privilege_System_Grant_Tables.md
[06.03.01]：06.03.01_User_Names_and_Passwords.md
[06.03.07]:06.03.07_Pluggable_Authentication.md
[04.04.07]:../Chapter_04/04.04.07_mysql_upgrade_Check_and_Upgrade_MySQL_Tables.md
[10.01.04]:../Chapter_10/10.01.04_Connection_Character_Sets_and_Collations.md
[02.10.02]:../Chapter_02/02.01.02_Choosing_Which_MySQL_Distribution_to_Install.md
[13.07.01]:../Chapter_13/13.07.01_Account_Management_Statements.md
[06.01.02.04]:06.01.02_Keeping_Passwords_Secure.md#06.01.02.04
[02.09.04]：../Chapter_02/02.09.04_MySQL_Source-Configuration_Options.md
[02.10.02]:../Chapter_02/02.10.02_Securing_the_Initial_MySQL_Accounts.md
[04.02.02]:../Chapter_04/04.02.02_Connecting_to_the_MySQL_Server.md
[04.04.07]:../Chapter_04/04.04.07_mysql_upgrade_Check_and_Upgrade_MySQL_Tables.md
[05.01.04]:../Chapter_05/05.01.04_Server_System_Variables.md
[05.01.07]:../Chapter_05/05.01.07_Server_SQL_Modes.md
[05.01.08.02]:../Chapter_05/05.01.08_Server_Plugins.md#05.01.08.02
[06.01.02]:06.01.02_Keeping_Passwords_Secure.md
[06.01.02.01]:06.01.02_Keeping_Passwords_Secure.md#06.01.02.01
[06.02.02]:06.02.02_Privilege_System_Grant_Tables.md
[06.02.04]:06.02.04_Access_Control_Stage_1_Connection_Verification.md
[06.03.05]:06.03.05_Assigning_Account_Passwords.md
[06.03.07]:06.03.07_Pluggable_Authentication.md
[06.03.07.02]：06.03.07_Pluggable_Authentication.md#06.03.07.02
[06.03.07.03]:06.03.07_Pluggable_Authentication.md#06.03.07.03
[06.03.07.04]:06.03.07_Pluggable_Authentication.md#06.03.07.04
[06.03.07.05]:06.03.07_Pluggable_Authentication.md#06.03.07.05
[06.03.07.05.01]:06.03.07_Pluggable_Authentication.md#06.03.07.05.01
[06.03.07.05.03]:06.03.07_Pluggable_Authentication.md#06.03.07.05.03
[06.03.07.06]:06.03.07_Pluggable_Authentication.md#06.03.07.06
[06.03.07.07]:06.03.07_Pluggable_Authentication.md#06.03.07.07
[06.03.07.08]:06.03.07_Pluggable_Authentication.md#06.03.07.08
[06.03.07.09]:06.03.07_Pluggable_Authentication.md:x#06.03.07.09
[06.03.08]:06.03.08_Proxy_Users.md
[06.03.09]:06.03.09_Using_SSL_for_Secure_Connections.md
[06.03.09.02]:06.03.09_Using_SSL_for_Secure_Connections.md#06.03.09.02
[06.03.09.03]:06.03.09_Using_SSL_for_Secure_Connections.md#06.03.09.03
[06.03.09.04]:06.03.09_Using_SSL_for_Secure_Connections.md#06.03.09.04
[06.03.10]:06.03.10_Connecting_to_MySQL_Remotely_from_Windows_with_SSH.md
[10.01.04]:../Chapter_10/10.01.04_Connection_Character_Sets_and_Collations.md
[12.13]：../Chapter_12/12.13.00_Encryption_and_Compression_Functions.md
[13.07.01]:../Chapter_13/13.07.01_Account_Management_Statements.md
[13.07.01.03]:../Chapter_13/13.07.01_Account_Management_Statements.md#13.07.01.03
[13.07.01.04]:../Chapter_13/13.07.01_Account_Management_Statements.md#13.07.01.04
[13.07.05.22]:../Chapter_13/13.07.05_SHOW_Syntax.md#13.07.05.22
[16.03.07]:../Chapter_16/16.03.07_Setting_Up_Replication_Using_SSL.md
[19.06]:../Chapter_19/19.06.00_Access_Control_for_Stored_Programs_and_Views.md
[22.02.05.05]：../Chapter_22/22.02.05_ConnectorNet_Programming.md#22.02.05.05
[22.08.07.33]：../Chapter_22/22.08.07_C_API_Function_Descriptions.md#22.08.07.33
[22.08.07.68]：../Chapter_22/22.08.07_C_API_Function_Descriptions.md#22.08.07.68
[23.02.04.09]：../Chapter_22/22.02.04_ConnectorNet_Tutorials.md#23.02.04.09
[23.02.04.09.04]：../Chapter_22/22.02.04_ConnectorNet_Tutorials.md#23.02.04.09.04
[E.09]:../Appendix_E/E.09.00_Restrictions_on_Pluggable_Authentication.md
