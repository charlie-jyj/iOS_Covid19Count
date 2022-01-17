## Goodbye corona19
> 코로나 현황판 앱 만들기

### 1) 기능
- 시도별 신규 확진자 수가 파이 차트로 표시 
- 도시 항목을 선택하면 상세 현황을 볼 수 있는 화면으로 이동

### 2) 기본 개념

#### 1) Alamofire
> Swift 기반 HTTP 네트워킹 라이브러리, URLSession 기반, 네트워크 작업 단순화, JSON parsing
- request response method 제공
- url json parameter encoding
- file data stream
- multipart data upload
- HTTP response 검증
- 단위 테스트, 통합 테스트 보장

##### URLSession vs Alamofire
```swift

import Foundation

// URL
var components = URLComponents(string: "https://api.mywebserver.com/v1/board")!
components.queryItems = ["title": "Gunter"].map { (key, value) in URLQueryItem(name: key, value: value)}

// Request
var request = try! URLRequest(url: components.url!)
request.httpMethod = "GET"
URLSession.shared.dataTask(with: components.url!) {
    (data, response, error) in
    do {
        guard let data = data,
              let response = response as? HTTPURLResponse, (200..<300) ~= response.statusCode,
              error == nil else {
            throw error ?? URLError(.badURL)
        }
        let res = try JSONDecoder().decode(Response.self, from: data)
        print("Success: \(response)")
    } catch {
        print("failure: \(error.localizedDescription)")
    }
}


```

```swift
AF.request("https://", method: .get, parameters: ["title":"yujin"])
    .validate()
    .responseJSON {
        response in
        switch response.result {
        case .success(let result):
            debugPrint("success \(result)")
        case .failure(let error):
            debugPrint("failure \(error)")
        }
    }

```

#### 2) Cocoapods
> 외부 라이브러리 사용
