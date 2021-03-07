# [iOS tcpdump抓包方法(需越狱)](https://www.cnblogs.com/wqxlcdymqc/p/3225276.html)

## 前提条件：机器要破解，cydia能打开

* 需要工具  
  1.openssh  
  2.tcpdump  

安装工具方法：  
1.连接网络，打开cydia  
2.确认Cydia设置为开发者模式(管理->设置->开发者)，在Cydia时面搜索openssh,tcpdump并安装  

连接方法：  
1.找到一台电脑与iPhone连接同一个Wifi,在PC能ping通iPhone  
2.在PC的命令行界面输入ssh root@iphoneip  
3.提示连接ssh,输入yes  
4.输入密码alpine  
5.正常显示登录iPhone成功命令行前面为"iphone手机名字"  
6.输入tcpdump，正常可以看到很多包信息显示  
7.ctrl+c停止抓包  
8.tcpdump带参简介  
  a.tcpdump -X -s0 -w /tmp.cap  //抓全包写文件  
  b.tcpdump -X -s0 host www.qq.com //主机全包  
  c.tcpdump -X -s0 port 14000   //抓端口全包  
  d.可以配合以上参数抓包  
9.在PC上用Ethereal分析抓包文件数据*







lsof -i -a -p <pid> #我们上面查看到的进程pid

-i表示列出所有网络连接

-a表示指定当前用户

-p表示某个进程（pid）
