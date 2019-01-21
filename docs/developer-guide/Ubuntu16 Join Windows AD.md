### 准备工作

- 下载 [PowerBroker Identity Services](https://github.com/BeyondTrust/pbis-open/releases)

### 开始

- 切换到root用户

```
    su root
```

- 创建临时账户，如 `ITC180012`

```
    useradd -g root -d /home/ITC180012 -m ITC180012
    passwd ITC180012
```

- 将新建的账户添加到域控中

```
    domainjoin-cli join --disable ssh itc.inventec ITC180012
    # domainjoin-cli leave --disable ssh ITC180012

    /opt/pbis/bin/config AssumeDefaultDomain true
    /opt/pbis/bin/config UserDomainPrefix itc
    # /opt/pbis/bin/config --show AssumeDefaultDomain
    # /opt/pbis/bin/config --show UserDomainPrefix
```

- 重启

```
    reboot
```

- 删除临时账户

```
    userdel -r ITC180012
```
