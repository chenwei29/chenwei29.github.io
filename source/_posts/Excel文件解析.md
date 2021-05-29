---
title: Excel文件解析
date: 2021-05-28 15:28:37
tags: 工作
---

在工作中需要解析某个excel文件,提取其中的数据在进行操作.在这里,是要解析excel文件,并将其数据放回前端显示出来,本环境是基于springboot下的项目.

### 开始

 首先是在pom文件添加依赖,

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.17</version>
</dependency>
```

 添加依赖后,利用文件的输入流来得到数据,并循环来获取数据

```java
public class ExcelUtils {
    private ExcelUtils() {
       
    }
    /**
     * 获取并解析excel文件，返回一个二维集合
     * @param file 上传的文件
     * @return 二维集合（第一重集合为行，第二重集合为列，每一行包含该行的列集合，列集合包含该行的全部单元格的值）
     */
    public static ArrayList<ArrayList<String>> analysis(MultipartFile file) {
        ArrayList<ArrayList<String>> row = new ArrayList<>();
        //获取文件名称
        String fileName = file.getOriginalFilename();
        System.out.println(fileName);
        InputStream in = null;
        try {
            //获取输入流
            in=file.getInputStream();
            //判断excel版本
            Workbook workbook = null;
            if (judegExcelEdition(fileName)) {
                workbook = new XSSFWorkbook(in);
            } else {
                workbook = new HSSFWorkbook(in);
            }
            //获取第一张工作表
            Sheet sheet = workbook.getSheetAt(0);
            //从第二行开始获取
            for (int i = 1; i < sheet.getPhysicalNumberOfRows(); i++) {
                //循环获取工作表的每一行
                Row sheetRow = sheet.getRow(i);
                //循环获取每一列
                ArrayList<String> cell = new ArrayList<>();
                //需要表头将j=1 改成j=0
                for (int j = 1; j < sheetRow.getPhysicalNumberOfCells(); j++) {
                    //将每一个单元格的值装入列集合
                    Cell cell1 = sheetRow.getCell(j);
                    //此处判断所在单元格是否和空,避免将空的一列给排除掉
                    if (cell1==null){
                        cell.add(null);
                    }else{
                    cell1.setCellType(CELL_TYPE_STRING);
                    cell.add(cell1.getStringCellValue());}
                }
                //将装有每一列的集合装入大集合
                row.add(cell);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
            System.out.println("===================未找到文件======================");
        } catch (IOException e) {
            e.printStackTrace();
            System.out.println("===================上传失败======================");
        }
        finally {
            //关闭资源
            if (in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return row;
    }

    /**
     * 判断上传的excel文件版本（xls为2003，xlsx为2017）
     * @param fileName 文件路径
     * @return excel2007及以上版本返回true，excel2007以下版本返回false
     */
    private static boolean judegExcelEdition(String fileName){
        if (fileName.matches("^.+\\.(?i)(xls)$")){
            return false;
        }else {
            return true;
        }
    }
    
}
```

 现在来进行测试,编写controller层的测试代码:

```java
//接受文件上传
@AResponseBody
@PostMapping("/upload")
public ResponseData uploadFile(MultipartFile file) {
    Map<String, Object> map = new HashMap<>(16);
    //调用工具类解析excel文件
    List<ArrayList<String>> row = ExcelUtils.analysis(file);
    //打印信息
    for (int i = 0; i < row.size(); i++) {
        List<String> cell = row.get(i);
        for (int j = 0; j < cell.size(); j++) {
            System.out.print(cell.get(j) + " ");
        }
        System.out.println();
    }
    return new ResponseData(row);
}
```