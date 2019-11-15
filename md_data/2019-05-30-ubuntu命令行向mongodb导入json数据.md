---
layout:     post
title:      ubuntu命令行向mongodb导入json数据
subtitle:   
date:       2019-05-30
author:     P
header-img: img/post-bg-map.jpg
catalog: true
tags:
    - python
---
** **

## **准备**

在学习本教程之前，请确保完成以下先决条件：

- Ubuntu 14.04 腾讯云[CVM](https://cloud.tencent.com/product/cvm?from=10680)。
- 一台已经设置好可以使用`sudo`命令的非root账号的Ubuntu**服务器**，并且已开启防火墙。没有服务器的同学可以在[这里购买](https://cloud.tencent.com/product/cvm?from=10680)，不过我个人更推荐您使用**免费**的腾讯云[开发者实验室](https://cloud.tencent.com/developer/labs?from=10680)进行试验，学会安装后再[购买服务器](https://cloud.tencent.com/product/cvm?from=10680)。
- 在Ubuntu 14.04上安装和配置MongoDB

除非另有说明，否则本教程中需要root权限的所有命令都应作为具有sudo权限的非root用户运行。

## **了解基础知识**

在继续本文之前，需要对此问题有一些基本的了解。如果您有使用[MySQL](https://cloud.tencent.com/product/cdb?from=10680)等流行的关系数据库系统的经验，那么在使用MongoDB时可能会发现一些相似之处。

您应该知道的第一件事是MongoDB使用[json](http://json.org/)和bson（二进制json）格式来存储其信息。Json是人类可读的格式，非常适合导出并最终导入数据。您可以使用任何支持json的工具进一步管理导出的数据，包括简单的文本编辑器。

一个示例json文档如下所示：

```
{"address":[
    {"building":"1007", "street":"Park Ave"},
    {"building":"1008", "street":"New Ave"},
]}
```

Json使用非常方便，但它不支持bson中可用的所有数据类型。这意味着如果使用json，将会出现所谓的保真度丢失。这就是备份/恢复的原因，最好使用能够更好地恢复MongoDB数据库的二进制bson。

其次，您不必担心显式创建MongoDB数据库。如果您指定用于导入的数据库尚不存在，则会自动创建该数据库。集合'（数据库表）结构的情况更好。与其他数据库引擎相比，在MongoDB中，再次在第一个文档（数据库行）插入时自动创建结构。

第三，在MongoDB中读取或插入大量数据（例如本文的任务）可能会占用大量资源并占用大量CPU，内存和磁盘空间。考虑到MongoDB经常用于大型数据库和大数据，这是至关重要的。解决此问题的最简单方法是在夜间运行导出/备份。

第四，如果您有一个繁忙的MongoDB服务器，其信息在数据库导出过程中发生变化，则信息一致性可能会有问题。这个问题没有简单的解决方案，但在本文的最后，您将看到有关进一步阅读有关复制的建议。

## **将信息导入MongoDB**

要了解如何将信息导入MongoDB，我们可以使用一个关于餐馆的流行示例MongoDB数据库。它是.json格式，可以使用`wget`以下方式下载：

```
wget https://raw.githubusercontent.com/mongodb/docs-assets/primer-dataset/primer-dataset.json
```

下载完成后，您应该在当前目录中有一个名为`primer-dataset.json`（12 MB大小）的文件。让我们将这个文件中的数据导入一个名为`newdb`的新数据库，并进入一个名为`restaurants`的集合。对于导入，我们将使用如下命令`mongoimport`：

```
sudo mongoimport --db newdb --collection restaurants --file primer-dataset.jsonmongoimport -d expedia -c hotel_info --file 文件名.json-d 指哪个database-c 哪个table/collection
```

结果应如下所示：

```
2016-01-17T14:27:04.806-0500    connected to: localhost
2016-01-17T14:27:07.315-0500    imported 25359 documents
```

如上面的命令所示，已导入25359个文档。因为我们没有调用`newdb`数据库，所以MongoDB会自动创建它。

让我们通过连接到这样调用的新创建的名为`newdb`的MongoDB数据库来验证导入：

```
sudo mongo newdb
```

您现在已连接到新创建的`newdb`数据库实例。请注意，您的提示已更改，表明您已连接到数据库。

使用以下命令计算restaurants集合中的文档：

```
db.restaurants.count()
```

结果应该显示`25359`，正好是导入文档的数量。为了更好的检查，您可以从餐馆集合中选择第一个文档，如下所示：

```
db.restaurants.findOne() 
```

结果应如下所示：

```
{
        "_id" : ObjectId("569beb098106480d3ed99926"),
        "address" : {
                "building" : "1007",
                "coord" : [
                        -73.856077,
                        40.848447
                ],
                "street" : "Morris Park Ave",
                "zipcode" : "10462"
        },
        "borough" : "Bronx",
        "cuisine" : "Bakery",
        "grades" : [
                {
                        "date" : ISODate("2014-03-03T00:00:00Z"),
                        "grade" : "A",
                        "score" : 2
                },
...
        ],
        "name" : "Morris Park Bake Shop",
        "restaurant_id" : "30075445"
}
```

这样详细的检查可以揭示文档的问题，例如它们的内容，编码等.json格式使用`UTF-8`编码，您的导出和导入应该在该编码中。如果您手动编辑json文件，请记住这一点。否则，MongoDB会自动为您处理。

要退出MongoDB提示，请在提示符处键入`exit`：

```
exit
```

您将以非root用户身份返回到正常的命令行提示符。

## **从MongoDB导出信息**

正如我们之前提到的，通过导出MongoDB信息，您可以获取包含数据的人类可读文本文件。默认情况下，信息以json格式导出，但您也可以导出到csv（逗号分隔值）。

要从MongoDB导出信息，请使用该命令`mongoexport`。它允许您导出非常精细的导出，以便您可以指定数据库，集合，字段，甚至可以使用查询进行导出。

一个简单的`mongoexport`例子是从我们之前导入的`newdb`数据库中导出餐馆集合。它可以这样做：

```
sudo mongoexport --db newdb -c restaurants --out newdbexport.json
```

在上面的命令中，我们用`--db`来指定数据库，`-c`集合以及`--out`文件里的数据将被保存。

成功的输出`mongoexport`应如下所示：

```
2016-01-20T03:39:00.143-0500    connected to: localhost
2016-01-20T03:39:03.145-0500    exported 25359 records
```

上面的输出显示已导入25359个文档 - 与导入的文档数相同。

在某些情况下，您可能只需要导出集合的一部分。考虑到餐馆json文件的结构和内容，让我们出口所有符合标准的餐厅，位于布朗克斯区，并有中国菜。如果我们想在连接到MongoDB时直接获取此信息，请再次连接到数据库：

```
sudo mongo newdb
```

然后，使用此查询：

```
db.restaurants.find( { borough: "Bronx", cuisine: "Chinese" } )
```

结果显示在终端上。要退出MongoDB提示，请`exit`在提示符处键入：

```
exit
```

如果要从sudo命令行而不是在连接到数据库时导出数据，请`mongoexport`通过为`-q`参数指定前面的查询部分，如下所示：

导出数据命令（导出的文件有两种格式：json/csv，此处导出的是json文件，对于导出CSV文件是需要额外指定一个变量 -field 对于的字段名称）：

mongoexport -h 数据库所在主机地址（若是本地则为127.0.0.1，若是远程则写为远程地址IP）-d 要导     出的数据库名称 -c 集合名称 -o 输出多的json文件路径

导出csv文件示例 ：mongoexport -h 主机地址 -d 数据库名称 -c 集合名称 --csv --field 字段列表 -o 输出地址

实际示例：mongoexport -h 127.0.0.1 -d test_new -c mycolle -o D:\Database\temp\mycolle.json

如果导出成功，结果应如下所示：

```
2016-01-20T04:16:28.381-0500    connected to: localhost
2016-01-20T04:16:28.461-0500    exported 323 records
```

以上显示已导出323条记录，您可以在我们指定的`Bronx_Chinese_retaurants.json`文件中找到它们。

## **结论**

本文向您介绍了从MongoDB数据库导入和导出信息的基本要素。

复制不仅对可伸缩性有用，而且对当前主题也很重要。复制允许您在从故障恢复主服务器时从MongoDB服务器中不间断地继续运行MongoDB服务。复制的一部分也是[操作日志（oplog）](https://docs.mongodb.org/manual/core/replica-set-oplog/)，它记录了修改数据的所有操作。就像在MySQL中使用二进制日志一样，您可以使用此日志在上次备份完成后恢复数据。回想一下，备份通常在夜间进行，如果您决定在晚上恢复备份，则会丢失自上次备份以来的所有更新。

更多Ubuntu教程请前往[腾讯云+社区](https://cloud.tencent.com/developer?from=10680)学习更多知识。

> 
参考文献：《How To Import and Export a MongoDB Database on Ubuntu 14.04》

