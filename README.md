# SpringMVC2-Login1


# 1. 로그인 요구사항 & 프로젝트 생성

### 로그인 요구사항

이전 프로젝트에서 새로 로그인 관련 기능이 추가되었다고 합시다.

- 홈 화면 - 로그인 전
    - 회원 가입
    - 로그인
- 홈 화면 - 로그인 후
    - 본인 이름(누구님 환영합니다.)
    - 상품 관리
    - 로그 아웃
- 보안 요구사항
    - 로그인 사용자만 상품에 접근하고, 관리할 수 있음
    - 로그인 하지 않은 사용자가 상품 관리에 접근하면 로그인 화면으로 이동
- 회원 가입, 상품 관리

**홈 화면 - 로그인 전**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ab972608-1f8c-453b-88ce-203166262019/Untitled.png)

홈 화면 로그인 후

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a98eac8f-c951-44c1-95ad-5cab48efadd4/Untitled.png)

회원 가입

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e8d201b7-a91c-4cf6-91fa-bbe0e631bc70/Untitled.png)

로그인 화면

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2f1ea43e-7a21-4ada-bedb-b3c4a695082b/Untitled.png)

로그인 된 화면에서 상품 관리 버튼을 누르면 상품 목록으로 이동하게 만들어야 합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b0590fd-f111-4701-91aa-5ce80951d371/Untitled.png)


### 프로젝트 생성

이전 프로젝트에 이어서 로그인 기능을 학습해봅니다.

이전 프로젝트를 그대로 사용합니다.

`HomeController`

```java
@Slf4j
@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "redirect:/items";
    }
}
```

`HomeController` 컨트롤러를 추가해줍니다. localhost:8080 에 진입하면 일단은 상품 목록 url 로 redirect 하도록 했습니다.

### 패키지 구조 설계

package 구조

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6361b883-743b-4adb-ab3e-e52e293b730e/Untitled.png)

- hello.login
    - domain
        - item
        - member
        - login
    - web
        - item
        - member
        - login

**도메인이 가장 중요합니다.**

도메인 = 화면, UI, 기술 인프라 등등의 영역은 제외한 시스템이 구현해야 하는 핵심 비즈니스 업무 영역

향후 web을 다른 기술로 바꾸어도 도메인은 그대로 유지할 수 있어야 한다.

이렇게 하려면 web 은 domain 을 알고있지만 domain 은 web 을 모르도록 설계해야 한다. 

이것을 web 은 domain 을 의존하지만, domain 은 web 을 의존하지 않는다고 표현한다. 

예를 들어 web 패키지를 모두 삭제해도 domain 에는 전혀 영향이 없도록 의존관계를 설계하는 것이 중요하다. 반대로 이야기하면 domain 은 web 을 참조하면 안된다.


# 2. 홈 화면

먼저 홈 화면을 타임리프로 만들어 봅시다. 

`HomeController` - `home()` 수정

```java
package hello.login.web;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Slf4j
@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

`templates/home.html` 추가

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="container" style="max-width: 600px">
    <div class="py-5 text-center">
        <h2>홈 화면</h2>
    </div>
    <div class="row">
        <div class="col">
            <button class="w-100 btn btn-secondary btn-lg" type="button"
                    th:onclick="|location.href='@{/members/add}'|">
                회원 가입
            </button>
        </div>
        <div class="col">
            <button class="w-100 btn btn-dark btn-lg"
                    onclick="location.href='items.html'"
                    th:onclick="|location.href='@{/login}'|" type="button">
                로그인
            </button>
        </div>
    </div>
    <hr class="my-4">
</div> <!-- /container -->
</body>
</html>
```

실행결과를 확인해 보면 아래와 같습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/83b46386-4941-419e-843a-e78300600cd8/Untitled.png)

아직 회원 가입 버튼이나 로그인 버튼을 눌렀을 때의 화면은 구성되지 않았습니다.

하나씩 차례대로 만들어 봅시다.

# 3. 회원 가입

`Member`

