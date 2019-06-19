# chapter 8 고급 라우팅과 파이어베이스의 통합
### 8.1 라우터 사용하기
- App 컴포넌트(예제 8.1)
    - App 컴포넌트가 자식 라우트를 위한 컨테이너가 되도록 함
    - children 컴포넌트로 전환하는 동작을 수행
    ```javascript
        render() {
            if (this.state.error) {
                return (
                    <div className="app">
                        <ErrorMessage error={this.state.error} />
                    </div>
                );
            }
            return (
                <div className="app">
                    <Nav user={this.props.user} />
                    {this.state.loading ? (
                        <div className="loading">
                            <Loader />
                        </div>
                    ) : (
                        this.props.children
                    )}
                </div>
            );
        }
    ```
<br>  

- Home 컴포넌트(예제 8.2)
    - Post 목록 보여주거나 새 포스트 등록 등
    ```javascript
        import React, { Component } from 'react';
        import parseLinkHeader from 'parse-link-header';
        import orderBy from 'lodash/orderBy';

        import * as API from '../shared/http';
        import Ad from '../components/ad/Ad';
        import CreatePost from '../components/post/Create';
        import Post from '../components/post/Post';
        import Welcome from '../components/welcome/Welcome';

        export class Home extends Component {
            constructor(props) {
                super(props);
                this.state = {
                    //...
                };
                this.getPosts = this.getPosts.bind(this);
                this.createNewPost = this.createNewPost.bind(this);
            }
            componentDidMount() {
                this.getPosts();
            }
            getPosts() {
                //...
            }
            createNewPost(post) {
                //...
            }
            render() {
                return (
                    //...
                );
            }
        }

        export default Home;
    ```
<br>  

- history 라이브러리 사용(예제 8.3)
    - history 인스턴스 생성
    - navigate 매서드와 history 인스턴스 모듈로 내보냄
    ```javascript
        import createHistory from 'history/createBrowserHistory';
        // 인스턴스 생성(전역), 앱 전체가 공유
        const history = createHistory();
        // 히스토리 인스턴스에 추가하는거
        const navigate = to => history.push(to);
        export { history, navigate };
    ```
<br>  

- index.js 에 라우트 설정(예제 8.4)
    - render에서 라우터와 라우트 생성, 컴포넌트 연결
    - history에 리스너 핸들러 정의
    ```javascript
        // Import
        import React from 'react';
        import { render } from 'react-dom';
        //...

        export const renderApp = (state, callback = () => {}) => {
            // Router 생성
            render(
                <Router {...state}>
                    <Route path="" component={App}>
                        <Route path="/" component={Home} />
                        <Route path="/posts/:postId" component={SinglePost} />
                        <Route path="/login" component={Login} />
                        <Route path="*" component={NotFound} />
                    </Route>
                </Router>,
                document.getElementById('app'),
                callback
            );
        };

        // 경로와 사용자를 추적할 상태 객체
        let state = {
            location: window.location.pathname,
            user: {
                authenticated: false,
                profilePicture: null,
                id: null,
                name: null,
                token: null
            }
        };

        // Render the app initially
        renderApp(state);

        // History 핸들러 정의
        history.listen(location => {
            const user = firebase.auth().currentUser;
            state = Object.assign({}, state, {
                location: user ? location.pathname : '/login'
            });
            renderApp(state);
        });
    ```
<br>  

### 8.1.1 포스트 읽기 페이지 만들기
- SinglePost 페이지 컴포넌트 구현(예제 8.5)
    ```javascript
        export class SinglePost extends Component {
            static propTypes = {
                params: PropTypes.shape({
                    // 미리 작성한 Post 컴포넌트 가져옴
                    postId: PropTypes.string.isRequired
                })
            };
            render() {
                return (
                    <div className="single-post">
                        // 라우터가 전달해 준 포스트 Id 
                        <Post id={this.props.params.postId} />
                        <Ad
                            url="https://www.manning.com/books/react-in-action"
                            imageUrl="/static/assets/ads/ria.png"
                        />
                    </div>
                );
            }
        }
    ```
