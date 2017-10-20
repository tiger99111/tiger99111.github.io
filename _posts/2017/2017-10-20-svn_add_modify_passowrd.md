---
layout: post
title: VisualSVN Server 增加自助修改密码页面(支持2.1-3.6最新版)
date: 2017-10-20 
tags: 工具
---

&ensp; &ensp; 如果不用VisualSVN客户端的话，VisualSVN Server只能在服务器端修改密码，对管理来说很不方便。
&ensp; &ensp; 网上大部分给 VisualSVN 增加自助修改密码的补丁都是基于 2.5.x 版本的，也有几个用于 3.5.x/3.6.x 版本，多数还是用 cgi 方式，而且要么像 csdn 那样藏着掖着，要么也没个详细的说明。
&ensp; &ensp; VisualSVN Server 帐号及密码保存在 htpasswd 文件里，除了 cgi 有以下几种修改方式：

&ensp; &ensp;1、使用 VisualSVN Server Manager 管理工具重置密码。 
&ensp; &ensp;2、通过 WMI 里用 PowerShell脚本更改。示例如下： 
&ensp; &ensp; 来源：[http://stackoverflow.com/questions/4354356/visualsvn-server-password-change](http://stackoverflow.com/questions/4354356/visualsvn-server-password-change)
```
$svnuser = Get-WmiObject -Namespace Root\VisualSVN `
-ComputerName svn.hostname.com `
-query "select * from VisualSVN_User where name = 'username'"
$svnuser.SetPassword('123456')
```
&ensp; &ensp;3、 使用 Windows 版 Apache 的 htpasswd.exe 命令更改。
&ensp; &ensp; 来源：[http://www.cnblogs.com/rgqancy/p/4679881.html](http://www.cnblogs.com/rgqancy/p/4679881.html)
&ensp; &ensp;该方法是使用 php 页面来调用 htpasswd.exe 修改密码，方便用户通过网页修改，下面讲解配置要点。
&ensp; &ensp;我使用的操作系统是 Windows Server 2008R2 x64，安装了 VisualSVN Server 3.5.6 x64 带 Apache 2.2.31 x64 的版本，默认安装路径。
&ensp; &ensp;从 Apache 官方网站下载完整的 Apache 2.2.31 x64 版本，从里面提取一个文件 htpasswd.exe 放到:
|C:\Program Files\VisualSVN Server\bin\htpasswd.exe|
| ------------- :|
      
&ensp; &ensp;Apache 2.2.x 要以 handler 方式加载 php 模块，只有 php 5.2-5.4 的 Thread Safe 版本才带 php5apache2_2.dll 文件，php 5.5 及之后的版本只能和 Apache 2.4.x 搭配了，所以选定 php 5.4 版本。
&ensp; &ensp;<font color=##FF0000>**特别注意:**如果用 VisualSVN Server x64 就必须找 x64 的 php！</font>
&ensp; &ensp;否则 Apache 加载 php 模块会提示错误 Cannot load php5apache2_2.dll into server因为 Apache x64 无法使用 php x86。
使用 32 位版本的 VisualSVN Server 比较简单，因为 php 官网都是 x86 版本:
[http://windows.php.net/downloads/releases/archives/](http://windows.php.net/downloads/releases/archives/)
使用 64 位版本的 VisualSVN Server 就得找第三方编译的 php x64 版本了，提供两个下载：
[https://www.anindya.com/php-5-4-12-and-5-3-22-x64-64-bit-for-windows/](https://www.anindya.com/php-5-4-12-and-5-3-22-x64-64-bit-for-windows/)
[http://www.apachelounge.com/viewtopic.php?t=6123](http://www.apachelounge.com/viewtopic.php?t=6123)
&ensp; &ensp;我下载的是 php-5.4.36-Win32-VC9-x64.zip，下载后解压到 C:\Program Files\VisualSVN Server\php 文件夹。把 php.ini-production 文件重命名为 php.ini 即可，其他不用配置。
&ensp; &ensp;修改空文件 <font color=###0000FF>C:\Program Files\VisualSVN Server\conf\httpd-custom.conf</font>
内容如下：
```
LoadModule php5_module "php/php5apache2_2.dll"
<IfModule php5_module>
    AddType application/x-httpd-php .php
	    DirectoryIndex index.html index.php
		</IfModule>
# 配置 php.ini 的路径
		PHPIniDir "php"
		```
		&ensp; &ensp;新建一个 php 文件放到 <font color=###0000FF>C:\Program Files\VisualSVN Server\htdocs\pw\index.php</font>
		内容如下（做了一点小修改，增加返回及跳转页面的处理）：
		```
		<?php
		/***************************************************************/
		$passwdfile="C:\Repositories\htpasswd";
		$htpasswdPath = "C:\Program Files\VisualSVN Server\bin\htpasswd.exe";
		/***************************************************************/

		$username = $_SERVER["PHP_AUTH_USER"]; //经过 AuthType Basic 认证的用户名
		$authed_pass = $_SERVER["PHP_AUTH_PW"]; //经过 AuthType Basic 认证的密码
		$input_oldpass = (isset($_REQUEST["oldpass"]) ? $_REQUEST["oldpass"] : ""); //从界面上输入的原密码
		$newpass = (isset($_REQUEST["newpass"]) ? $_REQUEST["newpass"] : ""); //界面上输入的新密码
		$repeatpass = (isset($_REQUEST["repeatpass"]) ? $_REQUEST["repeatpass"] : ""); //界面上输入的重复密码
		$action = (isset($_REQUEST["action"]) ? $_REQUEST["action"] : ""); //以hide方式提交到服务器的action

		if ($action!="modify") {
			    $action = "view";
		} else if ($authed_pass!=$input_oldpass) {
			    $action = "oldpasswrong";
		} else if (empty($newpass)) {
			    $action = "passempty";
		} else if ($newpass!=$repeatpass) {
			    $action = "passnotsame";
		} else{
			    $action = "modify";
		}
?>

<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=GBK">
	    <title>Subversion 在线自助密码修改</title>
		</head>
		<body>

		<?php
		//action=view 显示普通的输入信息
		if ($action == "view") {
			?>
				<script language = "javaScript">
				<!--
				function loginIn(myform) {
					    var newpass=myform.newpass.value;
						    var repeatpass=myform.repeatpass.value;

							    if (newpass=="") {
									        alert("请输入密码！");
											        return false;
													    }

								    if (repeatpass=="") {
										        alert("请重复输入密码！");
												        return false;
														    }

									    if (newpass!=repeatpass) {
											        alert("两次输入密码不一致，请重新输入！");
													        return false;
															    }
										return true;
				}
			//-->
			</script>

				<style type="text/css">
				<!--
				    table {
						        border: 1px solid #CCCCCC;
										        background-color: #f9f9f9;
												        text-align: center;
														        vertical-align: middle;
																        font-size: 9pt;
																		        line-height: 15px;
																				    }
			    th {
					        font-weight: bold;
							        line-height: 20px;
									        border-top-width: 1px;
											        border-right-width: 1px;
													        border-bottom-width: 1px;
															        border-left-width: 1px;
																	        border-bottom-style: solid;
																			        color: #333333;
																						           background-color: f6f6f6;
																								       }
				    input{
						        height: 18px;
										    }
					    .button {
							        height: 20px;
											    }
						-->
							</style>

							<br><br><br>
							<form method="post">
							<input type="hidden" name="action" value="modify"/>
							<table width="220" cellpadding="3" cellspacing="8" align="center">
							<tr>
							<th colspan=2>Subversion 密码修改</th>
							</tr>
							<tr>
							<td>用 户 名：</td>
							<td align="left"> <?php echo $username?></td>
							</tr>
							<tr>
							<td>原 密 码：</td>
							<td><input type=password size=12 name=oldpass></td>
							</tr>
							<tr>
							<td>用户密码：</td>
							<td><input type=password size=12 name=newpass></td>
							</tr>
							<tr>
							<td>确认密码：</td>
							<td><input type=password size=12 name=repeatpass></td>
							</tr>
							<tr>
							<td colspan=2>
							<input onclick="return loginIn(this.form)" class="button" type=submit value="修 改">
							<input name="reset" type=reset class="button" value="取 消">
							<a href="/"><input name="return" type=button class="button" value="返 回"></a>
							</td>
							</tr>
							</table>
							</form>
							<?php
		} else if ($action == "oldpasswrong") {
			    $msg="原密码错误！";
		} else if ($action == "passempty") {
			    $msg="请输入新密码！";
		} else if ($action == "passnotsame") {
			    $msg="两次输入密码不一致，请重新输入！";
		} else {
			//    $passwdfile="D:\SVN_Repositories\htpasswd";
			//    $command='"d:\VisualSVN Server\bin\htpasswd.exe" -b '.$passwdfile." ".$username." ".$newpass;
			    $command='"'. $htpasswdPath. '" -b '.$passwdfile." ".$username." ".$newpass;
				    system($command, $result);
					    if ($result==0) {
							        $msg_succ="用户[".$username."]密码修改成功，请用新密码登陆.";
									    } else {
											        $msg="用户[".$username."]密码修改失败，返回值为".$result."，请和管理员联系！";
													    }
		}

if (isset($msg_succ)) {
	?>
		<script language="javaScript">
		<!--
		alert("<?php echo $msg_succ?>");
	window.location.href="/"
		//-->
		</script>
		<?php
} else if (isset($msg)) {
	?>
		<script language="javaScript">
		<!--
		alert("<?php echo $msg?>");
	window.location.href="<?php echo $_SERVER["PHP_SELF"]?>"
		//-->
		</script>
		<?php
}
?>
</body>
</html>
```
&ensp; &ensp;在header增加超链接麻烦，所以我就在页脚增加修改密码的链接，
修改文件 <font color=###0000FF>C:\Program Files\VisualSVN Server\WebUI\index.html</font>
内容如下:
 ```
  <footer>
  Powered by <a href="http://www.visualsvn.com/server/">VisualSVN Server</a>. &copy; 2005-2016 VisualSVN Limited.
  <br /><br /><a href="/pw">自助修改密码</a>
  </footer>
  ```
  设置完成后，重新启动下Visual SVN server，然后浏览器进入即可看到效果。
      
  **以下为效果图：**
  ![效果图1](http://img.blog.csdn.net/20171020141743397?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvVGlnZXI5OTExMQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
      
  ![这里写图片描述](http://img.blog.csdn.net/20171020150426134?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvVGlnZXI5OTExMQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


  本文转自：(http://oicu.cc.blog.163.com/blog/static/1230394712016102312546504/)
