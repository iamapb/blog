## Linux SSH登录服务器报ECDSA host key "ip地址" for has changed and you have requested strict checking错误

### ssh问题
```xml
Linux SSH命令用了那么久，第一次遇到这样的错误：ECDSA host key "ip地址" for  has changed and you have requested strict checking.记录下方便记忆。
```
### 解决办法
```xml
    ssh-keygen -R "你的远程服务器ip地址"
```