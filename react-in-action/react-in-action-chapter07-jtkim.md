# chapter 7 리액트의 라우팅

### 7.1 라우팅의 기초
- 라우팅: 리소스 탐색을 위한 시스템
    - 예) 웹에서의 페이지 이동, 서버에서 데이터 리소스 연결 등
    - 이 책에서의 의미는 컴포넌트를 URL에 결합하는 방법을 의미  
<br>  

- 사용 이유
    - 페이지 단위로 모듈화
    - 특정 시점에 올려야 하는 앱의 크기를 줄일 수 있음  
#
### 7.1.1 최신 프런트엔트 웹 애플리케이션에서의 라우팅
- 비교(그림 7.1)
    - 과거의 아키텍처
        - 모든 요청에 대해 전체 앱이 다시 로드된다.  
    <br>  

    - 현대의 아키텍처
        - JS(리액트)가 뷰를 관리
        - 다른 데이터는 XHR/AJAX를 이용해 전송됨
        - 서버에서는 JSON 형태의 데이터로 클라이언트에 전송  
  
#
### 7.2 라우터 구현하기
- Router, Route 컴포넌트 개발
- 개발할 구조
    ```
            Router
                └Route
                    └Route
                    └...
                └...
    ```  
<br>  

- 컴포넌트: Route
    - 라우터들을 관리
    - 매개 변수를 정의  
<br>  

- 컴포넌트: Router
    - 컴포넌트를 URL과 매핑  
<br>  

- 컴포넌트: Link
    - 클라이언트 측 라우터에 탐색 기능을 구현(8장)  
<br>  

- 사용 예(예제 7.1)
    ```html
        <!-- 라우트를 정의, 렌더링될 적절한 컴포넌트를 리턴 -->
        <Router location="/">

            <!-- 각 Route 컴포넌트는 경로, 컴포넌트 연결, 아래처럼 중첩 가능 -->
            <Route path="/" component={App}>

                <!-- 컴포넌트 값에 동적으로 전달 가능 -->
                <Route path="posts/:post" component={SinglePost} />
                <Route path="login" component={Login} />
            </Route>
        </Router>
    ```  
<br>  

- 라우터 라이브러리: react-enroute
    - 작고 가벼움  
#
### 7.2.1 컴포넌트 라우팅
- children 컴포넌트 속성 사용하여 개발 (2장 참조)
    - React.createElement(type, props, children) 여기의 children을 말함  
#
### 7.2.2 <Route /> 컴포넌트의 개발
- Route 컴포넌트의 예제 코드
    - Router: 대부분 작업을 담당
    - Route: URL과 컴포넌트의 매핑에 필요한 데이터를 보관하는 역할  
<br>  

- Route 구현 예(예제 7.2)
    ```javascript
        import PropTypes from 'prop-types';
        import { Component } from 'react';
        // 특정 조건을 만족하는 경우 에러를 발생시키는 도구
        import invariant from 'invariant';

        class Route extends Component {
            // 경로와 함수를 전달 받음
            static propTypes = {
                path: PropTypes.string,
                // 둘 중 하나의 타입인지
                component: PropTypes.oneOfType([PropTypes.element, PropTypes.func]),
            }

            render() {
                return invariant(false, "<Route> 엘리먼트는 렌더되면 안됨");
            }
        }

        export default Route;
    ```
