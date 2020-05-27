# Demo

Demo: [連結](https://reurl.cc/3D50ml)(部署在heroku)

## 爬蟲

以Scrapy實作爬蟲，實作檔案```MySite/SpiderBot/SpiderBotspiders/NBASpider.py```

爬蟲自動定時抓取
實作檔案```MySite/SpiderBot/main.py```
使用方式```python main.py --freq 間隔秒數```
```python
#main.py
#line 7
def Job(frequence):
    Args = ["scrapy", "crawl", 'NBANews']
    while True:
        p = Process(target=cmdline.execute, args=(Args,))
        p.start()
        p.join()
        print('完成爬蟲')
        print(f'等待: {frequence} 秒')
        time.sleep(int(frequence))
```

## DB

使用SQLite搭配Django，使用的model為```MySite/api/model.py```
僅會儲存尚未儲存過的新聞
```Python
#api/models.py
#line 5
class News(models.Model):
    ID = models.AutoField(primary_key=True) #PK
    CrawlerAt = models.DateTimeField(auto_now=True) #新聞爬取時間
    Title = models.CharField(max_length = 100) #新聞標題
    CreateAt = models.DateTimeField() #新聞發布時間
    Content = models.CharField(max_length = 1000) #新聞內容
    Url = models.CharField(max_length = 1000) #原始連結
```
![image](Image/SQLite.PNG)

## API 

(1)新聞列表

如Demo連結，實作檔案```MySite/api/templates/list.html```

![image](Image/newslist.PNG)
```javascript
//list.html
//line 31
function GetNewsList(){
	const xhr = new XMLHttpRequest();
	xhr.open('GET', '/api/news', true);
	xhr.send();
	xhr.onreadystatechange = function() {
		if (this.readyState === 4 && this.status === 200) {
			let jsonData = JSON.parse(this.responseText);
			for(var item of jsonData){
				id = item['ID']
				title = item['Title']
				NewsListAdd(id, title)
			}
		}
	}
}
```


(2)新聞詳情頁面

如Demo連結，實作檔案```MySite/api/templates/news.html```

![image](Image/newsdetail.PNG)
```javascript
//news.html
//line 27
var LocationUrl = window.location.href.split('/')
var NewsID = LocationUrl[LocationUrl.length - 1]
const xhr = new XMLHttpRequest();
xhr.open('GET', '/api/news/' + NewsID, true);
xhr.send();
xhr.onreadystatechange = function() {
	if (this.readyState === 4 && this.status === 200) {
    let jsonData = JSON.parse(this.responseText);
    ChangeNews(jsonData['Title'], jsonData['CreateAt'], jsonData['Content'])
	}
}
```

## Websocket

使用channels搭配channels-redis

由前端與後端Handshake後加入Group

前端console會顯示連接成功

![Image](Image/websocketConnect.PNG)

有新的焦點新聞時，會由後端廣播至該Group所有前端

前端接收到通知後會顯示Windows Notify並重新撈取api

![Image](Image/notify.PNG)

實作檔案

```MySite/api/templates/list.html```

```MySite/api/consumers.py```

```MySite/SpiderBot/SpiderBot/pipelines.py```
```js
//list.html
//line 83
BroadcastSocket.onmessage = function(e){
	var data = JSON.parse(e.data);
	var message = data['message'];
	var Type = message['type']
	switch (Type){
		case 'Broadcast':{
			document.getElementById('news').innerHTML = '' //清空列表
			Notify('有新焦點新聞') //Windows通知
			GetNewsList() //重新撈取api
			break;
		}
	}
}
```

## Container

以docker部署至heroku，如Demo連結
