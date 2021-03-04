---
layout:     post
title:      POI和EasyExcel
subtitle:   POI和EasyExcel
date:       2021-03-03
author:     张三
header-img: img/spring.jpg
catalog: true
mathjax: true
tags:
    - SpringBoot
    - POI

---

> POI和easyExcel是比较流行的操作Excel的工具，POI提供API给Java程序对office文档进行读写

# Apache POI

官网：[http://poi.apache.org/](http://poi.apache.org/)

基本功能

| 类 | 功能 |
|--- | --- |
| HSSF | 提供读写excel文档的功能 |
| XSSF | 提供读写07版excel文档的功能 |
| HWPF | 提供读写word格式文档的功能 |
| HSLF | 提供读写ppt文档的功能 |
| HDGF | 提供读写visio文档的功能 |

# easyExcel

官网：[https://github.com/alibaba/easyexcel](https://github.com/alibaba/easyexcel)

> 阿里巴巴开源的excel处理工具，使用简单，节省内存，几乎不可能出现内存溢出


# POI写Excel

导入依赖

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>4.1.1</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.17</version>
</dependency>
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10.1</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.1</version>
</dependency>
```

**目标对象：**

- 工作簿
- 工作表
- 行
- 列

excel写
```java
    @Test
    public void writeExcel() throws IOException {
        // 创建工作簿
        Workbook workbook = new XSSFWorkbook();
        // Workbook workbook = new HSSFWorkbook();

        // 创建工作表
        Sheet sheet = workbook.createSheet("sheet01");
        // 创建一行
        Row row0 = sheet.createRow(0);
        // 创建单元格
        Cell cell0 = row0.createCell(0);
        cell0.setCellValue("00");
        Cell cell1 = row0.createCell(1);
        cell1.setCellValue("01");

        Row row1 = sheet.createRow(1);
        // 创建单元格
        Cell cell10 = row1.createCell(0);
        cell10.setCellValue("10");
        Cell cell11 = row1.createCell(1);
        cell11.setCellValue("11");

        FileOutputStream fileOutputStream = new FileOutputStream("D:\\test.xlsx");
        workbook.write(fileOutputStream);
        workbook.close();
        fileOutputStream.close();
    }
```

03与07版本的Excel，对象不同，方法一样

## 大数据量写入Excel

- HSSF
    + 最多只能写入65536行
    + 先操作缓存，最后一次性写入磁盘，速度快
- XSSF 
    + 没有数据量限制
    + 非常消耗内存
- SXSSF
    + 没有数据量限制
    + 写数据速度快，且占用内存小
    + 过程中会产生临时文件（默认在内存中保存100条数据，超过100条会写入临时文件）需要清理 : ((SXSSFWorkbook) workbook).dispose();
    + 可以通过 new SXSSFWorkbook(数量) 指定内存中数据的数量

# POI读Excel

```java
    @Test
    public void readExcel() throws IOException {
        FileInputStream fileInputStream = new FileInputStream("D:\\test.xls");
        HSSFWorkbook workbook = new HSSFWorkbook(fileInputStream);
        Sheet sheet = workbook.getSheetAt(0);
        
        int rowCount = sheet.getPhysicalNumberOfRows();
        for (int r = 0; r < rowCount; r++) {
            int cellCount = sheet.getRow(r).getPhysicalNumberOfCells();
            for (int c = 0; c < cellCount; c++) {
                System.out.println(sheet.getRow(r).getCell(c));
            }
        }
        fileInputStream.close();
        workbook.close();
    }
```

注意获取值的类型即可

## 读取不同数据类型

使用 cell.getCellTypeEnum();

> null、数字、字符串、公式、空白、布尔值、错误

```java
    /**
     * Unknown type, used to represent a state prior to initialization or the
     * lack of a concrete type.
     * For internal use only.
     */
    @Internal(since="POI 3.15 beta 3")
    _NONE(-1),

    /**
     * Numeric cell type (whole numbers, fractional numbers, dates)
     */
    NUMERIC(0),
    
    /** String (text) cell type */
    STRING(1),
    
    /**
     * Formula cell type
     * @see FormulaType
     */
    FORMULA(2),
    
    /**
     * Blank cell type
     */
    BLANK(3),
    
    /**
     * Boolean cell type
     */
    BOOLEAN(4),
    
    /**
     * Error cell type
     * @see FormulaError
     */
    ERROR(5);

```

可使用switch选择

## 公式

```java
    public void testForMula() throws IOException {
        FileInputStream fileInputStream = new FileInputStream(PATH + "/公式.xls");
        Workbook workbook = new HSSFWorkbook(fileInputStream);
        Sheet sheet = workbook.getSheetAt(0);

        Row row = sheet.getRow(4);
        Cell cell = row.getCell(0);

        // 拿到计算公式
        FormulaEvaluator formulaEvaluator = new HSSFFormulaEvaluator((HSSFWorkbook) workbook);

        // 输出单元格的内容
        int cellType = cell.getCellType();
        switch (cellType) {
            case (Cell.CELL_TYPE_FORMULA): // 公式
                String formula = cell.getCellFormula();
                System.out.println(formula); // SUM(A2:A4)

                // 计算
                CellValue evaluate = formulaEvaluator.evaluate(cell);
                String cellValue = evaluate.formatAsString();
                System.out.println(cellValue); // 1188.0
                break;
        }
    }
```

# EasyExcel

[EasyExcel](https://www.yuque.com/easyexcel/doc/write)

## 写

**excel示例**

![excel示例](/img/后端开发/POI和EasyExcel/excel示例.png)

**数据生成demo**

```java
    public List<EasyExcelDemoData> data() {
        List<EasyExcelDemoData> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            EasyExcelDemoData data = new EasyExcelDemoData();
            data.setString("字符串" + i);
            data.setDate(new Date());
            data.setDoubleData(0.56);
            list.add(data);
        }
        return list;
    }
```

**写excel**

```java
    @Autowired
    IEasyExcelDemoService easyExcelDemoService;
    /**
     * 最简单的写
     * <p>1. 创建excel对应的实体对象 参照{@link EasyExcelDemoData}
     * <p>2. 直接写即可
     */
    @Test
    public void simpleWrite() {
        String fileName = "D:\\easyexcel_test.xlsx";
        // 指定写用哪个class去写，然后写到第一个sheet，名字为模板 然后文件流会自动关闭
        EasyExcel.write(fileName, EasyExcelDemoData.class).sheet("模板").doWrite(easyExcelDemoService.data());
    }
```

**对象**

```java
@Data
public class EasyExcelDemoData {
    @ExcelProperty("字符串标题")
    private String string;
    @ExcelProperty("日期标题")
    private Date date;
    @ExcelProperty("数字标题")
    private Double doubleData;
    /**
     * 忽略这个字段
     */
    @ExcelIgnore
    private String ignore;
}
```

## 读

```java
    /**
     * 最简单的读
     * <p>1. 创建excel对应的实体对象 参照{@link EasyExcelDemoData}
     * <p>2. 由于默认一行行的读取excel，所以需要创建excel一行一行的回调监听器，参照{@link DemoDataListener}
     * <p>3. 直接读即可
     */
    @Test
    public void simpleRead() {
        // 有个很重要的点 DemoDataListener 不能被spring管理，要每次读取excel都要new,然后里面用到spring可以构造方法传进去
        String fileName = "D:\\easyexcel_test.xlsx";
        // 这里 需要指定读用哪个class去读，然后读取第一个sheet 文件流会自动关闭
        EasyExcel.read(fileName, EasyExcelDemoData.class, new DemoDataListener()).sheet().doRead();
    }
```

**对象**

```java
public class EasyExcelDemoData {
    private String string;
    private Date date;
    private Double doubleData;
}
```

**监听器**

监听器可以直接粘贴使用

```java
// 有个很重要的点 DemoDataListener 不能被spring管理，要每次读取excel都要new,然后里面用到spring可以构造方法传进去
public class DemoDataListener extends AnalysisEventListener<EasyExcelDemoData> {
    private static final Logger LOGGER = LoggerFactory.getLogger(DemoDataListener.class);
    /**
     * 每隔5条存储数据库，实际使用中可以3000条，然后清理list ，方便内存回收
     */
    private static final int BATCH_COUNT = 5;
    List<EasyExcelDemoData> list = new ArrayList<EasyExcelDemoData>();
    /**
     * 假设这个是一个DAO，当然有业务逻辑这个也可以是一个service。当然如果不用存储这个对象没用。
     */
    private DemoDAO demoDAO;
    public DemoDataListener() {
        // 这里是demo，所以随便new一个。实际使用如果到了spring,请使用下面的有参构造函数
        demoDAO = new DemoDAO();
    }
    /**
     * 如果使用了spring,请使用这个构造方法。每次创建Listener的时候需要把spring管理的类传进来
     *
     * @param demoDAO
     */
    public DemoDataListener(DemoDAO demoDAO) {
        this.demoDAO = demoDAO;
    }
    /**
     * 这个每一条数据解析都会来调用
     *
     * @param data
     *            one row value. Is is same as {@link AnalysisContext#readRowHolder()}
     * @param context
     */
    @Override
    public void invoke(EasyExcelDemoData data, AnalysisContext context) {
        LOGGER.info("解析到一条数据:" + JSON.toJSONString(data));
        list.add(data);
        // 达到BATCH_COUNT了，需要去存储一次数据库，防止数据几万条数据在内存，容易OOM
        if (list.size() >= BATCH_COUNT) {
            saveData();
            // 存储完成清理 list
            list.clear();
        }
    }
    /**
     * 所有数据解析完成了 都会来调用
     *
     * @param context
     */
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        // 这里也要保存数据，确保最后遗留的数据也存储到数据库
        saveData();
        LOGGER.info("所有数据解析完成！");
    }
    /**
     * 加上存储数据库
     */
    private void saveData() {
        LOGGER.info(list.size() + "条数据，开始存储数据库！");
        demoDAO.save(list);
        LOGGER.info("存储数据库成功！");
    }
}
```
