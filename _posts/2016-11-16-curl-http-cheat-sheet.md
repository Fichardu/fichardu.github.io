---
layout: post
title:  "curl CheatSheet"
date:   2016-12-13
categories: Program
---

1. GET

		curl http://example.com

2. GET with Headers

		curl -H 'key1: value1' -H 'key2: value2' http://example.com

3. POST

		curl -d 'data' http://example.com
		curl -X POST --data-urlencode "data" http://example.com

4. 指定输出文件

		curl -o filename http://example.com

5. 区间请求

		curl http://example.com/id/[123-125]
		curl http://example.com/id/{123,124,125}

6. 输出详细信息

		curl -v http://example.com

7. 只输出头信息

		curl -I http://example.com

8. 上传文件

		curl -X POST -F "filename=@path" -F "field=value" http://example.com



**其他参考**
	[https://bagder.github.io/curl-cheat-sheet/http-sheet.html](https://bagder.github.io/curl-cheat-sheet/http-sheet.html)
