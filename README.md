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


# 6. 쿠키와 보안 문제

이전 글에서 쿠키를 사용해서 로그인Id 를 전달해서 로그인을 유지할 수 있었습니다. 

**그런데 여기에는 심각한 보안 문제가 있습니다!!**

### 보안 문제

**쿠키 값은 임의로 변경할 수 있다.**

클라이언트가 쿠키를 강제로 변경하면 다른 사용자가 된다.

실제 웹브라우저 개발자모드 →  `Application` -> `Cookie` 변경으로 확인

`Cookie: memberId=1` → `Cookie: memberId=2` (다른 사용자의 이름이 보임)

**쿠키에 보관된 정보는 훔쳐갈 수 있다.**

만약 쿠키에 개인정보나, 신용카드 정보가 있다면?

이 정보가 웹 브라우저에도 보관되고, 네트워크 요청마다 계속 클라이언트에서 서버로 전달된다.

쿠키의 정보가 나의 로컬 PC에서 털릴 수도 있고, 네트워크 전송 구간에서 털릴 수도 있다.

**해커가 쿠키를 한번 훔쳐가면 평생 사용할 수 있다.**

해커가 쿠키를 훔쳐가서 그 쿠키로 악의적인 요청을 계속 시도할 수 있다.

### 대안

쿠키에 중요한 값을 노출하지 않고, 사용자 별로 예측 불가능한 임의의 토큰(랜덤 값)을 노출하고, 서버에서 토큰과 사용자 id를 매핑해서 인식한다. 그리고 서버에서 토큰을 관리한다.

토큰은 해커가 임의의 값을 넣어도 찾을 수 없도록 예상 불가능 해야 한다.

해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 해당 토큰의 만료시간을 짧게(예: 30분) 유지한다. 또는 해킹이 의심되는 경우 서버에서 해당 토큰을 강제로 제거하면 된다.

# 7. 로그인 처리하기 - 세션 동작방식

### 목표

**앞서 쿠키에 중요한 정보를 보관하는 방법은 여러가지 보안 이슈가 있었다.** 

이 문제를 해결하려면 결국 중요한 정보를 모두 서버에 저장해야 한다. 그리고 클라이언트와 서버는 추정 불가능한 임의의 식별자 값으로 연결해야 한다.

이렇게 **서버에 중요한 정보를 보관하고 연결을 유지하는 방법을 세션**이라 한다.

### 세션 동작 방식

세션을 어떻게 개발할지 먼저 개념을 이해해보자

**로그인**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/da6977c5-9814-470f-8547-9c6dcbc126d3/Untitled.png)

사용자가 `loginId` , `password` 정보를 전달하면 서버에서 해당 사용자가 맞는지 확인한다

**세션 생성**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/adb9ebad-3625-4184-b8b1-ce12533eee36/Untitled.png)

세션 ID를 생성하는데, 추정 불가능해야 한다.

**UUID는 추정이 불가능하다.** 아래 긴 값이 UUID 이다.

`Cookie: mySessionId=zz0101xx-bab9-4b92-9b32-dadb280f4b61`

생성된 세션 ID 와 세션에 보관할 값( `memberA` )을 서버의 세션 저장소에 보관한다.

**세션id를 응답 쿠키로 전달**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/484bca49-a760-4822-a768-51fba86f73e6/Untitled.png)

**클라이언트와 서버는 결국 쿠키로 연결이 되어야 한다.**

서버는 클라이언트에 `mySessionId` 라는 이름으로 세션ID 만 쿠키에 담아서 전달한다.

클라이언트는 쿠키 저장소에 `mySessionId` 쿠키를 보관한다.

### 중요

여기서 중요한 포인트는 **회원과 관련된 정보는 전혀 클라이언트에 전달하지 않는다는 것**이다.

**오직 추정 불가능한 세션 ID** 만 쿠키를 통해 클라이언트에 전달한다

**클라이언트의 세션id 쿠키 전달**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e40b8e5c-e6ee-44c8-82fd-29cb972b110c/Untitled.png)

클라이언트는 요청시 항상 `mySessionId` 쿠키를 전달한다.

서버에서는 클라이언트가 전달한 `mySessionId` 쿠키 정보로 세션 저장소를 조회해서 로그인시 보관한 세션 정보를 사용한다.

### 정리

세션을 사용해서 서버에서 중요한 정보를 관리하게 되었다. 덕분에 아래와 같은 보안 문제들을 해결할 수 있다.

- 쿠키 값을 변조 가능한 문제
    - 예상 불가능한 복잡한 세션Id를 사용해서 해결!
- 쿠키에 보관하는 정보는 클라이언트 해킹시 털릴 가능성이 있는 문제
    - 세션Id가 털려도 여기에는 중요한 정보가 없다.
- 쿠키 탈취 후 사용할 수 있는 문제
    - 해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 세션의 만료시간을 짧게(예: 30분) 유지한다.
    - 또는 해킹이 의심되는 경우 서버에서 해당 세션을 강제로 제거하면 된다.

그렇다면 세션을 직접 개발해서 적용해봅시다.


# 8. 로그인 처리하기 - 세션 직접 만들기

세션을 직접 개발해서 적용해보자.

**세션 관리는 크게 다음 3가지 기능을 제공하면 된다.**

- **세션 생성**
    - sessionId 생성 (임의의 추정 불가능한 랜덤 값)
    - 세션 저장소에 sessionId 와 보관할 값 저장
    - sessionId 로 응답 쿠키를 생성해서 클라이언트에 전달
- **세션 조회**
    - 클라이언트가 요청한 sessionId 쿠키의 값으로, 세션 저장소에 보관한 값 조회
- **세션 만료**
    - 클라이언트가 요청한 sessionId 쿠키의 값으로 세션 저장소에 보관한 sessionId 와 값 제거

`SessionManager` - 세션 관리

```java
package hello.login.web.session;

import org.springframework.stereotype.Component;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Arrays;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;

@Component
public class SessionManager {

    public static final String SESSION_COOKIE_NAME = "mySessionId";

    private Map<String, Object> sessionStore = new ConcurrentHashMap<>();

    // 세션 생성
    public void createSession(Object value, HttpServletResponse response) {

        // 세션 id 을 생성하고, 값을 세션에 저장
        String sessionId = UUID.randomUUID().toString();
        sessionStore.put(sessionId, value);

        // 쿠키 생성
        Cookie mySessionCookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
        response.addCookie(mySessionCookie);

    }

    // 세션 조회
    public Object getSession(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie == null) {
            return null;
        }
        return sessionStore.get(sessionCookie.getValue());
    }

    // 세션 만료
    public void expire(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie != null) {
            sessionStore.remove(sessionCookie.getValue());
        }
    }

    private Cookie findCookie(HttpServletRequest request, String cookieName) {
        if (request.getCookies() == null) {
            return null;
        }
        return Arrays.stream(request.getCookies())
                .filter(cookie -> cookie.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }

}
```

로직은 어렵지 않아서 쉽게 이해가 될 것이다.

`@Component` : 스프링 빈으로 자동 등록한다.

`ConcurrentHashMap` : `HashMap` 은 동시 요청에 안전하지 않다. 동시 요청에 안전한 `ConcurrentHashMap` 를 사용했다.

