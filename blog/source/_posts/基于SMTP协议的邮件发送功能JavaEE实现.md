本博客介绍基于SSM框架(Spring4.0+SpringMVC+Mybatis)组合的Javamail应用，邮箱的话基于腾讯的QQ邮箱，其实也是Foxmail邮箱

先要了解一下SMTP协议和SSL加密
SMTP：称为简单邮件传输协议（Simple Mail Transfer Protocal），目标是向用户提供高效、可靠的邮件传输。SMTP是一种请求响应的协议，也就是客户机向远程服务器发送请求，服务器响应，监听端口是25，所以其工作模式有两种：发送SMTP，接收SMTP

SSL加密：用来保障浏览器和网站服务器的安全性，其原理用译文解释就是：
当你的浏览器向服务器请求一个安全的网页(通常是 https://)

服务器就把它的证书和公匙发回来

浏览器检查证书是不是由可以信赖的机构颁发的，确认证书有效和此证书是此网站的。

使用公钥加密了一个随机对称密钥，包括加密的URL一起发送到服务器

服务器用自己的私匙解密了你发送的钥匙。然后用这把对称加密的钥匙给你请求的URL链接解密。

服务器用你发的对称钥匙给你请求的网页加密。你也有相同的钥匙就可以解密发回来的网页了

然后介绍怎么实现javamail发送邮件，先要下载javamail的jar：http://download.csdn.net/detail/u014427391/9721520

去充当服务器的QQ邮箱开启SMTP服务：
![这里写图片描述](http://img.blog.csdn.net/20161226170901273?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

写个发送邮件的业务类：

```
package com.appms.email;

import java.util.Date;
import java.util.Properties;

import javax.mail.Address;
import javax.mail.Message;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;

import com.sun.mail.util.MailSSLSocketFactory;

public class JavaEmailSender {
	

	public static void sendEmail(String toEmailAddress,String emailTitle,String emailContent)throws Exception{
		Properties props = new Properties();

        // 开启debug调试
        props.setProperty("mail.debug", "true");
        // 发送服务器需要身份验证
        props.setProperty("mail.smtp.auth", "true");
        // 设置邮件服务器主机名
        props.setProperty("mail.host", "smtp.qq.com");
        // 发送邮件协议名称
        props.setProperty("mail.transport.protocol", "smtp");

        /**SSL认证，注意腾讯邮箱是基于SSL加密的，所有需要开启才可以使用**/
        MailSSLSocketFactory sf = new MailSSLSocketFactory();
        sf.setTrustAllHosts(true);
        props.put("mail.smtp.ssl.enable", "true");
        props.put("mail.smtp.ssl.socketFactory", sf);

        //创建会话
        Session session = Session.getInstance(props);

        //发送的消息，基于观察者模式进行设计的
        Message msg = new MimeMessage(session);
        msg.setSubject(emailTitle);
        //使用StringBuilder，因为StringBuilder加载速度会比String快，而且线程安全性也不错
        StringBuilder builder = new StringBuilder();
        builder.append("\n"+emailContent);
        builder.append("\n时间 " + new Date());
        msg.setText(builder.toString());
        msg.setFrom(new InternetAddress("你的QQ邮箱"));

        Transport transport = session.getTransport();
        transport.connect("smtp.qq.com", "你的QQ邮箱", "你开启SMTP服务申请的独立密码");
		//发送消息
        transport.sendMessage(msg, new Address[] { new InternetAddress(toEmailAddress) });
        transport.close();
	}
}
```


然后写个SpringMVC框架的Controller类：

```

	/**
	 * 跳转到发送邮件页面
	 * @return
	 * @throws Exception
	 */
	@RequestMapping("/goSendEmail")
	public ModelAndView goSendEmail(HttpServletRequest request)throws Exception{
		ModelAndView mv = this.getModelAndView();
		String email = request.getParameter("email");
		if(email!=null&&!"".equals(email)){
			email = email.trim();
			mv.setViewName("member/send_email");
			mv.addObject("email", email);
		}
		return mv;
	}
	
	/**
	 * 发送邮件
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value="/sendEmail",produces="application/json;charset=UTF-8")
	@ResponseBody
	public Object sendEmail(HttpServletRequest request)throws Exception{
		Map<String,String> map = new HashMap<String,String>();
		String msg = "ok";		//发送状态
		String toEMAIL = request.getParameter("EMAIL");					//对方邮箱
		String TITLE = request.getParameter("TITLE");					//标题
		String CONTENT = request.getParameter("CONTENT");				//内容
		JavaEmailSender.sendEmail(toEMAIL, TITLE, CONTENT);
		map.put("result", msg);
		return map;
	}
```

这里用了Jquery TIP插件进行验证提示，所以需要引入相应的Jquery文件

```
<script type="text/javascript" src="source/js/jquery-1.7.2.js"></script>
	<!--提示框-->
	<script type="text/javascript" src="source/js/jquery.tips.js"></script>
```
Jquery表单验证和Ajax异步请求：
```
<!-- 发送邮件 -->
	<script type="text/javascript">
//发送
function sendEm(){
	
	if($("#TYPE").val()=="1"){
		$("#CONTENT").val(getContentTxt());
	}else{
		$("#CONTENT").val(getContent());
	}
	if($("#EMAIL").val()==""){
		$("#EMAIL").tips({
			side:3,
            msg:'请输入邮箱',
            bg:'#AE81FF',
            time:2
        });
		$("#EMAIL").focus();
		return false;
	}
	if($("#TITLE").val()==""){
		$("#TITLE").tips({
			side:3,
            msg:'请输入标题',
            bg:'#AE81FF',
            time:2
        });
		$("#TITLE").focus();
		return false;
	}
	if($("#CONTENT").val()==""){
		
		$("#nr").tips({
			side:1,
            msg:'请输入内容',
            bg:'#AE81FF',
            time:3
        });
		return false;
	}
	
	var EMAIL = $("#EMAIL").val();
	var TYPE  = $("#TYPE").val();
	var TITLE = $("#TITLE").val();
	var CONTENT = $("#CONTENT").val();

	$("#zhongxin").hide();
	$("#zhongxin2").show();
	
	$.ajax({
		type: "POST",
		url: 'retroaction/sendEmail.do?tm='+new Date().getTime(),
    	data: {EMAIL:EMAIL,TITLE:TITLE,CONTENT:CONTENT},
		dataType:'json',
		//beforeSend: validateData,
		cache: false,
		success: function(data){
			if("ok" == data.result){
				$("#msg").tips({
						side:3,
			            msg:'发送成功！',
			            bg:'#68B500',
			            time:5
				      });
				setTimeout("showdiv()",1000);	
			}else{
				$("#msg").tips({
						side:3,
			            msg:'发送失败!',
			            bg:'#68B500',
			            time:5
				      });
			}
			
		}
	});
	
}

</script>

```

JSP页面的调用：

```
<!-- 编辑邮箱  -->
		<div>
		<table style="width:98%;" >
			<tr>
				<td style="margin-top:0px;">
					 <div style="float: left;" style="width:81%"><textarea name="EMAIL" id="EMAIL" rows="1" cols="50" style="width:600px;height:20px;" placeholder="请选输入对方邮箱,多个请用(;)分号隔开" title="请选输入对方邮箱,多个请用(;)分号隔开">${email}</textarea></div>
					 <div style="float: right;" style="width:19%"><a class='btn btn-mini btn-info' title="编辑邮箱" onclick="dialog_open();">编辑邮箱</i></a></div>
				</td>
			</tr>
			<tr>
				<td>
					 <input type="text" name="TITLE" id="TITLE" value="" placeholder="请选输入邮件标题" style="width:98%"/>
				</td>
			</tr>
			<tr>
				<td id="nr">
					 <script id="editor" type="text/plain" style="width:650px;height:259px;"></script>
				</td>
			</tr>
			<tr>
				<td style="text-align: center;">
					<a class="btn btn-mini btn-primary" onclick="sendEm();">发送</a>
					<a class="btn btn-mini btn-danger" onclick="top.Dialog.close();">取消</a>
				</td>
			</tr>
		</table>
		</div>
		<div id="zhongxin2" class="center" style="display:none"><br/><img src="assets/images/jzx.gif" id='msg' /><br/><h4 class="lighter block green" id='msg'>正在发送...</h4></div>		
```

![这里写图片描述](http://img.blog.csdn.net/20161226172448445?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


