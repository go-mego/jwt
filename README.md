# JSON Web Token

JSON Web Token 套件能協助你簽發與讀取 JWT，簡單說就是透過一組密碼將資料簽名，接著以「明文方式」（因此避免存放機密資料）存放於客戶端，但因為客戶端不知道這組密碼所以也就無法變動其內容。而 JWT 能夠省去普通 Token 需存取資料庫的費時問題。

# 索引

* [安裝方式](#安裝方式)
* [使用方式](#使用方式)
    * [標準欄位](#標準欄位)
    * [解析 JWT](#解析 JWT)

# 安裝方式

打開終端機並且透過 `go get` 安裝此套件即可。

```bash
$ go get github.com/go-mego/jwt
```

# 使用方式

欲簽發一個新的 JWT 內容，首先透過 `NewSigner` 以指定的演算法與金鑰建立一個新的簽署者。接著透過 `Sign` 並夾帶官方資料欄位來簽署自訂的內容來產生一個 JWT。

```go
package main

import (
	"fmt"

	"github.com/go-mego/jwt"
)

type User struct {
	Username string
	Nickname string
}

func main() {
	// 透過 HS256 作為簽名的演算法並以此建立一個簽署者。
	// 其簽署金鑰是 `MySecret`。
	signer := jwt.NewSigner("MySecret", jwt.SigningMethodHS256)
	// 以簽署者簽署下列的官方標準與自訂資料。
	token, err := signer.Sign(jwt.Information{
		IssueAt:        1475874457,
		ExpirationTime: 1524087454,
	}, User{
		Username: "admin",
		Nickname: "Yami Odymel",
	})
	// 結果：「已簽發 JWT：eyJ0eXAiOiJKV1QiLCJhbGci...」
	fmt.Println("已簽發 JWT：" + token)
}
```

## 標準欄位

JWT 有下列標準欄位可供簽發時填寫。

```go
jwt.Information{
    // 簽發者名稱，通常是簽署者的網域名稱或 IP 位置。（*）
    Issuer: "www.example.com",
    // 主旨名稱（如：使用者名稱、用途）。（*）
    Subject: "admin",
    // 簽發對象名稱（如：客戶端的 IP 位置、電子信箱地址或是開發者的網域名稱）。（*）
    Audience: "admin@gmail.com",
    // 到期時間，此時間之後的 JWT 無效。（*）
    ExpirationTime: 1524087454,
    // 啟用時間，JWT 需要在這之後才能使用，這能夠讓你事先配發 JWT 而避免先被使用。
    NotBefore: 1495874457,
    // 簽發時間。
    IssueAt: 1475874457,
    // 可識別此 JWT 的獨立編號。
    JWTID: "e4654066-5df8-418c-ba2c-21a2bf027083",
}
```

`（*）` 為必要欄位。

## 解析 JWT

使用 `NewParser` 建立一個會以指定金鑰解析特定 JWT 的解析器。透過 `Parse` 來解析特定 JWT 並將完成的資料映射到本地的變數上。當 JWT 與當初簽署的金鑰不正確時會回傳錯誤表示不相符，以防客戶端擅自竄改內容。

```go
package main

import (
	"fmt"

	"github.com/go-mego/jwt"
)

type User struct {
	Username string
	Nickname string
}

func main() {
	var ctx User
	// 以指定金鑰建立 JWT 解析器。
	parser := jwt.NewParser("MySecret")
	// 傳入一個欲解析的 JWT 和欲映射的變數對象。
	err := jwtParser.Parse("eyJ0eXAiOiJKV1QiLCJhbGci...", &ctx)
	if err != nil {
		fmt.Printf("解析 JWT 時發生錯誤：%v", err)
	}
	// 結果：「JWT 裡的 Username 是：admin」
	fmt.Printf("JWT 裡的 Username 是：%v", ctx.Username)
}
```