`SessionManagerTest` - 테스트

```java
package hello.login.session;

import static org.assertj.core.api.Assertions.*;

class SessionManagerTest {

    SessionManager sessionManager = new SessionManager();

    @Test
    void sessionTest(){

        // 세션 생성
        MockHttpServletResponse response = new MockHttpServletResponse();
        Member member = new Member();
        sessionManager.createSession(member, response);

        // 요청에 응답 쿠키 저장
        MockHttpServletRequest request = new MockHttpServletRequest();
        request.setCookies(response.getCookies());

        // 세션 조회
        Object result = sessionManager.getSession(request);
        assertThat(result).isEqualTo(member);

        // 세션 만료
        sessionManager.expire(request);
        Object expired = sessionManager.getSession(request);
        assertThat(expired).isNull();

    }
}
```

여기서는 `HttpServletRequest`, `HttpServletResponse` 객체를 직접 사용할 수 없기 때문에 테스트에서 비슷한 역할을 해주는 가짜 `MockHttpServletRequest`, `MockHttpServletResponse` 을 사용했다.




# 9. 로그인 처리하기 - 직접 만든 세션 적용

지금까지 개발한 세션 관리 기능을 실제 웹 애플리케이션에 적용해보자.

**`LoginController` - `loginV2()`**

```java
@PostMapping("/login")
public String loginV2(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
    if (bindingResult.hasErrors()) {
        return "login/loginForm";
    }

    Member loginMember = loginService.login(form.getLoginId(), form.getPassword());
    log.info("login? {}", loginMember);

    if (loginMember == null) {
        bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
        return "login/loginForm";
    }

    // 로그인 성공 처리
    // 세션 관리자를 통해 세션을 생성하고, 회원 데이터 보관
    sessionManager.createSession(loginMember, response);

    return "redirect:/";
}
```

클래스 레벨에 `private final SessionManager sessionManager;` 주입

`sessionManager.createSession(loginMember, response);`
로그인 성공시 세션을 등록한다. 

세션에 `loginMember` 를 저장해두고, 쿠키도 함께 발행한다.

**`LoginController` - `logoutV2()`**

```java
@PostMapping("/logout")
public String logoutV2(HttpServletRequest request) {
    sessionManager.expire(request);
    return "redirect:/";
}
```

로그 아웃시 해당 세션의 정보를 제거한다.

**`HomeController` - `homeLoginV2()`**

```java
@GetMapping("/")
public String homeLoginV2(HttpServletRequest request, Model model) {

    // 세션 관리자에 저장된 회원 정보 조회
    Member member = (Member) sessionManager.getSession(request);
    if (member == null) {
        return "home";
    }
    // 로그인
    model.addAttribute("member", member);
    return "loginHome";
}
```

`private final SessionManager sessionManager;` 주입 

세션 관리자에서 저장된 회원 정보를 조회한다. 

만약 회원 정보가 없으면, 쿠키나 세션이 없는 것이므로 로그인 되지 않은 것으로 처리한다.

**정리**

이번 시간에는 세션과 쿠키의 개념을 명확하게 이해하기 위해서 직접 만들어보았습니다.

세션이라는 것이 특별한 것이 아니라 단지 쿠키를 사용하는데, 서버에서 데이터를 유지하는 방법일 뿐이라는 것입니다!

그런데 프로젝트마다 이러한 세션 개념을 직접 개발하는 것은 상당히 불편할 것입니다. 그래서 서블릿도 세션 개념을 지원합니다.

이제 직접 만드는 세션 말고, 서블릿이 공식 지원하는 세션을 알아봅시다. 

서블릿이 공식 지원하는 세션은 우리가 직접 만든 세션과 동작 방식이 거의 같습니다. 추가로 세션을 일정시간 사용하지 않으면 해당 세션을 삭제하는 기능을 제공합니다.

# 10. 로그인 처리하기 - 서블릿 HTTP 세션1

세션이라는 개념은 대부분의 웹 애플리케이션에 필요한 것입니다.어쩌면 웹이 등장하면서 부터 나온 문제이지요…

서블릿은 세션을 위해 `HttpSession` 이라는 기능을 제공하는데, 지금까지 나온 문제들을 해결해줄 수 있습니다.

우리가 직접 구현한 세션의 개념이 이미 구현되어 있고, 더 잘 구현되어 있다.

### HttpSession 소개

서블릿이 제공하는 `HttpSession` 도 결국 우리가 직접 만든 `SessionManager` 와 같은 방식으로 동작한다.

서블릿을 통해 `HttpSession` 을 생성하면 다음과 같은 쿠키를 생성한다. 쿠키 이름이 `JSESSIONID` 이고, 값은 추정 불가능한 랜덤 값이다.

`Cookie: JSESSIONID=5B78E23B513F50164D6FDD8C97B0AD05`

### **HttpSession 사용**

서블릿이 제공하는 `HttpSession` 을 사용하도록 개발해보자.

```java
package hello.login.web;

public class SessionConst {
    public static final String LOGIN_MEMBER = "loginMember";
}
```

`HttpSession` 에 데이터를 보관하고 조회할 때, 같은 이름이 중복 되어 사용되므로, 상수를 하나 정의했다.

**`LoginController` - `loginV3()`**

```java
@PostMapping("/login")
public String loginV3(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletRequest request) {
    if (bindingResult.hasErrors()) {
        return "login/loginForm";
    }

    Member loginMember = loginService.login(form.getLoginId(), form.getPassword());
    log.info("login? {}", loginMember);

    if (loginMember == null) {
        bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
        return "login/loginForm";
    }
    // 로그인 성공 처리

    // 세션이 있으면 있는 세션 반환, 없으면 신규 세션 생성
    HttpSession session = request.getSession();
    // 세션에 로그인 회원 정보 보관
    session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);
    return "redirect:/";
}
```

### **세션 생성과 조회**

세션을 생성하려면 `request.getSession(true)` 를 사용하면 된다.

`public HttpSession getSession(boolean create);`

세션의 `create` 옵션에 대해 알아보자.

`request.getSession(true)`

세션이 있으면 기존 세션을 반환한다.

세션이 없으면 새로운 세션을 생성해서 반환한다.

`request.getSession(false)`

세션이 있으면 기존 세션을 반환한다.

세션이 없으면 새로운 세션을 생성하지 않는다. null 을 반환한다.

`request.getSession()`: 신규 세션을 생성하는 `request.getSession(true)` 와 동일합니다.

### **세션에 로그인 회원 정보 보관**

`session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);`

세션에 데이터를 보관하는 방법은 `request.setAttribute(..)` 와 비슷하다. 

하나의 세션에 여러 값을 저장할 수 있다.

**`LoginController` - `logoutV3()`**

```java
@PostMapping("/logout")
public String logoutV3(HttpServletRequest request) {
    // 세션 삭제
    HttpSession session = request.getSession(false);

    if (session != null) {
        session.invalidate();
    }
    return "redirect:/";
}
```

`session.invalidate()` : 세션을 제거한다.

**`HomeController` - `homeLoginV3()`**

