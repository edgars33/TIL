# RxSwift 정리

1. 비동기처리란 다른쓰레드를 사용해서 내가 현재 진행하고있는 작업은 작업대로 진행하고 다른쓰레드에서 내가원하는작업을 비동기적으로 동시에  수행한다음에 다른쓰레드에서 멀티로처리한다음에 그 결과를 비동기적으로 받아서 처리를 하는것

2. Rxswift 에서 나중에 생기는 데이터를 Observable 라는 이름을 사용함 그래서 리턴할때 Observable.creat()  를 써줘야하고 result 값에 바로 사용하는게 아닌 result.onNext() 로 사용해줘야함

3. RxSwift는 비동기적으로 생기는 데이터를 컴플리션같은 클로저를 통해 전달하는게 아니라 리턴값으로 전달하기 위해 만들어진 유틸리티 이다. 그 리턴값을 사용할때는 ‘나중에오면’ 메서드를 호출하면된다 
      ‘나중에오면’은 subscribe다
    
4. 나중에 생기는 데이터는 subscribe인데 거기에 event 가 오고 종류가 3가지(next, completed, error)가온다. 데이터가 완전히 전달되면 컴플릿 데이터가 전달될때는 next 에러날때는 error다

5. dispose() 는 
  ```
  let disposable = downloadJson(MEMBER_LIST_URL)
            .subscribe { event in
                switch event {
                case let .next(json):
                    self.editView.text = json
                    self.setVisibleWithAnimation(self.activityIndicator, false)
                    
                case .completed:
                    break
                case .error:
                    break
            }
           
        }
        disposable.dispose()
  ```
  + 이렇게 event 문이 실행중이여도 dispose 해주면 동작을 취소시킬수가있음 
  + 여기서 f.onNext(json) 뒤에  f.onCompleted() 를 써주면 순환참조를 해결할수있음.
  
  6. Observable의 생명주기
   ```
  1. Create - create한다해서 데이터가 생성이 되거나 전달되거나 하지는 않음 그럼 언제 동작하느냐 ?
2. Subscribe - 이때 동작하게됨
3. OnNext
4. —끝—
5. onCompleted / onError
6. Disposed

특징 : 한번 동작이 끝난 Observable은 다시 사용을 못함
 ```
 7. RxSwift의 용도는 *_비동기적으로 생기는 데이터를 리턴값으로 전달하기위해서 나중에 생기는 데이터가 observable클래스로 감싸서 전달하는 것_*
 
 8. 근데 이렇게 만들면 코드들이 너무많아져서 지저분해지는데 이걸 해결하기위한 sugarAPI가 있다
 
 9. Just, from 의 사용법 ( create로하면 코드가 너무길어져서 쓰는것)
 + 원래 create 사용법
 ```
 func downloadJson(_ url:String) -> Observable<String?>  {
        return Observable.create() { emitter in
            emitter.onNext("Hello")
            emitter.onCompleted()
            return Disposables.create
        }
    }
 ```
 + just 사용법 - 한번에 출력 가능해짐.
 ```
 func downloadJson(_ url:String) -> Observable<String?>  {
        return Observable.just("Hello World")
    }
 ```
 + 두개의 단어를 한번에 출력하고싶다면 배열로 만들어주면됨 (string도 배열로만들어야함)
 ```
 func downloadJson(_ url:String) -> Observable<[String?]>  {
        return Observable.just(["Hello", "World"])
    }
 ```
 + from 사용법 - 배열없이도 두개의 단어를 출력가능하게해줌.
 ```
 func downloadJson(_ url:String) -> Observable<String?>  {
        return Observable.from("Hello", "World")
    }
 ```
 10. subscribe도 마찬가지로 코드가 길어져서 onNext로 표현가능 
 ```
 _ = downloadJson(MEMBER_LIST_URL)
        .subscribe(onNext: {print($0)}, onCompleted: { print("com") })
 ```
 
 11. DispatchQueue 도 없애는방법 (.observeOn 해주면 디스패치큐를 안써줘도됨)(operator 라고도부름)-본체와 섭스크라이브사이에서 바꿔치기 하는놈이라 오퍼레이터라함
 ```
 downloadJson(MEMBER_LIST_URL)
            .observeOn(MainScheduler.instance) // sugarApi (operator)
            .subscribe(onNext: { json in
               // DispatchQueue.main.async {
                    self.editView.text = json
                    self.setVisibleWithAnimation(self.activityIndicator, false)
                }
            })
 ```
 12. observeOn 은 다음줄에 영향을 주는것 , subscribeOn(위치상관x)은 첫번째 줄에 영향을줌
   (https://reactivex.io/documentation/operators/observeon.html)
 
 13. Subject는 Observable처럼 subscribe도해서 빼먹을수도있지만 외부에서 값을 통제할수도있다
 (https://reactivex.io/documentation/subject.html)
 
 14. bind 는 subscribe를 순환참조없이 1줄로 간단하게 표현이 가능하다
 ```
 .bind(to : tiemCountLabe.rx.text) 

==

.subscribe(onNext: { [weak self] in 
self?.totalPrice.text = $0
})
 ```
 15.  결국 모든처리는 뷰모델에서 진행하는게 rxswift 이다.  뷰컨트롤러나 테이블셀에서는 아무것도 안하는것. 
 16. RxRelay : UI용도 
    
    원래 subject로도 사용하는데 이건 에러가 생겨도 끊어지지가 않아서  사용해준다.
    
    그래서 complete, error 가 아예 호출이 안됨. 왜 ? 오로지 onNext밖에 없기때문에
    
    메서드명을 accept 로 해주면된다
    
 17.  UI용도로 사용하는건 항상 메인스레드에서 돌아가야하고 에러가 나더라도 끊어지지가 않아야한다.
 
| RxSwift | UI용도 |
| ------------ | ------------- |
| Observable | Driver |
| Subject | Relay |
 
 
 
 