```java
package hello.login.web.member;

import lombok.Data;

import javax.validation.constraints.NotEmpty;

@Data
public class Member {

    private Long id;

    @NotEmpty
    private String loginId; // 로그인 ID

    @NotEmpty
    private String name; // 사용자 이름

    @NotEmpty
    private String password; 

}
```

`MemberRepository`

```java
package hello.login.web.member;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Repository;

import javax.swing.text.html.Option;
import java.util.*;

@Slf4j
@Repository
public class MemberRepository {

    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L;

    public Member save(Member member) {
        member.setId(++sequence);
        log.info("save: member={}", member);
        store.put(member.getId(), member);
        return member;
    }

    public Member findById(Long id) {
        return store.get(id);
    }

    public Optional<Member> findByLoginId(String loginId) {
        return findAll().stream()
                .filter(m -> m.getLoginId().equals(loginId))
                .findFirst();
    }

    private List<Member> findAll() {
        return new ArrayList<>(store.values());
    }

    public void clearStore() {
        store.clear();
    }

}
```

`MemberController`

```java
package hello.login.web.member;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.validation.Valid;

@Controller
@RequiredArgsConstructor
@RequestMapping("/members")
public class MemberController {

    private final MemberRepository memberRepository;

    @GetMapping("/add")
    public String addForm(@ModelAttribute("member") Member member) {
        return "members/addMemberForm";
    }

    @PostMapping("/add")
    public String save(@Valid @ModelAttribute Member member, BindingResult result) {
        if (result.hasErrors()) {
            return "members/addMemberForm";
        }
        memberRepository.save(member);
        return "redirect:/";

    }

}
```

`templates/members/addMemberForm.html` - 회원 가입 뷰 템플릿

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        }

        .field-error {
            border-color: #dc3545;
            color: #dc3545;
        }
    </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center">
        <h2>회원 가입</h2>
    </div>
    <h4 class="mb-3">회원 정보 입력</h4>
    <form action="" th:action th:object="${member}" method="post">
        <div th:if="${#fields.hasGlobalErrors()}">
            <p class="field-error" th:each="err : ${#fields.globalErrors()}"
               th:text="${err}">전체 오류 메시지</p>
        </div>
        <div>
            <label for="loginId">로그인 ID</label>
            <input type="text" id="loginId" th:field="*{loginId}" class="formcontrol"
                   th:errorclass="field-error">
            <div class="field-error" th:errors="*{loginId}"/>
        </div>
        <div>
            <label for="password">비밀번호</label>
            <input type="password" id="password" th:field="*{password}"
                   class="form-control"
                   th:errorclass="field-error">
            <div class="field-error" th:errors="*{password}"/>
        </div>
        <div>
            <label for="name">이름</label>
            <input type="text" id="name" th:field="*{name}" class="formcontrol"
                   th:errorclass="field-error">
            <div class="field-error" th:errors="*{name}"/>
        </div>
        <hr class="my-4">
        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit">회원
                    가입
                </button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'"
                        th:onclick="|location.href='@{/}'|"
                        type="button">취소
                </button>
            </div>
        </div>
    </form>
</div> <!-- /container -->
</body>
</html>
```

[http://localhost:8080/members/add](http://localhost:8080/members/add) 실행 결과

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b9c1005d-8f15-41a0-beb9-7cdf960f13b1/Untitled.png)

### 회원용 테스트 데이터 추가

편의상 테스트용 회원 데이터를 추가하자.

`loginId` : `test`

`password` : `test!`

`name` : `테스터`

`TestDataInit`

```java
package hello.login;

import hello.login.domain.item.Item;
import hello.login.domain.item.ItemRepository;
import hello.login.web.member.Member;
import hello.login.web.member.MemberRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

@Component
@RequiredArgsConstructor
public class TestDataInit {

    private final ItemRepository itemRepository;
    private final MemberRepository memberRepository;

