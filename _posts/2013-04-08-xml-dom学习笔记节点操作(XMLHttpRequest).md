#前端学习之xml dom 系列之节点操作(XMLHttpRequest)
>[w3school xml dom](http://www.w3school.com.cn/xmldom/index.asp) 

>在浏览器中使用XMLHttpRequest对象实现异步交互，可以无需刷新页面即可实现页面数据更新

    1.创建实例
      vat xmlhttp;
      function loadXMLDoc(url) {
        xmlhttp = null;
        if(window.XMLHttpRequest) {
          xmlhtpp = new XMLHttpRequest();
        } else {
          xmlhttp = new ActiveXobject("Microsoft.XMLHTTP");
        }
        
        if(xmlhttp != null) {
          xmlhttp.readystatchage = stat_Change;
          xmlhttp.open('GET',url,true);
          xmlhttp.send(null);
        }
      }
      
      function stat_Change() {
        if(xmlhttp.readyStat ==4 )
          if(xmlhttp.code == 200) {
            //TODO 处理业务逻辑
          } else {
            alert("Problem retriving XML data");
          }
        }
      }
