# AJAX

## AJAX的核心是XMLHttpRequest。

### 一个完整的AJAX请求一般包括以下步骤：

    1.创建XMLHTTPRequest对象
    2.使用open方法创建http请求,并设置请求地址
    3.设置发送的数据，开始和服务器端交互
    4.注册事件
    5.获取响应并更新界面

## get\post例子

<html>

	<h2>ajax请求用户名的校验</h2>
	<p>用户名 : <input type="text" name="username" value="" onblur="f1()" /></p>
	<p>邮箱 : <input type="text" name="email" value="" /></p>
	<div id="result" style="color:green;"></div>
	<!-- <input type="button"  value="提交" onclick="f2()" /> -->
	
</html>

### get请求

<html>

	//请求函数
	function f1(){
		console.log('start');
		//1.创建AJAX对象
		var ajax = new XMLHttpRequest();
		
		//4.给AJAX设置事件(这里最多感知4[1-4]个状态)
		ajax.onreadystatechange = function(){
			//5.获取响应
			//responseText		以字符串的形式接收服务器返回的信息
			//console.log(ajax.readyState);
			if(ajax.readyState == 4 && ajax.status == 200){
				var msg = ajax.responseText;
				console.log(msg);
				//alert(msg);
				var divtag = document.getElementById('result');
				divtag.innerHTML = msg;
			}
		}
		
		//2.创建http请求,并设置请求地址
		var username = document.getElementsByTagName('input')[0].value;
		var email = document.getElementsByTagName('input')[1].value;
		username = encodeURIComponent(username);	//对输入的特殊符号(&,=等)进行编码
		email = encodeURIComponent(email);
		ajax.open('get','response.php?username='+username+'&email='+email);
		
		//3.发送请求(get--null    post--数据)
		ajax.send(null);
	}
	
</html>

### post请求

<html>

	//请求函数
	function f1(){
		//console.log('start');
		//1.创建AJAX对象
		var ajax = new XMLHttpRequest();
		
		//4.给AJAX设置事件(这里最多感知4[1-4]个状态)
		ajax.onreadystatechange = function(){
			//5.获取响应
			//responseText		以字符串的形式接收服务器返回的信息
			//console.log(ajax.readyState);
			if(ajax.readyState == 4 && ajax.status == 200){
				var msg = ajax.responseText;
				console.log(msg);
				//alert(msg);
				var divtag = document.getElementById('result');
				divtag.innerHTML = msg;
			}
		}
		
		//2.创建http请求,并设置请求地址
		ajax.open('post','response.php');
		//post方式传递数据是模仿form表单传递给服务器的,要设置header头协议
		ajax.setRequestHeader("content-type","application/x-www-form-urlencoded");
		
		//3.发送请求(get--null    post--数据)
		var username = document.getElementsByTagName('input')[0].value;
		var email = document.getElementsByTagName('input')[1].value;
		username = encodeURIComponent(username);	//对输入的特殊符号(&,=等)进行编码
		email = encodeURIComponent(email);
		var info = 'username='+username+'&email='+email;	//将请求信息组成请求字符串
		ajax.send(info);
	}
	
</html>

# ajax封装

## 将AJAX请求封装成ajax()方法，它接受一个配置对象params

<html>

    function ajax(params) {   
      params = params || {};   
      params.data = params.data || {};   
      // 判断是ajax请求还是jsonp请求
      var json = params.jsonp ? jsonp(params) : json(params);   
      // ajax请求   
      function json(params) {   
        //  请求方式，默认是GET
        params.type = (params.type || 'GET').toUpperCase(); 
        // 避免有特殊字符，必须格式化传输数据  
        params.data = formatParams(params.data);   
        var xhr = null;    

        // 实例化XMLHttpRequest对象   
        if(window.XMLHttpRequest) {   
          xhr = new XMLHttpRequest();   
        } else {   
          // IE6及其以下版本   
          xhr = new ActiveXObjcet('Microsoft.XMLHTTP');   
        }; 
        // 监听事件，只要 readyState 的值变化，就会调用 readystatechange 事件 
        xhr.onreadystatechange = function() {  
          //  readyState属性表示请求/响应过程的当前活动阶段，4为完成，已经接收到全部响应数据
          if(xhr.readyState == 4) {   
            var status = xhr.status;  
            //  status：响应的HTTP状态码，以2开头的都是成功
            if(status >= 200 && status < 300) {   
              var response = ''; 
              // 判断接受数据的内容类型  
              var type = xhr.getResponseHeader('Content-type');   
              if(type.indexOf('xml') !== -1 && xhr.responseXML) {   
                response = xhr.responseXML; //Document对象响应   
              } else if(type === 'application/json') {   
                response = JSON.parse(xhr.responseText); //JSON响应   
              } else {   
                response = xhr.responseText; //字符串响应   
              };  
              // 成功回调函数 
              params.success && params.success(response);   
           } else {   
              params.error && params.error(status);   
           }   
          };   
        };  
        
        // 连接和传输数据   
        if(params.type == 'GET') {
          // 三个参数：请求方式、请求地址(get方式时，传输数据是加在地址后的)、是否异步请求(同步请求的情况极少)；
          xhr.open(params.type, params.url + '?' + params.data, true);   
          xhr.send(null);   
        } else {   
          xhr.open(params.type, params.url, true);   
          //必须，设置提交时的内容类型   
          xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded; charset=UTF-8'); 
          // 传输数据  
          xhr.send(params.data);   
        }   
      }
      //格式化参数   
      function formatParams(data) {   
        var arr = [];   
        for(var name in data) { 
          //   encodeURIComponent() ：用于对 URI 中的某一部分进行编码
          arr.push(encodeURIComponent(name) + '=' + encodeURIComponent(data[name]));   
        };   
        // 添加一个随机数参数，防止缓存   
        arr.push('v=' + random());   
        return arr.join('&');   
      }

      // 获取随机数   
      function random() {   
        return Math.floor(Math.random() * 10000 + 500);   
      }
    }

</html>

## 使用实例

<html>

    ajax({   
      url: 'test.php',   // 请求地址
      type: 'POST',   // 请求类型，默认"GET"，还可以是"POST"
      data: {'b': '异步请求'},   // 传输数据
      success: function(res){   // 请求成功的回调函数
        console.log(JSON.parse(res));   
      },
      error: function(error) {}   // 请求失败的回调函数
    });

</html>

