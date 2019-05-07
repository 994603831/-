# 爬取捧腹网搞笑段子

## 进入捧腹网查找网页规律
    网址：https://www.pengfue.com
  
    规律：
      https://www.pengfue.com/xiaohua_1.html      1   下一页+1
      https://www.pengfue.com/xiaohua_2.html      2
      https://www.pengfue.com/xiaohua_3.html      3
    
## 查找网页的特性
    获取一个页面中，一个段子url特征: 
      <h1 class="dp-b"><a href="xxx链接" -- 10处
      去一个段子中的title 特征: <h1>title</h1> --> 有两处，取第一个
      去一个段子中的content 特征: <div class="content-txt pt10">段子内容<a id="prev" href="url">
    
## 实现步骤:
    第一，先获取页面规律，根据用户指定起始、终止页打开要爬取网页，并获取每一个中单个段子所对应的URL
    第二，依次将每一个段子对应的网页打开，读取标题和正文内容
    第三，将一个网页内容(10个段子)保存成一个 .txt文件。
    
## 具体实现：
    获取页面规律，根据用户指定起始、终止页打开要爬取网页
      //指定爬取起始、终止页
	    var start, end int
	    fmt.Print("请输入爬取的起始页(>=1) : ")
	    fmt.Scan(&start)
	    fmt.Printf("请输入爬取的终止页(>=start) : ")
	    fmt.Scan(&end)

	    toWork(start, end)
