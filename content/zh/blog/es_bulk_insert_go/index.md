---
title: "Elasticsearch 批量写入"
date: 2022-04-16T17:51:01+08:00
draft: false
# page title background image
bg_image: "images/backgrounds/page-title.jpg"
# about image
image: "images/blog/elasticsearch.png"
# meta description
description: "Elasticsearch 批量写入Bulk使用。"
# categories
categories: ["es"]
# tags
tags: ["es"]
# type
type: "post"

aliases: ""
---

本篇文章主要介绍如何使用 es 第三方库[olivere/elastic](https://github.com/olivere/elastic)来批量同步和异步写入数据。

## 1.新建 es 客户端链接

```go
package elastic

import (
    "context"
    "net/http"

    es "github.com/olivere/elastic"
    log "github.com/sirupsen/logrus"
)

const maxConn = 100

func NewESClient(endpoints ...string) (*es.Client, error) {
    tr := &http.Transport{
        MaxIdleConnsPerHost: maxConn,
        MaxIdleConns:        maxConn,
    }

    esClient, err := es.NewClient(
        es.SetURL(endpoints...),
        es.SetSniff(false),
        es.SetHttpClient(&http.Client{Transport: tr}),
        es.SetErrorlog(log.New()),
    )
    if err != nil {
        return nil, err
    }

    for _, endpoint := range endpoints {
        if _, _, err = esClient.Ping(endpoint).Do(context.Background()); err != nil {
            return nil, err
        }
    }

    return esClient, nil
}
```

## 2.同步写

```go
package main

import (
    "fmt"
    "strconv"

    es "github.com/olivere/elastic"
    log "github.com/sirupsen/logrus"

    elastic "goEs/elastic"
)

type Tweet struct {
    User    string `json:"user"`
    Message string `json:"message"`
}

func addAllBulkIndexReq(bulk *es.BulkService, msg *Tweet) {
    index := "twitter"
    /*
        set with assigned id

        elastic.NewBulkIndexRequest().Index(index).Type("tweet").Id(strconv.Itoa(n)).Doc(msg)
    */
    doc := es.NewBulkIndexRequest().Index(index).Type("tweet").Doc(msg)

    bulk.Add(doc)
}

func initBulkReq(client *es.Client) *es.BulkService {
    bulkRequest := client.Bulk()
    // set retry func
    bulkRequest.Retrier(es.NewBackoffRetrier(es.NewExponentiaBackoff(500*time.Millisecond, 5*time.Second)))
    return bulkRequest
}

func main() {
    client, err := elastic.NewClient()
    if err != nil {
        log.Errrof("New es client failed with err(%v)", err)
        return
    }

    bulkRequest := initBulkReq(client)

    twtMsgs := make([]Tweet, 1000)
    for i := 0; i < 1000; i++ {
        twtMsg := Tweet{User: fmt.Sprintf("user-%d", n), Message:  fmt.Sprintf("user-%d insert one message ", n)}
        addAllBulkIndexReq(bulkRequest, msg)
    }

    bulkSize := bulkRequest.NumberOfActions()
    outputResp := &BulkInsertCounterResp{
        BulkInsertResp: &BulkInsertResp{Succeeded: bulkSize}
    }

    if bulkSize != 0 {
        resp, err := bulkRequest.Do(context.Background())
        if err != nil {
            return nil, err
        }

        if resp != nil {
            log.Debuf("bulk insert msg into es succeeded=%d", len(resp.Succeeded()))
            if resp.Errors && len(resp.Failed()) > 0 {
                log.Errorf("bulk insert msg into es has failed, failed=%d", len(resp.Failed()))
            }
        }
    }
}
```

## 3.异步写入

```go
func addBulkProcessorReq(bulkProssor *es.BulkProcessor, msg *Tweet) {
    index := "twitter"
    doc := es.NewBulkIndexRequest().Index(index).Type("tweet").Doc(msg)
    bulkProssor.Add(doc)
}

func putMsgWithBulkProssor(client *es.Client, msgs []*Tweet) error {
    bulkProcessor, err := client.BulkProcessor().
        Workers(5).                        // 携程数
        BulkActions(1000).                 // 每个携程队列容量
        FlushInterval(time.Second*2).      // 刷新间隔
        Stats(true).                       // 是否获取统计
        After(getFailed).                  // 回调函数
        Do(context.Backgroud())
    if err = bulkProcessor.Start(context.Backgroud()); err != nil {
        return err
    }

    defer bulkProcessor

    for _, msg := range msgs {
        addBulkProcessorReq(bulkProcessor, msg)
    }

    return nil
}

func getFailed(executionID int64, req []es.BulkableRequest, resp *es.BulkResponse, err error) {
    if resp == nil {
        log.Error("get nil bulk processor response")
        return
    }

    log.Debugf("put msg into es with async (execID=%d): succeeded=%d, failed=%d, indexed=%d, took=%dms",
        executionID, len(resp.Succeeded()), len(resp.Failed()), len(resp.Indexed()), resp.Took)

    failedCnt := resp.GetFailed()
    if len(failedCnt) > 0 {
        for _, fc := range failedCnt {
            log.Errorf("failed case: index:%s, type=%s, id=%s, status=%d, result=%s, err=%v",
                fc.Index, f.Type, f.Id, f.Status, f.Result, f.Error)
        }
    }
}
```

## 4.参考

-   https://github.com/olivere/elastic
