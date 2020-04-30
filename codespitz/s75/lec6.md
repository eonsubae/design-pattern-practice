## 6강 내용정리

MVC 패턴의 문제점
  * 모델이 변경되면 뷰가 영향을 받는다
  * 뷰의 양이 많기 때문에 수정해야할 부분이 많아진다

개선안
  * 상호 의존성을 줄이는 것이 아니라 뷰가 모델을 모르게 하는 것이 초점
  * MVP, MVVM은 뷰가 진짜 모델을 모르게 하는 패턴

MVP
  * MVC에서 컨트롤러는 다섯 가지 책임을 가진 고유명사라고 할 수 있다
  * MVP에서 프레젠터도 한 단어로 설명할 수 없고 책임을 다 설명해야 알 수 있다

MVP의 특징
  * 뷰와 프레젠터는 서로를 알고 있다
  * 그런데 프레젠터는 View를 더 강하게 알고 있다
  * MVC에서는 뷰가 모델에 대해서 완전히 알아야지만 그림을 그릴 수 있었다
  * MVP에서 뷰는 모델에 대한 의존성이 거의 없어진다 
  * 리스너를 맡기기 위해서 프레젠터를 알 뿐이다

그럼 어떻게 하면 프레젠터가 뷰를 알게 하는가?
  * ### 뷰가 누가 나를 알던 간에 상관 없는 public한 getter, setter를 제공해줘야 한다
  * 그리고 인터랙션만 프레젠터에 위임

MVP의 결과
  * 프레젠터가 알아서 작업을 처리하고, View는 일종의 VO가 된다고 할 수있다
  * View는 서비스를 제공할 뿐 어떻게 사용할지는 프레젠터가 결정
  * 게터와 세터의 제공으로 뷰는 모델을 전혀 모른다
  * 게다가 어떤 프레젠터가 뷰를 게팅 세팅하는지도 관심이 없다
  * 모델의 필드가 name에서 username에서 바뀌어도 View의 getName, setName에는 영향이 없다

MVC의 App
```
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
```
기존의 라우터
  * 컨트롤러를 액션값에따라 호출하면 뷰가 리턴되는 로직
  * 리턴된 뷰는 parent에 끼워넣어지는 구조
  * 그런데 MVC, MVP를 떠나 컨트롤러의 액션이 반드시 뷰를 리턴해야 하는 제약을 없애고 싶음

개선된 App
```
const App = class {
  constructor(_parent = err()) {
    prop(this, {_parent, _table:new Map});
  }
  add(k = err(), controller = err()) {
    k = k.split(':');
    this._table.set(k[0], controller).set(`${k[0]}:base`, controller);
    if (k[1]) k[1].split(',').forEach(v => this._table.set(`${k[0]}:${v}`, controller));
  }
  route(path = err(), ...arg) {
    const [k, action = 'base'] = path.split(':');
    if(!this._table.has(k)) return;
    const presenter = this._table.get(k)();
    presenter[action](...arg);
    append(attr(sel(this._parent), 'innerHTML', ''), presenter.view);
  }
}
```
개선된 라우터
  * route 메소드의 4번째 줄을 보면 이제 presenter에 따라 호출만 하고 있을뿐 아무런 리턴값도 받고 있지 않음
  * 아까는 리턴값이 뷰가 되어 appendChild가 되었는데 이제는 호출했을 뿐이다
  * 실제 append는 presenter에 있는 view 객체(presenter.view)를 직접 넣어주는 것으로 변경됨
  * presenter[action](...arg)와 상관없이 프레젠터가 알고있던 view속성을 리턴하도록 변경됨
  * presenter는 view속성을 가지고 있어야 한다는 구조가 전제된 것

기존의 View와 Controller
```
const View = class {
  constructor(_controller = err(), isSingleton = false) {
    prop(this, {_controller});
    if(isSingleton) return singleton.getInstance(this);
  }
  render(model = null) { override(); }
};

const Controller = class {
  constructor(isSingleton) {
    if (isSingleton) return singleton.getInstance(this);
  }
  listen(model) {}
};
```
서로 아무런 관계가 없는 상태
  * 단지 컨트롤러를 상속한 객체들이 액션에 따라 메소드를 호출했을시 뷰를 리턴하는 책임이 있었을 뿐이었다
  * 그런데 MVP에서 원하는 것은 컨트롤러인 프레젠터가 view를 가지는 것