```java
@GetMapping("/")
public String homeLoginV3(HttpServletRequest request, Model model) {

    // 세션이 없으면 home
    HttpSession session = request.getSession(false);
    if (session == null) {
        return "home";
    }

    Member loginMember = (Member) session.getAttribute(SessionConst.LOGIN_MEMBER);

    // 세션에 회원 데이터가 없으면 home
    if (loginMember == null) {
        return "home";
    }

    // 세션이 유지되면 로그인으로 이동
    model.addAttribute("member", loginMember);
    return "loginHome";

}
```

`request.getSession(false)` : `request.getSession()` 를 사용하면 기본 값이 `create: true` 이므로, 로그인 하지 않을 사용자도 의미없는 세션이 만들어진다. 

따라서 세션을 찾아서 사용하는 시점에는 `create: false` 옵션을 사용해서 세션을 생성하지 않아야 한다.

`session.getAttribute(SessionConst.LOGIN_MEMBER)` : 로그인 시점에 세션에 보관한 회원 객체를 찾는다.

**실행 결과**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/db7038dd-fde3-4f0a-9b2b-20f70d3548ed/Untitled.png)

JSESSIONID 쿠키가 적절하게 생성되는 것을 확인할 수 있습니다.

# 11. 로그인 처리하기 -  서블릿 HTTP 세션2

### **@SessionAttribute**

스프링은 세션을 더 편리하게 사용할 수 있도록 `@SessionAttribute` 을 지원합니다.

이미 로그인 된 사용자를 찾을 때는 아래처럼 사용하면 됩니다. (참고로 이 기능은 세션을 생성하지는 않습니다.)

```java
@SessionAttribute(name="loginMember", required=false) Member loginMember
```

**`HomeController` - `homeLoginV3Spring()`**

```java
@GetMapping("/")
public String homeLoginV3Spring(
        @SessionAttribute(name = SessionConst.LOGIN_MEMBER, 
                required = false) Member loginMember,
        Model model
) {
    // 세션에 회원 데이터가 없으면 home 으로
    if (loginMember == null) {
        return "home";
    }

    // 세션이 유지되면 로그인으로 이동
    model.addAttribute("member", loginMember);
    return "loginHome";
}
```

이렇게 간단한 코드로 세션을 찾고, 세션에 들어있는 데이터를 찾는 번거로운 과정을 스프링이 한번에 편리하게 처리해주는 것을 확인할 수 있습니다.

required 의 기본값은 true → session 이 없거나 session 의 속성이 없으면 예외가 던져짐.

예외로 던지지 않고 없을 때는 null 로 처리하고 싶으면 false 로 해야 함.

### **TrackingModes**

로그인을 처음 시도하면 URL이 다음과 같이 `jsessionid` 를 포함하고 있는 것을 확인할 수 있습니다.

크롬으로 로그인을 시도했었다면 종료한 후에 다시 실행하여 로그인 해 봅시다.  

저는 사파리를 사용했습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eeadf2b8-70be-48da-9533-1037b9c76c91/Untitled.png)

```java
http://localhost:8080/;jsessionid=C5FEE3BA9DA52A3BF62D47019DCE1905
```

이것은 웹 브라우저가 쿠키를 지원하지 않을 때 쿠키 대신 URL을 통해서 세션을 유지하는 방법입니다.

이 방법을 사용하려면 URL에 이 값을 계속 포함해서 전달해야 합니다.

타임리프 같은 템플릿은 엔진을 통해서 링크를 걸면 `jsessionid` 를 URL에 자동으로 포함해줍니다.

서버 입장에서 웹 브라우저가 쿠키를 지원하는지 하지 않는지 최초에는 판단하지 못하므로, 쿠키 값도 전달하고, URL에 `jsessionid` 도 함께 전달합니다.

URL 전달 방식을 끄고 항상 쿠키를 통해서만 세션을 유지하고 싶으면 다음 옵션을 넣어주면 됩니다.

`application.properties`

```java
server.servlet.session.tracking-modes=cookie
```

이렇게 하면 URL에 `jsessionid` 가 노출되지 않지요!

변경한 것으로 서버를 다시 실행시킨 후 사파리를 껐다가 다시 로그인을 해보면 초기 URL 에도 `jsessionid` 가 포함되지 않는 것을 볼 수 있습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dfe32e52-68fb-4f7c-b56e-3f47da12bc23/Untitled.png)

# 12. 세션 정보와 타임아웃 설정

### 세션 정보 확인

세션이 제공하는 정보들을 아래 코드를 통해서 확인해봅시다.

`SessionInfoController`

```java
package hello.login.web.session;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.Date;

@Slf4j
@RestController
public class SessionController {

    @GetMapping("/session-info")
    public String sessionInfo(HttpServletRequest request) {
        HttpSession session = request.getSession(false);

        if (session == null) {
            return "세션이 없습니다.";
        }

        // 세션 데이터 출력
        session.getAttributeNames().asIterator()
                .forEachRemaining(name -> log.info("session name={}, value={}",
                        name, session.getAttribute(name)));

        log.info("sessionId={}", session.getId());
        log.info("maxInactiveInterval={}", session.getMaxInactiveInterval());
        log.info("creationTime={}", new Date(session.getCreationTime()));
        log.info("lastAccessedTime={}", new Date(session.getLastAccessedTime()));
        log.info("isNew={}", session.isNew());

        return "세션 출력";
    }

}
```

`sessionId` : 세션 Id, `JSESSIONID` 의 값입니다.. 
예) 34B14F008AA3527C9F8ED620EFD7A4E1

`maxInactiveInterval` : 세션의 유효 시간, 예) 1800초, (30분)

`creationTime` : 세션 생성일시

`lastAccessedTime`: 세션과 연결된 사용자가 최근에 서버에 접근한 시간. 
클라이언트에서 서버로 `sessionId` ( JSESSIONID )를 요청한 경우에 갱신됩니다.

`isNew` : 새로 생성된 세션인지, 아니면 이미 과거에 만들어졌고, 클라이언트에서 서버로`sessionId` ( JSESSIONID )를 요청해서 조회된 세션인지 여부.

### **세션 타임아웃 설정**

세션은 사용자가 로그아웃을 직접 호출해서 `session.invalidate()` 가 호출 되는 경우에 삭제된됩니다.

하지만 저를 포함한 대부분의 사용자는 로그아웃을 선택하지 않고, 그냥 웹 브라우저를 종료하지요.

이 경우의 문제는 HTTP가 비연결성(ConnectionLess)이므로 서버 입장에서는 해당 사용자가 웹 브라우저를 종료한 것인지 아닌지를 인식할 수 없습니다. 

따라서 서버에서 세션 데이터를 언제 삭제해야 하는지 판단하기가 어렵습니다.

이 경우 남아있는 세션을 무한정 보관하면 다음과 같은 문제가 발생할 수 있습니다.

세션과 관련된 쿠키( JSESSIONID )를 탈취 당했을 경우 오랜 시간이 지나도 해당 쿠키로 악의적인 요청을 할 수도 있습니다…

세션은 기본적으로 메모리에 생성되는데 메모리의 크기가 무한하지 않기 때문에 꼭 필요한 경우만 생성해서 사용해야 합니다. 그런데 만약 10만명의 사용자가 로그인하면 10만개의 세션이 생성되게 됩니다.

그래서 우리는 어느 시점에서는 세션이 종료되도록 해야 합니다.

