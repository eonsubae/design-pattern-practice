## 코드스피츠 디자인 패턴 5강 - 디자인패턴과 뷰패턴
* 싱글턴
  * MVC에서는 객체를 계속 만드는 경우와 하나만 유지하는 경우로 나뉨
    * 웹 개발에서는 메모리 컨텍스트에 따라서 세 가지 정도로 등장
      1. 어플리케이션 : 웹 서버가 죽을때까지 계속 떠있는 메모리
      2. 세션 : 유저가 로그인해서 로그아웃할 때까지 유지되는 메모리
      3. 리퀘스트 메모리 : 하나의 요청당 존재하는 메모리
    * 이 중 어플리케이션 메모리는 싱글턴(죽지 않음)
  * 자바스크립트에서는 Weakmap을 이용해 쉽게 구현 가능
    * ES6는 코어 객체를 상속받을 수 있음
    * 이는 ES3의 프로토타입 체인과 완전히 다른 형태로 객체를 만든다
    * 코어 상속이 가능한 이유는 new를 하면 부모 클래스의 인스턴스를 만들고 아래 애들을 참조하는 형태로 바뀜
    * 이런 변화는 new target이라는 것이 새로 생겼기 때문이다
    ~~~
    const Singleton = class extends WeakMap {
      has() { err(); }
      get() { err(); }
      set() { err(); }
      getInstance(v) {
        if(!super.has(v.constructor)) super.set(v.constructor, v);
        return super.get(v.constructor);
      }
    }
    ~~~
      * WeakMap의 특징은 Map과 다르게 객체를 키로 사용할 수 있다는 점이다(Map은 값으로만)
      * 싱글턴이 목적이지 WeakMap 사용이 목적이 아니므로 has, get, set은 사용금지시키고 부모측 레이어만 사용 
      * 생성자당 하나의 객체만 리턴하므로 결과적으로 클래스당 인스턴스 하나만 유지해주도록 동작
  * 싱글턴 사용 예제
    ~~~
    const singleton = new Singleton;

    const Test = class {
      constructor(isSingleton) {
        if(isSingleton) return singleton.getInstance(this);
      }
    };

    const test1 = new Test(true), test2 = new Test(true), test3 = new Test;
    console.log(test1 === test2); // true
    console.log(test2 === test3); // false
    ~~~
    * 테스트 객체는 싱글턴 사용여부를 생성자에서 받는다
    * 자바스크립트의 생성자 특성
      * 생성자가 new에 반응해서 아무것도 리턴하지 않거나 원시값을 리턴하면 원래 객체의 this가 반환 
      * 원시값이 아닌 객체를 반환하면 new의 결과가 해당 객체로 반환됨
