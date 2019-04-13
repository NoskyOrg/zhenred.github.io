---
title: GoLang Test
categories:                
- 代码
- coding
- go
date: 2019-04-07
---

## 此文档只是一个无聊的展示

```golang
package main

import (
	"database/sql"
	"flag"
	"fmt"
	"time"

	_ "github.com/go-sql-driver/mysql"
)

func main() {
	user := flag.String("u", "root", "user")
	password := flag.String("p", "abc123", "password")
	port := flag.String("P", "3306", "listen port")
	host := flag.String("h", "127.0.0.1", "host")
	database := flag.String("db", "", "database")

	flag.Parse()

	dbParameter := *user + ":" + *password + "@tcp(" + *host + ":" + *port + ")/" + *database
	// fmt.Println(dbParameter)

	// db, err := sql.Open("mysql", "root:abc123@tcp(172.16.130.135:3301)/ob")
	db, err := sql.Open("mysql", dbParameter)
	if err != nil {
		panic("database parameters is wrong, can not connect database")
	}
	defer db.Close()

	err = db.Ping()
	if err != nil {
		panic("can not ping database. " +
			"Check if the database is started or database parameters is wrong")
	} else {
		fmt.Println("Successful connection to the database")
	}

	stmtIns, err := db.Prepare("insert into user values (?, ?, ?)")
	if err != nil {
		panic(err.Error())
	}
	defer stmtIns.Close()

	go func() {
		for i := 1; i < 200000; i++ {
			//println(i)
			_, err = stmtIns.Exec(i, (i * i), (i * i * i))
			if err != nil {
				panic(err.Error())
			}
		}
	}()
	time.Sleep(time.Second * 30)
	println("Insert into values successful")

	rows, err := db.Query("select * from user")
	if err != nil {
		panic(err.Error())
	}
	defer rows.Close()

	for rows.Next() {
		var uid int
		var name, password string

		rows.Scan(&uid, &name, &password)
		fmt.Println("uid:", uid, "name:", name, "password", password)
	}
}
```