### **세션의 종료 시점**

그렇다면 세션의 종료 시점을 어떻게 정하면 좋을까요? 

가장 단순하게 생각해보면, 세션 생성 시점으로부터 30분 정도로 잡으면 될 것 같습니다. 

그런데 문제는 30분이 지나면 세션이 삭제되기 때문에, 열심히 사이트를 돌아다니다가 30분 후에는 또 로그인을 해서 세션을 생성해야 합니다. 그러니까 30분 마다 계속 로그인해야 하는 번거로움이 발생합니다..

더 나은 대안은 세션 생성 시점이 아니라 사용자가 서버에 최근에 요청한 시간을 기준으로 30분 정도를 유지해주는 것입니다. 이렇게 하면 사용자가 서비스를 사용하고 있으면, 세션의 생존 시간이 30분으로 계속 늘어나게 됩니다. 따라서 30분 마다 로그인해야 하는 번거로움이 사라지지요.

`HttpSession` 은 이 방식을 사용합니다!!

### **세션 타임아웃 설정**

스프링 부트로 글로벌 설정

`application.properties`

```java
server.servlet.session.timeout=60 
```

**최소값은 60초**이다, 기본은 1800(30분)

(글로벌 설정은 분 단위로 잘려서 설정됩니다.. 60(1분), 90(1분), 120(2분), ...)

특정 세션 단위로 시간을 설정할 수도 있습니다.

```java
... 
session.setMaxInactiveInterval(1800); //1800초
...
```

