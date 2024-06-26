# v8go를 사용한 JavaScript 사용

- `v8go`를 사용하여 golang 에서 JavaScript 코드를 실행하는 예제
- Json Data를 받아서 JavaScript 코드로 처리

## 준비 사항

- `v8go` 패키지 설치

### `v8go` 패키지 설치

```bash
go get -u rogchap.com/v8go
```

## 프로젝트 설정

### 1. 프로젝트 디렉토리 생성 및 초기화

```bash
mkdir v8go-example
cd v8go-example
go mod init v8go-example
```

### 2. `main.go` 파일 생성

`main.go` 생성하고, 다음 코드를 작성

```go
package main

import (
	"fmt"

	"rogchap.com/v8go"
)

type Response struct {
	Result string `json:"result"`
	Error  string `json:"error,omitempty"`
}

// JavaScript 코드를 JSON 데이터와 함께 실행하는 함수
func executeJavaScript(code string, jsonData string) (Response, error) {
	iso := v8go.NewIsolate() // V8 엔진초기화 (격리된 환경)
	defer iso.Dispose()

	ctx := v8go.NewContext(iso) // 실행 컨텍스트 생성
	defer ctx.Close()

	// JSON 데이터를 V8 컨텍스트에 로드
	if _, err := ctx.RunScript(fmt.Sprintf("var userInfo = %s;", jsonData), "userInfo.js"); err != nil {
		return Response{}, fmt.Errorf("잘못된 JSON 데이터: %v", err)
	}

	// 주어진 JavaScript 코드 실행
	value, err := ctx.RunScript(code, "code.js")
	if err != nil {
		return Response{Error: err.Error()}, nil
	}

	return Response{Result: value.String()}, nil
}

func main() {
	// 예제 JSON 데이터
	jsonData := `
	{
		"members": [
			{ "name": "Alice", "age": 30 },
			{ "name": "Bob", "age": 25 }
		]
	}`

	// 데이터를 처리할 예제 JavaScript 코드
	code := `
	var result = userInfo.members.map(function(member) {
		return member.name + " is " + member.age + " years old";
	}).join(", ");
	result;
	`

	// JSON 데이터와 JavaScript 코드를 사용하여 함수 실행
	response, err := executeJavaScript(code, jsonData)
	if err != nil {
		fmt.Printf("에러: %v\n", err)
		return
	}

	fmt.Printf("결과: %s\n", response.Result)
}

```

### 출력 예시

```
결과: Alice is 30 years old, Bob is 25 years old
```
