title: Jsoup爬取oschina博客
date: 2017-11-27 15:21:33
comments: true
tags:
 - Jsoup
 - 博客
 - 爬虫
categories: Java
----------

jsoup 是一款Java 的HTML解析器，可直接解析某个URL地址、HTML文本内容。它提供了一套非常省力的API，可通过DOM，CSS以及类似于jQuery的操作方法来取出和操作数据。

<!-- more -->

```java
public class OSChinaSpider {
	public static void main(String[] args) throws IOException {
		int num = 0;
		String currUrl = "";
		String baseUrl = "http://www.oschina.net/action/ajax/get_more_recommend_blog?classification=0&p=";
		boolean flag = true;
		OsChina osChina = null;
		Document document = null;
		Elements elements = null;
		List<OsChina> recommend = new ArrayList<OsChina>();
		long start = System.currentTimeMillis();
		System.out.println("开始爬取..");
		while(flag){
			num++;
			currUrl = baseUrl + num;
			document = Jsoup.connect(currUrl).userAgent("Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36").get();
			elements = document.select(".box.item");
			if(elements.size() < 20)
				flag = false;
			for (Element element : elements) {
				osChina = new OsChina();
				osChina.setHome(element.select(".box-fl a").attr("href"));
				osChina.setAvatar(element.select(".box-fl a img").attr("data-delay"));
				osChina.setName( element.select(".box-aw footer span").get(0).text());
				osChina.setTitle(element.select(".box-aw header a").attr("title"));
				osChina.setUrl(element.select(".box-aw header a").attr("href"));
				recommend.add(osChina);
			}
		}
		long end = System.currentTimeMillis();
		System.out.println("结束爬取.. 用时 " + ((end - start) / 1000) + " 秒,爬取 " + (--num) + "个页面,获取数据 " + recommend.size() + " 条");
		FileWriter writer = null;
		start = System.currentTimeMillis();
		System.out.println("\n开始写出..");
        try {
            writer = new FileWriter("F:/recommend.txt" , false);//true表示在源文件后追加，false表示覆盖，默认false
            for (OsChina oc : recommend) {
				writer.write(oc.toString());
			}
            writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }finally{
        	end = System.currentTimeMillis();
        	System.out.println("结束写出.. 用时 " + ((end - start) / 1000) + "秒");
        }
	}
}
```