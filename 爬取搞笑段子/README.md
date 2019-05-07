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
      //获取页面规律，根据用户指定起始、终止页打开要爬取网页
      func main() {
		//指定爬取起始、终止页
		var start, end int
		fmt.Print("请输入爬取的起始页(>=1) : ")
		fmt.Scan(&start)
		fmt.Printf("请输入爬取的终止页(>=start) : ")
		fmt.Scan(&end)

		toWork(start, end)
	}
	    
        //处理请求    
	func toWork(start, end int) {
		fmt.Printf("正在爬取 %d 到 %d 页...\n", start, end)

		page := make(chan int)
		for i := start; i <= end; i++ {
			//开启协程并发处理
			go SpiderPage(i, page)
		}

		for i := start; i <= end; i++ {
			fmt.Printf("第 %d 页爬取完毕\n", <-page)
		}
	}
	 
	
	//爬取一个段子页面的 title 和 content
	func SpiderJokePage(url string) (title, content string, err error) {
		result, err1 := HttpGet(url)
		if err1 != nil {
			err = err1
			return
		}
		//编译解析正则表达式	-- title
		ret1 := regexp.MustCompile(`<h1>(?s:(.*?))</h1>`)
		//提取title,有2处，取第一个
		alls := ret1.FindAllStringSubmatch(result, 1)
		for _, tempTitle := range alls {
			title = tempTitle[1]
			title = strings.Replace(title, "\n", "", -1)
			title = strings.Replace(title, "\t", "", -1)
			break
		}
		//编译解析正则表达式	-- content
		ret2 := regexp.MustCompile(`<div class="content-txt pt10">(?s:(.*?))<a id="prev" href="`)
		//提取title,有2处，取第一个
		alls2 := ret2.FindAllStringSubmatch(result, 1)
		for _, tempContext := range alls2 {
			content = tempContext[1]
			content = strings.Replace(content, "\n", "", -1)
			content = strings.Replace(content, "\t", "", -1)
			break
		}
		return
	}
	
	
	//获取一个网页所有的内容， result返回
	func HttpGet(url string) (result string, err error) {
		resp, err1 := http.Get(url)
		if err1 != nil {
			err = err1
			return
		}
		defer resp.Body.Close()
		buf := make([]byte, 4096)
		for {
			n, err2 := resp.Body.Read(buf)
			if n == 0 {
				break
			}
			if err2 != nil && err2 != io.EOF {
				err = err2
				return
			}
			result += string(buf[:n])
		}
		return
	}

	func SaveJokeToFile(idx int, fileTile, fileContent []string) {
		path := "D:/GoProject/src/爬虫/爬取搞笑段子/" + "第 " + strconv.Itoa(idx) + " 页.txt"
		f, err := os.Create(path)
		if err != nil {
			fmt.Println("Create err: ", err)
			return
		}
		defer f.Close()
		n := len(fileTile)
		for i := 0; i < n; i++ {
			f.WriteString(fileTile[i] + "\n" + fileContent[i] + "\n")
			f.WriteString("\n\n")
		}
	}

	//SpiderPage 爬取一个网页，带有 10 个段子 -- 10url
	func SpiderPage(idx int, page chan int) {
		//拼接url
		url := "https://www.pengfue.com/xiaohua_" + strconv.Itoa(idx) + ".html"
		//封装函数获取段子的url
		result, err := HttpGet(url)
		if err != nil {
			fmt.Println("HttpGet err: ", err)
			return
		}
		//解析编译正则
		ret := regexp.MustCompile(`<h1 class="dp-b"><a href="(?s:(.*?))"`)
		//提取需要信息，每一个段子的url
		alls := ret.FindAllStringSubmatch(result, -1)

		//创建用户存储title、content 的切片,初始容量为0
		fileTile := make([]string, 0)
		fileContent := make([]string, 0)

		for _, jokeURL := range alls {
			// fmt.Println("jokeURL: ", jokeURL[1])
			//爬取一个段子页面的title和内容
			title, content, err := SpiderJokePage(jokeURL[1])
			if err != nil {
				fmt.Println("SpiderJokePage err: ", err)
				continue
			}
			// fmt.Println("title: ", title)
			// fmt.Println("content: ", content)
			//将title、content 追加到切片末尾
			fileTile = append(fileTile, title)
			fileContent = append(fileContent, content)

		}
		SaveJokeToFile(idx, fileTile, fileContent)
		//防止主go协程提前结束
		page <- idx
	}


## 编译运行
	go run main.go 
	会并发执行并生成10个.txt文件，每个文件会有10条段子，每条段子都包含段子标题和段子内容
	
	
	
	
	
	
