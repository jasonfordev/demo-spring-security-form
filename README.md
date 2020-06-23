# demo-spring-security-form
  1. 스프링부트 웹 프로젝트 생성
  
    - 의존성 : web, thymeleaf
    
  2. 테스트 뷰 생성 : 정상 접근
  
    - admin, dashboard, index, info
      
  3. 스프링 시큐리티 의존성 추가
  
    - compile 'org.springframework.boot:spring-boot-starter-security'
    
  4. 스프링 시큐리티 동작 : 테스트 뷰 접근 시 로그인 필요
  
    - 톰캣 부팅 시 랜덤으로 user 계정의 비밀번호 생성 (로그에서 확인 가능)
    - Using generated security password: 9c62c883-1ccc-4754-b486-a170950b6492
    - 로그인 정보 : user / 톰캣에서 생성해준 비밀번호 
    
  5. 스프링 시큐리티 커스터마이징
  
  ```java
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests()
                    .mvcMatchers("/", "/info").permitAll()
                    .mvcMatchers("/admin").hasRole("ADMIN")
                    .anyRequest().authenticated();
            http.formLogin();
            http.httpBasic();
        }
    }
  ```
    
