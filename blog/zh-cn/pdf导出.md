---
title: freemaker+itext pdf导出网页 
date: 2019-07-23 
tags:
- pdf         
---
### 添加maven依赖
```xml
   <!-- pdfHTML -->
        <dependency>
            <groupId>com.itextpdf</groupId>
            <artifactId>html2pdf</artifactId>
            <version>1.0.2</version>
        </dependency>
        <!-- add all iText 7 Community modules -->
        <dependency>
            <groupId>com.itextpdf</groupId>
            <artifactId>itext7-core</artifactId>
            <version>7.0.5</version>
            <type>pom</type>
        </dependency>
    <!-- freemarker engine -->
        <dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.28</version>
        </dependency>
```
### freemaker根据模板生成html 
` FreeMarkerUtils.htmHandler(templatePath, templateName, htmlUrl, map);`
```java
  /**
     * @param templatePath 模板路径
     * @param templateName 模板名称
     * @param htmUrl       html页面全路径
     * @param paramMap     渲染ftl的数据模型
     * @throws Exception
     * @author xly
     */
    public static void htmHandler(String templatePath, String templateName,
                                  String htmUrl, Map<String, Object> paramMap) throws Exception {
        Configuration cfg = new Configuration(Configuration.VERSION_2_3_28);
        //设置字符编码
        cfg.setDefaultEncoding("UTF-8");
        //设置模板加载路径
        cfg.setDirectoryForTemplateLoading(new File(templatePath));
        //获取模板
        Template template = cfg.getTemplate(templateName);
        //构建html文件
        File outHtmFile = new File(htmUrl);
        //写出html文件
        Writer out = new BufferedWriter(new OutputStreamWriter(
                new FileOutputStream(outHtmFile), "UTF-8"));
        template.process(paramMap, out);

        out.close();
    }
```
### html2pdf 
```java
    private void html2pdf(String htmlUrl, String pdfName) throws IOException {
        File htmlSource = new File(htmlUrl);
        FileInputStream inputStream = new FileInputStream(htmlSource);
        String pdfPath = FreeMarkerUtils.getTemplatePath("pdf");
        FileUtils.createDirectory(pdfPath);
        //声明输出流
        FileOutputStream fileOutputStream = new FileOutputStream(pdfPath + pdfName + ".pdf");
        //写入流信息
        fileOutputStream.write(Html2PdfUtil.convert(inputStream));
        fileOutputStream.close();
    }
```
### 压缩文件夹打包zip导出
```java
	/**
	 * 压缩成ZIP 方法1
	 * @author xly
	 * @param srcDir           压缩文件夹路径
	 * @param out              压缩文件输出流
	 * @param KeepDirStructure 是否保留原来的目录结构,true:保留目录结构;
	 *                         false:所有文件跑到压缩包根目录下(注意：不保留目录结构可能会出现同名文件,会压缩失败)
	 * @throws RuntimeException 压缩失败会抛出运行时异常
	 */
	public static void toZip(String srcDir, OutputStream out, boolean KeepDirStructure)
			throws RuntimeException {

		long start = System.currentTimeMillis();
		ZipOutputStream zos = null;
		try {
			zos = new ZipOutputStream(out);
			File sourceFile = new File(srcDir);
			compress(sourceFile, zos, sourceFile.getName(), KeepDirStructure);
			long end = System.currentTimeMillis();
			System.out.println("压缩完成，耗时：" + (end - start) + " ms");
		} catch (Exception e) {
			throw new RuntimeException("zip error from ZipUtils", e);
		} finally {
			if (zos != null) {
				try {
					zos.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}

	}
```
### 前端导出进度展示
html 页面
```html
 <link rel="stylesheet" href="${ctxStatic}/components/jquery-circliful/css/jquery.circliful.css"/>
  <script type="text/javascript" src="${ctxStatic}/components/jquery-circliful/js/jquery.circliful.js"></script>
  
  
  <!-- Mask是遮罩，Progress是进度条 -->
<div>
    <div id="Mask"></div>
    <div id="Progress" data-dimension="200" data-text="0%" data-info="导出进度" data-width="15" data-fontsize="30"
         data-percent="0" data-fgcolor="##6633CC" data-bgcolor="#eee" data-fill="#ddd"></div>
</div>
```
js
```js
jseasyui.exam.examperiod.main.count;
jseasyui.exam.examperiod.main.batchDownload = function () {
    //校验
    var schoolYear = $('#schoolYear').val();
    var schoolTerm = $('#schoolTerm').val();
    var examType = $('#testType').val();
    if (schoolYear == '') {
        jseasyui.tips('提示', "请选择学年！", 5);
        return;
    }
    if (schoolTerm == '') {
        jseasyui.tips('提示', "请选择学期！", 5);
        return;
    }
    if (examType == '') {
        jseasyui.tips('提示', "请选择考试类型！", 5);
        return;
    }
    var param = {
        schoolYear: schoolYear,
        schoolTerm: schoolTerm,
        examType: examType
    };
    var asyncParam = {
        async: false
    };
    var count;
    jseasyui.ajaxJson('excel/getTotalCount', param, function (data) {
        count = data.data;
    }, asyncParam);
    if (count == 0) {
        jseasyui.tips('提示', "没有找到数据！", 5);
    } else {
        //导出
        window.location.href = 'excel/exportZip?schoolYear=' + schoolYear + '&schoolTerm=' + schoolTerm + '&examType=' + examType;
        /*** 进度条的显示  */
        jseasyui.exam.examperiod.main.showProgress();
        jseasyui.exam.examperiod.main.timer();
        isFirstExport = false;
    }
}
/**
 * 定时器 刷新进度
 */
jseasyui.exam.examperiod.main.timer = function () {
    window.setTimeout(function () {
        var timer = window.setInterval(function () {
            jseasyui.ajaxJson('excel/flushProgress', {}, function (data) {
                $("#Progress .circle-text").text(data.data.percentText);
                $("#Progress .circle-info").text("导出进度:" + data.data.curCount + "/" + data.data.totalCount);
                if (data.data.percent == "100") {
                    window.clearInterval(timer);
                    jseasyui.exam.examperiod.main.hideProgress();
                }
            })
        }, 800);
    }, 800);
}
//显示进度条
var isFirstExport = true;
jseasyui.exam.examperiod.main.showProgress = function () {
    $("#Mask").css("height", $(document).height());
    $("#Mask").css("width", $(document).width());
    $("#Mask").show();
    if (isFirstExport) {
        $("#Progress").circliful();
    } else {
        $("#Progress .circle-text").text("0%");
        $("#Progress .circle-info").text("导出进度");
        $("#Progress").show();
    }
}
//隐藏进度条
jseasyui.exam.examperiod.main.hideProgress = function () {
    $("#Mask").hide();
    $("#Progress").hide();
}
```
### 效果
![](https://xly2018.github.io/long/img/a6848d34.gif)