    /**
     * 테스트용 데이터 추가
     */
    @PostConstruct
    public void init() {
        itemRepository.save(new Item("itemA", 10000, 10));
        itemRepository.save(new Item("itemB", 20000, 20));

        Member member = new Member();
        member.setLoginId("test");
        member.setPassword("test!");
        member.setName("테스터");
        memberRepository.save(member);
        
    }

}
```

추후에 테스트할 것들을 위해 미리 추가해 놓은 것 뿐입니다. 

즉, 아직은 같은 정보로 회원가입을 진행하더라도 중복 관련 예외처리를 하지 않았기 때문에 회원가입이 됩니다.

# 4. 로그인 기능

이어서 로그인 기능을 개발해봅시다. 지금은 로그인 ID, 비밀번호를 입력하는 부분에 집중합니다.

`LoginService`

```java
package hello.login.domain.login;

import hello.login.web.member.Member;
import hello.login.web.member.MemberRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class LoginService {

    private final MemberRepository memberRepository;

    // return null 이면 로그인 실패
    public Member login(String loginId, String password) {
        return memberRepository.findByLoginId(loginId)
                .filter(m -> m.getPassword().equals(password))
                .orElse(null);
    }

}
```

로그인의 핵심 비즈니스 로직은 회원을 조회한 다음에 파라미터로 넘어온 `password`와 비교해서 같으면 회원을 반환하고, 만약 password가 다르면 `null` 을 반환한다

`LoginForm`

```java
package hello.login.domain.login;

import lombok.Data;

import javax.validation.constraints.NotEmpty;

@Data
public class LoginForm {

    @NotEmpty
    private String loginId;

    @NotEmpty
    private String password;
    
}
```

저번 글에서 배운 폼 객체입니다!

`LoginController`

```java
package hello.login.domain.login;

import hello.login.web.member.Member;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;

import javax.validation.Valid;

@Slf4j
@Controller
@RequiredArgsConstructor
public class LoginController {

    private final LoginService loginService;

    @GetMapping("/login")
    public String loginForm(@ModelAttribute("loginForm") LoginForm form) {
        return "login/loginForm";
    }

    @PostMapping("/login")
    public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult) {

        if (bindingResult.hasErrors()) {
            return "login/loginForm";
        }

        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        log.info("login? {}", loginMember);

        if (loginMember == null) {
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "login/loginForm";
        }

        // 로그인 성공 처리해야 함. TODO

        return "redirect:/";

    }
}
```

로그인 컨트롤러는 로그인 서비스를 호출해서 로그인에 성공하면 홈 화면으로 이동하고 로그인에 실패하면 `bindingResult.reject()` 을 사용해서 글로벌 오류 (`ObjectError`) 을 생성한다. 

그리고 정보를 다시 입력하도록 로그인 폼을 뷰 템플릿으로 사용한다.

아직 로그인 성공시 동작할 것을 만들지 않았다.

`templates/login/loginForm.html` - 로그인 폼 뷰 템플릿

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        }

        .field-error {
            border-color: #dc3545;
            color: #dc3545;
        }
    </style>
</head>
<body>
<div class="container">

    <div class="py-5 text-center">
        <h2>로그인</h2>
    </div>

    <form action="item.html" th:action th:object="${loginForm}" method="post">
        <div th:if="${#fields.hasGlobalErrors()}">
            <p class="field-error" th:each="err : ${#fields.globalErrors()}"
               th:text="${err}">전체 오류 메시지</p>
        </div>

        <div>
            <label for="loginId">로그인 ID</label>
            <input type="text" id="loginId" th:field="*{loginId}" class="formcontrol"
                   th:errorclass="field-error">
            <div class="field-error" th:errors="*{loginId}"/>
        </div>

      <p></p>
      <p></p>

        <div>
            <label for="password">비밀번호</label>
            <input type="password" id="password" th:field="*{password}"
                   class="form-control"
                   th:errorclass="field-error">
            <div class="field-error" th:errors="*{password}"/>
        </div>

        <hr class="my-4">

        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit">
                    로그인
                </button>
            </div>

            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'"
                        th:onclick="|location.href='@{/}'|"
                        type="button">취소
                </button>
            </div>

        </div>
    </form>
</div> <!-- /container -->
</body>
</html>
```

