---
title: 使用itext生成pdf
date: 2020-07-16 08:33:11
categories: miscellaneous
---

> itext是用来处理pdf内容的开源工具（商业使用需要收费），可以用来根据模板生成pdf。

<!-- more -->

### 简介

---

- Github参见[这里](https://github.com/itext/itext7)，官网参见[这里](https://itextpdf.com/en)，帮助文档上有一些[例子](https://kb.itextpdf.com/home/it7kb/examples)，另外如果有疑问可以在[知识库](https://kb.itextpdf.com/home)的搜索框里输入，有可能会找到对应的答案（知识库中的一些问题来自stackoverflow上的解答）。
- itext分为若干个组件，有的用于ocr，有的用于绘制pdf，有的用于html转换pdf，等等
- 这次要做的是利用itextpdf，利用调整好的html，转换成指定格式的pdf

### 上手代码

---

- 首先需要在pom中引入itextpdf的依赖如下

  ```
          <dependency>
              <groupId>com.itextpdf</groupId>
              <artifactId>html2pdf</artifactId>
              <version>${com.itextpdf.html2pdf.version}</version>
          </dependency>
  
          <dependency>
              <groupId>com.itextpdf</groupId>
              <artifactId>forms</artifactId>
              <version>${com.itextpdf.version}</version>
          </dependency>
  
          <dependency>
              <groupId>com.itextpdf</groupId>
              <artifactId>svg</artifactId>
              <version>${com.itextpdf.version}</version>
          </dependency>
  
          <dependency>
              <groupId>com.itextpdf</groupId>
              <artifactId>styled-xml-parser</artifactId>
              <version>${com.itextpdf.version}</version>
          </dependency>
  
          <dependency>
              <groupId>com.itextpdf</groupId>
              <artifactId>layout</artifactId>
              <version>${com.itextpdf.version}</version>
          </dependency>
  
          <dependency>
              <groupId>com.itextpdf</groupId>
              <artifactId>kernel</artifactId>
              <version>${com.itextpdf.version}</version>
          </dependency>
  
          <dependency>
              <groupId>com.itextpdf</groupId>
              <artifactId>io</artifactId>
              <version>${com.itextpdf.version}</version>
          </dependency>
  ```

  

- itext对于中文的支持，需要引入itext-asian包或者引入中文字体；这里通过引入微软雅黑字体解决：

  ```
   ConverterProperties props = new ConverterProperties();
   FontProvider fp = new FontProvider();
   fp.addStandardPdfFonts();
   // .ttf 字体所在目录
   String resources = 		Html2PdfUtil.class.getResource(FONT_RESOURCE_DIR).getPath();
   fp.addDirectory(resources);
   props.setFontProvider(fp);
  ```

- 由于需要使用html转换为pdf，同时对于部分章节需要另起一页，所以使用了如下方法，其中PAGE_SIZE为定义好的页面大小，可以设置为PAGE_SIZE.A4等等；

  ```
   //html转成元素
  List<IElement> elements = HtmlConverter.convertToElements(htmlContent, props);
  PdfDocument pdf = new PdfDocument(new PdfWriter(dest));
  Document document = new Document(pdf, PAGE_SIZE, false);
  ```

- 由于需要为每一页添加页眉页脚，参考[这个问题](https://kb.itextpdf.com/home/it7kb/faq/how-to-add-text-as-a-header-or-footer)，增加对应的handler，用于处理页眉页脚：

  ```
   /**
   * 添加页眉页脚文字与分割线
   * pdf绘制时坐标可以参考平面直角坐标系的第一象限
   */
   private static class HeaderFooterHandler implements IEventHandler {
   	protected String info;
      public void setInfo(String info) {
      	this.info = info;
      }
      public String getInfo() {
      	return info;
      }
  	@Override
  	public void handleEvent(Event event) {
   		PdfDocumentEvent docEvent = (PdfDocumentEvent) event;
          PdfPage page = docEvent.getPage();
   		Rectangle pageSize = page.getPageSize();
   		PdfDocument pdfDoc = ((PdfDocumentEvent) event).getDocument();
   		PdfCanvas pdfCanvas = new PdfCanvas(page.newContentStreamBefore(),page.getResources(), pdfDoc);
   		// 画页脚分割线
   		// MARGIN_LEFT为左侧留白，FOOTER_LINE_BOTTOM_DISTANCE为页脚线到底部距离，MARGIN_RIGHT为右侧留白
   		pdfCanvas.setStrokeColor(ColorConstants.GRAY) .moveTo(MARGIN_LEFT, FOOTER_LINE_BOTTOM_DISTANCE).lineTo(PAGE_SIZE.getWidth()- MARGIN_RIGHT, FOOTER_LINE_BOTTOM_DISTANCE).closePathStroke();
   		// 画页眉分割线
   		// HEADER_LINE_TOP_DISTANCE为页眉线到顶部距离
   		pdfCanvas.setStrokeColor(ColorConstants.GRAY).moveTo(MARGIN_LEFT, pageSize.getHeight() - HEADER_LINE_TOP_DISTANCE).lineTo(PAGE_SIZE.getWidth()- MARGIN_RIGHT, pageSize.getHeight() - HEADER_LINE_TOP_DISTANCE).closePathStroke();
   		try {
              // 加载字体
              PdfFont YaHeiFont = PdfFontFactory.createFont("/font/WeiRuanYaHei-1.ttf", PdfEncodings.IDENTITY_H,true);
              //header
              float headerFontSize = 14;
              String headerTextContent = "页眉文字";
              int headerTextLength = headerTextContent.length();
              // 这个Rectangle用于定位，定位点在矩形左下角
              // x坐标：长方形左侧边线
              // y坐标：长方形底部边线
              // MARGIN_RIGHT为右侧留白
              // HEADER_PADDING_BOTTOM为页眉文字到页眉线留白
              Rectangle headerRightPosition = new Rectangle((float) (PAGE_SIZE.getWidth() - MARGIN_RIGHT - Math.ceil(headerFontSize * headerTextLength)),
   pageSize.getHeight() - (HEADER_LINE_TOP_DISTANCE- (0.5f * headerFontSize)) +  HEADER_PADDING_BOTTOM , headerFontSize * headerTextLength, headerFontSize);
               Canvas headerRightCanvas = new Canvas(pdfCanvas, pdfDoc, headerRightPosition);
               // 设置下字体
               Text headerText = new Text(headerTextContent).setFont(YaHeiFont).setFontSize(headerFontSize).setFontColor(ColorConstants.GRAY);
               Paragraph headerParagraph = new Paragraph().add(headerText).setMarginRight(0);
               headerRightCanvas.add(headerParagraph);
               //footer
               // 页码
               float footerFontSize = 12;
               String pageNumTextContent = "P" + pdfDoc.getPageNumber(page);
               int pageNumTextLength = pageNumTextContent.length();
               // x坐标：左侧margin
               // y坐标：页脚线到页面底部-字体占位高度
               Rectangle pageNumPosition =
               new Rectangle(MARGIN_LEFT, FOOTER_LINE_BOTTOM_DISTANCE - footerFontSize,footerFontSize * pageNumTextLength, footerFontSize);
               Canvas pageNumCanvas = new Canvas(pdfCanvas, pdfDoc, pageNumPosition);
               Text pageNumText = new Text(pageNumTextContent).setFont(YaHeiFont).setFontSize(footerFontSize).setFontColor(ColorConstants.GRAY);
               Paragraph pageNumParagraph = new Paragraph().add(pageNumText).setMarginLeft(0);
               pageNumCanvas.add(pageNumParagraph);
               // 页脚文字
               int footerTextLength = info.length();
               // x坐标：页面宽度-右侧margin-字体占位宽度
               // y坐标：页脚线到页面底部-字体占位高度
               Rectangle footerRightPosition = new Rectangle(PAGE_SIZE.getWidth() - MARGIN_RIGHT  - footerFontSize*footerTextLength,
               FOOTER_LINE_BOTTOM_DISTANCE - footerFontSize, footerTextLength * footerFontSize, footerFontSize);
               Canvas footerRightCanvas = new Canvas(pdfCanvas, pdfDoc, footerRightPosition);
               footerRightCanvas.setFont(YaHeiFont);
               Text footerText = new Text(info).setFont(YaHeiFont).setFontColor(ColorConstants.GRAY).setFontSize(footerFontSize);
               Paragraph footerTextParagraph = new Paragraph().add(footerText).setMarginRight(0).setTextAlignment(TextAlignment.RIGHT);
               footerRightCanvas.add(footerTextParagraph);
   		}
      	catch (Exception e){
      		e.printStackTrace();
      	}
  	}
  }
  ```

- 同理，可以为每一页添加背景图片：

  ```
   /**
   * 添加背景图片
   */
   private static class BackgroundEventHandler implements IEventHandler {
       public static final String BACKGROUND_IMAGE = Html2PdfUtil.class.getResource("/pdfTemplate").getPath() + "/background.png";
       @Override
       public void handleEvent(Event event) {
           PdfDocumentEvent docEvent = (PdfDocumentEvent) event;
           PdfDocument pdfDoc = docEvent.getDocument();
           PdfPage page = docEvent.getPage();
           try {
               // 添加背景图片
               Image backgroundImage = new Image(ImageDataFactory.create(BACKGROUND_IMAGE));
               PdfCanvas background = new PdfCanvas(page.newContentStreamBefore(),
               page.getResources(), pdfDoc);
               Rectangle areaBackground = page.getPageSize();
               new Canvas(background, pdfDoc, areaBackground)
               .add(backgroundImage);
           } catch (Exception e){
           	e.printStackTrace();
           }
  
      }
  }
  ```

- 完整的生成pdf方法如下：

  ```
  /**
  * 保存风险评述，生成pdf方法
  * @param htmlContent html文本
  * @param dest        目的文件路径，如 /xxx/xxx.pdf
  * @throws IOException IO异常
  */
  public static void createPdf(String htmlContent, String dest) throws IOException {
      ConverterProperties props = new ConverterProperties();
      FontProvider fp = new FontProvider();
      fp.addStandardPdfFonts();
      // .ttf 字体所在目录
      String resources = Html2PdfUtil.class.getResource(FONT_RESOURCE_DIR).getPath();
      fp.addDirectory(resources);
      props.setFontProvider(fp);
      //html转成元素
      List<IElement> elements = HtmlConverter.convertToElements(htmlContent, props);
      PdfDocument pdf = new PdfDocument(new PdfWriter(dest));
      Document document = new Document(pdf, PAGE_SIZE, false);
      //设置下留白
      document.setMargins(MARGIN_TOP, MARGIN_LEFT, MARGIN_BOTTOM, MARGIN_RIGHT);
      //背景图片处理器
      BackgroundEventHandler backgroundEventHandler = new BackgroundEventHandler();
      pdf.addEventHandler(PdfDocumentEvent.END_PAGE, backgroundEventHandler);
      //页眉页脚分割线与文字处理器
      HeaderFooterHandler handler = new HeaderFooterHandler();
      handler.setInfo("页脚文字");
      pdf.addEventHandler(PdfDocumentEvent.START_PAGE, handler);
  
      //添加分页符，此处用于在指定的元素位置添加分页符，非必须
      for (IElement element : elements) {
          if (element instanceof Div) {
          Div div = (Div) element;
          List<IElement> children = div.getChildren();
          children.add(2,new AreaBreak());
          }
      }
  
      for (IElement element : elements) {
          if (element instanceof HtmlPageBreak) {
              // 分页符
          	document.add((HtmlPageBreak) element);
          } else if (element instanceof Paragraph) {
              //段落，这里可以统一设置文字对齐
              Paragraph p = (Paragraph) element;
              p.setTextAlignment(TextAlignment.JUSTIFIED);
              document.add(p);
          } else if (element instanceof Div) {
          	//块
              Div div = (Div) element;
              document.add(div);
          } else if (element instanceof com.itextpdf.layout.element.List) {
          	//列表
              com.itextpdf.layout.element.List list = (com.itextpdf.layout.element.List) element;
              document.add(list);
          } else {
          	document.add((IBlockElement) element);
          }
      }
      document.close();
  }
  ```

### 总结

---

调试下来，发现最有用的就是官网知识库还有源码，官网知识库能够解决怎么做的问题，对于一些细节的调整，可以查看源码并调试，就能得到想要的结果。