- 라우터에 개별 포스트 페이지 연결 추가
    ```javascript
        // index.js
        export const renderApp = (state, callback = () => {}) => {
            render(
                <Router {...state}>
                    <Route path="" component={App}>
                        <Route path="/" component={Home} />
                        <Route path="/posts/:postId" component={SinglePost} />
                    </Route>
                </Router>,
                document.getElementById('app'),
                callback
            );
        };
    ```

### 8.1.2 <Link /> 컴포넌트 작성하기
- history 유틸리티, 라우터와 함께 동작
- 접근성에 대한 정의
    - 인터페이스가 사용자에게 얼마나 유용한가
    - 책 내용의 관련 링크들 찾아가서 보기
    <br>  
    
- Link 컴포넌트 코드(예제 8.7)
    - to(대상 URL), children(컴포넌트) 를 받고  
      cloneElement로 복사  
      history 도구를 이용해 onClick 핸들러 props에 전달
  ```javascript
    // cloneElement(element, props, children)
    function Link({ to, children }) {
        // onClick 이 to로 라우팅하는 객체 생성
        return cloneElement(Children.only(children), {
            onClick: () => navigate(to)
        });
    }
    Link.propTypes = {
        to: PropTypes.string,
        children: PropTypes.node
    };
  ```
  <br>  
    
- Link 컴포넌트 통합(예제 8.8)
    ```javascript
        export class Post extends Component {
            static propTypes = {
                post: PropTypes.object
            };
            constructor(props) {
                super(props);
                this.state = {
                    post: null,
                    comments: [],
                    showComments: false,
                    user: this.props.user
                };
                this.loadPost = this.loadPost.bind(this);
            }
            componentDidMount() {
                this.loadPost(this.props.id);
            }
            loadPost(id) {
                API.fetchPost(id)
                    .then(res => res.json())
                    .then(post => {
                        this.setState(() => ({ post }));
                    });
            }
            render() {
                return this.state.post ? (
                    <div className="post">
                        <!-- Link 컴포넌트 통합부분 -- >
                        <RouterLink to={`/posts/${this.state.post.id}`}>
                            <span>
                                <UserHeader date={this.state.post.date} user={this.state.post.user} />
                                <Content post={this.state.post} />
                                <Image post={this.state.post} />
                                <Link link={this.state.post.link} />
                            </span>
                        </RouterLink>

                        {this.state.post.location && <DisplayMap location={this.state.post.location} />}
                        <PostActionSection showComments={this.state.showComments} />
                        <Comments
                            comments={this.state.comments}
                            show={this.state.showComments}
                            post={this.state.post}
                            handleSubmit={this.createComment}
                            user={this.props.user}
                        />
                    </div>
                ) : null;
            }
        }

        export default Post;
    ```
<br>  
    
### 8.1.3 <NotFound /> 컴포넌트 작성하기
- 잘못된 페이지 탐색 시 처리를 위해
- NotFound 컴포넌트(예제 8.9)
    ```javascript
        import React from 'react';
        import Link from '../components/router/Link';

        // 컴포넌트의 상태관리가 필요없으므로 함수형 컴포넌트로 선언(state 사용 안함)
        export const NotFound = () => {
            return (
                <div className="not-found">
                    <h2>Not found :(</h2>
                    <!-- 홈페이지로 돌아갈 수 있는 링크 제공 -->
                    <Link to="/">
                        <button>go back home</button>
                    </Link>
                </div>
            );
        };

        export default NotFound;
    ```
- NotFound 컴포넌트를 라우터에 통합
    ```javascript
        // index.js
        export const renderApp = (state, callback = () => {}) => {
            render(
                <Router {...state}>
                    <Route path="" component={App}>
                        <Route path="/" component={Home} />
                        <Route path="/posts/:postId" component={SinglePost} />
                        <Route path="/login" component={Login} />
                        <!-- path를 *로 주고 마지막에 배치한다 -->
                        <Route path="*" component={NotFound} />
                    </Route>
                </Router>,
                document.getElementById('app'),
                callback
            );
        };
    ```

#
# 8.2 파이어베이스와의 통합
### 8.2.1 사용자의 로그인 처리하기
#
# 8.3 요약