로그인 폼 뷰 템플릿에는 특별한 코드는 없다. `loginId` , `password` 가 틀리면 글로벌 오류가 나타난다.

**실행**

실행해보면 로그인이 성공하면 홈으로 이동하고, 로그인에 실패하면 "아이디 또는 비밀번호가 맞지 않습니다."라는 경고와 함께 로그인 폼이 나타난다.

그런데 아직 로그인이 되면 홈 화면에 고객 이름이 보여야 한다는 요구사항을 만족하지 못한다. 로그인의 상태를 유지하면서, 로그인에 성공한 사용자는 홈 화면에 접근시 고객의 이름을 보여주려면 어떻게 해야할까?



# 5. 로그인 처리하기 - 쿠키 사용

참고

여기서는 여러분이 쿠키의 기본 개념을 이해하고 있다고 가정한다. 쿠키에 대해서는 이전 글 “HTTP 웹 강의”를 참고 하면 좋습니다. 혹시 잘 생각이 안나면 쿠키 관련 내용을 다시 익히고 공부하는 것이 좋습니다.

그렇다면 쿠키를 사용해서 로그인, 로그아웃 기능을 구현해봅시다.

**로그인 상태 유지하기**

로그인의 상태를 어떻게 유지할 수 있을까?

로그인 정보를 쿼리 파라미터로 계속 유지하면서 보내는 것은 매우 어렵고 번거로운 작업입니다. 

쿠키를 사용해보자.

### 쿠키

서버에서 로그인에 성공하면 HTTP 응답에 쿠키를 담아서 브라우저에 전달하자. 그러면 브라우저는 앞으로 해당 쿠키를 지속해서 보내준다

**쿠키 생성**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/93f05ad5-b60d-4acb-bd73-ae20c76f8557/Untitled.png)

**클라이언트 쿠키 전달1**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/82a73379-c219-472b-a4d2-3e462f806b79/Untitled.png)

**클라이언트 쿠키 전달2**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/58bab3d6-5000-4ed3-903a-a9805a54cd57/Untitled.png)

### 쿠키에는 영속 쿠키와 세션 쿠키가 있다.

- **영속 쿠키**: 만료 날짜를 입력하면 해당 날짜까지 유지
- **세션 쿠키**: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지

**브라우저 종료시 로그아웃이 되길 기대하므로, 우리에게 필요한 것은 세션 쿠키이다.**

`LoginController` - `login()` 메서드 - 로그인 성공시 세션 쿠키를 생성하자.

```java
@PostMapping("/login")
    public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {

        if (bindingResult.hasErrors()) {
            return "login/loginForm";
        }

        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        log.info("login? {}", loginMember);

        if (loginMember == null) {
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "login/loginForm";
        }

        // 로그인 성공 처리해야 함. TODO

				// 쿠키 생성 로직 - 쿠키에 시간 정보를 주지 않으면 세션 쿠키 (브라우저 종료 시 모든 쿠키 종료)
        Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
        response.addCookie(idCookie);

        return "redirect:/";

    }
```

여기서 쿠키 생성 로직에 주목해봅시다.

로그인에 성공하면 쿠키를 생성하고 `HttpServletResponse` 에 담는다. 쿠키 이름은 `memberId` 이고, 값은 회원의 `id` 을 담아둡니다. 웹 브라우저는 종료 전까지 회윈의 id 을 서버에 계속 보내줄 것입니다.

실행을 하면 크롬 브라우저를 통해 HTTP 응답 헤더에 쿠키가 추가된 것을 확인할 수 있습니다.

우리가 이전에 `TestDataInit` 클래스에서 테스터 아이디로 넣어준 아이디 test , 비밀번호 test! 로 로그인 했을 때의 결과입니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7dcbf7e9-ae6c-4a1d-ab73-ae961aed0b8f/Untitled.png)

이제 요구사항에 맞추어 로그인에 성공하면 로그인 한 사용자 전용 홈 화면을 보여주자.

`HomeController` - 홈 → 로그인 처리

