```
```
# 셀에서 이미지 표현 프로세스

###  1.  이미지를 뽑아서 직접적으로 전달하지않고 url을 통해 전달하는이유 (테이블뷰를 빠르게 스크롤할때 생기는 문제점)
테이블뷰같은경우 셀들이 재사용된다. 예를들어 테이블뷰를 표시하다 위로 스크롤하면 이미 지나간 셀들을 다시 재사용해서 ***힙메모리***에 올라가있는 UITableView셀을 다시 활용을해 재활용을한다.
결국 밑에서 다시올라올수있도록 내부적으로 설계되어있기때문에 만약 빠르게 스크롤을 한다면 잘못표시가 되는경우가 있는데 
어떤경우냐면 이미지를 서버에 요청해 다시 받아오는 과정이 생각보다 오래걸리는 과정이기때문에 
셀이 재사용되면서 이미지를 그릴때 최상단에 있던 이미지를 하단에 표시되는 경우가 있음 왜냐하면 셀이 힙의 메모리에 하나만 존재하는데 서버에 이미지를 받아오는 동안에 셀이 밑으로 내려가는 경우가 있음
그래서 아주나중에 오래걸린다음 이미지를 표시하는 경우가 생김 그래서 최상단이미지는 이미지나갔어야했는데 갑자기 하단에서 출연하는경우가 생길수있음.

### 2. 그래서 이런 오류를 안나게 하는방법은?
우선 셀에다 이미지를 표기하는 작업은 cellForRowAt에 직접 구현하는게 아니라 밑에작성한 코드처럼 imageUrl만 전달하고 직접적으로 url을 가지고 서버랑 통신하는일은 테이블뷰셀에서 직접적으로 
일어나도록 구현해야함 그래야 ***셀 내부에서 url을 확인할수있는 코드***를 넣을수가있음

```
cell.imageUrl = musicArrays[indexPath.row].imageUrl
```

이렇게 작성해줘야 오류가 안나고잘된다
```
var imageUrl: String? {
        didSet {
       loadImage()
        }
    }
    
 ~~~
 
 private func loadImage() {
        guard let urlString = self.imageUrl, let url = URL(string: urlString)  else { return }
```

### 3. 셀 내부에서 url을 확인할수있는 코드
오랜작업이 걸리는동안 혹시나 url주소가 바뀔수도있어서 작성해주는코드.
이걸 작성해주면 해당 url주소가 맞는지 한번더 확인한후 맞다면 통과시켜서 오류날일이 없다

 ```
guard self.imageUrl! == url.absoluteString else {return} 
```       

### 4. 셀이 재사용될때 nil로 설정해서 오류가 안나게하는 코드
일반적으로 이미지가 바뀌는 것처럼 보이는 현상을 없애기 위해서 작성하는코드
```
override func prepareForReuse() {
        super.prepareForReuse()
        self.mainImageView.image = nil
    }
```


### 5. DispatchQueue를 사용하는경우 
보통 오래걸리는 작업을 다른 쓰레드에서 일을 시키기위해 작성해주는데 
```
guard let data = try? Data(contentsOf: url) else { return }
```
윗코드처럼 보통 try가 들어가면 DispatchQueue를 사용해줘 해당작업을 메인큐로 비동기적으로 보내게해 오류가 안난다.
```
DispatchQueue.global().async { ~
```
        
        
        
        
        
        
        
        
        
        
        
        
        
        