---
* MVC 개념
  * Controller
    * 이름 그대로 조정자의 역할  
    * 책임들 
      1. 모델을 생성하는 책임
         * 모델이란 순수한 데이터 원형
         * 모델을 조작해서 뷰를 그리는 것이 목적
      2. 모델의 옵저버
         * 모델의 변화가 있으면 컨트롤러가 수신
         * 모델과 컨트롤러를 강하게 바인딩하고 싶지 않기 때문
         * 모델이 Subject 컨트롤러가 Observer
         * 변화를 감지하면 다시 그림을 그림
      3. 뷰 생성
         * MVC는 뷰도 컨트롤러가 생성
         * 뷰는 모델에 기반해서 렌더링을 처리하는 역할을 맡음
         * MV*로 시작하는 아키텍쳐의 차이점은 뷰와 모델과의 관계에서 발생
         * 뷰가 모델을 아는 것이 MVC
      4. 뷰에게 모델을 전달   
         * 컨트롤러가 모델을 뷰에 전달 
         * 컨트롤러는 그림 그리는 행위에는 관여하지 않음
         * 모델과 뷰를 만들고 둘 사이를 연결하는 책임만 수행
      5. 뷰의 액션을 처리 
  * View
    * 책임들 
      1. 모델에 기반하여 렌더링 처리
         * 모델에 기반해서 그림을 그리는 것이 View기 때문에 Model의 변화를 View가 수신해야 할 것처럼 보인다
         * 그러나 뷰는 전체 공정중의 일부만 그리게 되어 있다
         * 컨트롤러가 전체 공정을 다 알고있고 추가작업을 거치므로 컨트롤러가 Model의 변화를 통지받음
      2. 입력의 처리를 컨트롤러에게 위임
         * 그런데 단지 View는 그림만 그리지 않음
         * ex) 웹프론트엔드에서는 View를 클릭하면 이벤트가 발생
         * View가 여러 EventListener들을 가지고 있지만 처리는 다시 컨트롤러에 위임  
         * 그러면 View는 Model도 알고 Controller도 아는 셈이 된다
         * 일반적인 MVC에서는 View가 Model과 Controller 모두를 알고 있다
         * Model은 그림을 그리기 위해서, Controller는 인터랙션을 위임하기 위해 아는 것이다
         * 그렇다는 것은 컨트롤러에도 뷰의 액션을 처리해야 하는 책임이 추가됨을 알 수 있다
  * Model
    * 순수한 데이터 원형  
  * 모델과 뷰는 컨트롤러가 만든 것인데 그럼 컨트롤러는 누가 만든 것인가?
    * Router, App으로 일반적으로 지칭
      * 라우팅 정보를 기반으로 컨트롤러를 생성
      * 이처럼 컨트롤러를 만드는 진입점이 필요하다
    * MVC는 이렇게 네 가지 역할모델이 등장한다
      * 여기에 추상층을 더 추가하자면 모델을 상황에 따라 공급해주는 서비스 등이 추가된다
  * 어쨌든 MVC에서는 Controller가 너무 많은 역할을 맡고 있다
    * 제왕적 컨트롤러 모델
    * MVC에 컨트롤러가 5번 역할만 처리하고 나머지는 Router나 App이 담당하는 경우가 있다 
      * 이렇게 약한 컨트롤러 모델에서는 뷰가 모델을 바로 수신할 수 있다
      * 원래 최초의 MVC 모델은 이처럼 컨트롤러가 약한 모델이었다
      * 그러나 현재의 주류 MVC 프레임워크들은 제왕적 컨트롤러 모델을 채택하고 있다
    * MVC 모델의 가장 큰 어려움은 많은 역할을 맡는 컨트롤러를 만들 뛰어난 개발자가 필요하다는 것   
