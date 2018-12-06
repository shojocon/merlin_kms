# 梅林软件中心插件开发教程详解
[（教程镜像链接）](https://www.zybuluo.com/9168131/note/994332)
------

### 下面我们将按照以下顺序进行详解
1．	梅林插件介绍

2．	插件目录结构

3．	插件的命名规范

4．	插件相关参数提交及保存

5．	插件如何调用shell执行相关功能

6．	插件安装&卸载脚本

7．	插件打包成离线安装包


------
# 1.梅林插件介绍 

>  梅林插件通俗来说是对路由器本身功能的一个扩展，让其原本不支持的功能通过扩展插件的方式实现，那么功能有多强呢？有句话叫：贫穷限制了我们的想象力。只有想不到，没有做不到！

------

# 2.插件目录结构

                                   kms.tar.gz
                                       |  
        -----------------------------------------------------
        |       |        |         |        |               |
       bin     res     script     web    install.sh     uninstall.sh
        |       |        |         |
     vlmcsd     |        |   Module_kms.asp
            icon-kms.png |
                       kms.sh
                       
> - bin：执行文件目录 ，二进制执行文件都放在里面
> - res：静态资源目录 ，件的图标、css样式、javascript脚本等
> - Script：shell脚本目录 ，当页面提交保存之后需要执行的shell脚本都放在里面
> - Web：插件的页面文件
> - install.sh：插件安装脚本
> - uninstall.sh：插件卸载脚本

---

# 3.插件的命名规范


> 以kms插件为例所有插件都必须以此规范开发
压缩包名：**kms**.tar.gz
Web页面：Module_**kms**.asp
Icon图标：icon-**kms**.png
以上3个文件必须以这种方式定义，**粗体**的可以更改为你要开发插件名。
例如 **xxx**.tar.gz
那么web页面就是: Module_**xxx**.asp
Bin、script不做限制，子页面不做限制

---


# 4.插件相关参数提交及保存


    首先说一下插件的运作原理：
    页面设置保存 – 提交至 dbus 保存参数–dbus执行提交的shell脚本名称 –        shell脚本通过dbus 获取所需执行的参数
    
### 这是kms插件的页面代码 Module_kms.asp


```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
	<head>
		<meta http-equiv="X-UA-Compatible" content="IE=Edge" />
		<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
		<meta HTTP-EQUIV="Pragma" CONTENT="no-cache" />
		<meta HTTP-EQUIV="Expires" CONTENT="-1" />
		<link rel="shortcut icon" href="images/favicon.png" />
		<link rel="icon" href="images/favicon.png" />
		<title>软件中心 - 系统工具</title>
		<link rel="stylesheet" type="text/css" href="index_style.css" />
		<link rel="stylesheet" type="text/css" href="form_style.css" />
		<link rel="stylesheet" type="text/css" href="usp_style.css" />
		<link rel="stylesheet" type="text/css" href="ParentalControl.css">
		<link rel="stylesheet" type="text/css" href="css/icon.css">
		<link rel="stylesheet" type="text/css" href="css/element.css">
		<script type="text/javascript" src="/state.js"></script>
		<script type="text/javascript" src="/popup.js"></script>
		<script type="text/javascript" src="/help.js"></script>
		<script type="text/javascript" src="/validator.js"></script>
		<script type="text/javascript" src="/js/jquery.js"></script>
		<script type="text/javascript" src="/general.js"></script>
		<script type="text/javascript" src="/switcherplugin/jquery.iphone-switch.js"></script>
		<script language="JavaScript" type="text/javascript" src="/client_function.js"></script>
		<script type="text/javascript" src="/dbconf?p=kms_&v=<% uptime(); %>"></script>
		<script>
			var $j = jQuery.noConflict();
			function init() {
				show_menu(menu_hook);
				buildswitch();
				version_show();
				var rrt = document.getElementById("switch");
				if (document.form.kms_enable.value != "1") {
					rrt.checked = false;
				} else {
					rrt.checked = true;
				}
				 $j('#kms_wan_port').val(db_kms_["kms_wan_port"]);
			}
			function done_validating() {
				return true;
			}
			
			function buildswitch(){
				$j("#switch").click(
					function(){
					if(document.getElementById('switch').checked){
						document.form.kms_enable.value = 1;
					}else{
						document.form.kms_enable.value = 0;
					}
				});
			}
			
			function onSubmitCtrl(o, s) {
				document.form.action_mode.value = s;
				showLoading(3);
				document.form.submit();
			}
			
			function reload_Soft_Center(){
				location.href = "/Main_Soft_center.asp";
			}
			
			function version_show(){
				$j("#kms_version_status").html("<i>当前版本：" + db_kms_['kms_version']);
			    $j.ajax({
			        url: 'https://raw.githubusercontent.com/koolshare/koolshare.github.io/acelan_softcenter_ui/kms/config.json.js',
			        type: 'GET',
			        success: function(res) {
			            var txt = $j(res.responseText).text();
			            if(typeof(txt) != "undefined" && txt.length > 0) {
			                //console.log(txt);
			                var obj = $j.parseJSON(txt.replace("'", "\""));
					$j("#kms_version_status").html("<i>当前版本：" + obj.version);
					if(obj.version != db_kms_["kms_version"]) {
						$j("#kms_version_status").html("<i>有新版本：" + obj.version);
					}
			            }
			        }
			    });
			}
			
			var enable_ss = "<% nvram_get("enable_ss"); %>";
			var enable_soft = "<% nvram_get("enable_soft"); %>";
			function menu_hook(title, tab) {
				tabtitle[tabtitle.length -1] = new Array("", "KMS");
				tablink[tablink.length -1] = new Array("", "Module_kms.asp");
			}
		</script>
	</head>
	<body onload="init();">
		<div id="TopBanner"></div>
		<div id="Loading" class="popup_bg"></div>
		<iframe name="hidden_frame" id="hidden_frame" src="" width="0" height="0" frameborder="0"></iframe>
		<form method="POST" name="form" action="/applydb.cgi?p=kms_" target="hidden_frame">
			<input type="hidden" name="current_page" value="Module_kms.asp" />
			<input type="hidden" name="next_page" value="Module_kms.asp" />
			<input type="hidden" name="group_id" value="" />
			<input type="hidden" name="modified" value="0" />
			<input type="hidden" name="action_mode" value="" />
			<input type="hidden" name="action_script" value="" />
			<input type="hidden" name="action_wait" value="5" />
			<input type="hidden" name="first_time" value="" />
			<input type="hidden" name="preferred_lang" id="preferred_lang" value="<% nvram_get(" preferred_lang "); %>"/>
			<input type="hidden" name="SystemCmd" onkeydown="onSubmitCtrl(this, ' Refresh ')" value="kms.sh" />
			<input type="hidden" name="firmver" value="<% nvram_get(" firmver "); %>"/>
			<input type="hidden" id="kms_enable" name="kms_enable" value='<% dbus_get_def("kms_enable", "0"); %>' />
			<table class="content" align="center" cellpadding="0" cellspacing="0">
				<tr>
					<td width="17">&nbsp;</td>
					<td valign="top" width="202">
						<div id="mainMenu"></div>
						<div id="subMenu"></div>
					</td>
					<td valign="top">
						<div id="tabMenu" class="submenuBlock"></div>
						<table width="98%" border="0" align="left" cellpadding="0" cellspacing="0">
							<tr>
								<td align="left" valign="top">
									<table width="760px" border="0" cellpadding="5" cellspacing="0" bordercolor="#6b8fa3" class="FormTitle" id="FormTitle">
										<tr>
											<td bgcolor="#4D595D" colspan="3" valign="top">
												<div>&nbsp;</div>
												<div style="float:left;" class="formfonttitle">系统工具 - 来自网络的胃軟系统工具</div>
												<div style="float:right; width:15px; height:25px;margin-top:10px">
													<img id="return_btn" onclick="reload_Soft_Center();" align="right" style="cursor:pointer;position:absolute;margin-left:-30px;margin-top:-25px;" title="返回软件中心" src="/images/backprev.png" onMouseOver="this.src='/images/backprevclick.png'" onMouseOut="this.src='/images/backprev.png'"></img>
												</div>
												<div style="margin-left:5px;margin-top:10px;margin-bottom:10px">
													<img src="/images/New_ui/export/line_export.png">
												</div>
												<div class="formfontdesc" id="cmdDesc">该工具用于“鸡或”“胃軟奥菲斯”和“胃軟操作系统”。</div>
												<div class="formfontdesc" id="cmdDesc"></div>
												<table style="margin:10px 0px 0px 0px;" width="100%" border="1" align="center" cellpadding="4" cellspacing="0" bordercolor="#6b8fa3" class="FormTable" id="kms_table">
													<thead>
														<tr>
															<td colspan="2">系统工具选项</td>
														</tr>
													</thead>
													<tr>
														<th>开启系统工具</th>
														<td colspan="2">
															<div class="switch_field" style="display:table-cell;float: left;">
																<label for="switch">
																	<input id="switch" class="switch" type="checkbox" style="display: none;">
																	<div class="switch_container">
																		<div class="switch_bar"></div>
																		<div class="switch_circle transition_style">
																			<div></div>
																		</div>
																	</div>
																</label>
															</div>
															<div id="kms_version_show" style="padding-top:5px;margin-left:230px;margin-top:0px;"><i>当前版本：<% dbus_get_def("kms_version", "未知"); %></i>
															</div>
															<div id="kms_install_show" style="padding-top:5px;margin-left:330px;margin-top:-25px;"></div>
															<a style="margin-left: 318px;" href="https://raw.githubusercontent.com/koolshare/koolshare.github.io/acelan_softcenter_ui/kms/Changelog.txt" target="_blank"><em>[<u> 更新日志 </u>]</em></a>
														</td>
													</tr>
													<tr id="port_tr">
														<th width="35%">外网开关</th>
														<td>
															<div style="float:left; width:165px; height:25px">
																<select id="kms_wan_port" name="kms_wan_port" style="width:164px;margin:0px 0px 0px 2px;" class="input_option">
																	<option value="0">关闭</option>
																	<option value="1">开启</option>
																</select>
															</div>
														</td>
													</tr>
												</table>
												<div class="apply_gen">
													<button id="cmdBtn" class="button_gen" onclick="onSubmitCtrl(this, ' Refresh ')">提交</button>
												</div>
												<div style="margin-left:5px;margin-top:10px;margin-bottom:10px">
													<img src="/images/New_ui/export/line_export.png">
												</div>
												<div id="NoteBox">
													<h2>使用说明：</h2>
													<h3>以管理员身份运行CMD输入以下命令，红色字体代表变量不是固定的，请参照自己的计算机修改。</h3>
													<h3>【1】 奥菲斯鸡或</h3>
													<p>CD <font color="red">X</font>:\Program Files<font color="red">(X86)</font>\Microsoft Office\Office<font color="red">14</font>
													</p>
													<p>cscript ospp.vbs /sethst:<font color="red">192.168.0.1</font>
													</p>
													<p>cscript ospp.vbs /act</p>
													<p>cscript ospp.vbs /dstatus</p>
													<h3>【2】 操作系统鸡或</h3>
													<p>slmgr /ipk <font color="red">MHF9N-XY6XB-WVXMC-BTDCT-MKKG7</font>
													</p>
													<p>slmgr /skms <font color="red">192.168.0.1</font>
													</p>
													<p>slmgr /ato</p>
													<h2>申明：本工具来自国外互联网 <a href="https://forums.mydigitallife.info/threads/50234-Emulated-KMS-Servers-on-non-Windows-platforms" target="_blank">点我跳转</a></h2>
												</div>
												<div style="margin-left:5px;margin-top:10px;margin-bottom:10px">
													<img src="/images/New_ui/export/line_export.png">
												</div>
												<div class="KoolshareBottom">
													<br/>论坛技术支持：
													<a href="http://www.koolshare.cn" target="_blank"> <i><u>www.koolshare.cn</u></i> 
													</a>
													<br/>后台技术支持： <i>Xiaobao</i> 
													<br/>Shell, Web by： <i>fw867</i>
													<br/>
												</div>
											</td>
										</tr>
									</table>
								</td>
								<td width="10" align="center" valign="top"></td>
							</tr>
						</table>
					</td>
				</tr>
			</table>
		</form>
		</td>
		<div id="footer"></div>
	</body>
</html>
```

---



### dbus中到底存了哪些数据？
> 我们可以通过http://192.168.x.x/dbconf?p=kms_
来查看dbus中保存以kms_保存的所有数据
下面是 dbus 中的数据：
```javascript
var db_kms_=(function() {
var o={};
o['kms_enable']='1';
o['kms_title']='奥菲斯激活工具';
o['kms_version']='1.4';
o['kms_wan_port']='0';
return o;
})();
```
---

### javascript 和 页面取值的方法

> 第25行，当你载入页面javascript默认会调用并取出所有kms的参数
```javascript
<script type="text/javascript" src="/dbconf?p=kms_&v=<% uptime(); %>"></script>
```
---
> Javascript中，我们用 db_kms_["kms_wan_port"] 来取值，比如：

```javascript
var $prot = db_kms_["kms_wan_port"];
```
---
> Html标签中我们用 <% dbus_get_def("kms_enable", "0"); %>

```html
<input id="kms_enable" name="kms_enable" value='<% dbus_get_def("kms_enable", "0"); %>' />
```
> 第一个参数代表 dbus中的数据名，第二参数：如果值为空，那么默认为 0 

---
> 获取路由器NVRAM中数据的方法 <% nvram_get(" firmver "); %>
```html
<input type="hidden" name="firmver" value="<% nvram_get("firmver"); %>"/>
```
---
那么除了页面载入获取数据我们可不可以用其他方式获取？当然可以！
```javascript
$.ajax({
	url: '/dbconf?p=kms_',
	dataType: 'html',
	error: function(xhr) {},
	success: function(response) {
		//do someting here...
	}
});
```
我们可以通过ajax获取！

---
### 如何提交保存？
> 我们可以看module_kms.asp 第 96 行：
```html
<form method="POST" name="form" action="/applydb.cgi?p=kms_" target="hidden_frame">
```
> 这里的 form 定义了提交到dbus的插件数据名（相当于数据库的表名） "*applydb.cgi?p=kms_*"
> 只要你点击提交那么input标签里面的值都会保存到dbus中。下面就是执行shell当你执行的时候其实也等于是提交保存。

---

# 5． 插件如何调用shell执行相关功能 

我们可以通过ajax提交并执行shell脚本！
```javascript
var dbus={};
dbus["SystemCmd"] = "kms.sh";
dbus["action_mode"] = " Refresh ";
dbus["current_page"] = "Module_kms.asp";
dbus["kms_enable"] = 1;
$.ajax({
	type: "POST",
	url: '/applydb.cgi?p=kms_',
	contentType: "application/x-www-form-urlencoded",
	dataType: 'text',
	data: dbus,
	success: function(response) {
		//do something here...
	}
});
```
---
那么提交之后我们总要让路由器做点什么！
```javascript
dbus["SystemCmd"] = "kms.sh";
dbus["action_mode"] = " Refresh ";
dbus["current_page"] = "Module_kms.asp";
```
将以上3个参数提交dbus会根据你提交的这3个参数执行以下动作：
> 
1. 执行kms/script/kms.sh
2. 执行完会刷新页面
3. 会跳转到当前页面 Module_kms.asp



除了ajax提交执行外页面也自带提交执行，我们来看 97、98、106行的代码

```html
<input name="current_page" value="Module_kms.asp" />
<input name="next_page" value="Module_kms.asp" />
<input name="SystemCmd" onkeydown="onSubmitCtrl(this, ' Refresh ')" value="kms.sh" />
```
当你提交之后那么页面也会提交这些参数并执行。

那么我们来看看kms.sh的神秘之处：kms.sh
```shell
#!/bin/sh
# load path environment in dbus databse
eval `dbus export kms`
source /koolshare/scripts/base.sh
CONFIG_FILE=/jffs/configs/dnsmasq.d/kms.conf
FIREWALL_START=/jffs/scripts/firewall-start

start_kms(){
	/koolshare/bin/vlmcsd
	echo "srv-host=_vlmcs._tcp.lan,`uname -n`.lan,1688,0,100" > $CONFIG_FILE
	nvram set lan_domain=lan
   	nvram commit
	service restart_dnsmasq
	# creating iptables rules to firewall-start
	mkdir -p /jffs/scripts
	if [ ! -f $FIREWALL_START ]; then 
		cat > $FIREWALL_START <<-EOF
		#!/bin/sh
	EOF
	fi

	# creat start_up file
	if [ ! -L "/koolshare/init.d/S97Kms.sh" ]; then 
		ln -sf /koolshare/scripts/kms.sh /koolshare/init.d/S97Kms.sh
	fi
}
stop_kms(){
	# clear start up command line in firewall-start
	killall vlmcsd
	rm $CONFIG_FILE
	service restart_dnsmasq
}

open_port(){
	ifopen=`iptables -S -t filter | grep INPUT | grep dport |grep 1688`
	if [ -z "$ifopen" ];then
		iptables -t filter -I INPUT -p tcp --dport 1688 -j ACCEPT
	fi

	if [ ! -f $FIREWALL_START ]; then
		cat > $FIREWALL_START <<-EOF
		#!/bin/sh
		EOF
	fi
	
	fire_rule=$(cat $FIREWALL_START | grep 1688)
	if [ -z "$fire_rule" ];then
		cat >> $FIREWALL_START <<-EOF
		iptables -t filter -I INPUT -p tcp --dport 1688 -j ACCEPT
		EOF
	fi
}

close_port(){
	ifopen=`iptables -S -t filter | grep INPUT | grep dport |grep 1688`
	if [ ! -z "$ifopen" ];then
		iptables -t filter -D INPUT -p tcp --dport 1688 -j ACCEPT
	fi

	fire_rule=$(cat $FIREWALL_START | grep 1688)
	if [ ! -z "$fire_rule" ];then
		sed -i '/1688/d' $FIREWALL_START >/dev/null 2>&1
	fi
}

case $ACTION in
start)
	if [ "$kms_enable" == "1" ]; then
		logger "[软件中心]: 启动KMS！"
		start_kms
		[ "$kms_wan_port" == "1" ] && open_port
	else
		logger "[软件中心]: KMS未设置开机启动，跳过！"
	fi
	;;
stop)
	close_port >/dev/null 2>&1
	stop_kms
	;;
*)
	if [ "$kms_enable" == "1" ]; then
		close_port >/dev/null 2>&1
		stop_kms
   		start_kms
   		[ "$kms_wan_port" == "1" ] && open_port
   	else
   		close_port
		stop_kms
	fi
	;;
esac

```
如何让脚本开机自启动呢？下面的脚本就是：
```shell
 # creat start_up file
    if [ ! -L "/koolshare/init.d/S97Kms.sh" ]; then 
        ln -sf /koolshare/scripts/kms.sh /koolshare/init.d/S97Kms.sh
    fi
```
其实就是将kms.sh 做了个软连接到init.d文件夹下并以S+数字的形式。
这样软件中心就会在开机自动执行该脚本的 $ACTION start方法。
# 6．插件安装&卸载脚本

> 插件的安装脚本：install.sh
```shell
#!/bin/sh

# stop kms first
enable=`dbus get kms_enable`
if [ "$enable" == "1" ];then
	restart=1
	dbus set kms_enable=0
	sh /koolshare/scripts/kms.sh
fi

# cp files
cp -rf /tmp/kms/scripts/* /koolshare/scripts/
cp -rf /tmp/kms/bin/* /koolshare/bin/
cp -rf /tmp/kms/webs/* /koolshare/webs/
cp -rf /tmp/kms/res/* /koolshare/res/

# delete install tar
rm -rf /tmp/kms* >/dev/null 2>&1

chmod a+x /koolshare/scripts/kms.sh
chmod 0755 /koolshare/bin/vlmcsd

# re-enable kms
if [ "$restart" == "1" ];then
	dbus set kms_enable=1
	sh /koolshare/scripts/kms.sh
fi
```
> 插件的卸载脚本：uninstall.sh
```shell
#!/bin/sh

rm /koolshare/bin/vlmcsd
rm /koolshare/res/icon-kms.png
rm /koolshare/scripts/kms.sh
rm /koolshare/webs/Module_kms.asp
```

# 7．插件打包成离线安装包

只要按照将先压缩成tar格式再压缩成gz就行了。
注意的是 install.sh 不能缺失，否则无法安装。










------

再一次感谢您花费时间阅读这份开发教程，这也是我第一次用makedown写教程，如果你觉得乱先将就着看，以后我会慢慢改进，如果你在开发过程中还有什么不理解的可以和我联系。

QQ：[9168131](http://wpa.qq.com/msgrd?v=3&uin=9168131&site=qq&menu=yes)
Telegram：https://t.me/JSMonkey

如果有大神愿意捐赠的欢迎点击：[捐赠链接](https://ubgame.net/?link=donate)

作者 [JSMonkey](https://ubgame.net)
2017 年 12月 21日    
