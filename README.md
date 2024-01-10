DASOM-OfficialSite-Backend
Dasom 동아리 공식 홈페이지 백엔드 팀 

## **프로젝트 환경 설정**

- start.spring.io
    - Gradle-Groovy
    - Java 17
    - jar
    - dependency: Spring Web, Thymeleaf, spring-boot-devtools
- intellij
    - open - 압축 푼 폴더의 gradle 파일 선택 후 open
    - setting - gradle - bulid 설정 intellij로 변경

## Spring Boot

### 프로젝트 코드 부분 구조

- src/main(상단)
    - java
        - project.name
            - controller : 동적 페이지 컨트롤러
            - domain : 실제 데이터를 주고 받는 동작 제어(Post 등)
            - repository : 데이터베이스 접근, DB 저장 및 관리
            - service : 핵심 기능(중복 방지 등)
            - projectNameApplication 파일 : 최상단 실행 파일
    - resources
        - static : 정적 페이지
        - templates : 동적 페이지
- src/test : 테스트 파일 저장

### Spring Bean

- 컴포넌트 스캔과 자동 의존관계 설정

```java
@Component
@Controller
@Service
@Repository
```

- 자바 코드로 직접 스프링 빈 등록
    - @Bean으로 Spring Bean 등록

```java
package hello.hellospring;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class SpringConfig {
 @Bean
 public MemberService memberService() {
	 return new MemberService(memberRepository());
 }
 @Bean
 public MemberRepository memberRepository() {
	 return new MemoryMemberRepository();
 }
}
```

- thymeleaf: 데이터 사용을 편하게 해줌

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
 <div>
 <table>
	 <thead>
		 <tr>
			 <th>#</th>
			 <th>이름</th>
		 </tr>
	 </thead>
	 <tbody>
		 <tr th:each="member : ${members}">
			 <td th:text="${member.id}"></td>
			 <td th:text="${member.name}"></td>
		 </tr>
	 </tbody>
 </table>
 </div>
</div> <!-- /container -->
</body>
</html>
```

DB

- JdbcTemplate 클래스 제공
- 사용전 JDBC

```java
implementation 'org.springframework.boot:spring-boot-starter-jdbc'
```

- resources/application.properties

```java
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
```

- JdbcTemplate 사용

```java
List<Member> results = jdbcTemplate.query(
	"select * from MEMBERS where EMAIL = ?",
	new RowMapper<Member>() {
		@Override
		public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
			Member member = new Member(rs.getString("Email"),
			rs.getStfing("PW"), rs.getString("NAME"), rs.getTimestamp("REGDATE"));
			member.setId(rs.getLong("ID");
			return member;
		}
	},
	email);
return results.isEmpty() ? null : results.get(0);
```

- JPA
    - 스프링 데이터 JPA라는 프레임워크를 이용
    - JPA: java에서 데이터 관리를 위해 제공되는 api

```java
package hello.hellospring.repository;
import hello.hellospring.domain.Member;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, 
Long>, MemberRepository {
 Optional<Member> findByName(String name);
}
```

```java
package hello.hellospring;
import hello.hellospring.repository.*;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class SpringConfig {
 private final MemberRepository memberRepository;
 public SpringConfig(MemberRepository memberRepository) {
	 this.memberRepository = memberRepository;
 }
 @Bean
 public MemberService memberService() {
	 return new MemberService(memberRepository);
 }
}
```

## Spring Security

- 스프링 하위의 어플리케이션 보안 관련 프레임워크
- AuthenticationFilter에 의해 요청의 인증여부를 확인하고 권한은 인가함

```java
implementation 'org.springframework.boot:spring-boot-starter-security'
```

### 주요모듈

- ****SecurityContextHolder****
    - 보안주체의 세부정보를 저장하는 객체
- ****SecurityContext****
    - Authentication(권한)을 저장하는 객체
- ****Authentication****
    - 현재 접근하는 주체의 정보와 권한을 저장하는 인터페이스

예제 코드

```java
public interface Authentication extends Principal, Serializable {
    // 현재 사용자의 권한 목록을 가져옴, principal: id, serializable: pw
    Collection<? extends GrantedAuthority> getAuthorities();
    
    // credentials(주로 비밀번호)을 가져옴
    Object getCredentials();
    
    Object getDetails();
    
    // Principal 객체를 가져옴.
    Object getPrincipal();
    
    // 인증 여부를 가져옴
    boolean isAuthenticated();
    
    // 인증 여부를 설정함
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

1. 로그인 요청 → UserPasswordAuthenticationToken발급
    1. id 와 password 유효성 검사
2. UserPasswordAuthenticationToken를 AuthenticationManager에게 전달
3. AuthenticationManager이 UserPasswordAuthenticationToken를 AuthenticaionProvider에게 전달
    1. db에서 원하는 정보 조회
    2. authenticate함수에서 인증 절차 수행
4. UserDetailsService에서 DB에 정보 전달 후 원하는 정보 검색
5. UserDetailsService에서 조회한 정보를 바탕으로 인증토큰 발급 후 AuthenticationManager에게 전달
6. 인증된 토큰을 AuthenticationFilter에 전달
7. 전달된 Authentication객체를 SecurityContextHolder에 저장

SpringSecurity 관련 예제 코드
```
https://github.com/MangKyu/SpringSecurity-Example/tree/formbased
https://github.com/spring-projects/spring-security-samples/tree/main/reactive/webflux/java/hello-security-explicit
```
