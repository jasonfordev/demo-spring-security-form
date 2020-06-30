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
  
  6. 인메모리 유저 추가
  
    방법1) application.properties에 추가
  ```yml
      spring.security.user.name=admin
      spring.security.user.password=123
      spring.security.user.roles=ADMIN
  ```
    
    방법2) WebSecurityConfigurerAdapter 커스터마이징
  ```java
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                // {noop} --> 123을 암호화 하지 않음
                .withUser("user").password("{noop}123").roles("USER")
                .and()
                .withUser("admin").password("{noop}!@#").roles("ADMIN");
    }
  ```
  
  7. JPA 연동
  ```
    7-1. 의존성 추가
    
      compile 'org.springframework.boot:spring-boot-starter-data-jpa'
      runtime 'com.h2database:h2'
  ```  
  ```      
    7-2. Entity, Repository, Service, Controller 생성
  ```
  
  ```java
    @Entity
    public class Account {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Integer id;

        @Column(unique = true)
        private String username;

        private String password;

        private String role;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        public String getUsername() {
            return username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public String getPassword() {
            return password;
        }

        public void setPassword(String password) {
            this.password = password;
        }

        public String getRole() {
            return role;
        }

        public void setRole(String role) {
            this.role = role;
        }
        
        public void encodePassword() {
            this.password = "{noop}" + this.password;
        }
    }
  ```
    
    
  ```java
    public interface AccountRepository extends JpaRepository<Account, Integer> {
        Account findByUsername(String username);
    }
  ```
    
    
  ```java
    @Service
    public class AccountService implements UserDetailsService {

        @Autowired
        AccountRepository accountRepository;

        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            Account account = accountRepository.findByUsername(username);
            if (account == null) {
                throw new UsernameNotFoundException(username);
            }

            return User.builder()
                    .username(account.getUsername())
                    .password(account.getPassword())
                    .roles(account.getRole())
                    .build();
        }
        
        public Account createNew(Account account) {
            account.encodePassword();
            return accountRepository.save(account);
        }
    }
  ```
  
  ```java
    @RestController
    public class AccountController {

        @Autowired
        AccountService accountService;

        @GetMapping("/account/{role}/{username}/{password}")
        public Account createAccount(@ModelAttribute Account account) {
            return accountService.createNew(account);
        }
    }
  ```
    
  8. PasswordEncoder
  
  ```java
    [DemoSpringSecurityFormApplication.java]
    
    // 비추: 비밀번호가 평문 그대로 저장됩니다.
    @Bean
    public PasswordEncoder passwordEncoder() {
      return NoOpPasswordEncoder.getInstance();
    }

    // 추천: 기본 전략인 bcrypt로 암호화 해서 저장하며 비교할 때는 {id}를 확인해서 다양한 인코딩을 지원합니다.
    @Bean
    public PasswordEncoder passwordEncoder() {
      return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
  ```
  
  ```java
    [AccountService.java]
    
    @Autowired PasswordEncoder passwordEncoder;
    
    public Account createNew(Account account) {
        account.encodePassword(passwordEncoder);
        return accountRepository.save(account);
    }
  ```
    
  ```java
    [Account.java]
    
    public void encodePassword(PasswordEncoder passwordEncoder) {
        this.password = passwordEncoder.encode(this.password);
    }
  ```