새로운 View와 Presenter
```
const View = class {
  constructor(_presenter = err(), _view = err(), isSingleton = false) {
    prop(this, {_presenter, _view});
    if(isSingleton) return singleton.getInstance(this);
  }
  get view() { return this._view; }
};

const Presenter = class {
  constructor(isSingleton) {
    if (isSingleton) return singleton.getInstance(this);
  }
  get view() { return this._view && this._view.view; }
  listen(model) {}
};
```
View가 변경된 점
  * 생성자에서 DOM객체를 _view로 받아놓았다가 view라는 게터를 호출하면 리턴해줌

Presenter가 하는 역할
  * view라는 게터를 호출하면 자신이 알고 있는 뷰의 DOM객체를 리턴해줌

기존 HomeBaseView
```
const HomeBaseView = class extends View {
  constructor(controller, isSingleton) {
    super(controller, isSingleton);
  }
  render(model = err()) {
    if(!is(model, HomeModel)) err();
    const {_controller:ctrl} = this;
    return append(el('div'), el('button', 'innerHTML', 'Add', 'addEventListener', ['click', _ => ctrl.$add()]),
      append(
      el('ul'), ...model.list.map(v => append(
      el('li'),
      el('a', 'innerHTML', v.title, 'addEventListener', ['click', _ => ctrl.$detail(v.id)]),
      el('button', 'innerHTML', 'x', 'addEventListener', ['click', _ => ctrl.$remove(v.id)])
    ))));
  }
};
```

변경된 HomeBaseView
```
const HomeBaseView = class extends View {
  constructor(presenter, isSingleton) {
    super(presenter, el('ul'), isSingleton);
  }
  set list(list) {
    const {_presenter:pres, view} = this; // 프레젠터와 뷰를 캐싱 
    append(el(view, 'innerHTML', ''), ...list.map(v => append( // 뷰 초기화하고 리스트를 map으로 돌면서 똑같이 추가
      el('li'),
      el('a', 'innerHTML', v.title, 'addEventListener', ['click', _ => pres.$detail(v.id)]),
      el('button', 'innerHTML', 'x', 'addEventListener', ['click', _ => pres.$remove(v.id)])
    )));
  }
}
```
변경된 점
  * super에 프레젠터 뿐만 아니라 DOM객체도 같이 보냄
  * DOM객체가 바로 앞서 보았던 _view가 되는 것
  * 이제 View는 생성자에서 자신의 부모가 될 DOM태그 엘리먼트를 보낼 책임이 생긴 것이다
  * View가 더이상 렌더링하는 메소드가 없고 게터와 세터로 대체
  * set list는 배열형식으로 원하는 리스트만 받으면 되고 모델을 모른다

기존의 Home
```
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
  add() {
    const view = new HomeAddView(this, true);
    const model = new HomeModel(true);
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
  edit(id) {
    const view = new HomeEditView(this, true);
    const model = new HomeModel(true).get(id);
    return view.render(model);
  }
  $add() { app.route('home:add'); }
  $list() { app.route('home'); }
  $detail(id) { app.route('home:detail', id); }
  $edit(id) { app.route('home:edit', id); }
  listen(model) {
    switch(true) {
      case is(model, HomeModel): return this.$list();
      case is(model, HomeDetailModel): return this.$detail(model.id);
    }
  }
  $addDetail(title, memo) {
    const model = new HomeModel(true);
    const id = [...model._list].pop()._id + 1;
    model.add(new HomeDetailModel(id, title, memo));
  }
  $removeDetail(id) {
    this.$remove(id);
    // this.$list();
  }
  $editDetail(id, title, memo) {
    const model = new HomeModel(true).get(id);
    model.addController(this);
    model.edit(title, memo);
  }
};
```

