---
title: "[Spring_Security] 애플리케이션 수준의 보안을 책임지는 스프링 시큐리티의 각 구성요소와 맞춤형 빈 및 설정"
date: 2024-06-09 15:14 +0900
lastmod: 2024-06-09 15:14 +0900
categories: Spring_Security
tags:
  [
    Authentication,
    Authorization,
    XSS,
    CSRF,
    SecurityContext,
    UsernamePasswordAuthenticationToken,
    UserDetails,
    SecurityFilterChain,
    AbstractHttpConfigurer
  ]
---

## 인증 / 인가의 정의 및 취약성

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 인증, 인가란
</div>

**인증`Authentication`**이란 애플리케이션이 <span style='color:rgb(196,58,26);'>사용자를 식별</span>하는 방법이다.  
즉, 서버 리소스의 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">호출자를 식별하는 프로세스</span>를 말한다.

**인가`Authorization`**란 식별된 호출자가 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">요청된 리소스에 Access할 권한이 있는지 결정하는 프로세스</span>를 말한다.

**☄️ 인증 취약성(Vulnerability)**  
if, 애플리케이션 서버가 사용자의 로그인 여부만을 확인하고, 사용자마다 역할을 주지 않는다면  
악의를 가진 사용자가 다른 사용자의 정보를 검색할 수 있다.  
즉, <span style='color:rgb(196,58,26);'>특정 역할을 가진 사용자</span>만이 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#BCD4E6;">특정 엔드포인트에 접근하도록 정의</span>해야 한다.

- 세션 고정의 취약성  
  : 이미 생성된 세션ID를 이용해 유효한 사용자를 가장할 수 있다.  
  : 애플리케이션이 인증 프로세스 중 교유한 세션ID를 할당하지 않고, 기존 세션ID를 재사용할 때 발생한다.

- 스크립트의 취약성, `XSS`  
  : <span style='color:rgb(196,58,26);'>클라이언트쪽에 악의적인 스크립트를 주입</span>해 다른 사용자가 이를 실행하도록 하는 공격이다.

- 쿠키의 취약성, `XSS`  
  : <span style='color:rgb(196,58,26);'>세션값을 쿠키에 저장</span>하는 경우, <span style='color:rgb(196,58,26);'>스크립트를 주입</span>해 피해자의 브라우저가 스크립트를 실행하도록 할 수 있다.

- 출처 미확인의 취약성, `CSRF`  
  : 특정 서버에서 <span style='color:rgb(196,58,26);'>작업을 호출하는 URL을 추출</span>하여, 외부에서 사용할 수 있을 때  
  : 서버가 요청의 <span style='color:rgb(196,58,26);'>출처를 확인하지 않고 실행</span>하면, 서버쪽에서 의도치 않은 동작을 실행할 수 있다.

- 쿼리문의 취약성, `SQL Injection`  
  : 쿼리를 통해 <span style='color:rgb(196,58,26);'>시스템의 데이터를 변경, 삭제 및 추출</span>할 수 있다.

- 응답 정보 노출의 취약성  
  : <span style='color:rgb(196,58,26);'>로그</span>를 통해 공개정보가 아닌 민감한 데이터를 노출시켜서는 안된다.  
  : <span style='color:rgb(196,58,26);'>서버가 클라이언트에 반환하는 정보</span>에 의해서 실행 컨텍스트를 분석할 수 있다. 반환하는 응답으로 입력이 무엇인지 추측하게 해서는 안된다.

## HTTP Basic

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 HTTP Basic을 이용하려면 모든 호출마다 자격증명을 전송해야 한다
</div>

<span style='color:rgb(196,58,26);'>HTTP Basic</span>이란 클라이언트와 서버 간의 통신에서 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">사용자 인증을 위해 사용되는 간단한 인증 방법</span>이다.  
이 때 모든 호출에 보내는 자격증명은 암호화되지 않는다.  
때문에, **Base64 Encoding**을 통해 <span style='color:rgb(196,58,26);'>암호화한 자격증명(사용자 이름, 암호)</span>를 전송한다.

```text
Using generated security password: b5e073a3-440e-4cf2-aaf9-438275185c3c
```

애플리케이션을 실행할 때마다 다음과 같은 암호가 생성된다.  
HTTP Basic 인증으로 애플리케이션의 엔드포인트를 호출하려면 이 암호를 사용해야 한다.

