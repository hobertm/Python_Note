# Crawler  

- 爬虫工作流程  
将种子URL放入队列  
从队列中获取URL，抓取内容  
解析抓取内容，将需要进一步抓取的URL放入工作队列，存储解析后的内容  

- 抓取策略  
深度优先  
广度优先  
PageRank  
大站优先策略  