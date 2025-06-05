以下是关于Web漏洞和注入攻击的中文总结：

### Web漏洞与注入攻击
- **定义**：注入攻击通过欺骗应用程序将未预期的命令包含在发送给解释器的数据中。解释器会将这些字符串解释为命令并执行。常见的解释器包括SQL、shell（如cmd.exe, bash）、LDAP、XPath等。
- **关键思想**：应用程序的输入数据被解释器执行为代码。

### SQL注入
- **过程**：
  1. 应用程序向用户发送表单。
  2. 攻击者提交带有SQL利用数据的表单。
  3. 应用程序构建包含利用数据的字符串。
  4. 应用程序将SQL查询发送到数据库。
  5. 数据库执行查询，包括利用部分，并将结果返回给应用程序。
  6. 应用程序将结果返回给用户。

- **示例代码**：
  ```php
  $link = mysql_connect($DB_HOST, $DB_USERNAME, $DB_PASSWORD) or die("Couldn't connect: " . mysql_error());
  mysql_select_db($DB_DATABASE);
  $query = "SELECT COUNT(*) FROM users WHERE username='$username' AND password='$password'";
  $result = mysql_query($query);
  ```

- **攻击类型**：
  - **未经授权访问**：通过构造恶意的SQL语句绕过身份验证。例如，设置密码为 `' OR 1=1--`，这会使查询变为 `SELECT COUNT(*) FROM users WHERE username='user' AND password='' OR 1=1--`，从而总是返回真，允许未经授权的访问。
  - **数据库修改**：通过构造多个SQL语句修改数据库。例如，设置密码为 `foo'; DELETE FROM table users WHERE username LIKE '%'`，这会使数据库执行两个SQL语句，删除所有用户的记录。

- **检测方法**：
  - 提交单引号作为输入，如果导致错误，则应用程序可能易受攻击。
  - 提交两个单引号，数据库使用 `''` 表示字面意义上的单引号，如果错误消失，则应用程序可能易受攻击。

- **常见入口点**：
  - `SELECT` 语句中的 `WHERE` 和 `ORDER BY` 子句，以及表名和列名。

### SQL注入的影响
- **敏感信息泄露**。
- **声誉下降**。
- **敏感信息修改**。
- **失去对数据库服务器的控制**。
- **数据丢失**。
- **拒绝服务**。

### 根本原因
- 构建SQL命令字符串时直接使用用户输入是危险的。

### 缓解措施
- **无效的缓解措施**：黑名单过滤，容易被绕过。
- **部分有效的缓解措施**：白名单过滤、预编译查询。

#### 黑名单
- **问题**：只能过滤已知的恶意字符，如单引号，但无法覆盖所有情况。例如，URL转义字符、Unicode编码字符等都可能导致绕过。

#### 白名单
- **优点**：只接受符合安全字符集的输入，拒绝不符合的输入，而不是尝试修复输入。

#### 预编译查询
- **原理**：创建一个带有占位符的SQL语句字符串，编译成内部形式，然后使用参数列表执行该预编译查询。
  - **不安全版本**：
    ```java
    Statement s = connection.createStatement();
    ResultSet rs = s.executeQuery("SELECT email FROM member WHERE name=" + formField);
    ```
  - **安全版本**：
    ```java
    PreparedStatement ps = connection.prepareStatement("SELECT email FROM member WHERE name=?");
    ps.setString(1, formField);
    ResultSet rs = ps.executeQuery();
    ```

### 其他类型的注入
- **Shell注入**。
- **脚本语言注入**。
- **文件包含**。
- **XML注入**。
- **XPath注入**。
- **LDAP注入**。
- **SMTP注入**。

### 跨站脚本（XSS）
- **定义**：恶意客户端脚本被注入到应用程序输出中，并由用户的浏览器执行。如果不过滤HTML和JavaScript，可能会导致严重的安全问题。
- **影响**：可以用于接管用户的浏览器，进行各种攻击。

### 总结
介绍了Web漏洞和注入攻击，特别是SQL注入的原理、攻击方式、检测方法及其影响。同时，还讨论了如何通过预编译查询、白名单过滤等措施来缓解这些攻击。此外，文档还提到了其他类型的注入攻击，如Shell注入、跨站脚本（XSS）等，强调了过滤用户输入的重要性。这些知识对于理解和防范Web应用中的安全威胁非常有帮助。