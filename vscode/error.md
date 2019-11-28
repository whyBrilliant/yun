# vscode 常见错误

1. 无法加载文件 D:\nodejs\node_global\xxx.ps1，因为在此系统上禁止运行脚本，有关详细信息，请参阅...

   ```
   解决报错：
   （1）以管理员身份运行vs code
   
   （2）在终端执行：get-ExecutionPolicy，显示Restricted（表示状态是禁止的）
   
   （3）在终端执行：set-ExecutionPolicy RemoteSigned
   
   （4）在终端执行：get-ExecutionPolicy，显示RemoteSigned
   
   ```

   

      

