# gedis
golang redis client  
##demo  
```
package main

import (
	. "client"
	"net"
	"fmt"
	"time"
)

func main() {
	testCluster()
	time.Sleep(time.Duration(100)*time.Second)
}
//集群版
func testCluster(){
	var node7000 = Node{"127.0.0.1:7000", "123456", 10}
	var node7001 = Node{"127.0.0.1:7001", "123456", 10}
	var node7002 = Node{"127.0.0.1:7002", "123456", 10}
	var node7003 = Node{"127.0.0.1:7003", "123456", 10}
	var node7004 = Node{"127.0.0.1:7004", "123456", 10}
	var node7005 = Node{"127.0.0.1:7005", "123456", 10}

	nodes := []*Node{&node7000, &node7001, &node7002, &node7003, &node7004, &node7005}
	var clusterConfig = ClusterConfig{nodes,10}
	cluster := NewCluster(clusterConfig)
	value,err:=cluster.Get("name")
	fmt.Println(value, err)
}

func getConn() *net.TCPConn {
	var config = ConnConfig{"127.0.0.1:6379", "root"}
	pool := NewSingleConnPool(1, config)
	conn, _ := GetConn(pool)
	return conn
}  

//基于客户端分片集群
func testSharding(){
	var s1 = Shard{"192.168.96.232:6379", "vs959yUyx3", 50,5,200}
	var s2 = Shard{"192.168.96.4:6379", "K5re9U#mX@", 50,5,200}
	shards := []*Shard{&s1, &s2}
	var shardConfig = ShardConfig{shards, 10}
	sharding := NewSharding(shardConfig)
	rand.Seed(time.Now().UnixNano())
	for i:=0;i<20;i++{
		go func() {
			for {
				value, err := sharding.Get("teams")
				log.Printf("请求结果：%s, err: %s",value, err)
				fmt.Println("查询结果",value)
				time.Sleep(time.Duration(rand.Intn(3))*time.Second)
			}
		}()
	}
	time.Sleep(1000*10)
}

//单机版
func getClient() *Client {
	var config = ConnConfig{"127.0.0.1:7002", "123456"}
	pool := NewSingleConnPool(1, config)

	return BuildClient(pool)
}
```  

1.0 特性：  
基于原生golang开发  
连接池管理  
keepalive  
redisTemplate提供多种命令支持  
2.0 特性   
cluster支持  
loadbalance支持  
heartBeat支持  
连接池的监控及动态扩容  
3.0 特性  
支持客户端分片集群方式  
一致性哈希  
（不支持多key操作）