---
### 모델Model
  ~~~
  const Model = class extends Set {
    constructor(isSingleton) {
      super();
      if(isSingleton) return singleton.getInstance(this);
    }
    add() { err(); }
    delete() { err(); }
    has() { err(); }
    addController(v) {
      if(!is(v, Controller)) err();
      super.add(v);
    }
    removeController(v) {
      if(!is(v, Controller)) err();
      super.delete(v);      
    }
    notify() {
      this.forEach(v => v.listen(this));
    }
  }
  ~~~
  * 모델은 전역에서 공유되는 모델도 있으므로 싱글턴을 지원
  * 옵저버를 지원 받아야 하므로 Set을 상속받음
    * 중복 요소를 제거 가능, 배열이라면 중복 검사를 해야함
    * Set으로 사용하길 원하지 않기 때문에 add, delete, has를 막음
  * 컨트롤러를 추가, 삭제, 통지하는 Subject 역할을 수행
  * notify가 이루어지면 모든 컨트롤러는 listen이라는 메서드를 실행
  ---
  ~~~
  const HomeDetailModel = class extends Model {
    constructor(_id = err(), _title = err(), _memo = '') {
      super();
      prop(this, {_id, _title, _memo});
    }
    get id() { return this._id; }
    get title() { return this._title; }
    get memo() { return this._memo; }
  };
  ~~~ 
  * 하나의 게시글을 의미하는 모델
  * super에 인자를 안보냈으니 무조건 인스턴스로 만들어지고 있음
  * 이 모델은 옵저버 기능 외에 타이틀, 아이디, 메모라는 데이터를 제공하는 기능만 가짐
  ---
  ~~~
  const HomeModel = class extends Model {
    constructor(isSingleton) {
      super(isSingleton);
      if(!this._list) prop(this, {_list : [
        new HomeDetailModel(1, 'todo1', 'memo1'),
        new HomeDetailModel(2, 'todo2', 'memo2'),
        new HomeDetailModel(3, 'todo3', 'memo3'),
        new HomeDetailModel(4, 'todo4', 'memo4'),
        new HomeDetailModel(5, 'todo5', 'memo5')
      ]});
    }
  };
  ~~~
  * 싱글턴을 지원
  * 싱글턴을 만들고 리스트가 없으면 리스트를 생성
  * 어려운 문제들
    * super에서 싱글턴을 만든 다음에 그 다음 상황인 자식의 생성자 상황은 뭘까
    * super가 만약 리턴을 배열로하면 prop 안에서 부터 this는 배열일까? 아니면 그냥 인스턴스가 this일까
    * this는 super를 호출하면 동적으로 변하는건가?        
    ~~~
    class A {
      constructor() {
        return [];
      }
    }

    class B extends A {
      constructor() {
        super();
        console.log(Array.isArray(this));
      }
    }
    const b = new B; // true
    ~~~ 
    * super를 호출하면 this가 변할 수 있다
    * 따라서 super에서 싱글턴으로 처리하면 자식은 이미 싱글턴이된다
  * 싱글턴에 리스트를 설정한적이 없으면 리스트에 기본값을 설정해줌
  * 물론 실제로는 ajax로 json을 받아와서 파싱하는 로직이 들어갈 것
  --- 
  ~~~
  const HomeModel = class extends Model {
    constructor(isSingleton) {
      super(isSingleton);
      if(!this._list) prop(this, {_list : [
        new HomeDetailModel(1, 'todo1', 'memo1'),
        new HomeDetailModel(2, 'todo2', 'memo2'),
        new HomeDetailModel(3, 'todo3', 'memo3'),
        new HomeDetailModel(4, 'todo4', 'memo4'),
        new HomeDetailModel(5, 'todo5', 'memo5')
      ]});
    }
    add(...v) { this._list.push(...v); }
    remove(id) {
      const {_list:list} = this;
      if(!list.some((v, i) => {
        if(v.id == id) {
          list.splice(i, 1);
          return true;
        }
      })) err();
      this.notify();
    }
  };
  ~~~
  * add와 remove를 추가
  * remove하려면 인자로 들어온 id값을 list와 비교해서 처리를 해야 한다
  * 이를 처리할 때 some이나 every를 사용할 수 있다
  * some, every의 특징은 중간에 루프를 끊을 수 있다
  * some은 true, every는 false를 리턴하는 순간 루프가 종료
  * 리턴값은 무조건 true or false
  * truthy한 값이 나와도 무조건 true를 리턴
  * id를 검색한 결과 없으면 false를 리턴해 if문이 true가 되어 에러를 발생시키는 구조
  * 모델은 뷰나 컨트롤러가 모델을 참조하므로 의존성이 없어 로직을 작성하기 비교적 쉽다
  --- 
  ~~~
  const HomeModel = class extends Model {
    constructor(isSingleton) {
      super(isSingleton);
      if(!this._list) prop(this, {_list : [
        new HomeDetailModel(1, 'todo1', 'memo1'),
        new HomeDetailModel(2, 'todo2', 'memo2'),
        new HomeDetailModel(3, 'todo3', 'memo3'),
        new HomeDetailModel(4, 'todo4', 'memo4'),
        new HomeDetailModel(5, 'todo5', 'memo5')
      ]});
    }
    add(...v) { this._list.push(...v); }
    remove(id) {
      const {_list:list} = this;
      if(!list.some((v, i) => {
        if(v.id == id) {
          list.splice(i, 1);
          return true;
        }
      })) err();
      this.notify();
    }
    get list() { return [...this._list]; }
    get(id) {
      let result;
      if(!this._list.some(v => v.id == id ? (result = v) : false)) err();
      return result;
    }
  };
  ~~~ 
  * 최종적으로 뷰가 모델을 가져다 써야하므로 list와 특정 id에 맞는 레코드를 보내주는 로직을 작성
