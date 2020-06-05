
+ scrapy startproject [**projectname**]
+ scrapy crawl [**spidername**] (爬取spidername，spider文件中的name属性)
    + scrapy crawl [**spidername**] -o [json file name] (保存爬取到的数据为json文件)
+ scrapy shell [**url**] (载入url的response数据)
    + In [1]:
        + *response.body*
        + *response.header*
        + *response.selector*