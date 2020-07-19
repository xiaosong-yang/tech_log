### 样式优先级
Html标签中的class可以有多个class，用空格隔开即可，多个class共同组合形成标签最后的样式，多个class之间时候有优先级的，后面被申明的优先级更高。
```css
<button  class="layui-btn layui-btn-sm types layui-btn-warm">Spring</button>
```
其中types为当前页面申明的class并且，在是在引入其他样式之后声明的，所以这里types优先级最高，这也解释了为什么标签内的style优先级最高，因为他是最后被声明的。



### 样式铺垫
如果希望某一个样式后面一个样式具有怎样的样式，可以如下设定：
```
         .types {

                   float:left;

                   margin: 3px 5px;

         }

         .types+.types {

                   float:left;

                   margin: 3px 5px;
         }
   ```
   这表示，具有types样式标签的后面一个具有types样式的标签，具有float：left，margin：3px 5px的样式。