헤더에 <span style='color:#9E0ADD;'>암호화된 자격증명이 노출</span>되고, 이는 **Base64 Decoding**을 통해 <span style='color:#9E0ADD;'>복호화</span>할 수 있기 때문에,  
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:rgba(193,151,210,0.7);">자격증명의 기밀성을 보장하지 않는다.</span>

HTTPS를 함께 이용할 때가 아니면, <span style='color:rgb(196,58,26);'>HTTP Basic 인증은 이용하지 않는다.</span>

## 스프링 시큐리티 (Spring Security)

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(35,43,47);color:white;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐁 스프링 시큐리티는 애플리케이션 수준의 보안이다.
</div>

⚠️ 스프링 시큐리티는 `스프링 컨테이너🥥`를 관리한다.

<span style='color:rgb(196,58,26);'>사용자 요청을 가로채서 식별</span>하고, <span style='color:rgb(196,58,26);'>권한을 가진 사용자</span>만이 보호된 리소스에 접근할 수 있도록 스프링 시큐리티 구성요소를 작성해야 한다.

## 스프링 시큐리티 구성요소

<div style="margin-bottom:15px;font-size:20px;background-color:#652DC1;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐙 스프링 시큐리티 구성요소
</div>

⚠️ 스프링 시큐리티의 각 구성요소가 관리하는 <span style='color:rgb(196,58,26);'>인증단계에 따라</span> 특정 인증객체 `Authentication`를 만든다.

![Alt text](https://i.esdrop.com/d/f/OAHra5CzfD/zzNQnM8wVT.png "Optional title"){: width="800" height="500"}

우선, 인증필터 `Authentication Filter`가 <span style='color:#9E0ADD;'>사용자 요청을 가로챈다.</span>  
인증 관리자 `Authentication Manager`에게 인증 책임이 위임된다.  
그러면 인증 관리자는 인증 공급자 `Authentication Provider`를 이용하여 <span style='color:rgb(196,58,26);'>인증 논리를 구현</span>한다.  
인증 공급자는 <span style='color:rgb(196,58,26);'>사용자를 찾고 암호인코더로 암호를 검증</span>하는 인증 논리가 정의되어 있다.  
검증이 완료되면, <span style='color:#9E0ADD;'>인증결과가 인증필터에 반환</span>되고  
인증된 엔티티에 관한 세부정보가 보안 컨텍스트 `Security Context`에 저장된다.  
보안 컨텍스트는 인증 프로세스 이후 <span style='color:#9E0ADD;'>인증데이터를 유지</span>한다.  
인증필터는 <span style='color:#9E0ADD;'>컨트롤러에게 보안컨텍스트를 위임</span>하고, 컨트롤러는 보안컨텍스트에 있는 세부정보를 이용할 수 있다.

- <span style="font-size: 16.48px;font-weight:bold;">사용자 이름/암호 인증 토큰</span> **`UsernamePasswordAuthenticationToken`**

- 인증관리자 `Authentication Manager`  
  : HTTP Filter 계층에서 <span style='color:rgb(196,58,26);'>사용자 요청을 수신</span>하여, 책임을 <span style='color:rgb(196,58,26);'>인증공급자에 위임</span>한다.

- 인증공급자 `Authentication Provider`  
  : <span style='color:rgb(196,58,26);'>인증 논리를 정의</span>하고, <span style='color:rgb(196,58,26);'>사용자와 암호의 관리를 위임</span>한다.
  : <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);">⚠️ 인증 객체를 정의한 다음 인증공급자를 구현한다.</span>  
  : 인증공급자가 구현하는 인증논리는 `AuthenticationServerProxy`를 통해 인증서버를 호출한다.

- 보안컨텍스트 `Security Context`  
  : 인증관리자는 인증 프로세스 완료 후, 요청이 유지되는 동안 <span style='color:rgb(196,58,26);'>인증객체를 저장</span>하는데, 이를 <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">저장하는 인스턴스를 '보안 컨텍스트'</span>라 한다.

- 계약구현을 나타내는 객체 `UserDetailsService`  
  : 사용자의 고유값으로 <span style='color:rgb(196,58,26);'>사용자를 찾는 역할만</span> 한다.  
  : 또한, **사용자의 세부정보를 관리**한다.

