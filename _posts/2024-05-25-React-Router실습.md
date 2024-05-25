---
title: React router 실습 해보기
date: 2024-05-25 20:40:00 +09:00
categories: [JS, React]
tags:
  [
    TIL,
    javascript,
    React,
    reduce,
    hook,
    react-router
  ]
image: ../assets/img/common/react-logo.png
---
__출처__ : 본 실습은 [Udemy : React 2024 완벽가이드](https://www.udemy.com/course/best-react/) 과정을 수강하며 클론 하였습니다.

코드 Github : [Study/react-routing-practice at main · ykdman/Study](https://github.com/ykdman/Study/tree/main/react-routing-practice)

현재 리액트 라우터를 사용하여 간단히 페이지 라우팅과, params 사용, NavLink를 이용한 페이지 전환을 적용한 상태

- Challange
    
    ```jsx
    // Challenge / Exercise
    
    // 1. Add five new (dummy) page components (content can be simple <h1> elements)
    //    - HomePage
    //    - EventsPage
    //    - EventDetailPage
    //    - NewEventPage
    //    - EditEventPage
    // : DONE
    // 2. Add routing & route definitions for these five pages
    //    - / => HomePage V
    //    - /events => EventsPage V
    //    - /events/<some-id> => EventDetailPage V
    //    - /events/new => NewEventPage V
    //    - /events/<some-id>/edit => EditEventPage
    // : DONE
    // 3. Add a root layout that adds the <MainNavigation> component above all page components
    // :Done
    // 4. Add properly working links to the MainNavigation
    // : DONE
    // 5. Ensure that the links in MainNavigation receive an "active" class when active
    // : DONE
    // 6. Output a list of dummy events to the EventsPage
    //    Every list item should include a link to the respective EventDetailPage
    // : DONE
    // 7. Output the ID of the selected event on the EventDetailPage
    // BONUS: Add another (nested) layout route that adds the <EventNavigation> component above all /events... page components
    // : DONE
    ```
    

이후 부터 차례대로 진행 하였다.

## loader() 를 이용한 데이터 가져오기

리액트 라우터를 사용할 때, 기존 컴포넌트에서 useEffect를 사용하여 fetched Data를 가져오는 방식에서, 라우팅 프로퍼티 loader() 를 사용하여, fetch한 data를 return 하여, 연결된 element Page Component에 값을 전달한 후 페이지를 렌더링 하는 방법이 존재한다.

- 기존의 Events 페이지
    
    ```jsx
    
    import EventsList from "../components/EventsList";
    
    function EventsPage() {
      const [isLoading, setIsLoading] = useState(false);
      const [fetchedEvents, setFetchedEvents] = useState();
      const [error, setError] = useState();
    
      useEffect(() => {
        async function fetchEvents() {
          setIsLoading(true);
          const response = await fetch("http://localhost:8080/events");
    
          if (!response.ok) {
            setError("Fetching events failed.");
          } else {
            const resData = await response.json();
            setFetchedEvents(resData.events);
          }
          setIsLoading(false);
        }
    
        fetchEvents();
      }, []);
     
      return (
        <>
          <div style={{ textAlign: "center" }}>
            {isLoading && <p>Loading...</p>}
            {error && <p>{error}</p>}
          </div>
          {!isLoading && fetchedEvents && <EventsList events={fetchedEvents} />}
    
        </>
      );
    }
    
    export default EventsPage;
    
    ```
    
    기존 리액트 컴포넌트에서 데이터를 fetch 하는 방식으로 useEffect 를 사용하여 부수효과를 발생시키는 방법으로 데이터를 가져오던 상태 였는데,
    
    App.js 에 설정된 라우팅 속성 중, loader 속성에 위에서 데이터를 가져오는 로직을 추가하고, App.js 컴포넌트의 코드는 최소화 된다. 
    
- App.js 의 추가된 loader 설정 코드
    
    ```jsx
    
    import { createBrowserRouter, RouterProvider } from "react-router-dom";
    
    import HomePage from "./pages/Home.js";
    import EditEventPage from "./pages/EditEvent.js";
    import EventDetailPage from "./pages/EventDetail.js";
    import EventsPage from "./pages/Events.js";
    import NewEventPage from "./pages/NewEvent.js";
    import RootLayout from "./pages/RootLayout.js";
    import EventsRootLayout from "./pages/EventsRoot.js";
    
    const router = createBrowserRouter([
      {
        path: "/",
        element: <RootLayout />,
        children: [
          { index: true, element: <HomePage /> },
          {
            path: "events",
            element: <EventsRootLayout />,
            children: [
              {
                index: true,
                element: <EventsPage />,
                loader: async () => {
                  const response = await fetch("http://localhost:8080/events");
    
                  if (!response.ok) {
                    //...
                  } else {
                    const resData = await response.json();
                    //페이지 안에서 사용할 수 있는 데이터로 return한다.
                    return resData.events; 
                  }
                },
              },
              { path: ":id", element: <EventDetailPage /> },
              { path: "new", element: <NewEventPage /> },
              { path: ":eventId/edit", element: <EditEventPage /> },
            ],
          },
        ],
      },
    ]);
    
    function App() {
      return <RouterProvider router={router} />;
    }
    
    export default App;
    
    ```
    
- Events.js 에서 loader 의 return 받은 값을 사용하는 코드
    
    ```jsx
    import { useLoaderData } from "react-router-dom";
    import EventsList from "../components/EventsList";
    
    function EventsPage() {
     
      // loader가 전달해준 값을 사용하기 위한 useLoaderData
      const events = useLoaderData();
      return (
        <>
          <EventsList events={events} />
        </>
      );
    }
    
    export default EventsPage;
    
    ```
    
    - loader로 반환 받은 값은, react-router-dom 의 `useLoaderData` 로 사용 가능하다.
    - useLoaderData는 페이지 뿐만 아니라 그에 자식에 해당하는 컴포넌트 들에서도 사용이 가능하다. (상위 컴포넌트 Or 페이지는 사용 못함)
        - 때문에 어떤 경우에 어떤 값이 반환될지 미리 정확히 인지하고 사용하는 것이 중요 하다.
        - 즉, loader를 부르는 Element 와 그 하위에서만 정상적인 호출이 가능하다!

### loader를 호출 할 수 있는 다른 방법

위에서 설명한 방식대로, 라우팅을 설정하는 속성 loader 에 함수 코드를 그대로 작성하는 방법이 기본형이지만, 역시나 import 를 통해서도 loader속성 안에 함수를 집어넣을 수 있다.

위에서 작성했던 Events.js에 loader 역할을 하는 함수를 추가해보았다.

- Events.js
    
    ```jsx
    import { useLoaderData } from "react-router-dom";
    import EventsList from "../components/EventsList";
    
    function EventsPage() {
    
      const events = useLoaderData();
      return (
        <>
         <EventsList events={events} />
        </>
      );
    }
    
    export default EventsPage;
    
    // loader 함수
    export async function loader() {
      const response = await fetch("http://localhost:8080/events");
    
      if (!response.ok) {
        //...
      } else {
        const resData = await response.json();
        return resData.events; //페이지 안에서 사용할 수 있는 데이터로 반환한다.
      }
    }
    
    ```
    

이제  export 로 내보낸 loader함수를 라우팅을 설정하는 App.js에서 사용해보자

- App.js
    
    ```jsx
    import { createBrowserRouter, RouterProvider } from "react-router-dom";
    
    import HomePage from "./pages/Home.js";
    import EditEventPage from "./pages/EditEvent.js";
    import EventDetailPage from "./pages/EventDetail.js";
    // 1. loader 함수를 불러온다.
    import EventsPage, {loader as eventLoader} from "./pages/Events.js";
    import NewEventPage from "./pages/NewEvent.js";
    import RootLayout from "./pages/RootLayout.js";
    import EventsRootLayout from "./pages/EventsRoot.js";
    
    const router = createBrowserRouter([
      {
        path: "/",
        element: <RootLayout />,
        children: [
          { index: true, element: <HomePage /> },
          {
            path: "events",
            element: <EventsRootLayout />,
            children: [
              {
                index: true,
                element: <EventsPage />,
                // 2. 불러온 loader함수를 loader 프로퍼티에 적용 시킨다.
                loader: eventLoader,
              },
              { path: ":id", element: <EventDetailPage /> },
              { path: "new", element: <NewEventPage /> },
              { path: ":eventId/edit", element: <EditEventPage /> },
            ],
          },
        ],
      },
    ]);
    
    function App() {
      return <RouterProvider router={router} />;
    }
    
    export default App;
    ```
    

## loader함수는 언제 실행 될까?

loader() 함수는 우리가 라우팅한 페이지에 접근하려고 하는 순간 실행이 되고, 해당 loader가 완전히 실행 된 후에 연결된 Element 가 렌더링 된다.

즉, 동기적인 진행이 보장 된다 (장점이자 단점)

때문에 페이지 진입시 발생하는 Loader() 가 수행 시간이 길다면, 사용자 입장에서는 

불편함을 겪을 수 있다.

- 위의 문제는 useNavigation 훅을 사용하여, 현재 브라우저의 상태에 따라 원하는 컴포넌트를 렌더링하여 어느정도 개선할 수 있다.

## 커스텀 오류로 loader 오류 처리하기

JS의 throw 를 사용하여 커스텀 에러를 발생 시키면, 라우팅 속성중 errorElement 를 만날때 까지

에러를 계속 `버블링` 시킨다 (상위로 계속 이동)

에러가 errorElement 속성을 가진 라우팅을 만나게 되면, 해당 errorElement 가 렌더링 된다.

- Events.js
    
    ```jsx
    import { useLoaderData } from "react-router-dom";
    import EventsList from "../components/EventsList";
    
    function EventsPage() {
      // loader가 전달해준 값을 사용하기 위한 useLoaderData
      const events = useLoaderData();
    
      return (
        <>
          <EventsList events={events} />
        </>
      );
    }
    
    export default EventsPage;
    
    export async function loader() {
    	// 잘못된 fetch URL 주소 사용
      const response = await fetch("http://localhost:8080/eventsasasad");
    
      if (!response.ok) {
    		 // 커스텀 에러 발생
        throw new Response(JSON.stringify({ messege: "에러가 발생해버려쓰" }), {
          status: 500,
        });
      } else {
        const resData = await response.json();
        return resData.events; //페이지 안에서 사용할 수 있는 데이터로 반환한다.
      }
    }
    ```
    
    위의 코드 처럼, 잘못된 fetch나 다른 에러 발생 시, throw 명령어를 통해 커스텀 에러를 발생 시켰다.
    

- App.js
    
    ```jsx
    
    import { createBrowserRouter, RouterProvider } from "react-router-dom";
    
    import HomePage from "./pages/Home.js";
    import EditEventPage from "./pages/EditEvent.js";
    import EventDetailPage from "./pages/EventDetail.js";
    import EventsPage, { loader as eventLoader } from "./pages/Events.js";
    import NewEventPage from "./pages/NewEvent.js";
    import RootLayout from "./pages/RootLayout.js";
    import EventsRootLayout from "./pages/EventsRoot.js";
    
    // Error 를 보여주기 위한 페이지
    import ErrorPage from "./pages/Error.js";
    
    const router = createBrowserRouter([
      {
        path: "/",
        element: <RootLayout />,
        // events 라우팅에서의 loader 가 에러가 발생했을 때, 
        // errorElement 속성을 만날때 까지 버블링되어, 
        // 해당 에러 컴포넌트가 렌더링 된다.
        errorElement: <ErrorPage />,
        children: [
          { index: true, element: <HomePage /> },
          {
            path: "events",
            element: <EventsRootLayout />,
            children: [
              {
                index: true,
                element: <EventsPage />,
                loader: eventLoader,
              },
              { path: ":id", element: <EventDetailPage /> },
              { path: "new", element: <NewEventPage /> },
              { path: ":eventId/edit", element: <EditEventPage /> },
            ],
          },
        ],
      },
    ]);
    
    function App() {
      return <RouterProvider router={router} />;
    }
    
    export default App;
    
    ```
    

- Error.js
    
    ```jsx
    import { useRouteError } from "react-router-dom";
    import PageContent from "../components/PageContent";
    
    function ErrorPage() {
      const error = useRouteError();
      console.log(error.data);
    
      return (
        <PageContent title="Error 발생">
          <p>에러 발생 해부렀으!</p>
        </PageContent>
      );
    }
    export default ErrorPage;
    
    ```
    
    라우팅에서 발생하는 Error 에 대해, `useRouteError` 를 사용하여, 발생한 에러 정보를 사용할 수 있다. (Response로 발생시킨 에러정보는 Error.data의 접근자로 사용 가능하다.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/5dc66f4b-2d22-4c3a-b8a9-955b7336085a/9eae8ec8-0e86-4c66-802a-470f13d1b625/Untitled.png)
    

### 커스텀 에러발생시 편의를 위한 json()기능

리액트 라우터에는 커스텀 에러시, 에러 정보전송을 json 형식으로 편하게 교환하기 위한 json 함수가 존재,

해당 기능을 적용하여 Error페이지와, Events 페이지의 loader함수를 변경해본다.

- Events.js
    
    ```jsx
    //... EventPage 컴포넌트 코드
    export default EventsPage;
    
    // loader 
    export async function loader() {
      const response = await fetch("http://localhost:8080/eventsasd");
    
      if (!response.ok) {
        throw json({ message: "무언가 잘못됨" }, { status: 500 });
      } else {
        const resData = await response.json();
        return resData.events; //페이지 안에서 사용할 수 있는 데이터로 반환한다.
      }
    }
    
    ```
    
- Error.js
    
    ```jsx
    import { useRouteError } from "react-router-dom";
    import PageContent from "../components/PageContent";
    import MainNavigation from "../components/MainNavigation";
    
    function ErrorPage() {
      const error = useRouteError();
      console.log(error.data);
      console.log(error.status);
    
      let title = "에러 발생 !";
      let message = "무언가 잘못 됐다...";
    
      if (error.status === 500) {
        message = error.data.message;
      }
    
      if (error.status === 404) {
        title = "Not Found!!!";
        message = "뭔가 찾을 수 없는 걸 찾으려고 했음";
      }
    
      return (
        <>
          <MainNavigation />
          <PageContent title={title}>
            <p>{message}</p>
          </PageContent>
        </>
      );
    }
    export default ErrorPage;
    
    ```
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/5dc66f4b-2d22-4c3a-b8a9-955b7336085a/019f52b6-7ed5-4f1b-b4f6-69c38921d26d/Untitled.png)
    

### loader에서 params, request 값 이용

loader는 컴포넌트 렌더링 이전에 페이지 요청이 발생할 때, 같이 즉시 실행된다.

때문에 브라우저가 요청받은 request나 params 값을 알 수 있는데

그 값은 아래와 같이 사용한다.

```jsx
export async function loader({ request, params }) {
  const id = params.eventId; // eventId라는 param을 받은 것
  // request.url 같은 것도 사용할 수 있다.
  const response = await fetch(`http://localhost:8080/events/${id}`);

  if (!response.ok) {
    throw json(
      { message: "선택된 이벤트에 대한 세부정보를 가져올 수 없음" },
      { status: 404 }
    );
  } else {
    return response.json();
  }
}
```

loader가 매칭된 URI 가 요청 될 때, 해당 요청 URI정보를 기반으로 작업을 수행하기 용이하다.

## Form 제출을 위한 action() 사용하기

loader 속성과 같이 라우팅에 사용되는 action 함수를 라우팅 속성에서 설정 가능하다.

또한 react-router-dom에서 제공하는 `<Form>` 을 통해 정보를 바로 전송하는 데이터를 획득할 수 있고, Form으로 인해 action과의 연결점이 생긴다.

- NewEvent.js
    
    ```jsx
    import { json, redirect } from "react-router-dom";
    import EventForm from "../components/EventForm";
    
    function NewEventPage() {
      return <EventForm />;
    }
    
    export default NewEventPage;
    
    // action 함수
    export async function action({ request }) {
      // Form 요청으로 발생하는 데이터를 가져온다.
      const data = await request.formData();
      const eventData = {
        title: data.get("title"),
        image: data.get("image"),
        date: data.get("date"),
        description: data.get("description"),
      };
    
      const response = await fetch("http://localhost:8080/evnets", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(eventData),
      });
    
      if (!response.ok) {
        throw json({ message: "이벤트 정보를 저장할 수 없음" }, { status: 500 });
      }
      return redirect('/events')
    }
    
    ```
    
    action 함수를 작성하여 export하였다. (App.js 에서 호출)
    
    또한 action 함수가 redirect를 반환하는데, 이는 react-router-dom의 기능으로 import 하여
    
    원하는 경로로 사용자를 다시 이동시켜주는 역할을 한다.
    

- EventForm.js
    
    ```jsx
    import { useNavigate, Form } from "react-router-dom";
    
    import classes from "./EventForm.module.css";
    
    function EventForm({ method, event }) {
      const navigate = useNavigate();
      function cancelHandler() {
        navigate("..");
      }
    
      return (
        <Form method="post" className={classes.form}>
          <p>
            <label htmlFor="title">Title</label>
            <input
              id="title"
              type="text"
              name="title"
              required
              defaultValue={event ? event.title : ""}
            />
          </p>
          <p>
            <label htmlFor="image">Image</label>
            <input
              id="image"
              type="url"
              name="image"
              required
              defaultValue={event ? event.img : ""}
            />
          </p>
          <p>
            <label htmlFor="date">Date</label>
            <input
              id="date"
              type="date"
              name="date"
              required
              defaultValue={event ? event.date : ""}
            />
          </p>
          <p>
            <label htmlFor="description">Description</label>
            <textarea
              id="description"
              name="description"
              rows="5"
              required
              defaultValue={event ? event.description : ""}
            />
          </p>
          <div className={classes.actions}>
            <button type="button" onClick={cancelHandler}>
              Cancel
            </button>
            <button>Save</button>
          </div>
        </Form>
      );
    }
    
    export default EventForm;
    
    ```
    
    react-router-dom에서 제공하는 `<Form>` 을 활용하여 해당 form이 react-router 의 action 함수와 연결되게 함
    
- App.js
    
    ```jsx
    import { createBrowserRouter, RouterProvider } from "react-router-dom";
    
    import HomePage from "./pages/Home.js";
    import EditEventPage from "./pages/EditEvent.js";
    import EventDetailPage, {
      loader as eventDetailLoader,
    } from "./pages/EventDetail.js";
    import EventsPage, { loader as eventLoader } from "./pages/Events.js";
    import NewEventPage, { action as newEventAction } from "./pages/NewEvent.js";
    import RootLayout from "./pages/RootLayout.js";
    import EventsRootLayout from "./pages/EventsRoot.js";
    import ErrorPage from "./pages/Error.js";
    
    const router = createBrowserRouter([
      {
        path: "/",
        element: <RootLayout />,
        errorElement: <ErrorPage />,
        children: [
          { index: true, element: <HomePage /> },
          {
            path: "events",
            element: <EventsRootLayout />,
            children: [
              {
                index: true,
                element: <EventsPage />,
                loader: eventLoader,
              },
              {
                path: ":eventId",
                id: "event-detail",
                loader: eventDetailLoader,
                children: [
                  {
                    index: true,
                    element: <EventDetailPage />,
                  },
                  { path: "edit", element: <EditEventPage /> },
                ],
              },
              { path: "new", element: <NewEventPage />, action: newEventAction },
              // action 추가
            ],
          },
        ],
      },
    ]);
    
    function App() {
      return <RouterProvider router={router} />;
    }
    
    export default App;
    
    ```


## 느낀점
React-Router를 실습 해보면서 느낀점은, 되게되게 유용할 것 같다는 생각이 들었다.
실제로 대형 웹서비스는, 하나의 페이지만 띄워주는 경우는 거의 없다보니, 해당 라이브러리를
사용하여 라우팅을 하는 것이 상당히 UX적 이점? 이 있을 것이 라는 생각이 들었다.

또한 페이지를 구성하면서 느낀점은, `재사용성` 이었다.
내가 만든 페이지(컴포넌트)를 다른 페이지에서도 재사용가능한가? 에 대한 궁금증을 가지며 페이지를 만드는 것과, 그렇지 않은 상태에서 개발하는 것은 개발자 경험적으로 중요한 경험인것 같다.

해당 과정을 실습하면서, 어떤 이유에선지, 자체적으로 띄워 놓던 Backend 코드에 CORS 에러도 발생 했었고, V3 백신이 서버측 app.js 코드를 바이러스로 생각하여 배포를 차단하는등, 상당히
억까... 가 많았다 

해당 실습을 통해 다음에는 나만의 프로젝트를 만들 때, 충분히 (필수로) 적용해야 되겠다는 생각을 가지게 되었고, loader와 action 등 react-router-dom 내부의 구조와 실행원리에 대해서도 좀 더 알아봐야 되겠다는 생각이 들었다.