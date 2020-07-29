# Python-视频信息爬取
一些常用网站视频信息爬取，内容信息都是基本的，没什么实际意义，主要是学习。
### 鹅厂视频网站
##### 说明
在学习笔记中根据教程写了一个基础爬虫，学习笔记中内容比较杂，去掉了一些多余注释以及稍微优化了一下，url内容更改成鹅厂视频网站任意一个相关链接应该都可以，我试过更换条目爬取。
##### 代码
代码如下：
```python
import re
from urllib import request
# 定义一个类Reptile
class Reptile():
    # 定义一个类变量url保存抓取网页的地址
    url = 'https://v.qq.com/channel/movie?listpage=1&channel=movie&itype=100012'
    root_pattern = '<div class="list_item" __wind>([\s\S]*?</div>[\s\S]*?</div>[\s\S]*?</div>[\s\S]*?</div>[\s\S]*?)</div>'
    name_pattern = ' class="figure_title figure_title_two_row bold">([\s\S]*?)</a>'
    score_pattern = '<div class="figure_score">([\s\S]*?)</div>'
    views_pattern = '</svg>([\s\S]*)$'
    # 定义一个私有方法获取网页内容
    def __fetch_content(self):
        r = request.urlopen(Reptile.url)
        htmls = r.read()
        htmls = str(htmls,encoding='utf-8')
        return htmls
    # 定义一个实例方法匹配内容
    def __analysis(self,htmls):
        root_html = re.findall(Reptile.root_pattern,htmls)
        films = []
        for html in root_html:
            name = re.findall(Reptile.name_pattern,html)
            score = re.findall(Reptile.score_pattern,html)
            views = re.findall(Reptile.views_pattern,html)
            film = {'name':name,'score':score,'views':views}
            films.append(film)
        return(films)
    # 定义一个方法提炼优化输出结果
    def __refine(self,films):
        refine = lambda film:{'name':film['name'][0],
        'score':film['score'][0].strip(),
        'views':film['views'][0]}
        return map(refine,films)
    # 指定排序规则
    def __sort_rule(self,film):
        num = re.findall('\d*',film['views'])
        number = float(num[0])
        if '万' in film['views']:
            number *= 10000
        return number
    # 进行排序
    def __sort(self,films):
        films = sorted(films,key=self.__sort_rule,reverse=True)
        return films
    # 展示出结果并加上序号
    def __show(self,films):
        for rank in range(0,len(films)):
            print(str(rank + 1) +'. '
            +'电影名：'+ '《' + films[rank]['name']+'》'+' 评分:'+films[rank]['score']+'分'
            +' 观看人数：'+films[rank]['views'])
    def entrance(self):
        htmls = self.__fetch_content()
        films = self.__analysis(htmls)
        films = list(self.__refine(films))
        films = self.__sort(films)
        self.__show(films)
reptile = Reptile()
reptile.entrance()
```
##### 结果
运行结果如下：
```
1. 电影名：《银河护卫队》 评分:8.9分 观看人数：8771万
2. 电影名：《钢铁飞龙之奥特曼崛起》 评分:6.6分 观看人数：8567万
3. 电影名：《黑猫警长之翡翠之星》 评分:7.4分 观看人数：8315万
4. 电影名：《末日之战》 评分:7.6分 观看人数：6830万
5. 电影名：《死亡飞车》 评分:8.6分 观看人数：5685万
6. 电影名：《星球大战8：最后的绝地武士》 评分:8.4分 观看人数：4990万
7. 电影名：《恐龙侵袭》 评分:6.5分 观看人数：3785万
8. 电影名：《星球大战9：天行者崛起》 评分:7.7分 观看人数：2452万
9. 电影名：《夺命五头鲨》 评分:6.3分 观看人数：1893万
10. 电影名：《惊涛迷局》 评分:7.8分 观看人数：400万
11. 电影名：《美人鱼》 评分:8.3分 观看人数：6亿
12. 电影名：《头号玩家》 评分:9.2分 观看人数：5亿
13. 电影名：《流浪地球》 评分:8.8分 观看人数：5亿
14. 电影名：《变形金刚5：最后的骑士》 评分:7.4分 观看人数：5亿
15. 电影名：《狂暴巨兽》 评分:8.1分 观看人数：4亿
16. 电影名：《复仇者联盟3：无限战争》 评分:8.9分 观看人数：4亿
17. 电影名：《复仇者联盟4：终局之战》 评分:9.2分 观看人数：3亿
18. 电影名：《环太平洋：雷霆再起》 评分:7.7分 观看人数：2亿
19. 电影名：《大黄蜂》 评分:8.4分 观看人数：2亿
20. 电影名：《复仇者联盟》 评分:9分 观看人数：2亿
21. 电影名：《变形金刚4：绝迹重生》 评分:8.2分 观看人数：2亿
22. 电影名：《哥斯拉2：怪兽之王》 评分:7.7分 观看人数：1亿
23. 电影名：《惊天魔盗团》 评分:8.7分 观看人数：1亿
24. 电影名：《钢铁飞龙之再见奥特曼》 评分:6.2分 观看人数：1亿
25. 电影名：《速度与激情：特别行动》 评分:8.1分 观看人数：1亿
26. 电影名：《星际穿越》 评分:9.4分 观看人数：1亿
27. 电影名：《超体》 评分:8.5分 观看人数：1亿
28. 电影名：《终结者：黑暗命运》 评分:8分 观看人数：1亿
29. 电影名：《蝙蝠侠：黑暗骑士》 评分:9.4分 观看人数：1亿
30. 电影名：《哥斯拉》 评分:8.1分 观看人数：1亿
```