---
### 뷰View
  * 컨트롤러는 모델, 뷰를 모두 알아야 하므로 앞서 모델을 만들었으니 뷰를 만들고 컨트롤러를 만드려는 것
  ~~~
  const View = class {
    constructor(_controller = err(), isSingleton = false) {
      prop(this, {_controller});
      if(isSingleton) return singleton.getInstance(this);
    }
    render(model = null) { override(); }
  }
  ~~~ 
    * 뷰는 우선 컨트롤러를 알아야 한다
    * 뷰도 싱글턴일 가능성이 높다
    * 렌더 메서드에는 모델을 인자로 받고 있다
    * 뷰가 싱글턴일 때 한 번만 생성할건데 그 때 모델을 넣는 것보다 인자로 받는 것이 낫다고 보기 때문 

  ---
  ~~~
  const HomeBaseView = class extends View {
    constructor(controller, isSingleton) {
      super(controller, isSingleton);
    }
    render(model = err()) {
      if(!is(model, HomeModel)) err();
      const {_controller:ctrl} = this;
      return append(el('ul'), ...model.list.map(v => append(
        el('li'),
        el('a', 'innerHTML', v.title, 'addEventListener', ['click', _ => ctrl.$detail(v.id)]),
        el('button', 'innerHTML', 'x', 'addEventListener', ['click', _ => ctrl.$remove(v.id)])
      )));
    }
  }
  ~~~
  * 모델로 리스트가 반드시 넘어와야 함
  * 컨트롤러를 꺼내서 이벤트 리스너를 위임해줌
  * 컨트롤러가 어떤 화면을 그릴 때도, 뷰의 액션에 반응할 때도 메서드가 반응할 것
  * 이 두가지를 구분하기 위해 액션 리스너에만 $표시를 붙임

  ---
  ~~~
  const HomeDetailView = class extends View {
    constructor(controller, isSingleton) {
      super(controller, isSingleton);
    }
    render(model = err()) {
      if(!is(model, HomeDetailModel)) err();
      const {_controller:ctrl} = this;
      return append(el('section'),
        el('h2', 'innerHTML', model.title),
        el('p', 'innerHTML', model.memo),
        el('button', 'innerHTML', 'delete', 'addEventListener', ['click', _ => ctrl.$removeDetail(model.id)]),
        el('button', 'innerHTML', 'list', 'addEventListener', ['click', _ => ctrl.$list()]),
      )
    }
  };
  ~~~
  * 리스트가 넘어오는 것, 싱글턴 설정 여부는 모두 같다
  * 렌더 메서드에서는 디테일용 모델을 받아야 한다

---
* MVC에서는 View가 모델을 알고 있으니 나쁜점이 있다
  * 데이터 모델링의 사정으로 필드가 추가, 수정, 삭제가 가능하다
  * 이 때마다 강력하게 커플링된 뷰가 항상 영향을 받아 다 고쳐져야 한다
  * 이를 해결하기 위해 뷰가 모델을 모르게하는 MVP, MVVM 등이 나온 것이다
