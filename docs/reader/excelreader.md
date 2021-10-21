# Excel Reader

`excelreader` 插件实现了从 Microsoft Excel 文件读取数据的能力。

## 配置

### 获取样例文件

从[这里下载](../assets/excel_reader_demo.zip)用于演示的Excel压缩文件，并解压缩放置到 `/tmp/in` 目录下。
三个文件夹的内容相同，其中

- `demo.xlsx` 是 Excel 新格式
- `demo.xls` 是 Excel 老格式
- `demo_gbk.xlsx` 是在Windows下创建，已GBK编码存储的文件

文件内容，如下图所示：

![excel demo](../images/excel_demo.png)

表头大致说明了单元数据的特征

### 创建 job 文件

创建如下 json 文件

=== "excel2stream.json"

	```json
	--8<-- "jobs/excelreader.json"
	```

将输出内容存保为到 `job/excel2stream.json` 文件中，执行采集命令：

```shell
$ bin/addax.sh job/excel2stream.json
```

如果没有报错，应该得到如下输出：

<details>
<summary>点击展开</summary>

```shell
--8<-- "output/excelreader.txt"
```

</details>

## 参数说明

| 配置项   | 是否必须 | 类型        | 默认值 | 描述                             |
| :------- | -------- | ----------- | ------ | -------------------------------- |
| path     | 是       | string/list | 无     | 指定要读取的文件夹，可以指定多个 |
| header   | 否       | boolean     | false  | 文件是否包含头                   |
| skipRows | 否       | int         | 0      | 要跳过前多少行                   |

### header

Excel 文件是否包含头，如果包含，则跳过

### skipRows

指定要跳过的行数， 默认为0，表示不跳过。这里要注意的是，假定 设置了 `header` 为 true，同时设置 `skipRows` 为 2。则表示前三行都跳过。
如果 `header` 为 false， 则表示跳过前两行。

### 支持的数据类型

Excel 读取功能的实现依赖于 [Apache POI](https://poi.apache.org/) 项目，该实现对单元格的数据类型定义很宽泛。
仅定义了布尔型(Boolean)，数值型（Double)，字符串型(String) 三种。其中数值型包括所有整数，小数和日期。
目前对于数值类型做了简单区分

1. 使用库工具类探测是否为日期类型，如果是，则判断为日期类型
2. 将数值转换为长整形，并和原值比较，如果大小相等，则判断为长整型(Long)
3. 否则判断为浮点型（Double）

## 限制

1. 当前仅读取文件的第一个 Sheet 而忽略其他 Sheets
2. 暂不支持指定列读取
3. 暂不支持跳过尾部行数（比如有总结的尾行可能并不符合要求）
4. 暂不判断每一行的列数是否相等，需要 Excel 自行保证
5. 仅会读取指定目录下文件后缀为 `xlsx` 或 `xls` 的文件，其他后缀文件将会忽略并给出告警消息