# Windows端口占用但无法查到相应进程问题  
在IDEA使用 **2233** 端口启动Java程序时，报错
```java
 java.net.BindException: Address already in use: NET_Bind
 ```
 这是由端口被占用造成的。  
 但当使用命令查看使用该端口的进程时，却没有查到任何结果。  

```text
PS C:\Users\15820> netstat -nao | findstr "2333"
PS C:\Users\15820>
```

经过搜索，确定原因是因为这个端口被Windows保留了。  
因为我在Windows中启用了Hyper-V，而Hyper-V 会将动态端口中的几段范围的端口保留给自己使用，用户的应用程序无法使用这些端口。
  
### **使用以下命令查看Windows中被保留的端口范围**
```text
netsh interface ipv4 show excludedportrange protocol=tcp
```
我的系统中被保留的范围是这些：  

 ```text
 PS C:\Users\15820> netsh interface ipv4 show excludedportrange protocol=tcp

协议 tcp 端口排除范围

开始端口    结束端口
----------    --------
      1037        1136
      1137        1236
      1340        1439
      1637        1736
      2009        2108
      2180        2279
     12528       12627
     12728       12827
     12828       12927
     12928       13027
     50000       50059     *

* - 管理的端口排除。
 ```

可以看到端口 **2233** 就在其中。
    
## **解决办法**  
### **办法一**  
更换程序端口，治标不治本，也许Windows下次更新把新端口也纳入保留范围了。  

### **办法二**
更改Windows保留端口的范围  
命令为：  
```shell
netsh int ipv4 set dynamicport tcp start=xxxx num=xxxx
```
以管理员身份打开PowerShell，设置tcp ipv4的动态端口范围为start至(start + num)，重启电脑。  
比如，start为2234，num为1000，最终范围为 **2234-3234**。  
这样Hyper-V就只能从这个范围中设置一些端口为自己保留。  

### **办法三**  
将制定端口从保留范围中释放出来。
管理员打开PowerShell，执行以下命令并重启：
```shell
# 临时禁用Hyper-V，需重启电脑
dism.exe /Online /Disable-Feature:Microsoft-Hyper-V

# 下列命令二选一
# 保留 2233~2234 这10个端口给应用程序使用
netsh int ipv4 add excludedportrange protocol=tcp startport=2233 numberofports=10
# 保留 2233 端口给应用程序使用
netsh int ipv4 add excludedportrange protocol=tcp startport=2233 numberofports=1

# 重新启用Hyper-V
dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All
```  
  
内容参考自 https://www.cnblogs.com/zsmumu/