* View가 모델을 모르게 하는 방법
  1. 뷰가 자기의 모든 필드에 대해서 getter, setter를 제공
     * 컨트롤러는 뷰를 바라보고 getter, setter만 호출하면 됨
     * 뷰는 모델도 모르고 다 모르게 됨
     * 이것이 MVP의 방식
     * 프레젠터라는 컨트롤러가 뷰를 바라보면 getter, setter 밖에 없음
     * 수동적인 뷰가 되고 코드 양이 엄청나게 많아짐
  2. 뷰가 모델을 알게 하는 것이 아니라 거꾸로 가짜 모델이 뷰를 알게끔 바인딩
     * 가짜 모델(뷰모델)이 주도적으로 뷰와 상호작용해서 뷰를 업데이트   
     * 이 경우 뷰가 뷰모델과 원활하게 상호작용 하기위한 시스템이 반드시 필요
     * 지능적으로 뷰를 뷰모델을 이용해서 업데이트할 수 있게 만들어야 함
     * 뷰모델이 뷰를 알게 만들어 뷰가 모델을 몰라도 되게 만드는 것
     * 그럼 어떻게 하면 뷰는 모델을 모르는데 뷰모델은 뷰를 컨트롤하는가?
     * ex. 앵귤러, 뷰 => 앵커 포인트 : 모델에 데이터를 집어 넣을 뷰만의 선언이 있음
     * 그런 뷰만의 선언된 필드들은 모델의 이름과는 관련없는 뷰를 바꾸기 위한 이름일 뿐
     * 뷰에서 뷰를 업데이트 하기 위한 데이터는 진짜 모델이 아닌 뷰를 업데이트 하기 위한 전용 구조체
     * 진짜 모델은 서버에서 받은 json 객체지만 이를 통해 바로 뷰를 업데이트 하는 것이 아님
     * 뷰에 해당되는 키들로 다시 재정리한 객체를 만들어서 업데이트
     * 이처럼 뷰를 업데이트 하기 위해 재정리한 모델을 뷰모델이라고 함 
     * 이 뷰모델이 뷰를 보고 데이터 바인딩을 통해서 해당 키에 넘겨줌 
     * MVVM을 사용하는 이유
       * 바인딩하는 엔진만 만들면 getter, setter 생성 작업이 없어짐 
       * 동시에 뷰가 모델을 몰라도됨
---
### 컨트롤러Controller
* 컨트롤러는 일반적으로 기본 바인딩을 해주는 경우가 많음
  ~~~
  const Home = class extends Controller {
    constructor(isSingleton) {
      super(isSingleton);
    }
    base() {
      const view = new HomeBaseView(this, true);
      const model = new HomeModel(true);
      return view.render(model);
    }
  }
  ~~~  
  * 컨트롤러를 클래스로 만들때 화면 하나당 클래스 하나를 만들기는 너무 힘든작업
  * 보통 클래스 하나의 메소드 하나가 화면 하나를 담당하게 하는 것이 일반적
    * ex. 게시판 : 게시판 클래스 안에 리스트, 디테일, 라이트 등으로 구성 
  * 아무것도 지정하지 않은 Home 컨트롤러의 기본 컨트롤러를 보여줄 때 이렇게 작성
  * base, basic, root 등의 이름을 지어줌
  * base는 위에서 언급한 다섯가지 역할을 잘 보여주고 있음
    1. 뷰를 만들고
    2. 뷰에 컨트롤러를 전달
    3. 모델을 만들고
    4. 모델을 뷰에게 전달
    5. 뷰의 액션을 처리(HomeBaseView, HomeDetailView의 리턴값인 append 함수의 마지막 부분) 
  * Home.base를 호출하면 최종적으로 HomeBaseView의 render가 그려준 ul이 리턴 
  * 컨트롤러는 이처럼 화면 전체를 다 그리는 것이 아님
  * 전체화면을 구성하는 것은 상위에 있는 App이 수행
* detail은 앱에게 디테일 화면으로 라우팅을 요청하면 됨
  ~~~
  const Home = class extends Controller {
    constructor(isSingleton) {
      super(isSingleton);
    }
    base() {
      const view = new HomeBaseView(this, true);
      const model = new HomeModel(true);
      return view.render(model);
    }
    $detail(id) { app.route('home:detail', id); }
  };
  ~~~
* remove는 id가 들어오면 모델을 가져오면 됨 
  ~~~
  const Home = class extends Controller {
    constructor(isSingleton) {
      super(isSingleton);
    }
    base() {
      const view = new HomeBaseView(this, true);
      const model = new HomeModel(true);
      return view.render(model);
    }
    $detail(id) { app.route('home:detail', id); }
    $remove(id) {
      const model = new HomeModel(true);
      model.remove(id);
      this.$list();
    }
  };
  ~~~
  * base의 model과 $remove안의 model은 같은 모델(싱글턴)
  * 지금은 옵저버로 수신하지 않고 직접 $list를 호출해서 다시 base로 돌아가게끔 작성해놓았음
  * 원래 옵저버로 작성해야 함
