# RxSwift(with.Woogie) 간단정리

### 1. 우선 원리부터 이해하기
맨먼저 뷰컨트롤러에서 데이터를 관리하지만 이건 결국 [Movie] 배열을 받아와서 건드리는것,

NetworkManager도포함. 그래서 결국 이 두개를 뷰컨트롤러 말고 뷰모델에서 선언해주고 정리를 해주면  그 데이터들을 뷰모델에서 건들수 있게된다.

### 2. 처음으로 뷰모델을 만들어준후 클래스 MovieViewModel 선언해준뒤그안에 
```
1. [Movie] 배열에 빈배열을 넣어주고
2. NetworkManager.shared를 선언 해주기. 
3. BehaviorRelay를 사용해 [Movie] 배열에 빈배열 넣어주기
4. 정지시켜주는 disposebag 에 DisposeBag 담아주기.
```
### 3. 그러면 가장먼저 앱을 실행시 가장먼저 init을 불러오는데 그래서 여기서 [Movie] 배열을 가져오는 코드를 작성해준뒤
```
init() {
        self.dataObservable()
            .subscribe(onNext: { data in
                self.movieDatas = data ?? []
                self.observeData.accept(data)
            })
            .disposed(by: disposebag)
    }
```

### 4. 그다음 API를 호출하는 부분에서 RxSwift를 이용해 코드작성해주기
```
func dataObservable() -> Observable<[Movie]?>  {
        return Observable<[Movie]?>.create { observe in
            self.networkManager.fetchMovie { result in
                switch result {
                case .success(let data):
                    observe.onNext(data)
                case .failure(let err):
                    observe.onError(err)
                }
            }
            return Disposables.create()
        }
    }
 ```
 
 ### 5. 뷰모델에서 extension으로 MovieViewModel을 해주고 거기서 TableView 에 있는 numberOfRowInSections을 관리해주는 코드만들어주기
 ```
    extension MovieViewModel {
    var numberOfRowInSections: Int {
        self.movieDatas.count
    }
}
```
    
### 6. 이제 뷰컨트롤러에서 MovieViewModel에서 [Movie] 배열을 들고있고 API를 다루고있는 MovieViewModel()을 들고와주고  정지시켜줄수있는 DisposeBag() 선언해주기
```
    let viewModel = MovieViewModel()
    let disposebag = DisposeBag()
```
### 7. 그리고 이상태로 실행시 테이블뷰를 리로드하지않아 오류가 생길수 있으므로 리로드해주기
```
    func setupData() {
        self.viewModel.observeData
            .subscribe(onNext: { _ in
                self.main.reloadData()
                
            })
            .disposed(by: disposebag)
    }
```
    
    
    
    
    
    
    
    
 