변경된 Home
```
const Home = class extends Presenter {
  constructor(isSingleton) {
    super(isSingleton);
  }
  listen(model) {
    switch(true) {
      case is(model, HomeModel): { 
        prop(this._view, {list:model.list});
        break;
      }
      case is(model, HomeDetailModel): {const {title}};
    }
  }
  base() {
    prop(this, {_view: new HomeBaseView(this, true)});
    const model = new HomeModel(true);
    model.addController(this); // 모델에 옵저버 등록
    model.notify();
  }
  $remove(id) {
    const model = new HomeModel(true);
    model.remove(id);
  }
};
```
  * base가 호출되면 _view에 방금만든 baseView가 들어옴
  * 모델을 잡는순간 옵저버 등록하고 바로 notify를 날림
  * 그러면 listen이 호출되고 지금은 모델이 HomeModel이므로 첫번째 케이스가 호출됨
  * Object.assign으로 this._view의 set list를 호출하고 있다(Object.assign하면 자동적으로 세터 게터가 호출됨)
  * list 세터에 model.list를 보내줘서 렌더링
  * 정리해보면 모델 안에 list라는 키가 있더라든지 list 속성 안에 내가 원하는 지식이 있다는 정보는 프레젠터가 소유
  * 뷰 입장에서는 그냥 list 객체를 받은것 뿐 모델에 대한 지식이 없다
  * 모델 코드에서 이름을 list가 아닌 array로 바꾸든 상관이 없다
  * 프레젠터 입장에서 뷰는 getter, setter하는 애로만 보임

기존의 DetailView
```
const HomeDetailView = class extends View {
  constructor(controller, isSingleton) {
    super(controller, isSingleton);
  }
  render(model = err()) {
    if(!is(model, HomeDetailModel)) err();
    const {_controller:ctrl} = this;
    const sec = el('section');
    return append(sec,
      el('h2', 'innerHTML', model.title, '@cssText', 'display:block', 'className', 'title'),
      el('p', 'innerHTML', model.memo, '@cssText', 'display:block', 'className', 'memo'),
      el('button', 'innerHTML', 'edit', 'addEventListener', ['click', _ => ctrl.$edit(model.id)]),
      el('button', 'innerHTML', 'delete', 'addEventListener', ['click', _ => ctrl.$removeDetail(model.id)]),
      el('button', 'innerHTML', 'list', 'addEventListener', ['click', _ => ctrl.$list()]),
    )
  }
};
```

변경된 DetailView
```
const HomeDetailView = class extends View {
  constructor(presenter, isSingleton) {
    super(presenter, el('section'), isSingleton);
    append(el(this.view, 'innerHTML', ''),
      el('input', '@cssText', 'display:block', 'className', 'title'),
      el('textarea', '@cssText', 'display:block', 'className', 'memo'),
      el('button', 'innerHTML', 'edit', 'addEventListener', 
          ['click', _ => presenter.$editDetail()]),
      el('button', 'innerHTML', 'delete', 'addEventListener', 
          ['click', _ => presenter.$removeDetail()]),
      el('button', 'innerHTML', 'list', 'addEventListener', 
          ['click', _ => presenter.$list()]),
    );
  }
  get title() { return sel('.title', this.view).value; }
  set title(title) { sel('.title', this.view).value = title; }
  get memo() { return sel('.memo', this.view).value; }
  set memo(memo) { sel('.memo', this.view).value = memo; }
};
```
  * 초반에 한 번만 뷰를 세팅해놓고 갱신만하면 되므로 생성자에서 기초작업
  * 그런데 edit과 remove에 id값이 없다
  * id는 프레젠터가 관리하고 있다
  * 게터, 세터만 제공하고 이를 언제 호출하는지는 프레젠터가 정한다
  * 뷰는 모델도 모르고 프레젠터도 리스너를 걸기 위해서만 알면 되므로 의존성이 줄었다
  * 대신 코드가 길어지고, 데이터를 한 번에 업데이트 안하고 게터 세터 호출하는만큼 라운드트립이 발생한다
  * MVP는 일반적으로 라운드트립이 일어나서 여러번 뷰와 프레젠터가 자동으로 통신한다
  * MVC에서는 뷰가 컨트롤러에게 모델을 받고나면 더이상 컨트롤러와 통신하지 않았다
  * MVP에서는 getter, setter할 때마다 통신이 일어남(repaint가 자주 일어남)

Home에 메소드 추가 
```
detail(id) {
  prop(this, {_id: id, _view:new HomeDetailView(this, true)});
  const model = new HomeModel(true).get(id);
  model.addPresenter(this);
  model.notify();
}
$removeDetail() {
  this.$remove(this._id);
}
$editDetail() {
  const model = new HomeModel(true).get(this._id);
  model.addPresenter(this);
  const {title, memo} = this._view;
  model.edit(title, memo);
}
```