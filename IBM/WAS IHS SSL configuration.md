
# WAS和IHS配置SSL 加密传输

## 1. 密钥库

1. IHS密钥库和证书 (IHSkey.kdb)  <was_install>/bin/ikeyman.bat

   1. 建立密钥数据库文件
      1. 进入新建密钥数据库文件界面。
      2. 密钥数据库类型选择CMS，确定文件名（IHSkey.kdb）及文件路径（/opt/IBM/secret/）。
      3. 设置该密钥数据库文件的密码(secret)，并选择将stashpassword to file 密码存储到文件。
      4. 生成密钥数据库文件，将在/opt/IBM/secret/下生成IHSkey.kdb、IHSkey.rdb、IHSkey.sth这3个文件。

   2. 新建自签名证书  (New Self-Signed)
      1. 进入新建自签署证书界面。
      2. 填写该界面各项数据。
         * 密钥标签： IHScert
         * 版本： X509 V3
         * 密钥大小：  2048
         * 算法：SHA256WithRSA
         * 公共名： your_host_name
         * 组织：  Your Organization
         * 国家或地区： CA
         * 有效期： 3650 天 
      3. 生成自签名证书。

   3. 抽取公用自签名证书
      1. 在密钥数据库文件内容中选择“个人证书”，并选择1.1.2中生成的自签署证书（IHScert）。
      2. 单击抽取证书。
      3. 在抽取证书界面选择数据类型“Base64 编码的 ASCII 数据”，并输入证书名（IHScert .arm）。
      4. 抽取证书。

2. WAS密钥库和证书 (WASKey.jks)

   1. 建立密钥库文件
      1. 进入  WebSphere Admin Console-> 安全性-》 SSL证书和密钥管理-》密钥库和证书-》新建。
      2. 确定密钥库文件名（WASkey.jks），路径（/opt/IBM/secret/ WASkey.jks）及类型（JKS）。Password:secret 
      3. 确定生成，在/opt/IBM/secret/下将生成WASkey.jks文件。

   2. 生成自签署证书
      1. 选择上步中生成的密钥库文件-WASkey.jks> 个人证书-创建自签署证书(Create Self-Signed Certificate)。
      2. 输入各数据项  别名：WAScert
      3. 生成自签署证书。

   3. 抽取公用自签名证书
      1. 选择前一步中建立的自签署证书（WAScert），点击抽取按钮Extract。
      2. 在抽取证书界面选择数据类型“Base64 编码的 ASCII 数据”，并输入证书名（WAScert.arm）。
      3. 确定抽取。默认保存在${was_install}/profiles/Dmgr01/etc
      4. Export "root"  from signer certificate to WASroot.arm

## 2. 交换证书

1. 在IHS中导入WAS中抽取的证书  (use ikeyman)
   1. 在密钥数据库文件内容中选择“签署人证书”，并点击“添加”按钮。
   2. 确定各项。
      * 类型：Base64 编码的 ASCII 数据
      * 文件名：1.2.3中抽取的自签名证书（WAScert .arm）
      * 位置：文件存放路径
   3. 确定进入下一步。
   4. 输入证书标识。
   5. 确定导入。
   6. 重复 1-5 同上导入(WASroot.arm) as Signer Certificates

2. 在WAS中导入IHS中抽取的证书
   1. 进入  WebSphere Admin Console > 安全性 > SSL 证书和密钥管理 > 密钥库和证书 > WASkey > 签署者证书。
   2. 单击Add按纽。
   3. 确定各项
      * 别名：IHScert
      * 文件名：1.1.3中抽取出的公用自签名证书（IHScert .arm）
      * 类型：Base64 编码的 ASCII 数据
   4. 确定导入。
   5. 重复1-iv 导入到 CellDefaultTrustStore

## 3. 配置WAS

1. 配置SSL
   1. 生成SSL配置
      1. 进入WebSphere Admin Console -> 安全性-》 SSL证书和密钥管理-》ssl  配置-》新建。
         * 名称:自由定义（WASssl）
         * 信任库名：1.2.i 中建立的密钥库文件（WASkey）
         * 密钥库名：1.2.i 中建立的密钥库文件（WASkey）
      2. 点获取证书别名按钮
         * 缺省服务器证书别名：选择在1.2.ii 中创建的自签署证书WAScert
         * 缺省客户机证书别名：WAScert
      3. 点确定生成SSL配置。

   2. 使用SSL配置  进入【Manage endpoint security configurations】
       1. Inbound>[your_cell]>nodes>[your_node]>servers>[your_server]
       2. Specific SSL configuration for this endpoint
          1. Check Override inherited values
          2. SSL配置，选为:3.1.1中生成的SSL配置（WASssl），Update certificate alias list  Alias 设置为“WAScert”然后确定。
       3. Outbound>[your_cell]>nodes>[your_node]>servers>[your_server]
       4. Specific SSL configuration for this endpoint
          1. Check Override inherited values
          2. SSL配置，选为:3.1.1中生成的SSL配置（WASssl），Update certificate alias list  Alias 设置为“WAScert”然后确定。

2. 配置plugin (CMSkeyStore)
   1. 进入WebSphere Admin Console -> 安全性-》 SSL证书和密钥管理 > CMSKeyStore
       1. Signer Certificates
          1. add /opt/IBM/secret/ WAScert.arm as WAScert
          2. add /opt/IBM/secret/WASroot.arm as WASroot
          3. Import Personal Certificates
             1. Key store file /opt/IBM/secret/IHSkey.kdb
             2. Key file password “secret” click 【Get Key File Aliases】
             3. Certificate alias to import select [IHScert]
             4. Imported as IHScert
             5. OK

   2. 进入服务器 > Web 服务器 > Web_server_name。
        1. 在“其他属性”下，单击插件属性 > 插件属性。
        2. 点击【Copy to Web server key store directory】（注意：生成与传播后plugin-cfg.xml文件路径与httpd.conf里的配置路径是否一致）。

## 4. 配置IHS

在IHS的httpd.conf文件中添加以下内容：

```ini
LoadModule ibm_ssl_module modules/mod_ibm_ssl.so  
<IfModule mod_ibm_ssl.c>
   Listen 443
   <VirtualHost *:443>
      SSLEnable
      SSLClientAuth Optional
      SSLServerCert IHScert
   </VirtualHost>
  </IfModule>
SSLDisable
KeyFile "/opt/IBM/secret/IHSkey.kdb"
```

***注意*** deploy application module 一定要同时选定web server 和 application server 否则 就会出现404 requested url was not found on this server.
