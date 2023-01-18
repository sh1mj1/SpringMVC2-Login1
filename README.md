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