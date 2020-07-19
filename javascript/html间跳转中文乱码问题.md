&ensp;&ensp;&ensp;&ensp;今天遇到一个问题,有两个html界面A、B，我从A跳转到B，并给B传来一个参数param,该参数可能为中文，当我跳转到B界面时，通过以下函数：
```javascript
/**

 * 获取链接参数

 * @param name 参数key

 * @returns 参数value

 */

function GetQueryString(name) 

{ 

     var reg = new RegExp("(^|&)"+ name +"=([^&]*)(&|$)"); 

     var r = window.location.search.substr(1).match(reg); 

     if(r!=null)return  unescape(r[2]); return null; 

}
```

获取参数param的值时，出现了param乱码的问题。

解决方法如下：
```javascript
/**

 * 获取链接参数(有中文的参数，英文也可以)

 * @param name 参数key

 * @returns 参数value

 */

function GetCNQueryString(name) 

{ 

     var reg = new RegExp("(^|&)"+ name +"=([^&]*)(&|$)"); 

     var r = encodeURI(window.location.search).substr(1).match(reg); 

     if(r!=null)return  decodeURI(unescape(r[2])); return null; 

}
```