- 시큐리티 아케텍처와 연결할 사용자 프로토타입을 나타내는 인터페이스 `UserDetails`  
  : UserDetails는 하나 이상의 권한을 가진다.  
  : 사용자에게 허가된 권한은 `GrantedAuthority` 인터페이스로 나타낸다.

- 시큐리티 사용자를 나타내는 객체 `User`  
  : `org.springframework.security.core.userdetails` 패키지에 있으며 <span style='color:rgb(196,58,26);'>사용자를 나타내는 객체를 만드는 빌더 클래스</span>이다.  
  : `User`클래스로 <span style='color:rgb(196,58,26);'>UserDetails의 변경이 불가능한 인스턴스</span>를 만들 수 있다.

  ```java
  UserDetails u = User.withUsername("peter")
      .password("12345")
      .authorities("read", "write")
      .accountExpired(false)
      .disabled(true)
      .build();
  ```

- 인증 요청 이벤트를 나타내는 인터페이스 `Authentication`  
  : 접근 요청한 엔티티의 세부정보를 담는다.  
  : <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);">⚠️ Authentication extends Principal</span>  
  : `Authentication` 인스턴스의 <span style="margin-bottom:16px;font-size:15px;background-color:#99CCFF;color:white;border-radius:5px;padding:2px 3px;">getName()</span> 메서드는 `Principal` 인터페이스에서 상속받은 것이다.  
  : `Authentication` 인터페이스는 `Principal`을 확장한 것이다.  
  : <span style="margin-bottom:16px;font-size:15px;background-color:#99CCFF;color:white;border-radius:5px;padding:2px 3px;">getCredentials()</span> <span style="margin-bottom:16px;font-size:15px;background-color:#99CCFF;color:white;border-radius:5px;padding:2px 3px;">getAuthorities()</span> <span style="margin-bottom:16px;font-size:15px;background-color:#99CCFF;color:white;border-radius:5px;padding:2px 3px;">getDetails()</span> <span style="margin-bottom:16px;font-size:15px;background-color:#99CCFF;color:white;border-radius:5px;padding:2px 3px;">getIsAuthenticated()</span>  
  : <span style="margin-bottom:16px;font-size:15px;background-color:#99CCFF;color:white;border-radius:5px;padding:2px 3px;">getAuthorities()</span>: 인증된 요청에 허가된 권한의 컬렉션 반환

  > 🍍 Principal  
  > : 애플리케이션에 접근을 요청하는 사용자를 나타내는 인터페이스  
  > : 최소한 이름이 있어야 한다, getName()

  `🛡️ Spring_Security, Authentication.java`

  ```java
  public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    Object getCredentials(); // 인증에 이용된 암호나 비밀을 반환
    Object getDetails();
    Object getPrincipal();
    boolean isAuthenticated(); // 인증 프로세스가 끝났으면 true, 진행중이면 false
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
  }
  ```

## 맞춤형 빈 재정의

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🏈 애플리케이션 상황에 따라 자동구성 빈을 상속받아 맞춤형 빈을 정의해서 기본 구성요소를 재정의할 수 있다
</div>

🦕 맞춤형 인증토큰 구축, `UsernamePasswordAuthenticationToken`

```java
public class CustomUsernamePasswordAuthenticationToken extends UsernamePasswordAuthenticationToken {
  public CustomUsernamePasswordAuthenticationToken( // Authentication 객체가 인증되었다, 인증 프로세스 완료
    Object principal,
    Object credentials,
    Collection<? extends GrantedAuthority> authorities) {
      super(principal, credentials, authorities);
  }

  public CustomUsernamePasswordAuthenticationToken( // 인증 인스턴스가 인증되지 않은 상태로 유지, 인증관리자는 요청을 인증할 올바를 인증공급자를 찾으려고 한다
    Object principal,
    Object credentials) {
      super(principal, credentials);
  }
}
```

---

🦕 맞춤형 계약 구축, `UserDetailsService`  
⚠️ `UserDetailsService`의 기본 구현을 대체할 때는 `PasswordEncoder`도 지정해야한다.  
⚠️ 인스턴스를 만들 때는, <span style='color:rgb(196,58,26);'>사용자의 이름, 암호, 하나 이상의 권한을 지정</span>해야 한다.

`🛡️ Spring_Security, UserDetailsService.java`

```java
public interface UserDetailsService {
  // 주어진 사용자 이름을 가진 사용자의 세부정보를 얻는다.
  UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

`☕ InMemoryUserDetailsService.java`

```java
public class InMemoryUserDetailsService implements UserDetailsService {
  private final List<UserDetails> users;

