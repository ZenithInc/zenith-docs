# Excel 导入导出

这篇文档描述了如何在 Java 应用中导入导出 Excel。本文使用的是阿里开源的 `EasyExcel`，正如其名，确实简化了 Excel 的导入导出。

### 安装

引入 Maven 的`pom.xml`， 版本如下:

```xml
<!-- https://mvnrepository.com/artifact/com.alibaba/easyexcel -->
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>easyexcel</artifactId>
	<version>3.3.2</version>
</dependency>
```

### 读取 Excel {id="reader"}

读取 Excel 可以简单也可以复杂，更为详细的用法请参考[官方文档](https://easyexcel.opensource.alibaba.com/)。

#### 简单读取 Excel {id="simple_reader"}

示例 Excel 如下图:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/9e5RDrQeTMjcO8RNckQy.png" alt="excel_contents"/>

值对象如下:

```java
import com.alibaba.excel.annotation.ExcelProperty;

@Getter
@Setter
@EqualsAndHashCode
public class SimpleBO {

    @ExcelProperty("user_id")
    private String userId;

    @ExcelProperty("age")
    private Integer age;

    @ExcelProperty("name")
    private String name;

    @ExcelProperty("balance")
    private BigDecimal balance;
}
```

读取的代码如下:

```java
ExcelReaderBuilder builder = EasyExcel.read(excelFile.getInputStream());
List<SimpleBO> rows = builder.head(SimpleBO.class).sheet().doReadSync();
rows.forEach(row -> {
	System.out.println("row.getName() = " + row.getName());
});
```

#### 读多个 Sheet

每一个 Sheet 创建对应的 Listener：

```java
public class ExcelSheetRowUserListener implements ReadListener<SimpleBO> {


    @Override
    public void invoke(SimpleBO simpleBO, AnalysisContext analysisContext) {
        // 解析一行数据
    }

    @Override
    public void doAfterAllAnalysed(AnalysisContext analysisContext) {
        // 所有数据解析完成后调用
    }
}
```

创建对应的值对象, 如上文所示。然后读取所有 Sheet 示例如下:

```java
try (ExcelReader reader = EasyExcel.read(excelFile.getInputStream()).build()) {
	ReadSheet userSheet = EasyExcel.readSheet("users")
	.head(SimpleBO.class)
	.registerReadListener(new ExcelSheetRowUserListener())
	.build();
	ReadSheet promotionSheet = EasyExcel.readSheet("promotion")
	.head(SimpleBO.class)
	.registerReadListener(new ExcelSheetRowUserListener())
	.build();
	reader.read(userSheet, promotionSheet);
}
```

### 输出 Excel

编写了一个简单的工具类，还蛮好用的，示例如下:

```java
public class ExcelUtil {
    @SneakyThrows
    public static <T> void export(List<T> rows, HttpServletResponse response, Class<T> clazz) {
        response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        response.setCharacterEncoding(StandardCharsets.UTF_8.name());
        response.setHeader("Content-disposition", "attachment;filename*=utf-8''" + "report" + ".xlsx");
        EasyExcel.write(response.getOutputStream(), clazz).sheet("sheet")
                .doWrite(rows);
    }
}
```

值对象就类似上文中的示例，加上 `@ExcelProperty` 以及 `@EqualsAndHashCode` 两个注解就可以了。