* detail은 디테일뷰를 만들고 model의 get id()를 호출해 디테일 모델을 가져옴 
  ~~~
  const Home = class extends Controller {
    constructor(isSingleton) {
      super(isSingleton);
    }
    base() {
      const view = new HomeBaseView(this, true);
      const model = new HomeModel(true);
      return view.render(model);
    }
    $detail(id) { app.route('home:detail', id); }
    $remove(id) {
      const model = new HomeModel(true);
      model.remove(id);
      this.$list();
    }
    detail(id) {
      const view = new HomeDetailView(this, true);
      const model = new HomeModel(true);
      return view.render(model.get(id));
    }
  };
  ~~~
  * 디테일뷰에서는 $removeDetail과 $list를 컨트롤러에 위임하고 있으니 작성해주어야 함
* $removeDetail, $list
  ~~~
  const Home = class extends Controller {
    constructor(isSingleton) {
      super(isSingleton);
    }
    base() {
      const view = new HomeBaseView(this, true);
      const model = new HomeModel(true);
      return view.render(model);
    }
    $detail(id) { app.route('home:detail', id); }
    $remove(id) {
      const model = new HomeModel(true);
      model.remove(id);
      this.$list();
    }
    detail(id) {
      const view = new HomeDetailView(this, true);
      const model = new HomeModel(true);
      return view.render(model.get(id));
    }
    $list() { app.route('home'); }
    $removeDetail(id) {
      this.$remove(id);
      this.$list();
    }
  };
  ~~~
  * $list에는 home에 :을 붙이지 않음(base로 감)
  * $removeDetail에서는 id를 보내 지운 다음에 list로 돌아감
  * list로 돌아가는 것은 나중에 옵저버로 고쳐야 함 
* 이렇게 컨트롤러를 만들면 화면 두개를 호스팅할 수 있음
* 화면에 대해서는 뷰가 책임지고 있음
* 데이터는 모델이 책임지고 있음
* 뷰의 액션은 컨트롤러가 책임지고 있음
### App
* 사용측 코드
  ~~~
  const app = new App('#stage');
  app.add('home:detail', _ => new Home(true));
  app.route('home');
  ~~~
  * 부모를 가지고 태어남
  * 라우팅 테이블을 등록
  * 뒤에 보조함수를 등록해놓음
    * 바로 클래스를 주면 new를 할 때처럼 개입을 할 수 없음
    * 위 코드에서도 Home 컨트롤러를 싱글턴으로 만드는 형태로 개입을 하고 있음 
  * 최초로 home으로 라우팅하고 있음(홈이 그려지면서 시작)
  * 그래서 mvc에서는 보통 App을 통해서 라우팅해서 장면을 전환
* App의 실제 구현
  ~~~
  const App = class {
    constructor(_parent = err()) {
      prop(this, {_parent, _table:new Map});
    }
    add(k = err(), controller = err()) {
      k = k.split(':');
      this._table.set(k[0], controller);
      (k[1] || '').split(',').concat('base').forEach(v => this._table.set(`${k[0]}:${v}`, controller));
    }
    route(path = err(), ...arg) {
      const [k, action = 'base'] = path.split(':');
      if(!this._table.has(k)) return;
      const controller = this._table.get(k)();
      append(attr(sel(this._parent), 'innerHTML', ''), controller[action](...arg));
    }
  }
  ~~~
  * 생성자에서 parent를 받고 table을 Map으로 만들어 놓음
  * add는 key를 받고, 컨트롤러 만들어 주는 것이 들어옴
  * key를 :으로 split해서 앞에 컨트롤러 이름 뒤에는 메소드 이름으로 분리
  * table에 key이름을 잡아주고 컨트롤러를 넣어서 기본생성자를 등록해줌
  * 앞서 split한 :의 뒤가 없으면 ''으로 있으면 k의 index 1을 선택 
  * 여기에 split으로 ,을 하면 detail, remove 등 여러개를 넣을 수 있음
  * 기본값이라는 개념을 넣고 있으므로 base를 넣어줌
  * 각각 루프를 돌면서 컨트롤러 등록
  * route는 path를 받아 이에 맞춰서 인자가 들어오면(ex. id) 받아줌
  * 그리고 액션을 받되 없으면 base로 지정
  * 테이블에 key가 있는지 확인해 없으면 리턴
  * key를 이용해 호출하면 등록했던 컨트롤러를 가져옴
  * 그럼 컨트롤러를 인자와함께 해당 액션을 호출해줌
  * 부모를 셀렉트해 비워주고 액션의 호출 결과를 넣어줌
  * 이런 역할들을 App이 하므로 그냥 컨트롤러가 화면을 갱신하거나 뷰가 모델을 수신해서 자신을 갱신할 수 없음
  * 반드시 라우팅을 경우해야 함
  * MVC의 개념 상으로는 뷰가 모델을 수신하지만 모델을 수신해도 화면을 갱신할 수 없음
  * 반드시 App을 거쳐야 하므로 컨트롤러를 경유하는 것   
