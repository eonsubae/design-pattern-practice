# 코드스피츠 디자인패턴 4강

* 지난 컴포지트 패턴내용 복기
  * Task를 컴포지트 패턴으로 만들었더니 Task를 렌더링하는 DomRenderer객체도 컴포지트화
    * 그런데 컴포지트는 Task만 가지고 있어도 상관없음
    * 왜 렌더러도 컴포지트를 같이 돌면서 그려야 하냐는 의문이 도출됨
    * 렌더러는 Composition하는 행동 자체를 모르고 최종 결과에 대한 그림만 그리고 싶음
    * 컴포지션의 결과에 방문해서 원하는 일(그림그리기)만 하기 위해 Visitor패턴이 필요 
    * 기존 코드처럼 Renderer에도 컴포지트를 만들면 Dom, Console 등 다양한 렌더링이 필요할 떄마다 컴포지트 로직을 추가해야 함
  * 컴포지트 패턴을 사용할 때 비지터를 같이 사용하지 않으면 컴포지트를 반복시킬 수 밖에 없다
    * 따라서 컴포지트가 사용되면 비지터가 같이 사용되는 경우가 많다
    * 컴포지트 전파를 막기 위해 반드시 비지터를 같이 사용하는 것이 좋다

* 다양한 사용처
  * 컴포지트 패턴 외에 일반 배열이나 Object에도 비지터를 사용할 수 있다
    * 배열을 순회하면서 특정 로직을 실행할 때도 자주 사용한다
    * ex. forEach의 인자로 들어가는 함수를 비지터라고 생각하면 된다

* 복잡성
  * 비지터를 작성하는 로직은 복잡하지 않다
  * 복잡한 것은 비지터를 컴포지트 구조에 넣어야 하므로 컴포지트 로직이 복잡해진다

* 기존 컴포지트 코드에 accept 함수를 추가
   ~~~js
   accept(sort, stateGroup, visitor) {
     visitor.start();
     this.getResult(sort, stateGroup).children.forEach(
       ({item}) => item.accept(sort, stateGroup, visitor)
     );
     visitor.end();
   }
   ~~~
  * Task가 visitor를 받아들이므로 accept라는 이름을 붙임
  * start와 end는 로직상 빠져나올 필요가 있을 때 넣는다

* 이전 강의에서 Renderer는 DomRenderer였다
  * 렌더러의 로직에 DOM이 포함되어 있었기 때문
  * Renderer로 이름을 바꾼 것은 그림을 그리는 일반적인 로직을 다 가지고 있기 때문
  * 구체적인 그림을 그리는 로직은 visitor에 위임하려는 의도를 포함하는 것이다
  * 따라서 렌더러를 이렇게 만들면 자식 렌더러를 만들 필요가 없다
  * 이제 렌더러는 Task와 협조하여 Task를 그릴수 있도록 visitor를 중개하는 역할을 맡는다
  * Renderer의 constructor에 _visitor는 mandatory다
    * _visitor는 렌더러(this)를 알고 있는데 렌더러가 Task를 중개해주는 메서드를 알기 때문이다
    * _visitor는 Task와 관련된 구체적인 행동을 렌더러에 위임하고 싶은 의도가 있다
  * 코드의 가독성은 알고리즘이 감춰지고 커뮤니케이션이 일어날 때 확보된다
  * 렌더러는 비지터를 소유하고 있다가 list를 그림그리는 순간 비지터를 배포한다
    * 순회하는 책임은 Task가 가지고 있고 순회하면서 비지터를 초대해서 추가적인 액션을 한다
    * 이 과정을 통해 렌더러에는 DOM에 대한 로직이 하나도 없고 서브클래스도 만들 필요가 없다
  * 렌더러에 비지터를 리셋하는 메서드가 있는데 이를 라이프사이클이라고 한다
    * 어떤 객체가 특정 단계별로 실행이 될때 
    * 실행되는 단계가 확정되어 있다면 그것을 라이프사이클이라 한다
    * 비지터의 라이프 사이클은 리셋 후에 시작과 끝을 반복한다

* 이제 DOM에 대한 지식은 비지터를 구현한 DomVisitor가 담당한다
  * DomRenderer에 있던 로직이 옮겨 온것을 알 수 있다
  * 그러나 돔렌더러가 가지고 있던 리스트 관리, 초기화 기능 등이 다 없어졌다
  * 렌더러에 초대를 받아 Task가 자신을 순회할 때 부분기능추가를 시작, 종료해주는 역할만 맡음
  * Task와 list 사이의 관계, Renderer와의 관계가 전부 사라짐 
  * 더 이상 컴포지트를 인식하지 않음
  * 렌더러 본인이 비지터가 되어도 되지만 변화율 때문에 분리를 시킴
  * 렌더러 로직이 바뀔 확률(Task의 데이터에 관한 로직)은 거의 없다
  * 반면에 DOM에 대한 지식은 font 등 수시로 수정요청이 들어올 수 있다
  * 변화율이 역할의 정체다
    * 변화율은 빈도수만 가지고 판단하는 것이 아니다
    * 수정되는 원인이 달라져도 변화율이 다른 것이므로 역할을 분리해야 한다
    * 렌더러가 변하는 원인은 Task라는 자료구조에 근본적인 변화가 생겼을 때
    * DomVisitor를 바꾸는 원인은 그림이 마음에 안들기 때문
  * 비지터를 만든 결과를 종합해보자
    * 그 전까지는 Renderer에 구체적인 DOM에 대한 지식이 분리되지 않았다
    * 더 중요한 것은 Renderer에 컴포지트 지식이 있었다
    * 비지터를 만듦으로써 변화율이 다른 DOM에 대한 지식이 분리되었다
    * 그리고 더 이상 컴포지트 오염이 일어나지 않는다