`application.properties` 에서 설정한 것을 테스트 해봅시다. 테스터 아이디로 로그인을 한 후에 [http://localhost:8080/session-info](http://localhost:8080/session-info) 이 링크로 세션을 출력해보면 아래와 같은 로그를 확인할 수 있습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d96f28e8-418c-47e7-91fc-8ab5a26bbdc9/Untitled.png)

로그에서 maxInactiveInterval 이 60 인 것을 확인할 수 있지요!

이 상태에서 60초를 기다린 후 [localhost:8080](http://localhost:8080) 으로 들어가게 되면 아래처럼 세션이 종료되어 로그아웃이 된 상태가 됩니다!!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a8182cfc-ab32-4238-8a49-e6e383c70a41/Untitled.png)

### **세션 타임아웃 발생**

세션의 타임아웃 시간은 해당 세션과 관련된 `JSESSIONID` 를 전달하는 HTTP 요청이 있으면 현재 시간으로 다시 초기화 된다. 

이렇게 초기화 되면 세션 타임아웃으로 설정한 시간동안 세션을 추가로 사용할 수 있다.

`session.getLastAccessedTime()` : 최근 세션 접근 시간

`LastAccessedTime` 이후로 timeout 시간이 지나면, WAS 가 내부에서 해당 세션을 제거한다.

### **정리**

서블릿의 `HttpSession` 이 제공하는 타임아웃 기능 덕분에 세션을 안전하고 편리하게 사용할 수 있다.

실무에서 주의할 점은 **세션에는 최소한의 데이터만 보관해야 한다**는 점이다. 

**보관한 데이터 용량 * 사용자 수**로 세션의 메모리 사용량이 급격하게 늘어나서 장애로 이어질 수 있다. 추가로 세션의 시간을 너무 길게 가져가면 메모리 사용이 계속 누적 될 수 있으므로 적당한 시간을 선택하는 것이 필요하다. 기본이 30 분이라는 것을 기준으로 고민하면 된다.


# ======= 로그인 처리 2 =======

# 1. 서블릿 필터 - 소개

이전 글 프로젝트에서 이어서 진행됩니다.

### **공통 관심 사항**

요구사항을 보면 로그인 한 사용자만 상품 관리 페이지에 들어갈 수 있어야 합니다.

이전 글에서 로그인을 하지 않은 사용자에게는 상품 관리 버튼이 보이지 않기 때문에 문제가 없어 보입니다.

그런데 로그인 하지 않은 사용자도 ‘http://localhost:8080/items’ 이 URL 을 직접 호출하면 상품 관리 화면에 들어갈 수 있습니다!!

상품 관리 컨트롤러에서 로그인 여부를 체크하는 로직을 하나하나 작성하는 방법도 있겠지만, 등록, 수정, 삭제, 조회 등, 상품관리의 모든 컨트롤러 로직에서 공통으로 로그인 여부를 확인해야 하는 반복의 문제가 있습니다.

더 큰 문제는 향후 로그인과 관련된 로직이 변경될 때에 있습니다. 이 때 작성한 모든 로직을 다 수정해야 할 것입니다.

이렇게 애플리케이션 여러 로직에서 공통으로 관심이 있는 있는 것을 공통 관심사(cross-cutting
concern)라고 합니다. 여기서는 등록, 수정, 삭제, 조회 등등 여러 로직에서 공통으로 인증에 대해서 관심을 가지고 있습니다.

이러한 공통 관심사는 스프링의 **AOP** 로도 해결할 수 있지만, 웹과 관련된 공통 관심사는 **서블릿 필터** 또는 **스프링 인터셉터**를 사용하는 것이 더 좋습니다!

웹과 관련된 공통 관심사를 처리할 때는 **HTTP의 헤더나 URL의 정보들**이 필요한데, 서블릿 필터나 스프링 인터셉터는 `HttpServletRequest` 를 제공합니다.

### **서블릿 필터 소개**

필터는 서블릿이 지원하는 일종의 수문장입니다. 

필터의 특성은 아래와 같습니다.

**필터 흐름**

```java
HTTP 요청 -> WAS-> 필터 -> 서블릿 -> 컨트롤러
```

필터를 적용하면 위에 적은 순서처럼 필터가 호출된 다음에 서블릿이 호출됩니다. 그래서 모든 고객의 요청 로그를 남기는 요구사항이 있다면 필터를 사용하면 된다. 

참고로 필터는 특정 URL 패턴에 적용할 수 있습니다.  `/*` 이라고 하면 모든 요청에 필터가 적용되는 것입니다.

여기서 말하는 서블릿은 스프링을 사용하는 경우 스프링의 디스패처 서블릿으로 생각하면 된다.

**필터 제한**

```java
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러 //로그인 사용자
HTTP 요청 -> WAS -> 필터(적절하지 않은 요청이라 판단, 서블릿 호출X) //비 로그인 사용자
```

필터에서 적절하지 않은 요청이라고 판단하면 필터 선에서 요청을 막을 수도 있습니다. 그래서 로그인 여부를 체크하기에 딱 좋습니다.

**필터 체인**

```java
HTTP 요청 ->WAS-> 필터1-> 필터2-> 필터3-> 서블릿 -> 컨트롤러
```

필터는 체인으로 구성되는데, 중간에 필터를 자유롭게 추가할 수 있습니다. 예를 들어서 로그를 남기는 필터를 먼저 적용하고, 그 다음에 로그인 여부를 체크하는 필터를 만들 수 있습니다.

**필터 인터페이스**

```java
public interface Filter {

		public default void init(FilterConfig filterConfig) throws ServletException { }
		public void doFilter(ServletRequest request, ServletResponse response,
												FilterChain chain) throws IOException, ServletException;
		public default void destroy() {}
}
```

필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고, 관리합니다..

`init()`: 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.

`doFilter()`: 고객의 요청이 올 때 마다 해당 메서드가 호출된다. 필터의 로직을 구현하면 된다.

`destroy()`: 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.

# 2. 서블릿 필터 - 요청 로그

필터가 정말 수문장 역할을 잘 하는지 확인하기 위해 가장 단순한 필터인, 모든 요청을 로그로 남기는 필터 코드를 개발하고 적용해봅시다.

`LogFilter` - 로그 필터

```java
package hello.login.web.filter;

import lombok.extern.slf4j.Slf4j;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.util.UUID;

@Slf4j
public class LogFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("log filter init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        String uuid = UUID.randomUUID().toString();
        try {
            log.info("REQUEST [{}][{}]", uuid, requestURI);
            chain.doFilter(request, response);
        } catch (Exception e) {
            throw e;
        }finally {
            log.info("RESPONSE [{}][{}]", uuid, requestURI);
        }
    }

    @Override
    public void destroy() {
        log.info("log filter destroy");
    }
}
```

필터를 사용하려면 `Filter` 인터페이스를 구현해야 한다.

`doFilter(ServletRequest request, ServletResponse response, FilterChain chain)` - HTTP 요청이 오면 doFilter 가 호출된다.

ServletRequest request 는 HTTP 요청이 아닌 경우까지 고려해서 만든 인터페이스이다.

HTTP를 사용하면 `HttpServletRequest httpRequest = (HttpServletRequest) request;` 와 같이 다운 케스팅 하면 된다.

`String uuid = UUID.randomUUID().toString();`

HTTP 요청을 구분하기 위해 각 요청당 임의의 uuid 를 생성해둔다.

`log.info("REQUEST [{}][{}]", uuid, requestURI);`

uuid 와 requestURI 를 출력한다.

`chain.doFilter(request, response);`

이 부분이 가장 중요하다. 다음 필터가 있으면 필터를 호출하고, 필터가 없으면 서블릿을 호출한다.

만약 이 로직을 호출하지 않으면 다음 단계로 진행되지 않는다.

`WebConfig` - 필터 설정

```java
package hello.login;

import hello.login.web.filter.LogFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.servlet.Filter;

@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean logFilter(){
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();

        filterRegistrationBean.setFilter(new LogFilter());
        filterRegistrationBean.setOrder(1);
        filterRegistrationBean.addUrlPatterns("/*");
        return filterRegistrationBean;
    }

}
```

필터를 등록하는 방법은 여러가지가 있지만, 스프링 부트를 사용하면(우리의 경우) `FilterRegistrationBean` 을 사용해서 등록하면 된다.

`setFilter(new LogFilter())` : 등록할 필터를 지정한다.

`setOrder(1)` : 필터는 체인으로 동작한다. 따라서 순서가 필요하다. 낮을 수록 먼저 동작한다.

`addUrlPatterns("/*")` : 필터를 적용할 URL 패턴을 지정한다. 한번에 여러 패턴을 지정할 수 있다.

> **참고** - URL 패턴에 대한 룰은 필터도 서블릿과 동일하다. 자세한 내용은 서블릿 URL 패턴으로 검색해보면 알 수 있다.
> 

```java
@ServletComponentScan @WebFilter(filterName = "logFilter", urlPatterns = "/*")
```

위 코드로도 필터 등록이 가능하지만 이 때는 필터 순서 조절을 할 수가 없다. 그러므로 컴포넌트 스캔이 아닌 `FilterRegistrationBean` 으로 수동 빈 등록을 해줍니다.

### 실행

서버를 실행시킨 후 [localhost:8080](http://localhost:8080) 을 실행하면 아래 로그를 볼 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2734cf2a-e63c-4261-9e2a-7ab926aa4178/Untitled.png)

필터를 등록할 때 `urlPattern` 을 `/*` 로 등록했기 때문에 모든 요청에 해당 필터가 적용된다

> **참고 -** 실무에서 HTTP 요청시 같은 요청의 로그에 모두 같은 식별자를 자동으로 남기는 방법도 있습니다. 필요하다면 “ logback mdc “ 로 검색하여 참고합시다.
>

# 3. 서블릿 필터 - 인증 체크

그렇다면 이제 실제 인증 필터를 적용해 봅시다.  

로그인 되지 않은 사용자는 상품 관리 뿐만 아니라 미래에 개발될 페이지에도 접근하지 못하도록 합니다.

`LoginCheckFilter` - 인증 체크 필터

```java
package hello.login.web.filter;

import hello.login.web.SessionConst;
import lombok.extern.slf4j.Slf4j;
import org.springframework.util.PatternMatchUtils;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

@Slf4j
public class LoginCheckFilter implements Filter {
    private static final String[] whitelist = {"/", "/members/add", "/login", "/logout", "/css/*"};

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        HttpServletResponse httpResponse = (HttpServletResponse) response;

        try {
            log.info("인증 체크 필터 시작 {}", requestURI);

            if (isLoginCheckPath(requestURI)) {
                log.info("인증 체크 로직 실행 {}", requestURI);
                HttpSession session = httpRequest.getSession(false);
                if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
                    log.info("미인증 사용자 요청 {}", requestURI);

                    // 로그인으로 redirect
                    httpResponse.sendRedirect("/login?redirectURL= " + requestURI);
                    return; // 중요한 부분임. 미인증 사용자는 다음으로 진행하지 않고 끝내게 한다.
                }
            }

            chain.doFilter(request, response);
        } catch (Exception e) {
            throw e; // 예외 로깅 가능하지만 톰캣까지 예외를 보내주어야 함.
        } finally {
            log.info("인증 체크 필터 종료 {}", requestURI);
        }

    }

    // 화이트 리스트의 경우 인증 체크 X
    private boolean isLoginCheckPath(String requestURI) {
        return !PatternMatchUtils.simpleMatch(whitelist, requestURI);
    }

}
```

`whitelist = {”/”, “/member/add”, “/login”, “/logout”, “/css/*”};`

인증 필터를 적용해도 홈, 회원가입, 로그인 화면, css 같은 리소스에는 접근할 수 있어야 합니다.

이렇게 화이트 리스트 경로는 인증과 무관하게 항상 허용됩니다. 

화이트 리스트를 제외한 나머지 모든 경로에는 인증 체크 로직을 적용합니다..

`isLoginCheckPath(requestURI)`

화이트 리스트를 제외한 모든 경우에 인증 체크 로직을 적용한다.

`httpResponse.sendRedirect("/login?redirectURL=" + requestURI);`

미인증 사용자는 로그인 화면으로 리다이렉트 한다. 

그런데 로그인 이후에 다시 홈으로 이동해버리면, 원하는 경로를 다시 찾아가야 하는 불편함이 있다. 예를 들어서 상품 관리 화면을 보려고 들어갔다가 로그인 화면으로 이동하면, 로그인 이후에 다시 상품 관리 화면으로 들어가는 것이 좋다. 

이런 부분이 개발자 입장에서는 좀 귀찮을 수 있어도 사용자 입장으로 보면 편리한 기능이다. 

이러한 기능을 위해 현재 요청한 경로인 `requestURI` 를 `/login` 에 쿼리 파라미터로 함께 전달한다. 물론 `/login` 컨트롤러에서 로그인 성공시 해당 경로로 이동하는 기능은 추가로 개발해야 한다.

`return;` 여기가 중요하다. 필터를 더는 진행하지 않는다. 이후 필터는 물론 서블릿, 컨트롤러가 더는 호출되지 않는다. 앞서 `redirect` 를 사용했기 때문에 `redirect` 가 응답으로 적용되고 요청이 끝난다.

`WebConfig` - `loginCheckFilter()` 추가

```java
@Bean
public FilterRegistrationBean loginCheckFilter(){
    FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();

    filterRegistrationBean.setFilter(new LoginCheckFilter());
    filterRegistrationBean.setOrder(2);
    filterRegistrationBean.addUrlPatterns("/*");

    return filterRegistrationBean;
}
```

`setFilter(new LoginCheckFilter())` : 로그인 필터를 등록한다.

`setOrder(2)` : 순서를 2번으로 잡았다. 로그 필터 다음에 로그인 필터가 적용된다.

`addUrlPatterns("/*")` : 모든 요청에 로그인 필터를 적용한다.

### **RedirectURL 처리**

로그인에 성공하면 처음 요청한 URL로 이동하는 기능을 개발해봅시다.

`LoginController` - `loginV4()`

```java
@PostMapping("/login")
public String loginV4(
        @Valid @ModelAttribute LoginForm form, BindingResult bindingResult,
        @RequestParam(defaultValue = "/") String redirectURL,
        HttpServletRequest request
) {

    if (bindingResult.hasErrors()) {
        return "login/loginForm";
    }

    Member loginMember = loginService.login(form.getLoginId(), form.getPassword());
    log.info("login? {}", loginMember);

    if (loginMember == null) {
        bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
        return "login/loginForm";
    }

    // 로그인 성공 처리

    // 세션이 있으면 세션 반환. 없으면 신규 세션 생성
    HttpSession session = request.getSession();

    // 세션에 로그인 회원 정보 보관
    session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);

    return "redirect:" + redirectURL;
}
```

로그인 체크 필터에서, 미인증 사용자는 요청 경로를 포함해서 `/login` 에 `redirectURL` 요청 파라미터를 추가해서 요청했다. 이 값을 사용해서 로그인 성공시 해당 경로로 고객을 redirect 한다.

실행하여 확인해 봅시다. 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3e7fbdfe-9926-4fe0-847f-99223208b47a/Untitled.png)

처음 테스터 아이디로 로그인을 한 후에 session 이 만료될 때 까지 기다립니다.

session 이 종료된 후 (60초 후)에 상품 관리 탭을 누릅니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4627c0fb-4f0c-4f3e-b6ff-c1ec31205425/Untitled.png)

session 이 종료되어서 로그인 창으로 넘어가는 모습입니다.

URL이 [`http://localhost:8080/login?redirectURL=/items`](http://localhost:8080/login?redirectURL=/items) 로  변한게 보입니다!!

여기서 로그인을 하면 원래 이동하려던 상품 관리 창으로 자동으로 이동하게 됩니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/79e212e2-d07c-48c3-b136-b8029e90484d/Untitled.png)

여기에 다시 테스터 아이디로 로그인을 해봅니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e026fda4-7b20-48b5-8dfb-5384bea5fb9a/Untitled.png)