  public InMemoryUserDetailsService(List<UserDetails> users) {
    this.users = users;
  }

  @Override
  public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    return users.stream()
      .filter(
        u -> u.getUsername().equals(username)
      )
      .findFirst()
      .orElseThrow(
        () -> new UsernameNotFoundException("User not found.");
      );
  }
}
```

사용자의 이름이 존재하지 않으면 `UsernameNotFoundException`을 투척한다.  
<span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);">⚠️ UsernameNotFoundException extends AuthenticationException</span>

`InMemoryUserDetailsManager`라는 구현을 이용할 수도 있다.  
: 메모리에 자격증명을 저장해서, 스프링 시큐리티가 요청을 인증할 때 이용할 수 있게 한다.

`🫎 ProjectConfig.java`

```java
@Configuration
public class ProjectConfig {
  @Bean
  public UserDetailsService userDetailsService() {
    var userDetailsService = new InMemoryUserDetailsManager();

    var user = User.withUsername("peter")
      .password("12345")
      .authorities("read")
      .build();

    userDetailsService.createUser(user); // UserDetailsService에서 관리하도록 사용자 추가
    return userDetailsService;
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance(); // 해시를 적용하지 않고, 일반 텍스트처럼 처리, 테스트용
  }
}
```

`PasswordEncoder`가 없으면 엔드포인트를 호출할 때, 예외발생한다.  
스프링 시큐리티가 암호를 관리하는 방법을 모른다고 인식하기 때문이다.

```text
java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"
```

`authorities`는 해당 사용자에게 허가된 작업이며, 일단 아무 문자열이나 지정하면 된다.

---

🦕 맞춤형 사용자 프로토타입 구축, `UserDetails`  
: `UserDetails` 계약을 구현하고 이를 이용해, 사용자를 스프링 시큐리티 아키텍처에 연결한다.

`🛡️ 스프링시큐리티, UserDetails.java`

```java
public interface UserDetails extends Serializable {
  // 사용자의 자격증명 반환
  String getUsername();
  String getPassword();

  // 권한부여, (사용자에게 부여된 )권한이 그룹을 반환
  Collection<? extends GrantedAuthority> getAuthorities();

  // 사용자 계정 활성화 or 비활성화
  boolean isAccountNonExpired(); // 계정만료 여부
  boolean isAccountNonLocked(); // 계정잠금 여부
  boolean isCredentialsNonExpired(); // 자격증명 만료 여부
  boolean isEnabled(); // 비활성화 여부
}
```

`☕ SecurityUser.java`

```java
public class SecurityUser implements UserDetails {
  private final User user; // User엔티티가 반드시 필요하다

  public SecurityUser(User user) {
    this.user = user;
  }

  @Override
  public String getUsername() {
    return user.getUsername();
  }

  @Override
  public String getPassword() {
    return user.getPassword();
  }

  @Override
  public Collection<? extends GrantedAuthority> getAuthorities() {
    return List.of(() -> user.getAuthority());
  }

  //...(생략)
}
```

---

🦕 UserDetailsService 계약 확장 `UserDetailsManager`  
: 사용자 로드뿐만 아니라 사용자 추가, 수정, 삭제 작업을 추가

`🛡️ Spring_Security, UserDetailsManager.java`

```java
public interface UserDetailsManager extends UserDetailsService {
  void createUser(UserDetails user);
  void updateUser(UserDetails user);
  void deleteUser(String username);
  void changePassword(String oldPassword, String newPassword);
  boolean userExists(String username);
}
```

---

🦕 맞춤형 인증공급자 구축, `AuthenticationProvider`
: <span style='color:rgb(196,58,26);'>인증논리</span>를 구현  
: `UserDetailsService` + `PasswordEncoder`를 맞춤 구성하는 방식

⚠️ 인증관리자는 인증공급자 중 하나에 인증을 위임한다.  
⚠️ 인증공급자는 특정 인증유형을 지원하지 않거나, 인증유형을 지원하지만 인증하는 방법을 모를 수 있다.

`🛡️ Spring_Security, AuthenticationProvider.java`

```java
public interface AuthenticationProvider {
  // 인증논리 정의, 지원되지 않는 인증객체를 받으면 null 반환
  Authentication authenticate(Authentication authentication) throws AuthenticationException;

