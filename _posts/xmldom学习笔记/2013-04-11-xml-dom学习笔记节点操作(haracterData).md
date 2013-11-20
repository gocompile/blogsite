#前端学习之xml dom 系列之节点操作(CharacterData)
>[w3school xml dom](http://www.w3school.com.cn/xmldom/index.asp) 

CharacterData
> 是Text和Comment节点的超接口，文档中不包含本对象，他们只包含Text和Comment

CharacterData 对象属性
> data:该节点包含的文本
> length:该节点包含的字符数

CharacterData 对象方法
> appendData(string):添加字符串到节点文本上
> deleteData(start,length):删除节点的指定文本
> insertData(start,string):在指定偏移量插入文本
> replaceData(start,length,string):字符串替换指定位置和字符数的文本
> substringData(start,length):返回子字符串


@date:2013-04-11 23:51 @author:gocompile