우리가 의도한 대로 잘 동작하는 것을 확인할 수 있습니다!

### **정리**

서블릿 필터를 잘 사용한 덕분에 로그인 하지 않은 사용자는 나머지 경로에 들어갈 수 없게 되었다.

공통 관심사를 서블릿 필터를 사용해서 해결한 덕분에 향후 로그인 관련 정책이 변경되어도 이 부분만 변경하면 됩니다.

**참고 -**  필터에는 다음에 설명할 스프링 인터셉터는 제공하지 않는, 아주 강력한 기능이 있는데 `chain.doFilter(request, response);` 를 호출해서 다음 필터 또는 서블릿을 호출할 때 `request` , `response` 를 다른 객체로 바꿀 수 있습니다. 

`ServletRequest` , `ServletResponse` 를 구현한 다른 객체를 만들어서 넘기면 해당 객체가 다음 필터 또는 서블릿에서 사용됩니다. 잘 사용하는 기능은 아니니 참고만 해둡시다.

# 4. 스프링 인터셉터 - 소개

스프링 인터셉터도 서블릿 필터와 같이 웹과 관련된 공통 관심 사항을 효과적으로 해결할 수 있는 기술이다. 

서블릿 필터가 서블릿이 제공하는 기술이라면, 스프링 인터셉터는 스프링 MVC가 제공하는 기술이다. 

둘다 웹과 관련된 공통 관심 사항을 처리하지만, 적용되는 순서와 범위, 그리고 사용방법이 다르다.

### 스프링 인터셉터 흐름

```java
HTTP 요청 -> WAS-> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러
```

스프링 인터셉터는 디스패처 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출 된다.

스프링 인터셉터는 스프링 MVC가 제공하는 기능이기 때문에 결국 디스패처 서블릿 이후에 등장하게 된다.

스프링 MVC의 시작점이 디스패처 서블릿이라고 생각해보면 이해가 될 것이다.

스프링 인터셉터에도 URL 패턴을 적용할 수 있는데, 서블릿 URL 패턴과는 다르고, 매우 정밀하게 설정할 수 있다.

### 스프링 인터셉터 제한

```java
// 로그인 사용자
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러 //로그인 사용자

// 비 로그인 사용자
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터(적절하지 않은 요청이라 판단, 컨트롤러 호출 X) 
```

인터셉터에서 적절하지 않은 요청이라고 판단하면 거기에서 끝낼 수도 있다. 그래서 로그인 여부를 체크하기에 딱 좋다.

### 스프링 인터셉터 체인

```java
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터1 -> 인터셉터2 -> 컨트롤러
```

스프링 인터셉터는 체인으로 구성되는데, 중간에 인터셉터를 자유롭게 추가할 수 있다. 

예를 들어서 로그를 남기는 인터셉터를 먼저 적용하고, 그 다음에 로그인 여부를 체크하는 인터셉터를 만들 수 있다.

지금까지 내용을 보면 서블릿 필터와 호출되는 순서만 다르고, 제공하는 기능은 비슷해 보인다. 앞으로 설명하겠지만, 스프링 인터셉터는 서블릿 필터보다 편리하고, 더 정교하고 다양한 기능을 지원한다.

### 스프링 인터셉터 인터페이스

스프링의 인터셉터를 사용하려면 `HandlerInterceptor` 인터페이스를 구현하면 된다.

```java
public interface HandlerInterceptor {

		default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
				throws Exception {
	
			return true;
		}
	
		default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
				@Nullable ModelAndView modelAndView) throws Exception {
		}
	
		default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
				@Nullable Exception ex) throws Exception {
		}

}
```

서블릿 필터의 경우 단순하게 `doFilter()` 하나만 제공되었습니다. 

반면에 인터셉터는 컨트롤러 호출 전( `preHandle` ), 호출 후( `postHandle` ), 요청 완료 이후( `afterCompletion` )와 같이 단계적으로 잘 세분화 되어 있습니다! 