* 비지터에서는 start, end와 같은 액션을 처리하는 것들을 operator라 총칭해서 부른다

* 그런데 비지터를 완성하고 나서도 문제가 생긴다
  * Task는 컴포지트 자료구조체인데 왜 비지터를 감당하는 문제를 가져야 하는가라는 의문이 생긴다
  * Task는 컴포지트만 알지 비지터에 대한 지식이 없는데 비지터를 감당해야 한다
  * 비지터는 구체적인 사용순서(start, end)가 중요하다
  * 그런데 Task가 비지터의 공개메서드만 알고 있다고해서 제대로 사용이 가능한가라는 문제
  * 따라서 비지터에 대한 지식을 컴포지트 구조체가 가지고 있는 것은 문제일 수 있다
  * 역할이 중복되어있는 결정적인 증거는 accept 안에서 getResult를 사용하는 것
    * accept도 컴포지트를 하는데 그 안에있는 getResult라는 컴포지트를 재활용하고 있음
    * 컴포지트가 2단으로 중복되어서 getResult의 컴포지트 방법이 바뀌었을때 accept하는 방식이 마음에 안들게 되면 2가지 컴포지트를 따로 관리하게 되어서 getResult를 관리하기 위한 result를 새로 만들어야 할 수도 있다
    * accept와 getResult가 만들어내는 컴포지트의 회전이 다를 수 있음
      * 원래부터 역할이 다르기 때문
  * 이 문제는 accept를 받아들여 루프 돌리는 역할과 컴포지트를 하는 두 가지 역할 중복 때문에 발생
  * 비지터의 역할을 비지터만 알고 Task는 getResult로 데이터를 공급하는 역할만 담당해야 한다
  * 여기에 대응하기 위해 accept에서 비지터를 사용하는 지식을 갖지 않기 위해 reverse visitor를 사용

* 역할을 분리하기 위해 반대로 비지터가 Task를 받아들이면 된다
  * 리버스 비지터는 start, end에 대한 지식을 자신이 알고 있다는 점에서 바람직하다
  * 물론 이 역시 비지터가 구상 클래스인 Task와 단단하게 바인딩되는 단점이 있다
    * 아까의 비지터는 Task에 대해서 전혀 모르고 추가적 액션만 처리한다는 장점이 있었다
    * 리버스 비지터는 task 외의 자료를 사용할 수 없다
    * 다만 task 한정이되 어떻게 작동해야 하는지에 대한 지식을 비지터가 소유할 수 있다
  * 따라서 상황에 따라 Trade-off한 상황이라고 할 수 있다
    * 자료구조는 확정되어있고 뷰의 변화가 잦으면 리버스 비지터가 유리
    * Task가 계속 변화할거고 비지터가 오히려 안정화되어있으면 Task가 비지터를 아는 것이 좋다
  
* 비지터를 사용해서 이렇게 코드를 분리해놔도 여전히 문제가 남아있다
  * 렌더러의 add, remove, toggle에는 마지막에 render메서드를 호출하는 로직이 있다
  * 그러나 Task에서 제대로 작업이 수행되었는지가 보장되지 않는다
  * Task의 메서드에서 validation을 통과하지 못하면 작업이 제대로 수행되지 않기 때문이다
  * 모델에게 요청을 했지만 모델의 변화는 오직 모델만이 알고 있다
  * 따라서 모델의 변화를 바깥으로 통지할 방법이 필요하다
  * 그러면 어떻게 객체의 캡슐화를 무너뜨리지 않으면서도 작업의 성공여부를 알 수 있을까

