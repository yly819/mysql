

### 6.3.9. 使用SSL进行安全连接 ###

MySQL支持安全(加密)连接在MySQL客户端和服务器之间使用安全套接字层(SSL)协议。本节讨论如何使用SSL连接。对于信息如何要求用户使用SSL连接,参考讨论关于`GRANT`语句的子句`REQUIRE`在[13.7.1.4,“授权语法”][13.07.01.04]。

标准配置MySQL的目的是要尽可能快,所以默认情况下不使用加密连接。对于应用程序来说,它需要所提供的安全性加密的连接,花费一定的时间来计算加密数据是值得的。

MySQL允许对每个连接加密。你可以根据单独的应用程序的要求选择一个未加密的连接或一个安全的加密的SSL连接。

安全连接是基于OpenSSL API并有效地通过MySQL C API。可以复制使用C API,所以可以在主从复制之间使用安全连接。参考[16.3.7,"设置主从间复制使用SSL”][16.03.07]。

另一种方法从内部使用SSH安全地连接到MySQL服务器主机。有一个相关的例子,请参考[6.3.10,“从Windows系统中使用SSH远程连接到MySQL”][06.03.10]。

#### 6.3.9.1. SSL基本概念 ####

了解MySQL使用SSL,需要解释SSL和X509一些基本的概念。那些熟悉这些概念的人可以跳过这个讨论部分。

默认情况下,MySQL在客户机和服务器之间使用未加密的连接。这意味着有人可以通过访问网络可以看到你所有的通信和查看被发送或收到了的数据。他们甚至可以在传输时改变客户端和服务器之间的数据。提高一点安全性,你可以压缩客户机和服务器流量当调用客户端程序的时候通过使用`--compress`选项。然而,这并不能防止有心攻击者。

当你需要移动信息网中使用一种安全的方式,一个加密连接是不可能就够的。加密是使任何类型的数据不可读的一种方式。加密算法必须包括安全元素抵抗多种已知的攻击如改变加密消息的命令或重播两次数据。

SSL是一种协议,它使用不同的加密算法,以确保接收到的数据在一个公共的网络可以被信任。它有机制来检测发生变化的数据,丢失的,或被重新发送的。SSL还包含有算法,提供使用X509标准的身份验证。

X509它可以识别有人在互联网上。它是最常用的电子商务应用程序。在基本条款,应该有一些实体称为“证书权威”(CA)分配电子证书给所有需要他们的人。依赖不对称加密证书算法有两个加密密钥(公钥和密钥)。一个证书所有者可以显示证书给另一方作为身份证明。证书有主人的公钥。任何数据的加密与解密这个公钥可以只使用相应的密钥,这是的所有者所持有的证书。

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

这个`REQUIRE`语句允许其他ssl相关限制。这些可以用于比`REQUIRE SSL`更严格需求。在[13.07.01.04. “授权语法”][13.07.01.04]有描述`REQUIRE`, 提供补充的细节部分,SSL命令选项可能或必须由客户指定用于连接使用的账户创建使用各种REQUIRE的选项。

* `--ssl-ca=file_name`

PEM格式的、可信的SSL官方证书及文件路径。这个选项的含义和`——ssl`一样。

如果你使用SSL当建立一个客户端连接,你可以告诉客户端进行身份验证的服务器证书通过指定可以用`——ssl ca`也也可以用`ssl-capath`。服务器还是验证客户端根据在客户端建立使用`GRANT`语句的需求,它仍然是使用`——ssl ca/——ssl-capath`的值, 在启动时传递给服务器。

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
[06.03.07.05.02.01]：06.03.07_Pluggable_Authentication.md#06.03.07.05.02.01
[06.03.07.05.02.02]:06.03.07_Pluggable_Authentication.md#06.03.07.05.02.02
[06.03.07.05.03]:06.03.07_Pluggable_Authentication.md#06.03.07.05.03
[06.03.07.06]:06.03.07_Pluggable_Authentication.md#06.03.07.06
[06.03.07.06.01]:06.03.07_Pluggable_Authentication.md#06.03.07.06.01
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
