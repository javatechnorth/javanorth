#### 解析复杂表格

在实际开发中，上传excel文件是十分常见的问题，一般情况下，解析的思路无非1. 固定表头进行解析；2. 每一行进行解析。但是偶尔会碰一下一些格式比较复杂的表格，用以上方式解析就 得不到我们想要的结果了。
例如以下这张表，乍一看是不是有种心态崩溃的感觉，

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/feb2f1a039178e315d59dfff20536880.png#pic_center)



面对这种复杂表格，就需要采取特殊的方式了，首先，还是将思路，实现放到最后再说；1.按照每一行去解析，但是在解析时，需要判断是否为单元格；2. 得到数据后，还需要根据行号进行过滤，然后对每一行单元格数据进行合并操作；3. 得到数据后，最后需要根据列号进行过滤，对每一列单元格进行合并操作。

实现
话不多说，上代码：

#### 判断是否为单元格：

```java
/**
	 *
	 *
	 * @param sheet
	 *            表单
	 * @param cellRow
	 *            被判断的单元格的行号
	 * @param cellCol
	 *            被判断的单元格的列号
	 * @return row: 行数；col列数
	 * @throws IOException
	 * @Author zhangxinmin
	 */
	private static Map<String, Integer> getMergerCellRegionRow(Sheet sheet, int cellRow,
											  int cellCol) {
		Map<String, Integer> map = new HashMap<>();
		int retVal = 0, retCol= 0 ;
		int sheetMergerCount = sheet.getNumMergedRegions();
		for (int i = 0; i < sheetMergerCount; i++) {
			CellRangeAddress cra = (CellRangeAddress) sheet.getMergedRegion(i);
			int firstRow = cra.getFirstRow(); // 合并单元格CELL起始行
			int firstCol = cra.getFirstColumn(); // 合并单元格CELL起始列
			int lastRow = cra.getLastRow(); // 合并单元格CELL结束行
			int lastCol = cra.getLastColumn(); // 合并单元格CELL结束列
			if (cellRow >= firstRow && cellRow <= lastRow) { // 判断该单元格是否是在合并单元格中
				if (cellCol >= firstCol && cellCol <= lastCol) {
					retVal = lastRow - firstRow + 1; // 得到合并的行数
					retCol = lastCol - firstCol + 1; // 得到合并的列数
					break;
				}
			}
		}
		map.put("row", retVal);
		map.put("col", retCol);
		return map;
	}

private static Integer isMergedRegion(Sheet sheet,int row ,int column) {
		int sheetMergeCount = sheet.getNumMergedRegions();
		for (int i = 0; i < sheetMergeCount; i++) {
			CellRangeAddress range = sheet.getMergedRegion(i);
			int firstColumn = range.getFirstColumn();
			int lastColumn = range.getLastColumn();
			int firstRow = range.getFirstRow();
			int lastRow = range.getLastRow();
			if(row >= firstRow && row <= lastRow){
				if(column >= firstColumn && column <= lastColumn){
					return i;
				}
			}
		}
		return -1;
	}

```

解析代码：
CellRowAndColDTO定义类：

```java
@Getter
@Setter
public class CellRowAndColDTO {
    private String cellValue;
    private Integer row;
    private Integer col;
    private Integer cellRow;
    private Integer cellCol;
}

```

```java
public static List<CellRowAndColDTO> readDiffDataBySheet(Sheet sheet, int startRows){
		List<CellRowAndColDTO> result = new ArrayList<>();
		for (int rowIndex = startRows, z = sheet.getLastRowNum(); rowIndex <= z; rowIndex++) {
			Row row = sheet.getRow(rowIndex);
			if (row == null) {
				continue;
			}

			int rowSize = row.getLastCellNum();
			for (int columnIndex = 0; columnIndex < rowSize; columnIndex++) {
				CellRowAndColDTO dto = new CellRowAndColDTO();
				Cell cell = row.getCell(columnIndex);
				if (cell != null){
					// 读取单元格数据格式（标记为字符串）
					cell.setCellType(CellType.STRING);
					String value = cell.getStringCellValue();
					if(0 != isMergedRegion(sheet, rowIndex,columnIndex)){//判断是否合并格
						// 处理有值的cell
//						if (StringUtils.isEmpty(value)) {
//							continue;
//						}
						dto.setRow(rowIndex);
						dto.setCol(columnIndex);
						Map<String, Integer> map = getMergerCellRegionRow(sheet, rowIndex, columnIndex);//获取合并的行列
						dto.setCellCol(map.get("col") == 0? 1:map.get("col"));
						dto.setCellRow(map.get("row") == 0? 1:map.get("row"));
						dto.setCellValue(value);
						result.add(dto);

					}else{
						dto.setRow(rowIndex);
						dto.setCol(columnIndex);
						Map<String, Integer> map = getMergerCellRegionRow(sheet, rowIndex, columnIndex);//获取合并的行列
						dto.setCellCol(1);
						dto.setCellRow(1);
						dto.setCellValue(value);
						result.add(dto);
					}

				}
			}
		}
		List<CellRowAndColDTO> dtos = new ArrayList<>();
		Map<Integer, List<CellRowAndColDTO>> map = result.stream().collect(Collectors.groupingBy(CellRowAndColDTO::getRow));//根据行进行分组
		map.forEach((k,v) -> {
			for(int i =0;i<v.size();i++){
				if(i!=0){
					Integer col = dtos.get(dtos.size()-1).getCol()+dtos.get(dtos.size()-1).getCellCol();
					if(v.get(i).getCol() == col){
						dtos.add(v.get(i));
						continue;
					}
				}else{
					dtos.add(v.get(i));
				}

			}
		});

		List<CellRowAndColDTO> dtos2 = new ArrayList<>();
		Map<Integer, List<CellRowAndColDTO>> map2 = dtos.stream().collect(Collectors.groupingBy(CellRowAndColDTO::getCol));//根据列分组
		map2.forEach((k,v) -> {
			for(int i =0;i<v.size();i++){
				if(i!=0){
					if(v.get(i).getCellRow() != 1){
						if(v.get(i).getCellCol() == v.get(i-1).getCellCol() && v.get(i).getCellRow() == v.get(i-1).getCellRow()){
							if(v.get(i).getCellRow() == 1 && v.get(i).getCellCol() == 1){
								dtos2.add(v.get(i));
								continue;
							}else{
								if(StringUtils.isBlank((v.get(i).getCellValue()))){
									continue;
								}
							}
						}
					}

				}
				dtos2.add(v.get(i));
			}
		});
		return dtos2;
	}

```