  // 지원하는 인증 유형
  boolean supports(Class<?> authentication);
}
```

`☕ CustomAuthenticationProvider.java`

```java
@Component // spring에서 관리하는 컨테이너에 포함되도록
public class CustomAuthenticationProvider implements AuthenticationProvider {
  @Autowired
  private UserDetailsService userDetailsService;

  @Autowired
  private PasswordEncoder passwordEncoder;

  @Override
  public Authentication authenticate(Auhtentication authentication) {
    String username = authentication.getName();
    String password = authentication.getCredentials().toString();

    UserDetails u = userDetailsService.loadUserByUsername(username);

    if(passwordEncoder.matches(password, u.getPassword())) {
      return new UsernamePasswordAuthenticationToken(
          username,
          password,
          u.getAuthorities()
        );
    } else {
      throw new BadCredentialsException("Something went wrong!!");
    }
  }

  @Override
  public boolean supports(Class<?> authenticationType) {
    return authenticationType.equals(UsernamePasswordAuthenticationToken.class);
  }
}
```

<span style="font-size:17px;">🍪 supports 메서드에서 `AuthenticationProvider`가 `Authentication`객체로 제공된 형식을 지원하면 true를 반환한다.<span style='color:rgb(196,58,26);font-weight:bold;'>\* </span><span style='color:rgb(196,58,26);font-weight:bold;'>\* </span></span>

- `UserDetails u = userDetailsService.loadUserByUsername(username);`  
  : ⚠️ 사용자가 존재하지 않으면, `AuthenticationException` 투척, 401 권한없음

- **`BadCredentialsException extends AuthenticationException`**

- `UsernamePasswordAuthenticationToken implements Authentication`  
  : 사용자의 이름과 암호를 이용하는 <span style='color:rgb(196,58,26);'>표준 인증 요청</span>

`🫎 ProjectConfig.java`  
<span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);">⚠️ 구성클래스에 AuthenticationProvider 등록</span>

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
  @Autowired
  private AuthenticationProvider authenticationProvider;

  @Override
  protected void configure(AuthenticationManagerBuilder auth) {
    auth.authenticationProvider(authenticationProvider);
  }
}
```

## SecurityFilterChain

<div style="margin-bottom:15px;font-size:20px;background-color:#9E0ADD;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 WebSecurityConfigurerAdapter 클래스는 완전히 사라졌다
</div>

스프링 시큐리티에서는 <span style="margin-bottom:16px;font-size:15px;background-color:#E1FEE5;color:rgb(45,204,112);font-weight:bold;border-radius:5px;padding:2px 3px;">SecurityFilterChain</span>을 이용한 설정을 권장하고 있다.

```java
@Configuration
public class ProjectConfig {
  //...(생략)

  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeRequests(authz ->
      authz.anyRequest().permitAll() // 모든 요청에 인증없이 요청할 수 있다
      //anyRequest().authenticated() 모든 요청에 인증이 필요하다
      return http.build();
    );
  }
}
```

## 역할과 권한

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 사용자가 서버 리소스로 수행할 수 있는 작업을 역할 또는 권한이라 한다
</div>

`GrantedAuthority` 인터페이스로 <span style='color:rgb(196,58,26);'>권한</span> 또는 <span style='color:rgb(196,58,26);'>역할</span>을 나타낼 수 있는데
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">사용자 권한에 따라 모든 엔드포인트에 Access 규칙을 적용하는 것을 인가</span>라고 한다.

**역할**은 <span style='color:rgb(196,58,26);'>권한을 그룹화</span>한 것이므로, 역할은 권한의 개념보다 크다.  
즉, 역할이란 <span style='color:rgb(196,58,26);'>작업그룹에 속한 이용권리를 제공</span>하는 것으로써
역할을 이용하면 권한을 정의할 필요가 없다.

역할을 정의할 때 이름은 **`ROLE_ `**이라는 접두사로 시작한다.
단, 역할을 선언할 때만 이 접두사를 쓰지, 역할을 이용할 때는 접두사를 붙이지 않는다.

⚠️ User 빌더 클래스로 사용자를 구축할 때는 roles() 라는 메서드로 역할을 지정한다.  
(`GrantedAuthority` 객체를 만들고, 지정한 이름에 `ROLE_`접두사를 추가한다.)  
(`SimpleGrantedAuthority` 클래스로 `GrantedAuthority` 형식 변경이 불가능한 인스턴스를 만들 수 있다.)