서블릿 필터의 경우 단순히 `request` , `response` 만 제공했지만, 인터셉터는 어떤 컨트롤러( `handler` )가 호출되는지 호출 정보도 받을 수 있습니다. 그리고 어떤 `modelAndView` 가 반환되는지 응답 정보도 받을 수 있습니다.

### 스프링 인터셉터 호출 흐름

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8c0bafb7-b08f-4e96-ac12-62852e4affa4/Untitled.png)

**정상 흐름**

`preHandle` : 컨트롤러 호출 전에 호출된다. (더 정확히는 핸들러 어댑터 호출 전에 호출된다.)

`preHandle` 의 응답값이 `true` 이면 다음으로 진행하고, `false` 이면 더는 진행하지 않는다. `false` 인 경우 나머지 인터셉터는 물론이고, 핸들러 어댑터도 호출되지 않는다. 그림에서 1번에서 끝이 나버린다.

`postHandle` : 컨트롤러 호출 후에 호출된다. (더 정확히는 핸들러 어댑터 호출 후에 호출된다.)

`afterCompletion` : 뷰가 렌더링 된 이후에 호출된다.

### **스프링 인터셉터 예외 상황**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/730ada94-e92c-4efc-bc28-206bf805fb63/Untitled.png)

위 그림처럼 핸들러를 호출할 때 **예외가 발생시**

`preHandle` : 컨트롤러 호출 전에 호출된다.

`postHandle` : 컨트롤러에서 예외가 발생하면 `postHandle` 은 호출되지 않는다.

`afterCompletion` : `afterCompletion` 은 항상 호출된다. 이 경우 예외( ex )를 파라미터로 받아서 어떤 예외가 발생했는지 로그로 출력할 수 있다.

`afterCompletion`은 예외가 발생해도 호출된다.

예외가 발생하면 `postHandle()` 는 호출되지 않으므로 예외와 무관하게 공통 처리를 하려면
`afterCompletion()` 을 사용해야 한다.

예외가 발생하면 `afterCompletion()` 에 예외 정보( ex )를 포함해서 호출된다.

### **정리**

인터셉터는 스프링 MVC 구조에 특화된 필터 기능을 제공한다고 이해하면 된다. 

스프링 MVC를 사용하고, 특별히 필터를 꼭 사용해야 하는 상황이 아니라면 인터셉터를 사용하는 것이 더 편리하다.

# 5. 스프링 인터셉터 - 요청 로그

`LogInterceptor` - 요청 로그 인터셉터

```java
package hello.login.web.interceptor;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.UUID;

@Slf4j
public class LogInterceptor implements HandlerInterceptor {

    public static final String LOG_ID = "logId";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();

        String uuid = UUID.randomUUID().toString();
        request.setAttribute(LOG_ID, uuid);

        // @RequestMapping : HandlerMethod
        // 정적 리소스 : ResourceHttpRequestHandler
        if (handler instanceof HandlerMethod) {
            HandlerMethod handlerMethod = (HandlerMethod) handler; // 호출할 컨트롤러 메서드의 모든 정보가 포함되어 있음.
        }

        log.info("REQUEST [{}][{}][{}]", uuid, requestURI, handler);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandler [{}]", modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        String requestURI = request.getRequestURI();
        String logId = (String) request.getAttribute(LOG_ID);
        log.info("RESPONSE [{}][{}]", logId, requestURI);

        if (ex != null) {
            log.error("afterCompletion error!", ex);
        }
    }
}
```

코드를 천천히 살펴 봅시다.

`String uuid = UUID.randomUUID().toString()`

요청 로그를 구분하기 위한 uuid 를 생성한다.

`request.setAttribute(LOG_ID, uuid)`

서블릿 필터의 경우 지역변수로 해결이 가능하지만, 스프링 인터셉터는 호출 시점이 완전히 분리되어 있다. 따라서 `preHandle` 에서 지정한 값을 `postHandle` , `afterCompletion` 에서 함께 사용하려면 어딘가에 담아두어야 한다. 

`LogInterceptor` 도 싱글톤 처럼 사용되기 때문에 맴버변수를 사용하면 위험하다. 따라서 `request` 에 담아두었다. 

이 값은 `afterCompletion` 에서 `request.getAttribute(LOG_ID)` 로 찾아서 사용한다.

`return true`

`true` 면 정상 호출이다. 다음 인터셉터나 컨트롤러가 호출된다.

**HandlerMethod**

핸들러 정보는 어떤 핸들러 매핑을 사용하는가에 따라 달라진다. 스프링을 사용하면 일반적으로
`@Controller` , `@RequestMapping` 을 활용한 핸들러 매핑을 사용하는데, 이 경우 핸들러 정보로 `HandlerMethod` 가 넘어온다.

**ResourceHttpRequestHandler**

`@Controller` 가 아니라 `/resources/static` 와 같은 정적 리소스가 호출 되는 경우 `ResourceHttpRequestHandler` 가 핸들러 정보로 넘어오기 때문에 타입에 따라서 처리가 필요하다.

**postHandle, afterCompletion**

종료 로그를 `postHandle` 이 아니라 `afterCompletion` 에서 실행한 이유는, 예외가 발생한 경우 `postHandle` 가 호출되지 않기 때문이다. `afterCompletion` 은 예외가 발생해도 호출 되는 것을 보장한다.

