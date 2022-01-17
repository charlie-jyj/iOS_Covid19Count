## Goodbye corona19
> 코로나 현황판 앱 만들기

### 1) 기능
- 시도별 신규 확진자 수가 파이 차트로 표시 
- 도시 항목을 선택하면 상세 현황을 볼 수 있는 화면으로 이동

### 2) 기본 개념

#### (1) Alamofire
> Swift 기반 HTTP 네트워킹 라이브러리, URLSession 기반, 네트워크 작업 단순화, JSON parsing
- request response method 제공
- url json parameter encoding
- file data stream
- multipart data upload
- HTTP response 검증
- 단위 테스트, 통합 테스트 보장

- all examples require *import Alamofire* somewhere in the source file

- making request

```swift
open func request<Parameters: Encodable>(_ convertible: URLConvertible,
                                         method: HTTPMethod = .get,
                                         parameters: Parameters? = nil,
                                         encoder: ParameterEncoder = URLEncodedFormParameterEncoder.default,
                                         headers: HTTPHeaders? = nil,
                                         interceptor: RequestInterceptor? = nil) -> DataRequest
```

- http method

```swift
public struct HTTPMethod: RawRepresentable, Equatable, Hashable {
    public static let connect = HTTPMethod(rawValue: "CONNECT")
    public static let delete = HTTPMethod(rawValue: "DELETE")
    public static let get = HTTPMethod(rawValue: "GET")
    public static let head = HTTPMethod(rawValue: "HEAD")
    public static let options = HTTPMethod(rawValue: "OPTIONS")
    public static let patch = HTTPMethod(rawValue: "PATCH")
    public static let post = HTTPMethod(rawValue: "POST")
    public static let put = HTTPMethod(rawValue: "PUT")
    public static let trace = HTTPMethod(rawValue: "TRACE")

    public let rawValue: String

    public init(rawValue: String) {
        self.rawValue = rawValue
    }
}


```
=> these values can be passed as the method argument to the AF.request API

```swift
AF.request("https://httpbin.org/get")
AF.request("https://httpbin.org/post", method: .post)
AF.request("https://httpbin.org/put", method: .put)
AF.request("https://httpbin.org/delete", method: .delete)

```

https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#making-requests

- response handling

``` swift
// Response Handler - Unserialized Response
func response(queue: DispatchQueue = .main, 
              completionHandler: @escaping (AFDataResponse<Data?>) -> Void) -> Self

// Response Serializer Handler - Serialize using the passed Serializer
func response<Serializer: DataResponseSerializerProtocol>(queue: DispatchQueue = .main,
                                                          responseSerializer: Serializer,
                                                          completionHandler: @escaping (AFDataResponse<Serializer.SerializedObject>) -> Void) -> Self

// Response Data Handler - Serialized into Data
func responseData(queue: DispatchQueue = .main,
                  dataPreprocessor: DataPreprocessor = DataResponseSerializer.defaultDataPreprocessor,
                  emptyResponseCodes: Set<Int> = DataResponseSerializer.defaultEmptyResponseCodes,
                  emptyRequestMethods: Set<HTTPMethod> = DataResponseSerializer.defaultEmptyRequestMethods,
                  completionHandler: @escaping (AFDataResponse<Data>) -> Void) -> Self

// Response String Handler - Serialized into String
func responseString(queue: DispatchQueue = .main,
                    dataPreprocessor: DataPreprocessor = StringResponseSerializer.defaultDataPreprocessor,
                    encoding: String.Encoding? = nil,
                    emptyResponseCodes: Set<Int> = StringResponseSerializer.defaultEmptyResponseCodes,
                    emptyRequestMethods: Set<HTTPMethod> = StringResponseSerializer.defaultEmptyRequestMethods,
                    completionHandler: @escaping (AFDataResponse<String>) -> Void) -> Self

// Response Decodable Handler - Serialized into Decodable Type
func responseDecodable<T: Decodable>(of type: T.Type = T.self,
                                     queue: DispatchQueue = .main,
                                     dataPreprocessor: DataPreprocessor = DecodableResponseSerializer<T>.defaultDataPreprocessor,
                                     decoder: DataDecoder = JSONDecoder(),
                                     emptyResponseCodes: Set<Int> = DecodableResponseSerializer<T>.defaultEmptyResponseCodes,
                                     emptyRequestMethods: Set<HTTPMethod> = DecodableResponseSerializer<T>.defaultEmptyRequestMethods,
                                     completionHandler: @escaping (AFDataResponse<T>) -> Void) -> Self


```

    - the response handler does NOT evaluate any of the response data. It merely forwards on all information directly from the URLSessionDelegate. It is the Alamofire equivalent of using cURL to execute a request
    - to leverage the other response serializers taking advantage of Response and Result types
    - the *responseData* handler uses a *DataResponseSerializer* to extract and validate the *Data* returned by the server. If no errors occur and Data is returned, the response Result will be *.success* and the value will be the Data returned from the server

```swift
AF.request("https://httpbin.org/get").responseData { response in
    debugPrint("Response: \(response)")
}


```


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

#### (2) Cocoapods
> iOS 개발 외부 라이브러리 관리를 위한 의존성 도구 

https://cocoapods.org/

- init pod 이후에 생성된 pod file 에 사용할 라이브러리를 작성한다
```
use_frameworks!
    pod ‘라이브러리이름’, ‘버전’
    pod 'Alamofire', '~> 5.5'
```

#### (3) charts library
https://github.com/danielgindi/Charts

*doc*
https://weeklycoding.com/mpandroidchart-documentation/

#### (4) escaping closure
> 클로저가 함수를 탈출한다.

- 클로저가 전달인자로 전달되지만 
- 함수가 return된 후에 클로저의 내용이 실행 (인자가 함수 영역을 탈출하여 함수 바깥에서 사용)
- 기존 변수의 scope 개념 무시 (함수 블록 안의 local 영역을 뛰어 넘어 사용되기 때문)
- 주로 비동기 작업할 경우 completion handler 에서 사용
- 보통 네트워크는 비동기 작업이기 때문에 responseData 함수의 completionHandler는 자신을 호출한 함수(fetchCovidOverview)가 반환된 이후에 실행된다. 
    - 서버에서 언제 데이터를 응답하여 줄지 알 수 없기 때문

### 3) 새롭게 배운 것

- cocoa pod ruby 명령어로 install 되지 않아서 당황했는데 그냥 home-brew 로 설치했다.

```console

$ brew install cocoapods
$ pod init 
$ pod install
```

https://stackoverflow.com/questions/63650689/package-configuration-for-libffi-is-not-found-in-macos-while-installing-travis-c

- 라이브러리 설치 후 xcworkspace 확장자 파일이 생겼다. 앞으로의 작업은 여기에서 하게 된다

- static table view
‘설정’ 과 같은 정적인 테이블 뷰를 구현할 때 
UI table view 에서 사용 가능 하다. 
Table view content : dynamic prototypes => static cells 
    - static이기 때문에 value를 DataSource 통해서가 아닌 outlet 변수로 detail 에 표시

- Table View Controller 로 파일 생성하기
Datasource 와 delegate 프로토콜을 처음부터 채택하고 있다.
필수 구현 함수들이 override 되어 있고 delegate 함수들도 주석 처리 되어있다.

-Result<Class, Protocol>

-Alarmofire의 responseData 메서드의 completionHandler는 main thread 에서 동작한다.

- indicatorView
    - startAnimating() : fetch 전에 call
    - stopAnimating(): completionHandler 안에서 (서버 response를 받은 후)