기존 home() 에 있는 @GetMapping("/") 은 주석 처리하자.
@CookieValue 를 사용하면 편리하게 쿠키를 조회할 수 있다.
로그인 하지 않은 사용자도 홈에 접근할 수 있기 때문에 required = false 를 사용한다

```java
package hello.login.web;

import hello.login.web.member.Member;
import hello.login.web.member.MemberRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.CookieValue;
import org.springframework.web.bind.annotation.GetMapping;

@Slf4j
@Controller
@RequiredArgsConstructor
public class HomeController {

    private final MemberRepository memberRepository;

    @GetMapping("/")
    public String home() {
        return "home";
    }

    @GetMapping("/")
    public String homeLogin(
            @CookieValue(name = "memberId", required = false) Long memberId, Model model
    ) {
        if (memberId == null) {
            return "home";
        }

        // 로그인
        Member loginMember = memberRepository.findById(memberId);
        if (loginMember == null) {
            return "home";
        }

        model.addAttribute("member", loginMember);
        return "loginHome";
    }
}
```

기존 `home()` 에 있는 `@GetMapping("/")` 은 주석 처리하자.

`@CookieValue` 를 사용하면 편리하게 쿠키를 조회할 수 있다.

로그인 하지 않은 사용자도 홈에 접근할 수 있기 때문에 `required = false` 를 사용한다

### 로직 분석

로그인 쿠키( `memberId` )가 없는 사용자는 기존 `home` 으로 보낸다. 추가로 로그인 쿠키가 있어도 회원이 없으면 `home` 으로 보낸다.

로그인 쿠키( `memberId` )가 있는 사용자는 로그인 사용자 전용 홈 화면인 `loginHome` 으로 보낸다. 추가로 홈 화면에 화원 관련 정보도 출력해야 해서 `member` 데이터도 모델에 담아서 전달한다.

`templates/loginHome.html`  홈 - 로그인 사용자 전용

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="container" style="max-width: 600px">
    <div class="py-5 text-center">
        <h2>홈 화면</h2>
    </div>
    <h4 class="mb-3" th:text="|로그인: ${member.name}|">로그인 사용자 이름</h4>
    <hr class="my-4">
    <div class="row">
        <div class="col">
            <button class="w-100 btn btn-secondary btn-lg" type="button"
                    th:onclick="|location.href='@{/items}'|">
                상품 관리
            </button>
        </div>
        <p></p>
        <div class="col">
            <form th:action="@{/logout}" method="post">
                <button class="w-100 btn btn-dark btn-lg" type="submit">
                    로그아웃
                </button>
            </form>
        </div>

    </div>
    <hr class="my-4">
</div> <!-- /container -->
</body>
</html>
```

`th:text="|로그인: ${member.name}|"` : 로그인에 성공한 사용자 이름을 출력한다.

상품 관리, 로그아웃 버튼을 노출한다.

**실행**

로그인에 성공하면 사용자 이름이 출력되면서 상품 관리, 로그아웃 버튼을 확인할 수 있다. 

로그인에 성공시 세션 쿠키가 지속해서 유지되고, 웹 브라우저에서 서버에 요청시 `memberId` 쿠키를 계속 보내준다

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0d56b1ef-134a-4f8f-b019-6e656833afc1/Untitled.png)

### 로그아웃 기능

이번에는 로그아웃 기능을 만들어보자. 로그아웃 방법은 다음과 같다.

- 세션 쿠키이므로 웹 브라우저 종료시
- 서버에서 해당 쿠키의 종료 날짜를 0으로 지정

`LoginController` - `logout` 기능 추가

```java
@PostMapping("/logout")
public String logout(HttpServletResponse response) {
    expireCookie(response, "memberId");
    return "redirect:/";
}

private void expireCookie(HttpServletResponse response, String cookieName) {
    Cookie cookie = new Cookie(cookieName, null);
    cookie.setMaxAge(0);

    response.addCookie(cookie);
}
```

**실행** 

로그아웃도 응답 쿠키를 생성하는데 `Max-Age=0` 를 확인할 수 있다. 해당 쿠키는 즉시 종료된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aa7d893b-3137-4755-b68c-38e6309f3366/Untitled.png)