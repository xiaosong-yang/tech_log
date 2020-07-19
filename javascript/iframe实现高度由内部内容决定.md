&ensp;&ensp;&ensp;&ensp;通常的div块，在不设置高度情况下，父元素的大小，由内部元素大小决定，这样设置比较灵活，当div内部为float浮动元素时，父div需要添加overflow:hidden这个css属性才能实现父div高度由内部大小决定。但是当我们使用iframe时却不能通过上面的两种方式实现iframe的大小由内部大小来决定。所以这里找了一个比较可靠地方式，如下：
```js
var browserVersion = window.navigator.userAgent.toUpperCase();

var isOpera = browserVersion.indexOf("OPERA") > -1 ? true : false;

var isFireFox = browserVersion.indexOf("FIREFOX") > -1 ? true : false;

var isChrome = browserVersion.indexOf("CHROME") > -1 ? true : false;

var isSafari = browserVersion.indexOf("SAFARI") > -1 ? true : false;

var isIE = (!!window.ActiveXObject || "ActiveXObject" in window);

var isIE9More = (! -[1, ] == false);

 

 

function reinitIframe(iframeId, minHeight) {

    try {

        var iframe = document.getElementById(iframeId);

        var bHeight = 0;

        if (isChrome == false && isSafari == false)

            bHeight = iframe.contentWindow.document.body.scrollHeight;

 

        var dHeight = 0;

        if (isFireFox == true)

//            dHeight = iframe.contentWindow.document.documentElement.offsetHeight + 2;

  bHeight = iframe.contentWindow.document.body.scrollHeight;

        else if (isIE == false && isOpera == false)

            dHeight = iframe.contentWindow.document.documentElement.scrollHeight;

        else if (isIE == true && isIE9More) {//ie9+

            var heightDeviation = bHeight - eval("window.IE9MoreRealHeight" + iframeId);

            if (heightDeviation == 0) {

                bHeight += 3;

            } else if (heightDeviation != 3) {

                eval("window.IE9MoreRealHeight" + iframeId + "=" + bHeight);

                bHeight += 3;

            }

        }

        else//ie[6-8]、OPERA

            bHeight += 3;

 

        var height = Math.max(bHeight, dHeight);

        if (height < minHeight) height = minHeight;

        iframe.style.height = height + "px";

    } catch (ex) { }

}

function startInit(iframeId, minHeight) {

    eval("window.IE9MoreRealHeight" + iframeId + "=0");

    window.setInterval("reinitIframe('" + iframeId + "'," + minHeight + ")", 100);

}

var minHeight = $(window).height();

startInit('leftframe', minHeight);
```
html如下：
```html
<iframe style="float: left; width: 21%; padding: 0; margin-left: 10px; height: 100%;"

   frameBorder="0" src="/blog/common/left.html" scrolling="no" name="leftframe" id="leftframe"></iframe>
```
其中style中的float，width，padding，margin-left都不是必须的。