说明一下： 首先我先获取每一行，然后对该行的每一个单元格cell对象进行判断处理，判断时候为单元格，如果是，则将行号，列号，合并行数，合并列数，数值进行存储，存储到List集合；然后，对该集合进行过滤操作，通过java8 stream流的方式先根据行号进行分组，然后获取下一个格的位置col ，然后进行判断，如果是下一个格则进行存储；如果是该单元格内的空格，则跳出循环。然后再根据列进行分组，根据行号列号进行对合并格其他空格单元格进行过滤，最后完成数据库存储，完成解析操作。

普通表格按行解析



```java
public static Workbook readWorkBook(String fileType, InputStream is) throws IOException {
	if ("xls".equals(fileType)) {
		return new HSSFWorkbook(is);
	} else if ("xlsx".equals(fileType)) {
		return new XSSFWorkbook(is);
	} else {
		throw new IllegalArgumentException("不支持的文件类型，仅支持xls和xlsx");
	}
}

public static List<String[]> readData(String fileType, int startRows, boolean ignoreRowBlank, InputStream is) throws IOException {
		List<String[]> result = new ArrayList<>();

		Workbook wb = readWorkBook(fileType, is);
		for (int sheetIndex = 0; sheetIndex < wb.getNumberOfSheets(); sheetIndex++) {
			Sheet sheet = wb.getSheetAt(sheetIndex);

			for (int rowIndex = startRows, z = sheet.getLastRowNum(); rowIndex <= z; rowIndex++) {
				Row row = sheet.getRow(rowIndex);
				if (row == null) {
					continue;
				}

				int rowSize = row.getLastCellNum();
				String[] values = new String[rowSize];
				boolean hasValue = false;
				for (int columnIndex = 0; columnIndex < rowSize; columnIndex++) {
					String value = "";
					Cell cell = row.getCell(columnIndex);
					if (cell != null) {
						// 注意：一定要设成这个，否则可能会出现乱码,后面版本默认设置
						switch (cell.getCellType()) {
							case HSSFCell.CELL_TYPE_STRING:
								value = cell.getStringCellValue();
								break;
							case HSSFCell.CELL_TYPE_NUMERIC:
								if (HSSFDateUtil.isCellDateFormatted(cell)) {
									Date date = cell.getDateCellValue();
									if (date != null) {
										value = new SimpleDateFormat("yyyy-MM-dd")
												.format(date);
									} else {
										value = "";
									}
								} else {
									//value = new DecimalFormat("0").format(cell.getNumericCellValue());
									if (HSSFDateUtil.isCellDateFormatted(cell)) {
										value = String.valueOf(cell.getDateCellValue());
									} else {
										cell.setCellType(Cell.CELL_TYPE_STRING);
										String temp = cell.getStringCellValue();
										// 判断是否包含小数点，如果不含小数点，则以字符串读取，如果含小数点，则转换为Double类型的字符串
										if (temp.indexOf(".") > -1) {
											value = String.valueOf(new Double(temp)).trim();
										} else {
											value = temp.trim();
										}
									}
								}
								break;
							case HSSFCell.CELL_TYPE_FORMULA:
								// 导入时如果为公式生成的数据则无值
								if (!cell.getStringCellValue().equals("")) {
									value = cell.getStringCellValue();
								} else {
									value = cell.getNumericCellValue() + "";
								}
								break;
							case HSSFCell.CELL_TYPE_BLANK:
								break;
							case HSSFCell.CELL_TYPE_ERROR:
								value = "";
								break;
							case HSSFCell.CELL_TYPE_BOOLEAN:
								value = (cell.getBooleanCellValue() == true ? "Y"

										: "N");
								break;
							default:
								value = "";
						}
					}
					values[columnIndex] = value;
					if (!value.isEmpty()) {
						hasValue = true;
					}
				}
				if (!ignoreRowBlank || hasValue) {//不为忽略空行模式或不为空行
					result.add(values);
				}
			}
		}
		return result;
	}

```



这里我就不过多叙述这个按行解析了，代码思路比较简单一看就能懂。

<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.17</version>
</dependency>

总结
该文章为我总结平时开发过程中解决的难题的经验和思路；如果有更好的解决办法希望能不吝赐教。大家携手在开发的道路上越走越远。不喜勿喷。

原文链接：https://blog.csdn.net/weixin_42803027/article/details/110189928