- [invariant](https://github.com/zertosh/invariant) 라이브러리 
    - sandbox에서 사용결과
        ```javascript
            // 첫번째 인자가 false면 에러
            invariant(false, `error msg`);
        ```                        
        - "error msg"가 콘솔에 뜰 줄 알았는데 안뜸.
        - componentDidCatch()에도 안잡힘
        - 에러 발생은 됨  
<br>  

- 구성될 컴포넌트 구조(그림 7.2)
- Router 안에 Route 들이 구현된 모습
- 라우트 매개변수 문법으로 아래와 같이 사용 가능
    ```
        path="/posts/:postID"
        -> "/posts/111"
    ```  
#
### 7.2.3 <Router/> 컴포넌트의 개발
- 리액트만으로 강력하고 유연한 기능을 개발 가능하다는 것을 보여주는 목적의 예제  
<br>  

- react-router 라이브러리?
    - 리액트 앱에 라우팅을 적용하기 위한 가장 대중적인 방법
    - 이번 장에서는 사용하지 않고 12장에서 사용한다고 함  
<br>  

- Router 구현
    - 기본 뼈대와 유틸리티 함수(예제 7.3)
        ```javascript
            export default class Router extends Component {
                static propTypes = {
                    // 컴포넌트
                    children: PropTypes.object,
                    location: PropTypes.string.isRequired
                };

                constructor(props) {
                    super(props);
                    // Route는 Router에 객체 형태로 저장
                    this.routes = {};
                }

                render() {}
            }

            // 슬래시 중첩 제거 유틸함수
            cleanPath(path){
                return path.replace(/\/\//g, '/');
            }
            // 부모와 자식 라우트로 path 조합 유틸함수
            normalizeRoute(path, parent) {
                return (path[0] === '/') || (parent == null)
                    ? path : `${parent.route}/${path}`;
            }
        ```  
#
### 7.2.4 URL 경로 매칭과 매개변수의 처리
- 매개변수에 대한 예시, 일단 ':name' 문법만 사용
    - 다른 것들은 [path-to-regexp](https://github.com/pillarjs/path-to-regexp) 라이브러리 조사  
<br>  

- [enroute](https://github.com/lapwinglabs/enroute) 라이브러리 적용한 라우터 예제(예제 7.6)
    ```javascript
        import enroute from 'enroute';
        import invariant from 'invariant';

        export class Router extends Component {
            static propTypes = {
                // 컴포넌트
                children: PropTypes.element.isRequired,
                location: PropTypes.string.isRequired
            };

            // !!! 이것들이 현재 컴포넌트 프로퍼티로 있다고 이해함
            // routes
            // router
            // children
            // location

            // 컴포넌트 최초상태 설정, enroute 초기화
            constructor(props) {
                super(props);

                // Route는 Router에 객체 형태로 저장
                this.routes = {};

                // 여기서 routes 객체가 생성된다고 가정

                // 라우트를 enroute 라이브러리에 전달
                this.router = enroute(this.routes);
            }

            render() {
                // 현재 위치를 라우터 속성으로 전달
                const { location } = this.props;

                invariant(location, '<Router/> 동작할 location이 필요함');

                // 라우터를 이용해 위치와 매핑되는 컴포넌트 리턴
                return this.router(location);
            }

            // 슬래시 중첩 제거 유틸함수
            cleanPath(path){
                return path.replace(/\/\//g, '/');
            }
            // 부모와 자식 라우트로 path 조합 유틸함수
            normalizeRoute(path, parent) {
                return (path[0] === '/') || (parent == null)
                    ? path : `${parent.route}/${path}`;
            }
        }
    ```  
#
### 7.2.5 Router 컴포넌트에 라우트 추가하기
- 라우트 추가함수 작성  
<br>  

- 리액트 없이 [enroute](https://github.com/lapwinglabs/enroute) 사용 예(예제 7.7)
    ```javascript
        function edit_user (params, props) {
            return Object.assign({}, params, props);
        }
        function create_user (){
            // ...
        }
        function find_user (){
            // ...
        }
        function not_found (){
            // ...
        }

        // 키: 지원경로
        // 값: 처리할 함수
        const router = enroute({
            '/users/new': create_user,
            '/users/:slug': find_user,
            '/users/:slug/edit': edit_user,
            '*': not_found
        });
        // slug: https://zetawiki.com/wiki/%EC%8A%AC%EB%9F%AC%EA%B7%B8_slug

        // 위치, [추가데이터|실행함수]
        enroute('/users/mark/edit', { additional: 'props' });
    ```  
<br>  

- React.Children 객체가 제공하는 기본적인 매서드
    - React.Children.map(children, function)
        - 직속 자식만 탐색(Array.map과 유사)
    - React.Children.forEach(children, function)
        - map과 비슷하지만 리턴없음
    - React.Children.count(children)
        - 자식 컴포넌트의 수
    - React.Children.only(children)
        - 하나의 자식 컴포넌트 리턴
        - 하나가 아니면 에러
    - React.Children.toArray(children)
        - 자식들을 배열로 리턴  
<br>  

- 리액트의 자기 소멸(self-destructing, self-eradicating) 컴포넌트
    - render 메서드에서 배열 리턴 가능(16버전부터)
        ```javascript
            // A) 기존 방법, 최상위 태그로 감싸야 함

            export const Parent = () => {
                return (
                    <Flex>
                        <Sidebar />
                        <Main />
                        <LinksCollection />
                    </Flex>
                );
            }
            export const LinkCollection = () => {
                return (
                    // div로 감싸서 리턴
                    <div>
                        <User />
                        <Group />
                        <Org />
                    </div>
                );
            }

            //////////////////////////////////

            // B) 16 버전 이후

            // 자식 컴포넌트만을 렌더링하고 자신은 소멸
            // CSS 레이아웃 기법에 영향을 주지 않고 컴포넌트 분리 가능
            export const SelfEradicating = (props) => props.children

            export const Parent = () => {
                return (
                    <Flex>
                        <Sidebar />
                        <Main />
                        <LinkCollection />
                    </Flex>
                );
            }

            export const LinksCollection = () => {
                return (
                    <SelfEradicating>
                        <User />
                        <Group />
                        <Org />
                    </SelfEradicating>
                )
            }
        ```  
<br>  

- 라우터에 라우트를 추가하는 과정(그림 7.3)
    - 라우터 안의 라우트들은 path, component 속성을 enroute에 전달
    - 라우트 안의 라우트 들도 마찬가지로 enroute에 전달됨  
<br>  

- Router 컴포넌트 최종 코드(예제 7.10)
    ```javascript
        import PropTypes from 'prop-types';
        import React, { Component } from 'react';
        import enroute from 'enroute';
        import invariant from 'invariant';

        export default class Router extends Component {
            static propTypes = {
                children: PropTypes.array,
                location: PropTypes.string.isRequired
            }

            constructor(props) {
                super(props);
                this.routes = {};

                // 라우트 자식 추가
                this.addRoutes(props.children);

                // enroute에 매핑
                this.router = enroute(this.routes);
            }

            // 라우트 자식들 추가 매서드
            addRoutes(routes, parent) {
                React.Children.forEach(routes, route => this.addRoute(route, parent));
            }

            // 라우트 추가 매서드
            addRoute(element, parent) {
                // element.props에서 속성들 추출
                const { component, path, children } = element.props;
                // 예외처리
                invariant(component, `Route ${path} "path" 속성이 없음`);
                invariant(typeof path !== 'string', `Route ${path} 가 string이 아님`);

                // 라우트 매개변수와 추가데이터를 enroute에 전달하기 위한 함수
                const render = (params, renderProps) => {
                    // 속성 병합하여 새 컴포넌트 생성
                    const finalProps = Object.assign({ params }, this.props, renderProps);
                    const children = React.createElement(component, finalProps);

                    // 부모가 있으면 부모의 render 메서드 호출
                    // 그렇지 않으면 자식 리턴
                    return parent ? parent.render(params, { children }) : children;
                };

                // path 추출
                const route = this.normalizeRoute(path, parent);
                if (children) {
                    // 현재 route 컴포넌트에 자식 컴포넌트가 있으면 반복함
                    this.addRoutes(children, { route, render });
                }

                // 라우트 객체의 경로 생성 후 
                this.routes[this.cleanPath(route)] = render;
            }

            // 유틸함수들
            // 슬래시 중첩제거
            cleanPath(path) {
                return path.replace(/\/\//g, '/');
            }
            // 부모 포함한 path 조합
            normalizeRoute(path, parent) {
                return (path[0] === '/') || (parent == null)
                    ? path 
                    : `${parent.route}/${path}`;
            }

            render() {
                const { location } = this.props;
                invariant(location, '<Router /> 에 location 정보가 필요함');
                return this.router(location);
            }
        }
    ```  
#
### 7.3 요약
- 라우팅으로 페이지 새로고침 줄이고 서버 부담 줄이게 됨(일반적인 웹 설명)  
<br>  

- 리액트는 내장된 라우팅 라이브러리가 없음
  라이브러리를 사용하거나 구현해서 사용  
<br>  

- 리액트는 자식 컴포넌트 데이터 구조와 관련 유틸리티 지원  



