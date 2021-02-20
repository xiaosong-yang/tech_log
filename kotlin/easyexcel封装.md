&ensp;&ensp;&ensp;&ensp;项目中有大量的报表并且有对应的报表导出功能，导出功能都是基于报表当前的查询条件，导出所有页的记录，流程相对单一，所以封装了一个阿里的easyexcel，只需要传入指定参数即可，隔离了文件相关处理流程：
```kotlin

import com.alibaba.excel.EasyExcel
import com.alibaba.excel.ExcelWriter
import com.alibaba.excel.metadata.Table
import com.alibaba.excel.write.metadata.WriteSheet
import com.alibaba.excel.write.metadata.WriteTable
import io.zhudy.roar.admin.dto.PageDto
import io.zhudy.roar.admin.web.qo.GrantQoTypeEnum
import io.zhudy.roar.admin.web.qo.PageQo
import org.springframework.stereotype.Component
import java.io.File

/**
 *@Description excel工具

 *@Author xiaosong.yang

 *@Date2021/2/8 9:57

 *@Version V1.0

 **/
@Component
class ExcelHelper(

) {

    /**
     * @param dataMethod 获取数据的方法
     * @param fileName 导出的excel文件名称
     * @param excel excel映射对象类
     * @param covertMethod 数据转换成excel的方法
     */
    fun <T, V> export(
        dataMethod: (PageQo) -> PageDto<T>,
        fileName: String,
        excel: Class<V>,
        covertMethod: (T) -> V
    ): File {
        val pageInfo = dataMethod(PageQo(1, 999))

        val tempFile = File.createTempFile(fileName, ".xlsx")
        var excelWriter: ExcelWriter? = null
        try {
            // 这里 需要指定写用哪个class去写
            excelWriter = EasyExcel.write(tempFile, excel).build()
            // 这里注意 如果同一个sheet只要创建一次
            val writeSheet = EasyExcel.writerSheet(fileName).build()
            writeList(pageInfo.items, excelWriter, writeSheet, covertMethod)
            for (page in 2..pageInfo.totalPage) {
                val items = dataMethod(PageQo(page, 999)).items
                writeList(items, excelWriter, writeSheet, covertMethod)
            }
        } finally {
            // 千万别忘记finish 会帮忙关闭流
            excelWriter?.finish()
        }
        return tempFile
    }


    /**
     * @param dataMethod 获取数据的方法
     * @param fileName 导出的excel文件名称
     * @param covertMethod 数据转换成excel的方法
     * @param table 动态定义表头
     */
    fun <T, V> export(
        dataMethod: (PageQo) -> PageDto<T>,
        fileName: String,
        covertMethod: (T) -> V,
        table: WriteTable
    ): File {
        val pageInfo = dataMethod(PageQo(1, 999))

        val tempFile = File.createTempFile(fileName, ".xlsx")
        var excelWriter: ExcelWriter? = null
        try {
            // 这里 需要指定写用哪个class去写
            excelWriter = EasyExcel.write(tempFile).build()
            // 这里注意 如果同一个sheet只要创建一次
            val writeSheet = EasyExcel.writerSheet(fileName).build()
            writeList(pageInfo.items, excelWriter, writeSheet, covertMethod, table)
            for (page in 2..pageInfo.totalPage) {
                val items = dataMethod(PageQo(page, 999)).items
                writeList(items, excelWriter, writeSheet, covertMethod, table)
            }
        } finally {
            // 千万别忘记finish 会帮忙关闭流
            excelWriter?.finish()
        }
        return tempFile
    }


    private fun <T, V> writeList(
        list: List<T>,
        excelWriter: ExcelWriter,
        writeSheet: WriteSheet,
        covertMethod: (T) -> V,
        table: WriteTable? = null
    ) {
        val writeList = mutableListOf<V>()
        list.forEach {
            writeList.add(
                covertMethod(it)
            )
        }
        if (table == null) {
            excelWriter.write(writeList, writeSheet)
        } else {
            excelWriter.write(writeList, writeSheet, table)
        }
    }

}
```
PageDto，如下：
```kotlin
data class PageDto<out T>(
    val items: List<T> = emptyList(),
    val totalItems: Long = 0L,
    @JsonIgnore
    val pageable: PageQo
) {
    val page get() = pageable.page

    val size get() = pageable.size

    val totalPage = ceil(totalItems.toDouble() / pageable.size.toDouble()).toInt()
}
```
PageQo,如下：
```kotlin
data class PageQo(
    var page: Int = 1,
    var size: Int = 15
) {

    companion object {
        /**
         * 分页最大的记录数。
         */
        var maxSize = 1000
    }

    init {
        if (page <= 0) {
            throw BizCodeException(PubBizCodes.C_999, "page 必须大于 0")
        }
        if (size <= 0) {
            throw BizCodeException(PubBizCodes.C_999, "size 必须大于 0")
        }
        if (size >= maxSize) {
            throw BizCodeException(PubBizCodes.C_999, "size 不能大于 $maxSize")
        }
    }

    val qPage get() = page - 1
}
```


