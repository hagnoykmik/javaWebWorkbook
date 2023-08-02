## 1.3 Web MVC 방식

### MVC 구조와 서블릿/JSP

**서블릿 코드**

- 장점 - 자바코드를 그대로 이용할 수 있고, 상속이나 인터페이스 처리도 가능하다
- 단점 - HTTP로 전달된 메시지를 구성하는 HTML을 처리할 때 상당히 많은 양의 코드를 작성해야 한다.

**JSP**

- 장점 - HTML코드를 바로 사용할 수 있으므로 HTTP 메세지에 적합
- 단점 - 자바 코드를 재사용하는 문제나 자바 코드와 HTML이 혼재

**해결방안**

1. 브라우저의 요청(Request)은 해당 주소를 처리하는 서블릿에 전달 → **Controller**(컨트롤러의 역할을 하는 서블릿은 JSP에 필요한 데이터를 가공하는 역할을 하는데 이때 필요한 데이터를 제공하는 객체를 **Model**이라고 한다.**)**
2. 서블릿 내부에서는 응답에 필요한 재료 데이터들을 준비한다.
3. 서블릿은 준비한 데이터를 JSP로 전달하고
4. JSP에서는 EL을 이용해서 최종적인 결과 데이터(Response)를 생성한다.  → **View**

### MVC 구조의 설계 (GET)

`Browser`→ `InputController`→ `/WEB-INF/calc/input.jsp`

![Untitled](https://user-images.githubusercontent.com/109258144/257753833-64dc4484-c433-4d68-8d67-005d7661355b.png)

```java
// InputController -> GET 매핑
@WebServlet(name = "inputController", urlPatterns = "/calc/input") // 브라우저가 호출하는 경로
public class InputController extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        System.out.println("InputController...doGet...");

        // 서블릿에 전달된 요청을 다론쪽으로 전달하는 객체          내부적으로 input.jsp에 배포
        RequestDispatcher dispatcher = req.getRequestDispatcher("/WEB-INF/calc/input.jsp"); // WEB-INF는 브라우저에서 직접 접근이 불가능한 경로(브라우저에서 jsp로 직접 호출 불가능)

        dispatcher.forward(req, resp);
    }
}
```

![Untitled](https://user-images.githubusercontent.com/109258144/257754047-2d3cb66b-b6c2-4e69-913c-6c657ac2f5ac.png)

### POST

```java
// CalcController -> POST

@WebServlet(name = "calcController", urlPatterns = "/calc/makeResult")
public class CalcController extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        String num1 = req.getParameter("num1");
        String num2 = req.getParameter("num2");

        System.out.printf("num1: %s", num1);
        System.out.printf("num2: %s", num2);

    }
}
```

```java
// input.jsp

<form action="/calc/makeResult" method="post">
  <input type="number" name="num1">
  <input type="number" name="num2">
  <button type="submit">SEND</button>
</form>
```

### sendRedirect

- 브라우저 창에서 앞선 방법과 같이 다시 호출할 수 있기 때문에 POST 방식의 처리는 끝난 후에 다른 페이지로 브라우저 화면 이동시켜야 한다.
- 브라우저의 주소가 아예 변경되기 때문에 사용자의 새로 고침과 같은 요청을 미리 방지할 수 있다.
- 특정한 작업이 완전히 끝나고 새로 시작하는 흐름을 만들 수 있다.

```java
@WebServlet(name = "calcController", urlPatterns = "/calc/makeResult")
public class CalcController extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        String num1 = req.getParameter("num1");
        String num2 = req.getParameter("num2");

        System.out.printf("num1: %s", num1);
        System.out.printf("num2: %s", num2);

        resp.sendRedirect("/index");

    }
}
```

### PRG 패턴 (Post-Redirect-GET)

1. 사용자는 컨트롤러에 원하는 작업을 POST 방식으로 처리하기를 요청
2. POST방식을 컨트롤러에서 처리하고 브라우저는 다른 경로로 이동(GET)하라는 응답(Redirect)
3. 브라우저는 GET 방식으로 이동

### 예시

**Browser ↔ Server**

→ GET 방식으로 입력 화면 요청

← 입력 화면 결과

→ POST 방식으로 처리요청

← 서버 내부에서 결과 처리 후 브라우저는 다른 경로로 redirect 하라고 전달
    브라우저는 다른 경로로 이동하므로 다시 POST 요청할 수 없게 처리

→ GET 방식으로 다른 경로 요청

← 결과

- 반복적으로 POST 호출이 되는 상황을 막을 수 있다.
- 사용자 입장에서 처리가 끝나고 다시 처음 단계로 돌아간다는 느낌을 받는다.