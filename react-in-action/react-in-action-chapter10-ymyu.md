# 10 장 리액트 애플리케이션 아키텍처
>## 복잡한 애플리케이션을 만들기위한 전략

 * 가장널리 사용되는 플럭스패턴
    * 플럭스패턴의 성공적인 변형으로 널리알려진 리덕스

>> 플럭스

* 플럭스 개요
    + 뷰 -> 액션 -> 디스패처 -> 스토어 -> 뷰

    1. 스토어 : 애플리케이션의 상태와 로직을 구현하는 부분 , 하지만 데이터베이스의 레코드를 표현하는 것이 아니라 여러객체의 상태를 관리한다.
    2. 액션 : 플럭스 앱은 상태를 직접 갱신하는 것이 아니라 상태를 변경하는 액션을 생성해서 상태를 수정한다
    3. 뷰 : 사용자 인터페이스
    4. 디스패처 : 액션과 스토어의 갱신 사이에서 둘을 조율

    + 단방향 데이터흐름으로 MVC와는 다르다
    + MVC스타일에서는 데이터의 흐름을 한곳에서 관리할 수 없다.
    + 상태가 산재되어있는 MVC패턴의 관리의 어려움과 버그유발- 시간에걸쳐 발생한 변경에대하여 추론이어려움

* 리액트와 플럭스간의 공통점
    + 두방식 모두 변경을 추적하기 쉬우며, 거추장스러운 방법이 아닌 자율적이고 강력한 방법으로 UI개발
    + 플럭스는 패러다임이기 때문에 많은 라이브러리들이 이를 응용해 구현하고있음.
    + 리덕스 역시 플럭스에서 영감을 얻어 만들어진 라이브러리 중 하나이다.

>> 리덕스
* 핵심
    + 액션 : 변경과 관련된 정보를 가지고 있는 객체
    + 리듀서 : 상태의 변경 방식을 결정
    + 스토어 : 리덕스가 중앙에서 관리하는 상태 객체. 신뢰할 수 있는 데이터 원본
    + 미들웨어 : 어떤과정에 임의의 동작을 주입

* 리덕스 알아보기 245p 그림10.2 참조
    + 자바스크리트 앱을 위한 예측 가능한 상태 보관소
    + 플럭스의 개념을 자신만의 방식으로 해석해 구현한 라이브러리다.
    + 리덕스는 단일 스토어를 사용한다(전역스토어).
        + 플럭스의 개념에서는 여러 스토어를 정의할수 있음.
    + 리덕스는 리듀서를 활용한다.
        + 리듀서는 데이터의 변경을 불변 객체를 이용해 처리하는 방법이다.
    + 리덕스는 미들웨어를 활용한다.
        + 액션과 데이터가 단방향으로 흐르기 때문에 리덕스 앱에 미들웨어를 추가해서 데이터의 갱신에 대응할 임의동작을 주입할 수 있다.
    + 리덕스 액션은 스토어와는 분리되어 있다.
        + 액션을 생성한 모듈은 스토어에 아무것도 전달하지 않는다. 대신 중앙의 디스 패처가 사용하는 액션 객체를 리턴한다.
    + 리덕스는 아키텍처 패러다임이지만 우리가 설치할 수 있는 라이브러리이기도 하다. 이부분이 '순수한' 플럭스 구현체보다 리덕스가 월등한 부분이다.

>>> 리덕스 액션

    1. 액션은 스토어에 데이터를 전달하는 유일한 방법이다.
    2. 기본적으로 액션은 순수한 자바스크립트 객제이며 액션은 그종류에 따라 유일한 type 키를 가져야한다.
+ type은 문자열 상수로 정의하며 마음대로 지정해도 되지만  일관성을 고려하지 않으면 다른개발자가 보기에 혼란스러울 수 있음
<pre>예시<code>
{ 
    type: 'UPDATE_USER_PROFILE'
    payload:{
        email:'hello@ifelese.io'  // 추가적인 데이터도 담을수 있음.
    }
}
</code></pre>
    3. 액션 타입정의를 통해 객체로 묶을수도 있다. 
<pre>예시) src/constants/types.js<code>
export const app = {
    CREATE : 'letter-social/app/create',
    GET:'letter-social/app/get',
    LOADING:'letter-social/app/loading',
    LOADED:'letter-social/app/loaded',
    .
    .
}
</code></pre>
    4. 액션은 액션 객체를 리턴하는 함수인 '액션 생성자'가 생성하며 , dispatch 함수에 의해 스토어로 전달된다.
<pre>예시<code>
import * as types from'../constants/types'
export function loading() {
    return{
        type:types.app.LOADING
    };
}
</code></pre> 
    5. 액션 생성자는 단지 객체만 리턴을 하며, 제 역할을 하게 하러면 리덕스가 제공하는 디스패처를 사용해야한다. 디스패처는 스토어를 셋업해야 사용가능

>>> 리덕스 스토어

    1. 스토어는 디스패처, 리듀서, 상태관리를 포함한다.
    2. 스토어를 셋업하기전에 루트 리듀서파일을 먼저 생성해야함. 
        - 리덕스가 제공하는 combineReducers 함수를 사용해서 여러개의 리듀서를 하나로 결합한다.
        - 리듀서를 결합하는 기능이 없으면 리듀서를 병합하거나 여러 액션으로 라우트하는 방법을 찾아내야함.
 <pre>루트 스토어 생성하기<code>
import {combineReducers} from 'redux';
const rootReducer = combineReducers({});
export default rootReducer;
</code></pre>
    3. getState 함수는 리덕스 스토어에 저장된 상태의 스냅샷을 가져오는 함수
    4. dispatch 리덕스 스토어에 액션을 보내는 함수이다. dispatch 메서드를 호출할 때 액션 생성자가 리턴하는 액션을 전달하면 됨. dispath는 리덕스에서 상태의 변경을 시작하는 유일한방법

>>> 미들웨어

    1. 리덕스는 자체적으로 비동기 액션을 지원하지 않는다. redux-thunk라는 라이브러리를 이용하여 비동기 액션을 지원. redux-thunk는 미들웨어 라이브러리.
    2. 리덕스 미들웨어는 '액션을 보내는 지점과 액션이 리듀서에 도착하는 순간 사이에 위치하는 서드파티 확장 기능' 리듀서가 액션을 처리하기전에 액션을 조작할수 있는 기회
    applyMiddleware()
    3. redux-thunk 로 비동기액션생성자를 만들수있게됨. 액션자체는 동기, Promise객체를 평가할수있음. 리듀서에 비동기식으로 전달


## 리덕스를 사용해야하는가 ? 하지 않을 것인가?
* 애플리케이션의 다른 영역에서도 이 상태나 기능에 대해 알아야 할 필요가 있는가? 그렇다면 리덕스 스토어에 저장해야한다. 온전히 컴포넌트 내부에서만 사용되는 정보(ex.드롭다운리스트) 라면 리덕스 스토어에 저장할 필요가 없다 
* 지금 다루고 있는 상태 데이터가 단순한 것인가, 아니면 리덕스를 통해 더 잘 표현할 수 있는 것인가?
* 사용자 인증 로직처럼 다른 부분에서도 공통적으로 쓰이는 것은 리덕스에 통합하는것이 적합
