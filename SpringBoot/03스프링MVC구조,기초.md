# SpringMVC 구조

![image](https://user-images.githubusercontent.com/65898555/178188819-76470b03-74fe-4e42-ba71-3ffb1db96f41.png)

스프링 MVC는 프론트 컨트롤러 패턴으로 구현되어 있다. 스프링 MVC의 프론트 컨트롤러가 바로 디스패처 서블릿(DispatcherServlet)이다. 그리고 이 디스패처 서블릿이 바로 스프링 MVC의 핵심이다.

동작 순서
1. 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다.
4. 핸들러 실행: 핸들러 어댑터가 실제 핸들러를 실행한다.
5. ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
6. viewResolver 호출: 뷰 리졸버를 찾고 실행한다. JSP의 경우: InternalResourceViewResolver 가 자동 등록되고, 사용된다.
7. View 반환: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다. JSP의 경우 InternalResourceView(JstlView) 를 반환하는데, 내부에 forward() 로직이 있다.
8. 뷰 렌더링: 뷰를 통해서 뷰를 렌더링 한다.


# 스프링 MVC 시작

스프링은 애노테이션을 활용한 매우 유연하고, 실용적인 컨트롤러를 만들었는데 이것이 바로 @RequestMapping 애노테이션을 사용하는 컨트롤러이다. 
과거에는 스프링 프레임워크가 MVC 부분이 약해서 스프링을 사용하더라도 MVC 웹 기술은 스트럿츠 같은 다른 프레임워크를 사용했었다. 그런데 @RequestMapping 기반의 애노테이션 컨트롤러가
등장하면서, MVC 부분도 스프링의 완승으로 끝이 났다.

@RequestMapping
- RequestMappingHandlerMapping
- RequestMappingHandlerAdapter

가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터는 RequestMappingHandlerMapping, RequestMappingHandlerAdapter 이다.
@RequestMapping 의 앞글자를 따서 만든 이름인데, 이것이 바로 지금 스프링에서 주로 사용하는 애노테이션 기반의 컨트롤러를 지원하는 핸들러 매핑과 어댑터이다. 
실무에서는 99.9% 이 방식의 컨트롤러를 사용한다.


```java
import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Controller
public class SpringMemberSaveControllerV1 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @RequestMapping("/springmvc/v1/members/save")
  public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    Member member = new Member(username, age);
    System.out.println("member = " + member);
    
    memberRepository.save(member);
    
    ModelAndView mv = new ModelAndView("save-result");
    mv.addObject("member", member);
    
    return mv;
  }
}
```
**@Controller**

스프링이 자동으로 스프링 빈으로 등록한다. (내부에 @Component 애노테이션이 있어서 컴포넌트 스캔의 대상이 됨)
스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다.

**@RequestMapping** 

요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출된다. 애노테이션을 기반으로 동작하기 때문에, 메서드의 이름은 임의로 지으면 된다.

**ModelAndView** 

모델과 뷰 정보를 담아서 반환하면 된다.
스프링이 제공하는 ModelAndView 를 통해 Model 데이터를 추가할 때는 addObject() 를 사용하면 된다. 이 데이터는 이후 뷰를 렌더링 할 때 사용된다.


RequestMappingHandlerMapping 은 스프링 빈 중에서 @RequestMapping 또는 @Controller가 클래스 레벨에 붙어 있는 경우에 매핑 정보로 인식한다.

```java
/**
 * 클래스 단위 -> 메서드 단위
 * @RequestMapping 클래스 레벨과 메서드 레벨 조합
 */
@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  
  @RequestMapping("/new-form")
  public ModelAndView newForm() {
    return new ModelAndView("new-form");
  }
  
  @RequestMapping("/save")
  public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    Member member = new Member(username, age);
    memberRepository.save(member);
    ModelAndView mav = new ModelAndView("save-result");
    mav.addObject("member", member);
    return mav;
  }
  
  @RequestMapping
  public ModelAndView members() {
    List<Member> members = memberRepository.findAll();
    ModelAndView mav = new ModelAndView("members");
    mav.addObject("members", members);
    return mav;
  }
}
```
클래스 레벨 @RequestMapping("/springmvc/v2/members")

메서드 레벨 @RequestMapping("/new-form") -> /springmvc/v2/members/new-form

메서드 레벨 @RequestMapping("/save") -> /springmvc/v2/members/save

메서드 레벨 @RequestMapping -> /springmvc/v2/members



```java
/**
 * v3
 * Model 도입
 * ViewName 직접 반환
 * @RequestParam 사용
 * @RequestMapping -> @GetMapping, @PostMapping
 */
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  
  @GetMapping("/new-form")
  public String newForm() {
    return "new-form";
  }
  
  @PostMapping("/save")
  public String save( @RequestParam("username") String username, @RequestParam("age") int age, Model model) {
    Member member = new Member(username, age);
    memberRepository.save(member);
    model.addAttribute("member", member);
    return "save-result";
  }
  
  @GetMapping
  public String members(Model model) {
    List<Member> members = memberRepository.findAll();
    model.addAttribute("members", members);
    return "members";
  }
}
```
ViewName 직접 반환

@RequestParam 사용 : 스프링은 HTTP 요청 파라미터를 @RequestParam 으로 받을 수 있다. GET 쿼리 파라미터, POST Form 방식을 모두 지원한다.

@RequestMapping @GetMapping, @PostMapping : @RequestMapping(value = "/new-form", method = RequestMethod.GET)을 @GetMapping , @PostMapping 으로 더 편리하게 사용. 참고로 Get, Post, Put, Delete, Patch 모두 애노테이션이 준비되

