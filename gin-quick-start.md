# 用go和gin开发一个RESTful API

这里只包含了一些路由request，解析request，封装JSON响应等基础部分，更多详细内容参考[Gin Web Framework(Gin)](https://gin-gonic.com/docs/)。

## 规划API接入点

接入点的规划要简单清晰易懂，用户通过接入点访问相关的资源。这些是教程中会用到的接入点

/albums

- GET - 获取相册列表，通过JSON返回
- POST - 通过JSON添加新的相册

/albums/:id

- GET - 根据id获取相册，通过JSON返回

## 准备数据

这里为了方便将使用的数据存储在内存，但一般情况下API都是跟数据库交互的。

- 声明数据结构

    ```go
    // album represents data about a record album.
    type album struct {
        ID     string  `json:"id"`
        Title  string  `json:"title"`
        Artist string  `json:"artist"`
        Price  float64 `json:"price"`
    }
    ```

- 定义一个实际用到的数据

    ```go
    // albums slice to seed record album data.
    var albums = []album{
        {ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
        {ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
        {ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
    }
    ```

## 编码

1. 一个返回所有相册的函数`getAlbums`

    ```go
    // getAlbums responds with the list of all albums as JSON.
    func getAlbums(c *gin.Context) {
        c.IndentedJSON(http.StatusOK, albums)
    }
    ```

    - `gin.Context`包含request，验证和JSON等信息，是Gin中最重要的部分。

    - `Context.IdentedJSON`将结构体序列化成JSON然后写进response中。

2. 添加handler并启动监听

    ```go
    func main() {
        router := gin.Default()
        router.GET("/albums", getAlbums)

        router.Run("localhost:8080")
    }
    ```

    - 这里用`Default`函数初始化一个Gin router。

    - 用`GET`绑定`/albums`这个接入点上的GET方法的处理函数。

## 运行

用`curl`测试运行的服务

```shell
root@c3f5bdb3bd8f:~/projects/playground# curl http://localhost:8080/albums
[
    {
        "id": "1",
        "title": "Blue Train",
        "artist": "John Coltrane",
        "price": 56.99
    },
    {
        "id": "2",
        "title": "Jeru",
        "artist": "Gerry Mulligan",
        "price": 17.99
    },
    {
        "id": "3",
        "title": "Sarah Vaughan and Clifford Brown",
        "artist": "Sarah Vaughan",
        "price": 39.99
    }
]
```

## 添加新记录

1. 添加新记录的代码

    ```go
    // postAlbums adds an album from JSON received in the request body.
    func postAlbums(c *gin.Context) {
        var newAlbum album

        // Call BindJSON to bind the received JSON to
        // newAlbum.
        if err := c.BindJSON(&newAlbum); err != nil {
            return
        }

        // Add the new album to the slice.
        albums = append(albums, newAlbum)
        c.IndentedJSON(http.StatusCreated, newAlbum)
    }
    ```

    - 用`Context.BindJSON`将request的body绑到变量`newAlbum`中。

    - 将解析获得的album信息拼接到列表中。

    - response中的201和JSON信息表示对应的album添加成功。

2. 改变`main`函数

    ```go
    func main() {
        router := gin.Default()
        router.GET("/albums", getAlbums)
        router.POST("/albums", postAlbums)

        router.Run("localhost:8080")
    }
    ```

    - 将postAlbums函数与`/albums`的POST方法绑定。

3. 检测

    ```shell
    root@c3f5bdb3bd8f:~/projects/playground# curl http://localhost:8080/albums \
        --include \
        --header "Content-Type: application/json" \
        --request "POST" \
        --data '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
    # 返回结果如下
    HTTP/1.1 201 Created
    Content-Type: application/json; charset=utf-8
    Date: Sat, 04 May 2024 15:31:15 GMT
    Content-Length: 116

    {
        "id": "4",
        "title": "The Modern Sound of Betty Carter",
        "artist": "Betty Carter",
        "price": 49.99
    }

    root@c3f5bdb3bd8f:~/projects/playground# curl http://localhost:8080/albums
    [
        {
            "id": "1",
            "title": "Blue Train",
            "artist": "John Coltrane",
            "price": 56.99
        },
        {
            "id": "2",
            "title": "Jeru",
            "artist": "Gerry Mulligan",
            "price": 17.99
        },
        {
            "id": "3",
            "title": "Sarah Vaughan and Clifford Brown",
            "artist": "Sarah Vaughan",
            "price": 39.99
        },
        {
            "id": "4",
            "title": "The Modern Sound of Betty Carter",
            "artist": "Betty Carter",
            "price": 49.99
        }
    ]
    ```

## 获取指定的记录

1. 获取指定记录的函数

    ```go
    // getAlbumByID locates the album whose ID value matches the id
    // parameter sent by the client, then returns that album as a response.
    func getAlbumByID(c *gin.Context) {
        id := c.Param("id")

        // Loop over the list of albums, looking for
        // an album whose ID value matches the parameter.
        for _, a := range albums {
            if a.ID == id {
                c.IndentedJSON(http.StatusOK, a)
                return
            }
        }
        c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
    }
    ```

    - 从`URL`中解析出id变量。

    - 从现有列表中检索对应id的记录。

2. 检测

    ```go
    root@c3f5bdb3bd8f:~/projects/playground# curl http://localhost:8080/albums/2
    {
        "id": "2",
        "title": "Jeru",
        "artist": "Gerry Mulligan",
        "price": 17.99
    }
    ```

******

关于Gin的更多信息，参考[Gin Web Framework package documentation](https://pkg.go.dev/github.com/gin-gonic/gin)或[Gin Web Framework docs](https://gin-gonic.com/docs/)。