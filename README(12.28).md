# TIL
<h5>비동기 메서드와 콜백함수의 설계



**1-1.리턴형으로 처리하면 안된다. 리턴하고싶은 타입을  콜백함수를 전달하면서 뮤직어레이를 전달**

---
```

func getMethod(completion: @escaping ([Music]?) -> Void) {
    guard let url = URL(string: "https://itunes.apple.com
		/search?media=music&term=jazz") else {
        print("Error: cannot create URL")
        completion(nil)
        return
    }
    
    var request = URLRequest(url: url)
    request.httpMethod = "GET"

    
    
    URLSession.shared.dataTask(with: request) { data, response, error in
        guard error == nil else {
            print("Error: error calling GET")
            print(error!)
            completion(nil)
            return
        }
        guard let safeData = data else {
            print("Error: Did not receive data")
            completion(nil)
            return
        }
        guard let response = response as? HTTPURLResponse, 
				(200 ..< 299) ~= response.statusCode else {
            print("Error: HTTP request failed")
            completion(nil)
            return
        }
        do {
            let decoder = JSONDecoder()
            let musicData = try decoder.decode(MusicData.self, from: safeData)
            completion(musicData.results)
            return
        } catch {
            
        }
    }.resume()     // 시작


}



getMethod { musicArray in
    guard let array = musicArray else { return }
    dump(array)
}
```

1. 배열이 생기는 시점에 다른함수를 호출해가지고 다른함수를 실행시키면서 결과값을 전달을 해야한다.
2. completion(musicData.results) 여기서 배열이 생겼을때 콜백함수인  ([Music]?) -> Void)를 실행시킴 그래서 일반적으로 콜백함수에 전달하고싶은 데이터형태를 넣으면되고 리턴타입은 거의 대부분 Void로 설정하면됨

3. completion(nil)을 에러가 나는상황에 넣어주면되고 정상적으로 실행되는곳에는 completion(musicData.results)  를 써주면됨.

결국 말하자면 비동기처리일때 이런식(콜백함수)으로 함수를 설계해야 데이터를 받아 비동기처리가 끝난시점에 데이터를 받을수있음

-----
1-2. 서버에서 주는 데이터가 없을수도있기 때문에 옵셔널로 처리
```

struct Music: Codable {
    let songName: String?
    let artistName: String?
    let albumName: String?
    let previewUrl: String?
    let imageUrl: String?
    private let releaseDate: String?
```

##
<h5>앱UI 구현과 MVC패턴 설계
---
열거형과 구조체의 차이

```
public enum MusicApi { //열거형
	//var name = "hi" -> 안됨
    static let requestUrl = "https://itunes.apple.com/search?"
    static let mediaParam = "media=music"
}


// 사용하게될 Cell 문자열 묶음
public struct Cell {
    static let musicCellIdentifier = "MusicCell"
    static let musicCollectionViewCellIdentifier = "MusicCollectionViewCell"
    private init() {}
}

```
1. 열거형에서는  저장속성을 가질수없지만 타입저장속성은 만들수있음 이건 데이터 영역에 생기기때문에.
2. 케이스(case)자체가 데이터역할을 하기때문에 이자체가 메모리공간을 만들고 데이터를 저장하기때문에 저장속성을 만들수가없음

1. 구조체에서는 다좋지만 마지막에     private init() {}  를 선언해줘야함.

---

속성감시자 구현

```
var imageUrl: String? {
        didSet {
            loadImage()
        }
    }
```
didset : 저장속성에 값이 변하거나 할당받으면 didset 함수가 실행됨

---
네트워킹매니저에 통신할수있는 코드를 전부 때려박는이유 , 그리고 싱글톤으로 만드는이유
```
static let shared = NetworkingManager()

private init() {}
```
그 이유는 자기자신을 생성해서 타입저장속성에 할당하는것 그래서 이 변수(shared)는 데이터 영역에 존재하고 싱글톤인 NetworkingManager()는 힙 영역에 존재하게됨

private init은 싱글톤으로 만들때 다른곳에서 네트워킹매니저를 또 생성하지말라고 작성해주는코드

---
에러타입도 열거형으로 처리
```
enum NetworkError: Error {
    case networkingError
    case dataError
    case parseError
}
```

---
⭐️테이블뷰 리로드 코드는 꼭넣어야함 

```
func setupDatas() {
        // 네트워킹의 시작
        networkManager.fetchMusic(searchTerm: "jazz") { result in
            print(#function)
            switch result {
            case .success(let musicDatas):
                // 데이터(배열)을 받아오고 난 후
                self.musicArrays = musicDatas
                // 테이블뷰 리로드
		       ⭐️⭐️⭐️   DispatchQueue.main.async {
                    self.musicTableView.reloadData()
                }
            case .failure(let error):
                print(error.localizedDescription)
            }
        }
    }

```
배열에 아이템이 들어있더라도 테이블뷰를 그리지않는다면 아무것도 표시가 되지않음.

그래서 이 코드를 넣어줘야 화면을 다시그려서 띄워줄수있음

---

컬렉션뷰의 레이아웃을 담당하는 객체 ( 간편하게 컬렉션뷰의 줄갯수를 바꾸는법)

```
let flowLayout = UICollectionViewFlowLayout()


func setupCollectionView() {
        // 컬렉션뷰의 레이아웃을 담당하는 객체
        //let flowLayout = UICollectionViewFlowLayout()
        
        collectionView.dataSource = self
        collectionView.backgroundColor = .white
        // 컬렉션뷰의 스크롤 방향 설정
        flowLayout.scrollDirection = .vertical
        
        let collectionCellWidth = (UIScreen.main.bounds.width - CVCell.spacingWitdh * (CVCell.cellColumns - 1)) / CVCell.cellColumns
        
        flowLayout.itemSize = CGSize(width: collectionCellWidth, height: collectionCellWidth)
        // 아이템 사이 간격 설정
        flowLayout.minimumInteritemSpacing = CVCell.spacingWitdh
        // 아이템 위아래 사이 간격 설정
        flowLayout.minimumLineSpacing = CVCell.spacingWitdh
        
        // 컬렉션뷰의 속성에 할당
 ⭐️⭐️⭐️  collectionView.collectionViewLayout = flowLayout // 꼭 적어줘야하는코드
        
    }

```

```
// 컬렉션뷰 구성을 위한 설정
public struct CVCell {
    static let spacingWitdh: CGFloat = 1
    static let cellColumns: CGFloat = 3 // 3줄을 만든다는 의미
    private init() {}
}

```

중요한건  UICollectionViewFlowLayout을 담당하는 layout 객체가 따로있다. 얘를 가지고 설정해야만 컬렉션뷰를 우리가 원하는형태로 조절할수있음.