```java
GrantedAuthority ga = new SimpleGrantedAuthority("READ");
```

⚠️ 권한에 부여한 이름을 바탕으로 권한 부여 규칙을 작성한다.

```java
public interface GrantedAuthority extends Serializable {
  String getAuthority(); // 권한의 이름
}
```

⚠️ 인증이 완료되면 사용자 세부정보가 보안컨텍스트에 저장되고, 요청이 <span style='color:rgb(196,58,26);'>권한 부여 필터로 위임</span>된다.

<span style="font-size:17px;font-weight:bold;">🍪 SecurityConfig에서 엔드포인트에 따른 역할 및 권한 정의 및 제어<span style='color:rgb(196,58,26);font-weight:bold;'>\* </span><span style='color:rgb(196,58,26);font-weight:bold;'>\* </span></span>

- authorizeRequests()  
  : 특정 엔드포인트에 <span style='color:rgb(196,58,26);'>권한 부여 규칙</span>을 지정

- anyRequest()  
  : 모든 요청에 <span style='color:rgb(196,58,26);'>권한 부여 규칙</span>을 지정

- hasAuthority() / hasRole()  
  : 하나의 권한/역할만 매개변수로 받는다.  
  : 해당 권한/역할이 있는 사용자만 엔드포인트를 호출할 수 있다.

- hasAnyAuthority() / hasAnyRole()  
  : 하나 이상을 권한/역할을 매개변수로 받을 수 있다.  
  : 주어진 권한/역할 중 하나라도 있는 사용자는 엔드포인트를 호출할 수 있다.

- access()
  : SpEL(Spring Expression Language)를 기반으로 권한부여 규칙을 정의  
  : <span style="margin-bottom:16px;font-size:15px;background-color:#ffdce0;color:rgb(196,58,26);font-weight:bold;border-radius:5px;padding:2px 3px;">권고x</span>

## 예제: 구성클래스 책임 나누기 및 보안 공유객체 설정

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(45,204,112);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    ⚔️ 예제: 구성클래스 책임 나누기 및 보안 공유객체 설정
</div>

`🫎 UserManagementConfig.java`

```java
@Configuration
public class UserManagementConfig {
  @Bean
  public UserDetailsService userDetailsService() {
    var userDetailsService = new InMemoryUserDetailsManager();

    var user = User.withUsername("peter")
    // ... (생략)

    userDetailsService.createUser(user);
    return userDetailsService;
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
    return NoOpsPasswordEncoder.getInstance();
  }
}
```

`🫎 AuthorizationConfig.java`

```java
@Configuration
public class AuthorizationConfig {
  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.csrf()
      .disable()
      .formLogin()
      .disable()
      .httpBasic()
      .disable()
      .sesseionManagement(sessionManagement -> sessionManagement.sessionCreaatePolicy(STATELESS));

    http.apply(new MyCustomDsl());

    http.authorizeRequests().anyRequest().authenticated();
    return http.build();
  }

  public class MyCustomDsl extends AbstractHttpConfigurer<MyCustomDsl, HttpSecurity> {
    @Override
    public void configure(HttpSecurity http) throws Exception {
      AuthenticationManager authenticationManager = http.getSharedObject(AuthenticationManager.class);

      // ...(생략)
    }
  }
}
```

<div style="margin-bottom:15px;font-size:15px;background-color:rgba(0,0,0,0.03);border-radius:5px;padding:7px;"><span style="font-weight:bold;">📘 AbstractHttpConfigurer란</span><br>
스프링 시큐리티에서 사용되는 <span style='color:rgb(196,58,26);'>HTTP 보안 설정을 커스텀할 때 사용되는 클래스</span>이다.<br>
<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;">HTTP 보안 구성을 템플릿화</span>하는데 사용되며, 공통적인 보안설정을 재사용할 수 있다.<br>
<span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#DEDEDE;">HttpSecurity.getSharedObject()</span> 메서드는 보안 설정을 구성하는 다양한 컴포넌트들이 서로 정보를 주고 받기 위한 공유객체를 가져올 수 있다.<br>
공유객체를 사용하면, 설정 간 <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#DEDEDE;">결합도</span>를 낮추고 <span style="margin-bottom:15px;padding:0 3px;font-size:16px;border-radius:5px;background-color:#DEDEDE;">재사용성🏆</span>을 높일 수 있다.
</div>