`WebConfig` - 인터셉터 등록

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "/*.ico", "/error");
    }
		...
}
```

인터셉터와 필터가 중복되지 않도록 필터를 등록하기 위한 이전에 만든 `logFilter()` 의 `@Bean` 은 주석처리합니다.

`WebMvcConfigurer` 가 제공하는 `addInterceptors()` 를 사용해서 인터셉터를 등록할 수 있습니다. 코드를 간단히 뜯어봅시다.

`registry.addInterceptor(new LogInterceptor())` : 인터셉터를 등록한다.

`order(1)` : 인터셉터의 호출 순서를 지정한다. 낮을 수록 먼저 호출된다.

`addPathPatterns("/**")` : 인터셉터를 적용할 URL 패턴을 지정한다.

`excludePathPatterns("/css/**", "/*.ico", "/error")` : 인터셉터에서 제외할 패턴을 지정한다.

필터와 비교해보면 인터셉터는 `addPathPatterns` , `excludePathPatterns` 로 매우 정밀하게 URL 패턴을 지정할 수 있습니다!!

실행 결과 — 서버 실행 후 회원 가입을 눌러봅니다.

```java
REQUEST [75be8a57-a33f-41cf-a251-09efc12b6250]
	[/members/add][hello.login.web.member.MemberController#addForm(Member)]
postHandler [ModelAndView [view="members/addMemberForm"; 
model={member=Member(id=null, loginId=null, name=null, password=null), 
	org.springframework.validation.BindingResult.member
		=org.springframework.validation.BeanPropertyBindingResult: 0 errors}]]
RESPONSE [75be8a57-a33f-41cf-a251-09efc12b6250][/members/add]
```

**스프링의 URL 경로**

스프링이 제공하는 URL 경로는 서블릿 기술이 제공하는 URL 경로와 완전히 다르다. 더욱 자세하고, 세밀하게 설정할 수 있습니다.

자세한 내용은 아래를 참고하세요.

**PathPattern** 공식 문서

```java
? 한 문자 일치
* 경로(/) 안에서 0개 이상의 문자 일치
** 경로 끝까지 0개 이상의 경로(/) 일치
{spring} 경로(/)와 일치하고 spring이라는 변수로 캡처
{spring:[a-z]+} matches the regexp [a-z]+ as a path variable named "spring" {spring:[a-z]+} regexp [a-z]+ 와 일치하고, "spring" 경로 변수로 캡처
{*spring} 경로가 끝날 때 까지 0개 이상의 경로(/)와 일치하고 spring이라는 변수로 캡처
  /pages/t?st.html — matches /pages/test.html, /pages/tXst.html but not /pages/
  toast.html
  /resources/*.png — matches all .png files in the resources directory
  /resources/** — matches all files underneath the /resources/ path, including /
  resources/image.png and /resources/css/spring.css
  /resources/{*path} — matches all files underneath the /resources/ path and
  captures their relative path in a variable named "path"; /resources/image.png
  will match with "path" → "/image.png", and /resources/css/spring.css will match
  with "path" → "/css/spring.css"
  /resources/{filename:\\w+}.dat will match /resources/spring.dat and assign the
  value "spring" to the filename variable
```

스프링 사이트에서 더 자세하게 알아볼 수 있습니다.


# 6. 스프링 인터셉터 - 인증 체크

서블릿 필터에서 사용했던 인증 체크 기능을 스프링 인터셉터로 개발해보자.

`LoginCheckInterceptor`

```java
package hello.login.web.interceptor;

import hello.login.web.SessionConst;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

@Slf4j
public class LoginCheckInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String requestURI = request.getRequestURI();

        log.info("인증 체크 인터셉터 실행 {}", requestURI);
        HttpSession session = request.getSession(false);

        if (session == null
                || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
            log.info("미인증 사용자 요청");

            // 로그인으로 redirect
            response.sendRedirect("/login?redirectURL=" + requestURI);
            return false;
        }
        return true;
    }

}
```

서블릿 필터와 비교해서 코드가 매우 간결하다. 인증이라는 것은 컨트롤러 호출 전에만 호출되면 된다. 따라서 `preHandle` 만 구현하면 된다.

**순서 주의, 세밀한 설정 가능**

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "/*.ico", "/error");

        registry.addInterceptor(new LoginCheckInterceptor())
                .order(2)
                .addPathPatterns("/**")
                .excludePathPatterns(
                        "/", "/members/add", "/login", "/logout",
                        "/css/**", "/*.ico", "/error"
                );
    }
		...
}
```

인터셉터와 필터가 중복되지 않도록 이전에 작성했던 필터를 등록하기 위한 메서드 `logFilter()` , `loginCheckFilter()` 의 `@Bean` 은 주석처리합니다.

인터셉터를 적용하거나 하지 않을 부분은 `addPathPatterns` 와 `excludePathPatterns` 에 작성하면 됩니다. 

기본적으로 모든 경로에 해당 인터셉터를 적용하되 ( `/**` ), 홈( `/` ), 회원가입( `/members/add` ), 로그인( `/login` ), 리소스 조회( `/css/**` ), 오류( `/error` )와 같은 부분은 로그인 체크 인터셉터를 적용하지 않습니다. 

실행해보면 이전 포스팅에서 필터를 사용했던 것과 같이 작동합니다. 물론 매커니즘은 조금 다르지요. 중요한 점은 서블릿 필터와 비교해보면 매우 편리한 것을 알 수 있습니다!

### **정리**

서블릿 필터와 스프링 인터셉터는 웹과 관련된 공통 관심사를 해결하기 위한 기술입니다.

서블릿 필터와 비교해서 스프링 인터셉터가 개발자 입장에서 훨씬 편리하다는 것을 코드로 이해했을 것입니다. 

특별한 문제가 없다면 인터셉터를 사용하는 것이 좋습니다!!!!

# 7. ArgumentResolver 활용

우리는 이전에 `ArgumentResolver` 를 학습했습니다. 바로 ****6. 스프링 MVC - 기본 기능 - 스프링 MVC 1편**** ([https://sh1mj1-log.tistory.com/60](https://sh1mj1-log.tistory.com/60) ) 에서 배웠습니다.

이번 시간에는 해당 기능을 사용해서 로그인 회원을 조금 편리하게 찾아봅시다.

`HomeController` - 추가

```java

@GetMapping("/")
public String homeLoginV3ArgumentResolver(@Login Member loginMember, Model model) {

    // 세션에 회원 데이터가 없으면 home
    if (loginMember == null) {
        return "home";
    }

    // 세션이 유지되면 로그인으로 이동
    model.addAttribute("member", loginMember);
    return "loginHome";

}
```

현재 이 상태에서는 컴파일 오류가 발생할 것입니다. 아직 `@Login`에 대한 정보가 없기 때문이죠.

즉, `@Login` 애노테이션은 직접 만들어야 합니다.

`@Login` 애노테이션이 있으면 직접 만든 `ArgumentResolver` 가 동작해서 자동으로 세션에 있는 로그인 회원을 찾아주고, 만약 세션에 없다면 `null` 을 반환하도록 개발해봅시다.

**@Login 애노테이션 생성**

```java
package hello.login.web.argumentresolver;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Login {

}
```

`@Target(ElementType.PARAMETER)` : 파라미터에만 사용

`@Retention(RetentionPolicy.RUNTIME)` : 리플렉션 등을 활용할 수 있도록 런타임까지 애노테이션 정보가 남아있음

MVC1에서 학습한 `HandlerMethodArgumentResolver` 를 구현해봅시다.

```java
package hello.login.web.argumentresolver;

import hello.login.web.SessionConst;
import hello.login.web.member.Member;
import lombok.extern.slf4j.Slf4j;
import org.springframework.core.MethodParameter;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

@Slf4j
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        log.info("supportsParameter 실행");

        boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);

        boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());

        return hasLoginAnnotation && hasMemberType;
    }

    @Override
    public Object resolveArgument(
            MethodParameter parameter, ModelAndViewContainer mavContainer, 
            NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {

        log.info("resolverArgument 실행");

        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
        HttpSession session = request.getSession(false);

        if (session == null) {
            return null;
        }

        return session.getAttribute(SessionConst.LOGIN_MEMBER);

    }
}
```

`supportsParameter()` : `@Login` 애노테이션이 있으면서 `Member` 타입이면 해당 ArgumentResolver 가 사용된다.

`resolveArgument()` : 컨트롤러 호출 직전에 호출 되어서 필요한 파라미터 정보를 생성해준다.

여기서는 세션에 있는 로그인 회원 정보인 `member` 객체를 찾아서 반환해준다. 

이후 스프링MVC는 컨트롤러의 메서드를 호출하면서 여기에서 반환된 `member` 객체를 파라미터에 전달해준다.

`WebMvcConfig` 에 설정 추가

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new LoginMemberArgumentResolver());
    }
```

앞서 개발한 `LoginMemberArgumentResolver` 를 등록합니다.

**실행**

실행해보면, 결과는 동일하지만, 더 편리하게 로그인 회원 정보를 조회할 수 있다. 

이렇게 `ArgumentResolver` 를 활용하면 공통 작업이 필요할 때 컨트롤러를 더욱 편리하게 사용할 수 있습니다.


