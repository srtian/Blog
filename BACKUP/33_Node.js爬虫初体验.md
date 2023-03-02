# [Node.js爬虫初体验](https://github.com/srtian/Blog/issues/33)


### 一、准备阶段

当我们需要使用Node.js进行爬虫爬取网页时，我们通常需要下载两个库request和cheerio来帮助我们队网页进行爬取：

```
cnpm i request cheerio
```

其中request帮助我们对网页进行加载，而cheerio则是为服务器特别定制的，快速、灵活、实施的jQuery核心实现。有了这两个库，爬取简单的网页就没有太大的问题了。



### 二、网页分析

本次我的目标是爬取豆瓣电影Top250的第一页，因此打开浏览器对其页面结构进行了一番分析，就比如第一步电影——《肖申克的救赎》：<br />
![](https://images.gitee.com/uploads/images/2018/0726/154313_2415be33_1575229.png#align=left&display=inline&height=687&originHeight=687&originWidth=712&status=done&width=712)

通过上面的页面结构，我们不难看出，每一部电影都是一个li，且li下面的class都是item,因此当我们需要爬取一部电影的数据时，可以先取得item，再对内部的数据进行获取。其次，每部电影的数据的class命名很明确，因此当我们获取数据时，直接可以根据页面的class命名进行数据获取。而根据对页面的分析，我打算爬取的数据如下：

- 电影名称——name
- 评分——score
- 评语——quote
- 排名——ranking
- 封面地址——coverUrl



### 三、代码编写

既然确定所要获取的数据，我们就可以着手写代码了，首先我们需要引入上面我们下载好的两个包：

```javascript
const request = require('request')
const cheerio = require('cheerio')
```

然后我们要创造一个类，用以保存我们想要获取的数据：

```javascript
const Movie = function() {
    this.name = ''
    this.score = 0
    this.quote = ''
    this.ranking = 0
    this.coverUrl = ''
}
```

然后我们就能根据我们在上面所创造的类以及利用cheerio来定义一个函数，来通过传入的元素对数据进行获取：

```javascript
const getMovieFromDiv = (div) => {
    const movie = new Movie()
    const load = cheerio.load(div)
    const pic = load('.pic')
    movie.name = load('.title').text()
    movie.score = load('.rating_num').text()
    movie.quote = load('.inq').text()
    movie.ranking = pic.find('em').text()
    movie.coverUrl = pic.find('img').attr('src')
    return movie
}
```

将数据获取到了后，当然就需要将其保存了，我们可以调用Node.js的fs模块来对数据进行保存。在这里我们同样也可以定义一个函数，用以保存数据，鉴于在前端界数据通常是JSON，因此在这里就将数据保存为JSON格式：

```javascript
const saveMovie = (movies) => {
    const fs = require('fs')
    const path = 'DouBanTop25.json'
    const s = JSON.stringify(movies, null, 2)
    fs.writeFile(path, s, (error) => {
        if (error === null) {
            console.log('保存成功')
        } else {
            console.log('保存文件错误', error)
        }
    })
}
```

好了，上面两步主要为了处理数据以及保存数据。下面就是主要部分了，我们需要下载页面，并执行上面两个函数，已达到爬取网页数据并保存的目的：

```javascript
const getMoviesFromUrl = (url) => {
    request(url, (error, response, body) => {
        if (error === null && response.statusCode == 200) {
            const load = cheerio.load(body)
            const movieDiv = load('.item')
            const movies = []
            for(let i = 0; i < movieDiv.length; i++) {
                let element = movieDiv[i]
                const div = load(element).html()
                const movie = getMovieFromDiv(div)
                movies.push(movie)
            }
            saveMovie(movies)
        } else {
            console.log('请求失败', error)
        }
    })
}
```

在上面，当我们下载好页面后，先利用cheerio.load解析页面，然后我们创建一个数组，用于保存电影的数据。再然后通for循环遍历页面的item，并通过getMovieFromDiv来对每个item内的数据进行获取，然后push到数组内。在循环结束后，使用saveMovie将存有数据的数组进行保存。

基本的爬取数据的代码完成了，现在让我们来启动这些函数进行页面爬取吧！

```javascript
const getMovie = () =>{
    const url = 'https://movie.douban.com/top250'
    getMoviesFromUrl(url)
}

getMovie()
```

最后：

```
node doubantop25.js
```