### 옵저버
* 컨트롤러 변경
  ~~~
  const Home = class extends Controller {
    constructor(isSingleton) {
      super(isSingleton);
    }
    base() {
      const view = new HomeBaseView(this, true);
      const model = new HomeModel(true);
      model.addController(this); // 모델에 옵저버 등록
      return view.render(model);
    }
    $remove(id) {
      const model = new HomeModel(true);
      model.remove(id);
      // this.$list();
    }
    detail(id) {
      const view = new HomeDetailView(this, true);
      // const model = new HomeModel(true);
      const model = new HomeModel(true).get(id);
      model.addController(this); // 모델에 옵저버 등록
      // return view.render(model.get(id));
      return view.render(model);
    }
    $list() { app.route('home'); }
    $detail(id) { app.route('home:detail', id); }
    listen(model) {
      switch(true) {
        case is(model, HomeModel): return this.$list();
        case is(model, HomeDetailModel): return this.$detail(model.id);
      }
    }
  };
  ~~~
  * base와 detail에 옵저버를 등록하는 코드를 넣음
  * 그러면 $remove하면 옵저버를 타고 listen이 호출됨
  * listen에는 모델이 들어옴
  * 모델을 가지고 어떤 것이 notify를 일으켰는지 알 수 있음
  * list형 모델이냐 detail형 모델이냐에 따라 다른 액션 호출
  * 액션은 앱을 통해서 라우팅을 거쳐 이루어짐
* 디테일뷰 수정
  ~~~
  const HomeDetailView = class extends View {
    constructor(controller, isSingleton) {
      super(controller, isSingleton);
    }
    render(model = err()) {
      if(!is(model, HomeDetailModel)) err();
      const {_controller:ctrl} = this;
      const sec = el('section');
      return append(sec,
        el('input', 'value', model.title, '@cssText', 'display:block', 'className', 'title'),
        el('textarea', 'innerHTML', model.memo, '@cssText', 'display:block', 'className', 'memo'),
        el('button', 'innerHTML', 'edit', 'addEventListener', ['click', _ => ctrl.$editDetail(model.id, sel('.title', sec).value, sel('.memo', sec).value)]),
        el('button', 'innerHTML', 'delete', 'addEventListener', ['click', _ => ctrl.$removeDetail(model.id)]),
        el('button', 'innerHTML', 'list', 'addEventListener', ['click', _ => ctrl.$list()]),
      )
    }
  };
  ~~~
  * h2, p 태그가 아닌 input, textarea, button을 추가 
  * 입력창에 입력후 버튼을 누르면 detail 모델이 바뀜(갱신 가능)
  * 이를 처리하기 위해 Home 컨트롤러에 $editDetail 메서드가 추가됨
    ~~~
    const Home = class extends Controller {
      (...)
      $editDetail(id, title, memo) {
        const model = new HomeModel(true).get(id);
        model.addController(this);
        model.edit(title, memo);
      }
    }
    ~~~ 
    * 모델을 가져와서 모델이 알아서 수정하게끔 만든다
    * 실제 모델인 HomeDetailModel에 edit이 필요해짐
    ~~~
    const HomeDetailModel = class extends Model {
      (...)
      edit(_title = '', _memo = '') {
        prop(this, {_title, _memo});
        this.notify();
      }
    }
    ~~~
     