* 옵저버 패턴
  * 크게 옵저버와 서브젝트로 나뉜다
  * 서브젝트는 신문사 옵저버는 신문 구독자로 생각하면 된다
  * 신문을 뿌리는 것을 notify, 구독을 addObserver, 해지를 removeObserver라고 생각하면 된다
  * 옵저버를 만드는 요령 : 일단 인자를 보내지 말고 만들어본다
    * 대부분의 옵저버는 통지만으로 충분한 경우가 많기 때문이다
    * 쓸데없는 값이 없으면 동기화, 멀티쓰레드 문제도 없다
    * 실제로 데이터를 보내야 하는 경우는 그렇게 많지 않으니 잘 생각해봐야 한다
  * 현대 객체지향 언어들은 대부분 단일 상속으로 되어있다
    * 그런데 Task는 옵저버와 서브젝트 모두에 해당된다고 할 수 있다(동시 상속 불가능)
      * 옵저버 : 자식의 변화를 알아차리기 위해서 필요(부모가 자식을 listening)
      * 서브젝트 : 모델의 변화를 외부에 통지하는 것이 주된 역할
    * 이런 경우 해결방법의 하나는 상속의 계층 구조를 이용해 해결할 수 있다
    * 상속의 계층구조를 만들기 싫다면 상속 하나 소유 하나로 해결할 수 있다
      * 이를 이용하면 다수의 소유를 통해 상속을 해결할 수 있다
    * Task의 경우 서브젝트가 주된 역할이므로 서브젝트를 상속받고 옵저버를 소유한다
       ~~~
       const TaskObserver = class extends Observer {
         constructor(_task) {
           super();
           prop(this, {_task});
         }
         observe() {
           this._task.notify();
         }
       } 
       ~~~
       * 생성할 때 Task를 가지고 있는데 이 Task를 이용해 notify한다
         * 내가 자식한테 notify를 받으면 나도 다시 notify를 할 것이라는 의미
         * 2개를 상속받을 수 없으니 notify를 중개해주는 것
  ~~~
  const Task = class extends Subject {
    (...)
    add(task) {
      if (!is(task, Task)) err(); 
      this._list.push(task);
      task.addObserver(this._observer); 
      this.notify();
    }
    // 자식에 나를 알고 있는 옵저버를 추가함으로써 변경을 통지받고 상위로 변경을 통지한다 

    remove(task) {
      const list = this._list;
      if (list.includes(task)) err();
      list.splice(list.indexOf(task), 1);
      task.removeObserver(this._observer);
      this.notify();
    }
    // task를 제거시 옵저버도 제거하고 제거한 사실을 통지한다
  }
  ~~~

  * 여기서 사용한 코드는 정석적인 옵저버 패턴에 해당한다
  ~~~
  const Observer = class {
    observe() { override(); }
  };

  const Subject = class {
    constructor() {
      this._observers = new Set;
    }
    addObserver(o) {
      if (!is(o, Observer)) err();
      this._observers.add(o);
    }
    removeObserver(o) {
      if (!is(o, Observer)) err();
      this._observers.delete(o);
    }
    notify() {
      this._observers.forEach(o => o.observe());
    }
  };
  ~~~
  * 실제는 Event Dispatcher, addEventListener, removeEventListner 등의 이름으로 사용한다
    * 옵저버의 다른 이름을 Listener로 많이 사용
    * addEventListener의 경우 옵저버패턴이라 부르지 않고 채널 패턴이라 부른다
      * 옵저버 패턴의 확장인데, click, mouseover 등 채널에 따른 다양한 확장이 가능
      * 채널 패턴이 더 일반적으로 많이 쓰인다   
  * 이 코드는 Subject가 notify하면 Observer가 수신하는 형태로 작성되었다
    * 이를 푸쉬형이라 부른다(Subject가 밀어내면 Observer가 받는 형태)
  * 푸쉬형 옵저버의 반대는 풀(Pull)형 옵저버가 있다
    * 옵저버가 정보를 당겨오는 것(ex. 브라우저) 
    * naver의 웹서버를 Subject로 생각해보면 브라우저가 요청할 때 데이터를 보내줌
  * 푸쉬형 vs 풀형
    * 연결된 노드 수가 적으면 루프를 도는 부담이 적으니 푸쉬형이 유리하다
    * 많은 노드를 비정기적으로 다룰 때는 풀형이 압도적으로 자원을 적게 쓴다
    * Rx같은 경우 푸쉬형과 풀형을 모두 합쳐놓았다
  * 정리 : 옵저버를 도입한 이유
    * 모델인 Task의 변화는 Task만 알고 있는데 Renderer가 변화여부와 상관없이 렌더링
    * 그렇다고 Task가 변화를 통지하기 위해 다른 객체를 알고 싶지는 않은 상태
    * 그래서 중간에 프로토콜 층인 옵저버를 넣어서 통지하는 것

  ~~~
  const Renderer = class {
    constructor(_list = err(), _visitor = err()) {
      prop(this, { _list, _visitor: prop(_visitor, { renderer : this}), _sort: 'title'});
      _list.addObserver(this);
    }
    observe() {
      this.render();
    }
    add(parent, title, date) {
      if (!is(parent, Task)) err();
      parent.add(new TaskItem(title, date));
      // this.render();
    }
    remove(parent, task) {
      if (!is(parent, Task) || !is(task, Task)) err();
      parent.remove(task);
      // this.render();
    }
    toggle(task) {
      if (!is(task, Task)) err();
      task.toggle();
      // this.render();
    }
    render() {
      this._visitor.reset();
      this._visitor.operation(Task[this._sort], true, this._list);
    }
  }; 
  ~~~
  * 마지막으로 Renderer의 생성자에 자신이 받아들인 list의 변화를 통지받기위해 옵저버를 등록
    * 진짜 변화가 생겼을 때만 observe()를 호출해 렌더링
    * 따라서 add, remove, toggle에 this.render()가 사라짐 