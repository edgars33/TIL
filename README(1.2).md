## 컬렉션뷰 클론하면서 정리한것.
### 1. 컬렉션뷰셀을 3등분하고 여백사이를 균등하게 나누는방법
```
class ViewController: UIViewController {
    
    
 private enum  Metrics {
      static let inset: CGFloat = 4
    }
    
    private let collectionView: UICollectionView = {
        let layout = UICollectionViewFlowLayout()
        let cv = UICollectionView(frame: .zero, collectionViewLayout: layout)
        cv.backgroundColor = .white
  ❗️    layout.minimumLineSpacing = Metrics.inset
  ❗️    layout.minimumInteritemSpacing = Metrics.inset
        return cv
    }()
```
❗️ 표시한 코드가 참고사진에서 저사이를 균등하게 나눠주는코드


### 2. 컬렉션뷰셀을 만들때 뭔가 오류가난다면 넣어주는코드
```
self.addSubview(self.imageView)
self.imageView.translatesAutoresizingMaskIntoConstraints = false
```

### 3. URL을 보내는방식, 그리고 여기에다가 API Request를 쏘게하는방법
```
final class CatService {
    
    func getCats(page: Int, limit: Int) {
        var components = URLComponents(string: "https://api.thecatapi.com/v1/images/search")!
        components.queryItems = [
        URLQueryItem(name: "page", value: "\(page)"),
        URLQueryItem(name: "limit", value: "\(limit)")
        ]
        
        var request = URLRequest(url: components.url!)
        request.httpMethod = "GET"
```
여기서 요청을 보내는 api콜을 할건데 어떻게하냐면 요런코드를 쓰게된다   
```
let task = URLSession.shared.dataTask(with: request) {
```
그래서 리퀘스트 결과를 데이터 response형태로 받아올수있음

### 4. 이미지서비스는 어디서나 쓸수있게 싱글톤으로 작성

### 5. 성공할때 UIImage를 실패할때 Error를 갖는 함수
```
func downloadImage(url: String, completion: @escaping(Result< UIImage, Error>) -> Void ) {
        
    }
```

### 6. func downloadImage를 활용해서 UIImageView에다 이미지를 세팅하는함수 만들기
```
func setImage(view: UIImageView, urlString: String){
        self.downloadImage(urlString: urlString) { result in
            
            DispatchQueue.main.async {
                switch result {
                case .failure(let error):
                    return
                case .success(let image):
                    view.image = image
 }}}}
```

### 7. ImageService를 셀(catcell)에서 가져다쓸수있게 세팅해주기
```
private let service = ImageService.shared 세팅해주고

------------
func setupData(urlString: String) {
        
        service.setImage(view: self.imageView, urlString: urlString)
    }
```

### 8. 스크롤하면 무한으로 새로운데이터를 가져오는 코드짜기
이걸짜기위한 method가 있는데 맨마지막 셀이 보일때쯤 더보기API 요청을 보내가지고 새로운 데이터를 덧붙이는 방법으로 구현가능
```
var isLoading: Bool = false
    
    func loadMoreIfNeeded(index: Int) { 
        if index > data.count - 6 { // 셀이 6줄이되면 밑에 load()함수를 호출해 api데이터를 가져오게됨.
            self.load()
        }
        
    }
func load() { //그래서 여기 load함수에서 데이터를 계속 덧붙여주니깐 정상적으로 실행가능
							//근데 이 로드함수를 여러번 돌리면 api를 여러번 부르니깐 isLoading변수를 넣음
        guard !isLoading else { return } //그래서 isLoading이면 아무것도 하지않게하고
        self.isLoading = true //
        self.service.getCats(page: self.currentPage, limit: self.limit) { result in
               
            DispatchQueue.main.async { //여기부터 큐가바껴서 메인큐로 옮겨주고
                
                switch result {
                case .failure(let error):
                    break
                case .success(let response):
                    self.data.append(contentsOf: response)
                    self.currentPage += 1
                    self.delegates.forEach { $0.loadComplete() }
                }
                self.isLoading = false //그후 여기서 isloading상태를 풀고
		            }               //데이터에 append를 할거임 그리고 이제 계속 페이지로 바뀌어야 되니까 
                                //페이지도 하나씩 추가되는걸 구현함 (currentPage +=1)
                
        }
    }
```
### 9. 그런데 8번을 실행하면 계속해서 사진이 실시간으로 바뀜. 
이유는 전데이터들이 덮어씌워지면서 이러한 현상이 발생됨 , 왜그러냐면 view를 내부적으로 재활용하고있어서.. 재활용할때 이 데이터가 계속 다운받아지는 도중에 전데이터들이 덮어씌워지고있어서

이런걸 해결하기위해서는 task에 cancel()를 해줘야함

### 10. 근데 또 9번을 실행하면 속도가 느림
그이유는 캐싱을 안하고 실시간으로 데이터를 받아와서그런거임 

이럴때는 보통 NSCache를 사용해줌 작동하는방식은 NSCache는 담아두었다가 만약 메모리가 부족해지면 내부적으로 날려버리는 처리를 겸하고있음 그러면 여기다가 성공했으면 캐시에다가 오브젝트로
이미지를 urlstring을 키로해가지고 가져오게할거임 그리고 만약에 캐시가 오브젝트를 이미가지고 있다면, urlstring을 키로하는 그러면 그것을 그냥 세팅을 해주고 (반환하는 task는 optional로 두고) ㅇ이제 그렇지 않다면 이미지를 받는식으로 작업해야 될거다 그럼 이제 순환차조를 회피하기 위해서 weak self를 넣어야하고 이제 빌드하면됨 
### 11. layout에 스크롤방향도 horizontal로 만들기
layout.scrollDirection = .horizontal
















