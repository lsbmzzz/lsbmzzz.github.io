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

