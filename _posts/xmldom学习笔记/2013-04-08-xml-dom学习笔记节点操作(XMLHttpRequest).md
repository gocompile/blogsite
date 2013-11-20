#前端学习之xml dom 系列之节点操作(XMLHttpRequest)
>[w3school xml dom](http://www.w3school.com.cn/xmldom/index.asp) 

>在浏览器中使用XMLHttpRequest对象实现异步交互，可以无需刷新页面即可实现页面数据更新

    1.创建实例
      vat xmlhttp;
      //通过XMLHttpRequest请求数据
      function loadXMLDoc(url) {
        xmlhttp = null;
        if(window.XMLHttpRequest) {	//for all new browser
          xmlhtpp = new XMLHttpRequest();
        } else {
          xmlhttp = new ActiveXobject("Microsoft.XMLHTTP");//for ie5,ie6
        }
        
        if(xmlhttp != null) {
          xmlhttp.readystatchage = stat_Change;	//制定处理请求完成的结果
          xmlhttp.open('GET',url,true);	//通过GET发生请求，'true' 指定为异步交互
          xmlhttp.send(null);	//开始执行请求
        }
      }
      //请求结果处理函数
      function stat_Change() {
        if(xmlhttp.readyStat ==4 )
          if(xmlhttp.code == 200) {
            //TODO 处理业务逻辑
	    document.write(xmlhtpp.responseText);
          } else {
            alert("Problem retriving XML data");
          }
        }
      }

	2.修改div中的值
	
	  <html>
		<head>
			<script type="text/javascript">
				var xmlhttp;
				function loadXMLDoc(url) {
					xmlhttp = null;
					if(window.XMLHttpRequest) {
						xmlhttp = new XMLHttpRequest();
					} else {
						xmlhttp = new ActiveXObect("Microsoft.XMLHTTP");
					}
					if(xmlhttp != null) {
						xmlhttp.readystatchange = state_Chane;
						xmlhttp.open("GET",url,true);
						xmlhttp.send(null);
					} else {
						alert("Your browser does not support XMLHTTP");
					}
				}
				function state_Change() {
					if(xmlhttp.readyStat == 4) {
						if(xmlhttp.status == 200 ) {
							document.getElementById("T1").innerHTML = xmlhttp.responseText;
						} else {
							alert("Problem retrieving data:"+xmlhttp.statusText);
						}
					}
				}
			</script>
		</head>
		<body>
			<div id="T1" style="border:1px solid black;heighy:40;weith:300;padding:5"></div>
			<br/>
			<button onclick="loadXMLDoc('http://www.w3school.com.cn/example/xdom/books.xml')">Click</button>
		</body>
	  </html>
	
	3.打印headers

	<html>
		<head>
			<script type="text/javascript">
				var xmlhttp;
				function loadXMLDoc(url) {
					xmlhttp = null;
					if(window.XMLHttpRequest) {
						xmlhttp = new XMLHttpRequest();
					} else {
						xmlhttp = new ActiveXObect("Microsoft.XMLHTTP");
					}
					if(xmlhttp != null) {
						xmlhttp.readystatchange = state_Chane;
						xmlhttp.open("GET",url,true);
						xmlhttp.send(null);
					} else {
						alert("Your browser does not support XMLHTTP");
					}
				}
				function state_Change() {
					if(xmlhttp.readyStat == 4) {
						if(xmlhttp.status == 200 ) {
							document.getElementById("T1").innerHTML = xmlhttp.getAllResponseHeaders();//处理xmlhttp从服务端返回来的数据，可以访问response头信息，以及数据主体
						} else {
							alert("Problem retrieving data:"+xmlhttp.statusText);
						}
					}
				}
			</script>
		</head>
		<body>
			<div id="T1" style="border:1px solid black;heighy:40;weith:300;padding:5"></div>
			<br/>
			<button onclick="loadXMLDoc('http://www.w3school.com.cn/example/xdom/books.xml')">Click</button>
		</body>
	  </html>
	
	4.

