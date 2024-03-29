+++
title = "redis发布订阅"
author = ["zakudriver"]
description = "golang实现redis发布订阅"
date = 2020-07-02
lastmod = 2022-07-22T10:16:26+08:00
tags = ["golang"]
categories = ["code"]
draft = false
+++

## redis发布订阅 {#redis发布订阅}

<div class="EXPLAIN">

相比rabbitmq等专业消息队列的缺陷: 没有相应的机制保证消息的可靠消费，如果发布者发布一条消息，而没有对应的订阅者的话，这条消息将丢失，不会存在内存中。

</div>

```go
package main

import (
  "context"
  "fmt"
  "log"
  "strconv"
  "time"

  "github.com/gomodule/redigo/redis"
)

// ConsumeFunc consumes message at the channel.
type ConsumeFunc func(channel string, message []byte) error

// RedisClient represents a redis client with connection pool.
type RedisClient struct {
  pool *redis.Pool
}

// NewRedisClient returns a RedisClient.
func NewRedisClient(addr string, passwd string) *RedisClient {
  pool := &redis.Pool{
    MaxIdle:     10,
    IdleTimeout: 300 * time.Second,
    Dial: func() (redis.Conn, error) {
      c, err := redis.Dial("tcp", addr, redis.DialPassword(passwd), redis.DialDatabase(0))
      if err != nil {
        return nil, err
      }
      return c, nil
    },
    TestOnBorrow: func(c redis.Conn, t time.Time) error {
      if time.Since(t) < time.Minute {
        return nil
      }
      _, err := c.Do("PING")
      return err
    },
  }

  log.Printf("new redis pool at %s", addr)
  client := &RedisClient{
    pool: pool,
  }
  return client
}

// Close closes connection pool.
func (r *RedisClient) Close() error {
  err := r.pool.Close()
  return err
}

// Publish publishes message to channel.
func (r *RedisClient) Publish(channel, message string) (int, error) {
  c := r.pool.Get()
  defer c.Close()
  n, err := redis.Int(c.Do("PUBLISH", channel, message))
  if err != nil {
    return 0, fmt.Errorf("redis publish %s %s, err: %v", channel, message, err)
  }
  return n, nil
}

// Subscribe subscribes messages at the channels.
func (r *RedisClient) Subscribe(ctx context.Context, consume ConsumeFunc, channel ...string) error {
  psc := redis.PubSubConn{Conn: r.pool.Get()}

  log.Printf("redis pubsub subscribe channel: %v", channel)
  if err := psc.Subscribe(redis.Args{}.AddFlat(channel)...); err != nil {
    return err
  }
  done := make(chan error, 1)
  // start a new goroutine to receive message
  go func() {
    defer psc.Close()
    for {
      switch msg := psc.Receive().(type) {
      case error:
        done <- fmt.Errorf("redis pubsub receive err: %v", msg)
        return
      case redis.Message:
        if err := consume(msg.Channel, msg.Data); err != nil {
          done <- err
          return
        }
      case redis.Subscription:
        if msg.Count == 0 {
          // all channels are unsubscribed
          done <- nil
          return
        }
      }
    }
  }()

  ch <- 0

  // health check
  tick := time.NewTicker(time.Minute)
  defer tick.Stop()
  for {
    select {
    case <-ctx.Done():
      if err := psc.Unsubscribe(); err != nil {
        return fmt.Errorf("redis pubsub unsubscribe err: %v", err)
      }
      return nil
    case err := <-done:
      return err
    case <-tick.C:
      if err := psc.Ping(""); err != nil {
        return err
      }
      log.Println("over")
      return nil
    }
  }

}

func myConsumer(channel string, message []byte) error {
  log.Printf("receive message[%s] at the channel[%s]\n", string(message), channel)
  return nil
}

// ch 用于保证发布线程在订阅线程启动成功后才开始发布消息
var ch = make(chan int)

func main() {
  redisClient := NewRedisClient("127.0.0.1:6300", "zyhua1122")
  defer redisClient.Close()

  go func() {
    var subscriber int
    <-ch
    for i := 0; i < 3; i++ {
      subscriber, _ = redisClient.Publish("testx", "hello world"+strconv.Itoa(i))
      log.Printf("there is %d subscriber.\n", subscriber)
    }

  }()

  ctx, cancel := context.WithCancel(context.Background())
  err := redisClient.Subscribe(ctx,
    func(channel string, message []byte) error {
      log.Printf("receive message[%s] at the channel[%s]\n", string(message), channel)
      if string(message) == "goodbye" {
        cancel()
      }
      return nil
    },
    "testx")

  if err != nil {
    fmt.Printf("get error: %v\n", err)
  }

  return
}
```