### 使用
里面包含两种方式导出，一种表格格式固定，需要有一个bean来映射excel和数据之间的关系：
##### 静态表
```kotlin
        val file = excelHelper.export(
            //获取数据的方法，就是分页列表的那个函数
            dataMethod = {
                findPage(userId, GrantQoTypeEnum.INCOME.name, it)
            }, 
            //导出文件名
            fileName = "xxxxx记录", 
            //映射实体
            excel = GrantIncomeExcel::class.java, 
            //将获取的数据转换成映射实体的方法
            covertMethod = {
                GrantIncomeExcel(
                    time = it.time.toString("yyyy-MM-dd HH:mm:ss"),
                    grantUser = it.adminName,
                    gratedUserId = it.userId,
                    gratedNickname = it.nickname,
                    income = it.num.toLong(),
                    remark = it.remark
                )
            }
        )
```
对应的数据映射bean：GrantIncomeExcel
```kotlin
data class GrantIncomeExcel(
    @ExcelProperty("发放时间", index = 0)
    val time: String,
    @ExcelProperty("发放人", index = 1)
    val grantUser: String,
    @ExcelProperty("发放对象", index = 2)
    val gratedUserId: Int,
    @ExcelProperty("发放对象昵称", index = 3)
    val gratedNickname: String,
    @ExcelProperty("收入", index = 4)
    val income: Long,
    @ExcelProperty("备注", index = 5)
    val remark: String
)
```
有时候我们会有要多级表头的情况：
![easyexcel多级表头](..\picture_back_up\easyexcel多级表头.png)
这就需要修改@ExcelProperty的value属性：
```kotlin
data class GrantIncomeExcel(
    @ExcelProperty(value=["发放时间","发放时间"], index = 0)
    val time: String,
    @ExcelProperty(value=["总额","总额"], index = 1)
    val grantUser: String,
    @ExcelProperty(value=["生产","生产1"], index = 2)
    val gratedUserId: Int,
    @ExcelProperty(value=["生产","生产2"], index = 3)
    val gratedNickname: String,
    @ExcelProperty(value=["生产","生产3"], index = 4)
    val income: Long,
    @ExcelProperty(value=["生产","生产4"], index = 5)
    val remark: String
)
```
就能实现多级表头，需要注意的是，虽然第一列只有一级表头，但是由于占了上下两个单元格，所以需要写两遍重复的，会自动合并成要给大单元格，如果不写成两个，则会生成文件报错。


##### 静态表
和上面静态生产不同的则是动态表，这种不需要去定义数据映射的实体，但每次生成文件时，需要指定表的形式：
```kotlin
        val file = excelHelper.export(
        //数据获取的方法
        dataMethod = {
            PageDto( listOf("aaa","bbb"),10000, it)
        }, 
        //文件名
        fileName = "xx月报", 
        //数据转换函数
        covertMethod = { log ->
            listOf("aaa","bbb","ccc")
        }, 
        //表的格式
        table = WriteTable().also {
            //这里动态生成表头
            val head = mutableListOf<List<String>>()
            head.add(listOf("第一列","第一列"))
            head.add(listOf("第二列","第二小列1"))
            head.add(listOf("第二列","第二小列2"))
            it.head = head
        })
```
这个测试案例也是多级表头，形式，导出结果：
![动态生成excel表](..\picture_back_up\动态生成excel表.png)