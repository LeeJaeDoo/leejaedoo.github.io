---
date: 2020-07-27 18:40:40
layout: post
title: 토비의 스프링
subtitle: 5. 서비스 추상화
description: 5. 서비스 추상화
image: https://leejaedoo.github.io/assets/img/spring.jpeg
optimized_image: https://leejaedoo.github.io/assets/img/spring.jpeg
category: spring
tags:
  - spring
  - 책 정리
paginate: true
comments: true
---
# 사용자 레벨 관리 기능 추가

* 사용자의 레벨은 BASIC, SILVER, GOLD
* 사용자가 처음 가입하면 BASIC 레벨이 되며, 이후 활동에 따라 한 단계씩 업그레이드 될 수 있다.
* 가입 후 50회 이상 로그인 시, BASIC에서 SILVER 레벨이 된다.
* SILVER 레벨이 30번 이상 추천을 받으면 GOLD 레벨이 된다.
* 사용자 레벨의 변경 작업은 일정한 주기를 갖고 일괄적으로 진행된다. 변경 작업 전에는 조건을 충족하더라도 즉각 레벨의 변경이 일어나진 않는다.

## 필드 추가
### Level Enum
User 클래스에 사용자의 레벨을 저장할 필드를 추가한다.<br>
DB에는 varchar 타입으로 선언하고 'BASIC', 'SILVER', 'GOLD'라고 문자를 넣을 수도 있지만, 코드화하여 숫자를 넣는 것이 DB 용량도 절약하고 가벼워서 좋다.

> 그렇다면 User 클래스에 프로퍼티도 숫자로? -> 의미 없는 숫자를 프로퍼티에 사용하면 타입이 안전하지 않아서 위험할 수 있다.
> ```java
>   user1.setLevel(other.getSum());
> ```
> 위와 같이 다른 종류의 정보를 넣는 실수를 하여도 컴파일러가 체크해주지 못한다. 따라서 숫자 타입보다는 자바5 이상에서 제공하는 `enum`을 활용하는 것이 안전하고 편리하다.

```java
pubic enum Level {
    BASIC(1),
    SILVER(2),
    GOLD(3);    //  3개의 enum 오브젝트 정의

    private final int value;

    Level(int value) {  //  DB에 저장할 값을 넣어줄 생성자
        this.value = value;
    }

    public int intValue() {     //  값을 가져오는 메소드
        return value;
    }

    public static Level valueOf(int value) {    //  값으로부터 Level 타입 오브젝트를 가져오도록 만든 스태틱 메소드
        switch(value) {
            case 1: return BASIC;
            case 2: return SILVER;
            case 3: return GOLD;
            default: throw new AssertionError("Unknown value: " + value);
        }
    }
}
```

enum타입을 사용하게 되면 내부에는 DB에 저장할 int 타입을 갖지만 외부로는 Level 타입의 오브젝트이기 떄문에 안전하게 사용할 수 있다.
### User 필드 추가
```java
public class User {
    ...
    Level level;
    int login;
    int recommend;

    public Level getLevel() {
        return level;
    }

    public void setLevel(Level level) {
        this.level = level;
    }
    ...
}
```
### UserDaoTest 테스트 수정
기존 코드에 새로운 기능을 추가하려면 테스트를 먼저 만드는 것이 안전하다.

* 수정된 테스트 픽스처

```java
public class UserDaoTest {
    ...
    @Before
    public void setUp() {
        this.user1 = new User("user1", "이재두", "springno1", Level.BASIC, 1, 0);
        this.user2 = new User("user2", "정인철", "springno2", Level.SILVER, 55, 10);
        this.user3 = new User("user3", "이학영", "springno3", Level.GOLD, 100, 40);
    }
}
```

* 추가된 필드를 파라미터로 포함하는 생성자

```java
class User {
    ...
    public User(String id, String name, String password, Level level, int login, int recommend) {
        this.id = id;
        this.name = name;
        this.password = password;
        this.level = level
        this.login = login;
        this.recommend = recommend;
    }
}
```

* 새로운 필드를 포함하는 User 필드 값 검증 메소드

```java
private void checkSameUser(User user1, User user2) {
    assertTrue(user1.getId(), is(user2.getId());
    assertTrue(user1.getName(), is(user2.getName());
    assertTrue(user1.getPassword(), is(user2.getPassword());
    assertTrue(user1.getLevel(), is(user2.getLevel());
    assertTrue(user1.getLogin(), is(user2.getLogin());
    assertTrue(user1.getRecommend(), is(user2.getRecommed());
}
```

두 개의 User 오브젝트 필드 값이 모두 같은지 비교하는 메소드이다.

* checkSameUser() 메소드를 사용하도록 만든 addAndGet() 메소드

```java
@Test
public void addAndGet() {
    ...
    User userget1 = dao.get(user1.getId());
    checkSameUser(userget1, user1);

    User userget2 = dao.get(user2.getId());
    checkSameUser(userget2, user2);
}
```
### UserDaoJdbc 수정

* 추가된 필드를 위한 UserDaoJdbc의 수정 코드

```java
public class UserDaoJdbc implements UserDao {
    ...
    private RowMapper<User> userMapper = new RowMapper<User>() {
        public User mapRow(ResultSet rs, int rowNum) throws SQLException {
            User user = new User();
            user.setId(rs.getString("id"));
            user.setName(rs.getString("name"));
            user.setPassword(rs.getString("password"));
            user.setLevel(Level.valueOf(rs.getInt("level"));
            user.setLogin(rs.getInt("login"));
            user.setRecommend(rs.getInt("recommend"));

            return user;
        }
    };

    public void add(User user) {
        this.jdbcTemplate.update(
            "insert into users(id, name, password, level, login, recommend) " + "values(?,?,?,?,?,?)", user.getId(), user.getName(), user.getPassword(), user.getLLevell().intValue(), user.getLogin(), user.getRecommend()
        );
    }
}
```

여기서 눈여겨 볼 부분은 Level 타입의 level 필드를 사용하는 부분이다. Level enum 은 오브젝트이므로 DB에 저장될 수 있는 SQL타입이 아니기 때문에 DB에 저장가능한 정수형 값으로 변환해줘야 한다.
## 사용자 수정 기능 추가
### 수정 기능 테스트 추가

* 사용자 정보 수정 메소드 테스트

```java
@Test
public void update() {
    dao.deleteAll();
    dao.add(user1);

    //  픽스처에 들어있는 정보를 변경하여 수정 메소드를 호출한다.
    user1.setName("이재두");
    user1.setPassword("spring04");
    user1.setLevel(Level.GOLD);
    user1.setLogin(1000);
    user1.setRecommend(999);
    dao.update(user1);

    User user1update = dao.get(user1.getId());
    checkSameUser(user1, user1update);
}
```

### UserDao와 UserDaoJdbc 수정
### 수정 테스트 보완

```java
@Test
public void update() {
    dao.deleteAll();

    dao.add(user1);     //  수정할 사용자
    dao.add(user2);     //  수정하지 않을 사용자

    user1.setName("이재두");
    user1.setPassword("springno6");
    user1.setLevel(Level.GOLD);
    user1.setLogin(1000);
    user1.setRecommend(999);

    dao.update(user1);

    User user1update = dao.get(user1.getId());
    checkSameUser(user1, user1update);
    User user2same = dao.get(user2.getId());
    checkSameUser(user2, user2same);
}
```

## UserService.upgradeLevels()
레벨 관리 기능은 UserDao의 getAll() 메소드로 사용자를 다 가져와서 사용자별로 레벨 업그레이드 작업을 진행하면서 UserDao의 update()를 호출해 DB에 결과를 넣어주면 된다.<br>
그럼 사용자 관리 로직은 어디에 추가하는 것이 좋을까? UserService 클래스를 새로 생성하여 사용자 관리 비즈니스 로직을 담는다.<br>
인터페이스 타입으로 userDao 빈을 DI 받아 사용한다. UserSerivce는 UserDao의 구현 클래스가 바뀌어도 영향받지 않도록 해야 한다. 따라서 DAO의 인터페이스를 사용하고 DI를 적용해야 한다.
![UserService의 의존관계](../../assets/img/userservice.jpeg)
### UserService 클래스와 빈 등록

```java
public class UserService {
    UserDao userDao;    //  UserDao 오브젝트를 저장해둘 인스턴스 변수 선언

    public void setUserDao(UserDao userDao) {   //  UserDao 오브젝트의 DI가 가능하도록 수정자 메소드 추가
        this.userDao = userDao;
    }
}
```

### UserServiceTest 테스트 클래스

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserServiceTest {

    @Autowired
    UserService userService;    //  테스트 대상인 UserService 빈을 제공받을 수 있도록 @Autowired가 붙은 변수로 선언
                                //  UserService는 컨테이너가 관리하는 스프링 빈이므로 스프링 테스트 컨텍스트를 통해 주입받을 수 있음.

    @Test
    public void bean() {
        assertTrue(this.userService, is(notNullValue());    //  userService 빈이 생성돼어 userService 변수에 주입되는지 확인하는 테스트 메소드
    }
}
```

### upgradeLevels() 메소드
```java
public void upgradeLevels() {
    List<User> users = userDao.getAll();
    for(User user : users) {
        Boolean changed = null; //  레벨의 변화가 있는지 확인하는 flag
        if (user.getLevel() == Level.BASIC && user.getLogin() >= 50) {
            //  BASIC 레벨 업그레이드 로직   
        } else if (user.getLevel() == Level.SILVER && user.getRecommend() >= 30) {
            //  SILVER 레벨 업그레이드 로직
        } else if (user.getLevel() == Level.GOLD) {
            //  GOLD 레벨 처리 로직
        } else {...}

        if (changed) {
            userDao.update(user);   //  레벨의 변경이 있는 경우 udpate 처리 로직
        }
    }
}
```
### upgradeLevels() 테스트

```java
class UserServiceTest {
    ...
    List<User> users;   //  테스트 픽스처

    @Before
    public void setUp() {
        users = Arrays.asList(
            new User("hello", "이재두", "p1", Level.BASIC, 49, 0),
            new User("myu", "정인철", "p2", Level.BASIC, 60, 29),
            new User("naea", "이학영", "p3", Level.SILVER, 50, 0),
            new User("hello1", "김봉규", "p4", Level.SILVER, 60, 30),
            new User("todyou", "박주봉", "p5", Level.GOLD, 100, 100)
        );
    }

    @Test
    public void upgradeLevels() {
        userDao.deleteAll();
        for(User user : users)  userDao.add(user);

        userService.upgradeLevels();

        //  각 사용자별로 업그레이드 후의 예상 레벨을 검증
        checkLevel(users.get(0), Level.BASIC);
        checkLevel(users.get(1), Level.SILVER);
        checkLevel(users.get(2), Level.SILVER);
        checkLevel(users.get(3), Level.GOLD);
        checkLevel(users.get(4), Level.GOLD);
    }

    //  DB에서 사용자 정보를 가져와 레벨을 확인하는 코드가 중복되므로 헬퍼 메소드로 분리
    private void checkLevel(User user, Level expectedLevel) {
        User userUpdate = userDao.get(user.getId());
        assertThat(userUpdate.getLevel(), is(expectedLevel));
    }
}
```
적어도 가능한 모든 조건을 하나씩은 확인해봐야 하기 때문에 사용자 레벨 수 3가지, 그리고 GOLD를 제외한 나머지 레벨의 업그레이드 경우, 총 5가지의 경우의 수를 모두 테스트해봐야 하기 때문에 5개의 인스턴스를 테스트 케이스로 생성했다.

## UserService.add()
처음 가입한 사용자의 회원 레벨에 대한 로직을 추가해야 한다.
```java
@Test
public void add() {
    userDao.deleteAll();

    User userWithLevel = users.get(4);  //  GOLD 레벨

    //  레벨이 비어 있는 사용자. 로직에 따라 등록 중에 BASIC 레벨도 설정
    User userWithoutLevel = users.get(0);
    userWithoutLevel.setLevel(null);

    userService.add(userWithLevel);
    userService.add(userWithoutLevel);

    //  DB에 저장된 결과를 가져와 확인
    User userWithLevelRead = userDao.get(userWithLevel.getId());
    User userWithoutLevelRead = userDao.get(userWithoutLevel.getId());

    assertThat(userWithLevelRead.getLevel(), is(userWithLevel.getLevel()));
    assertThat(userWithoutLevelRead.getLevel(), is(Level.BASIC));
}
```
## 코드 개선
* 코드에 중복된 부분은 없는가?
* 코드가 무엇을 하는 것인지 이해하기 불편하지 않은가?
* 코드가 자신이 있어야 할 자리에 있는가?
* 앞으로 변경이 일어난다면 어떤 것이 있을 수 있고, 그 변화에 쉽게 대응할 수 있게 작성되어 있는가?

### upgradeLevels() 메소드 코드의 문제점
for 루프 속에 조건문이 복잡하다.
### upgradeLevels() 리팩토링
기존 upgradeLevels() 자주 변경될 가능성이 있는, 추상적인 로직의 흐름을 분리한다. 먼저 기본 흐름이 담긴 메소드를 구현한다.

* 기본 작업 흐름만 남겨둔 upgradeLevels()

```java
public void upgradeLevels() {
    List<User> users = userDao.getAll();
    for (User user : users) {
        if (canUpgradeLevel(user)) {
            upgradeLevel(user);
        }   
    }
}
```

그리고 나서 하나씩 구체적인 내용을 담은 메소드를 만든다.<br>
상태에 따라서 업그레이드 조건만 비교하면 되므로, 역할과 책임이 명료해진다.

* 업그레이드 가능 확인 메소드

```java
private boolean canUpgradeLevel(User user) {
    Level currentLevel = user.getLevel();
    switch(currentLevel) {
        case BASIC:     return (user.getLogin() >= 50);
        case SILVER:    return (user.getRecommend() >= 30);
        case GOLD:      return false;
        default: throw new IllegalArgumentException("Unknown Level: " + currentLevel);
    }
}
```

* 업그레이드 작업 메소드

```java
private void upgradeLevel(User user) {
    if (user.getLevel() == Level.BASIC) user.setLevel(Level.SILVER);
    else if (user.getLevel() == Level.SILVER)   user.setLevel(Level.GOLD);
    userDao.update(user);
}
```

위 처럼 업그레이드 작업용 메소드를 따로 분리함으로써 추후 추가되는 작업이 있더라도 어느 곳을 수정해야 할지 명확해진다.<br>
하지만 위 upgradeLevels() 메소드는 다음 단계가 무엇인지 알려주는 로직과 그 때 사용자 오브젝트의 level 필드를 변경해준다는 로직이 함께 있는데다 노골적으로 드러나있다. 게다가 예외상황에 대한 처리도 없다.<br>
그래서 위와 같이 레벨의 순서와 다음 단계 레벨이 무엇인지 결정하는 일은 Level이 맡도록 분리하였다.

* 업그레이드 순서를 담고 있도록 수정한 Level

```java
public enum Level {
    GOLD(3, null),
    SILVER(2, GOLD),
    BASIC(1, SILVER);

    private final int value;
    private final Level next;

    Level(int value, Level next) {
        this.value = value;
        this.next = next;
    }

    public int intValue() {
        return value;
    }

    public Level nextLevel() {
        return this.next;
    }

    public static Level valueOf(int value) {
        switch(value) {
            case 1: return BASIC;
            case 2: return SILVER;
            case 3: return GOLD;
            default: throw new AssertionError("Unknown value: " + value);
        }
    }
}
```

이렇게 구현함으로써 레벨의 업그레이드 순서는 Level enum 안에서 관리할 수 있다. 다음 단계의 레벨이 무엇인지를 일일이 if 조건식을 만들어서 비즈니스 로직에 담아둘 필요가 없다.<br>
이제 사용자 정보가 바뀌는 부분을 UserService에서 User로 이동시킨다. User의 내부 정보가 변경되는 부분은 UserService보다는 User가 스스로 다루는 게 적절하기 때문이다.

* User의 레벨 업그레이드 작업용 메소드

```java
public void upgradeLevel() {
    Level nextLevel = this.level.nextLevel();
    if (nextLevel == null) {
        throw new IllegalStateException(this.level + "은 업그레이드가 불가능합니다.");
    } else {
        this.level = nextLevel;
    }
}
```

덕분에 UserService는 User 오브젝트에게 알아서 업그레이드에 필요한 작업을 수행하라고 요청만 해주면 되기 때문에, upgradeLevel() 메소드가 아래처럼 간결해진다.

* upgradeLevel()

```java
private void upgradeLevel(User user) {
    user.upgradeLevel();
    userDao.update(user);
}
```

이로써 if 문장이 많던 이전 코드보다 간결하고 작업 내용이 명확하게 드러나면서 각 오브젝트가 해야할 책임도 깔끔하게 분리되었다.<br>

지금 개선한 코드를 보면 핵심은 각 오브젝트와 메소드가 각각 자기 몫의 책임을 맡아 일하는 구조로 만들어졌음을 알 수 있다. 각자 자기 책임에 충실한 작업만 하게되니 코드를 이해하기도 쉽다.<br>
`객체지향적인 코드는 다른 오브젝트의 데이터를 가져와서 작업하는 대신 데이터를 갖고 있는 다른 오브젝트에게 작업을 해달라고 요청한다.` 오브젝트에게 데이터를 요구하지 말고 작업을 요청하라는 것이 객체지향 프로그래밍의 가장 기본이 되는 원리이기도 하다.
### User 테스트

* User 테스트

```java
...
public class UserTest {
    User user;

    @Before
    public void setUp() {
        user = new User();
    }

    @Test()
    public void upgradeLevel() {
        Level[] levels = Level.values();
        for (Level level : levels) {
            if (level.nextLevel() == null) continue;
            user.setLevel(level);
            user.upgradeLevel();
            assertThat(user.getLevel(), is(level.nextLevel()));
        }
    }

    @Test(expected=IllegalStateException.class)
    public void cannotUpgradeLevel() {
        Level[] levels = Level.values();
        for(Level level : levels) {
            if (level.nextLevel() != null) continue;
            user.setLevel(level);
            user.upgradeLevel();
        }
    }
}
```

User 클래스에 대한 테스트는 굳이 스프링의 테스트 컨텍스트를 사용하지 않아도 된다. User 오브젝트는 스프링이 IoC로 관리해주는 오브젝트가 아니기 때문이다. 컨테이너가 생성한 오브젝트를 @Autowired로 가져오는 대신 생성자를 호출해서 테스트할 User 오브젝트를 만들면 된다.

# 트랜잭션 서비스 추상화
## 모 아니면 도
지금까지 구현해본 테스트 코드 실행 중 중간에 예외가 발생하여 작업이 중단된다면 결과가 어떻게 될까?<br>
장애가 발생했을 때 일어나는 현상 중 하나의 예외를 의도적으로 발생시키는 코드를 짜서 테스트 해보자.
### 테스트용 UserService 대역
가장 간단한 방법은 예외를 강제로 발생시키도록 애플리케이션 코드를 수정하는 것이지만, 테스트를 위해 함부로 코드를 수정하는 것은 좋지 못한 방법이다. 테스트용으로 특별히 만든 UserService의 대역을 사용하는 방법이 좋다. 즉, UserService를 대신할 테스트 클래스를 만들자는 얘기다.<br>
UserService를 상속해서 테스트에 필요한 기능을 추가하도록 일부 메소드를 오버라이딩 하는 방법이 좋을 듯 하다.

* UserService의 테스트용 대역 클래스

```java
static class TestUserService extends UserService {
    private String id;
    
    private TestUserService(String id) {    //  예외를 발생시킬 User 오브젝트의 id를 지정할 수 있게 만든다.
        this.id = id;
    }

    protected void upgradeLevel(User user) {    //  UserService의 메소드를 오버라이드한다.
        if (user.getId().equals(this.id)) {
            throw new TestUserServiceException();   //  지정된 id의 User 오브젝트가 발견되면 예외를 던져서 작업을 강제로 중단시킨다.
        }
        super.upgradeLevel(user);        
    }
}
```

* 테스트용 예외

```java
static class TestUserServiceException extends RuntimeException {...}
```

### 강제 예외 발생을 통한 테스트

* 예외 발생 시 작업 취소 여부 테스트

```java
@Test
public void upgradeAllOrNothing() {
    UserService testUserService = new TestUserService(users.get(3).getId());    //  예외 발생 시, 4번째 사용자의 id를 넣어 테스트용 UserService 대역 오브젝트를 생성한다.
    testUserService.setUserDao(this.userDao);   //  userDao를 수동 DI 해준다.

    userDao.deleteAll();
    for (User user : users) userDao.add(user);

    try {
        testUserService.upgradeLevels();
        fail("TestUserServiceException expected");  //  TestUserService는 업그레이드 작업 중에 예외가 발생해야 한다.
                                                    //  정상 종료라면 문제가 있으니 실패.
    } catch (TestUserServiceException e) {      //  TestUserService가 던져주는 예외를 잡아서 계속 진행되도록 한다. 그외의 예외라면 테스트 실패
    }

    checkLevelUpgraded(users.get(1), false);    //  예외가 발생하기 전에 레벨 변경이 있었던 사용자의 레벨이 처음 상태로 바뀌었나 확인
}
```

TestUserService는 upgradeAllOrNothing() 테스트 메소드에서만 특별한 목적으로 사용하는 것이니, 번거롭게 스프링 빈으로 등록할 필요는 없다.
결과는
```text
java.lang.AssertionError:
Expected: is <BASIC>
    got: <SILVER>
```
두 번째 사용자의 레벨이 BASIC에서 SILVER로 바뀐 것이 네 번째 사용자 처리 중 예외가 발생했지만 그대로 유지되고 있다.
### 테스트 실패의 원인
트랜잭션이 원인이다. 모든 사용자의 레벨을 업그레이드하는 작업인 upgradeLevel() 메소드가 하나의 트랜잭션 안에서 동작하지 않았기 때문이다.
## 트랜잭션 경계설정
DB는 그 자체로 완벽한 트랜잭션을 지원한다. 하나의 SQL 명령을 처리하는 경우는 DB가 트랜잭션을 보장해준다고 믿을 수 있다.<br>
하지만, 여러 개의 SQL이 사용되는 작업을 하나의 트랜잭션으로 취급해야 하는 경우, 즉 두 번의 SQL이 실행되는데 첫 번째는 성공적으로 실행했지만 두 번째 SQL이 성공하기 전에 장애가 생겨 작업이 중단되면서 앞에서 처리한 SQL도 취소가 필요한 경우, 이런 취소 작업을 `트랜잭션 롤백`이라고 한다.<br>
반대로 여러 개의 SQL을 하나의 트랜잭션으로 처리하는 경우에 모든 SQL 수행작업이 성공적으로 마무리됐다고 DB에 알려줘야 하는데 이를 `트랜잭션 커밋`이라고 한다.
### JDBC 트랜잭션의 트랜잭션 경계설정
모든 트랜잭션은 시작하는 지점과 끝나는 지점이 있다. 시작하는 방법은 한 가지이지만 끝나는 방법은 두 가지다.

* 모든 작업을 무효화하는 롤백
* 모든 작업을 다 확정하는 커밋

애플리케이션 내에서 트랜잭션이 시작되고 끝나는 위치 위치를 트랜잭션의 경계라고 부른다. 트랜잭션의 경계는 하나의 Connection이 만들어지고 닫히는 범위 안에 존재한다. 이렇게 하나의 DB 커넥션 안에서 만들어지는 트랜잭션을 로컬 트랜잭션이라고도 한다.
### 비즈니스 로직 내의 트랜잭션 경계설정
앞에서 말한 5개의 사용자 오브젝트를 순차적으로 가져와 회원 등급 업그레이드를 실행할 때 2번째 사용자의 레벨은 변경하는 도중 예외가 발생했을 떄, 5명 모두 롤백을 하기 위해서는 5명의 사용자의 레벨 업그레이드 로직이 하나의 트랜잭션 경계에 있어야 한다.<br>
하지만 이렇게 되면 기껏 그동안 비즈니스 로직과 데이터 로직을 성격과 책임에 따라 분리하고, 느슨하게 연결하여 확장성을 좋게 해온 수고가 헛수고가 된다.<br>
따라서, 이런 방식이 아닌 UserService와 UserDao는 그대로 둔 채로 트랜잭션을 적용하려면 결국 트랜잭션의 경계설정 작업을 UserService 쪽으로 가져와야 한다.<br>
트랜잭션 경계를 upgradeLevels() 메소드안에 두고 DB 커넥션도 이 메소드 안에서 만들고, 종료시켜야 한다.

* upgradeLevels의 트랜잭션 경계설정 구조

```java
public void upgradeLevels() throws Exception {
    (1) DB Connection 생성
    (2) 트랜잭션 시작
    try {
        (3) DAO 메소드 호출
        (4) 트랜잭션 커밋
    } catch (Exception e) {
        (5) 트랜잭션 롤백
        throw e;
    } finally {
        (6) DB Connection 종료
    }
}
```

여기서 생성된 Connection 오브젝트를 갖고 데이터 액세스 작업을 진행하는 코드는 UserDao의 update() 메소드 안에 있어야 한다. UserDao의 update() 메소드는 반드시 upgradeLevels() 메소드에서 만든 Connection을 사용해야 같은 트랜잭션 안에서 동작한다.<br>
기존 JdbcTemplate처럼 매번 새로운 Connection 오브젝트를 만들어버리면, upgradeLevels() 안에서 시작한 트랜잭션과는 무관한 별개의 트랜잭션이 만들어지기 때문이다.<br>
UserService에서 만든 Connection 오브젝트를 UserDao에서 사용하려면 DAO 메소드를 호출할 떄마다 Connection 오브젝트를 파라미터로 전달해줘야한다.

### UserService 트랜잭션 경계설정의 문제점
이로써 트랜잭션 문제는 해결하지만 그 밖에 다른 문제가 있다.
#### 더 이상 JdbcTemplate를 활용할 수 없다.
#### DAO의 메소드와 비즈니스 로직을 담고 있는 UserService의 메소드에 Connection 파라미터가 추가돼야 한다.
같은 Connection 오브젝트가 사용돼야 트랜잭션이 유지되기 때문이다. UserService는 스프링 빈으로 선언해서 싱글톤으로 돼있기 때문에 UserService에 Connection 을 저장해뒀다가 다른 메소드에서 사용할 수도 없다. 멀티스레드 환경에서는 문제가 되기 때문이다.<br>
결국, 트랝개션이 필요한 작업에 참여하는 UserService의 메소드는 Connection파라미터로 지저분해질 것이다.
#### Connection 파라미터가 UserDao 인터페이스 메소드에 추가되면 UserDao는 더 이상 데이터 액세스 기술에 독립적일 수 없다.
JPA나 하이버네이트로 UserDao 구현 방식을 변경하려고 할 때마다 UserDao 인터페이스와 UserService 코드도 함께 수정돼야 할 것이다.
#### DAO 메소드에 Connection 파라미터를 받게 되면 테스트 코드에도 영향을 미친다.
테스트 코드에서도 직접 Connection 오브젝트를 일일이 만들어서 DAO 메소드를 호출하도록 모두 변경해야 한다.
## 트랜잭션 동기화
객체지향 관점에 따라 분리되어 깔끔하게 정리된 코드와 트랜잭션 기능 두마리 토끼를 모두 잡을 수 있는 방법은 없을까?
### Connection 파라미터 제거
upgradeLevels() 가 트랜잭션 경계설정을 해야 하는 것은 피할 수 없기 때문에 그 안에서 Connection을 생성하고 트랜잭션 시작과 종료를 관리해야 되긴 하지만, Connection 오브젝트를 계속 메소드의 파라미터로 전달하다가 DAO를 호출할 때 사용하는 것을 피하고 싶다.<br>
이를 위해 스프링이 제안하는 방법은 독립적인 `트랜잭션 동기화(Transaction Synchronization)`방식이다.<br>
트랜잭션 동기화란 UserService에서 트랜잭션을 시작하기 위해 만든 Connection 오브젝트를 특별한 저장소에 보관해두고, 이후에 호출되는 DAO의 메소드에서는 저장된 Connection 오브젝트를 가져다 사용하는 것이다.<br>
정확히는 DAO가 사용하는 JdbcTemplate이 트랜잭션 동기화 방식을 이용하도록 하는 것이다. 그리고 트랜잭션이 모두 종료되면, 그 때는 동기화를 마치면 된다.
![트랜잭션 동기화를 사용한 경우의 작업흐름](../../assets/img/transaction_sync.jpeg)
(1) UserService에서 Connection 생성
(2) 이를 트랜잭션 동기화 저장소에 저장해두고 트랜잭션을 시작시킨 후 본격적으로 DAO의 기능을 이용
(3) 첫 번째 update() 메소드가 호출되고, 
(4) update() 메소드 내부에서 이용하는 JdbcTemplate 메소드에서는 트랜잭션 동기화 저장소에 현재 시작되있는 트랜잭션을 가진 Connection 오브젝트가 존재하는지 확인한다. (2) upgradeLevels() 메소드 시작 부분에서 저장해둔 Connection을 발견하고 이를 가져온다.
(5) 가져온 Connection 을 이용해 PreparedStatement 를 만들어 수정 SQL을 실행한다. 트랜잭션 동기화 저장소에서 DB 커넥션을 가져왔을 때는 JdbcTemplate은 Connection을 닫지 않은 채로 작업을 마친다. 이렇게 첫 번째 DB 작업을 마치고 여전히 Connection은 열려 있고 트랜잭션도 진행중인 채로 트랜잭션 동기화 저장소에 저장돼있다.
(6) 두 번째 update()가 호출되면 이 때도 마찬가지로 
(7) 트랜잭션 동기화 저장소에서 같은 Connection을 가져와 (8) 사용한다.
(9) 마지막 update()도 (10) 같은 트랜잭션을 가진 Connection을 가져와 (11) 사용한다.
(12) 트랜잭션 내 모든 작업이 정상적으로 끝났으면 Connection의 commit()을 호출해서 트랜잭션을 완료시킨다.
(13) 마지막으로 트랜잭션 저장소가 더 이상 Connection 오브젝트를 저장해두지 않도록 이를 제거한다. 어느 작업 중에라도 예외상황이 발생하면 UserService는 즉시 Connection의 rollback()을 호출하고 트랜잭션을 종료할 수 있다. 물론 이 떄도 트랜잭션 저장소에 저장된 동기화된 Connection 오브젝트는 제거해줘야 한다.

> 트랜잭션 동기화 저장소는 작업 스레드마다 독립적으로 Connection 오브젝트를 저장하고 관리하기 때문에 다중 사용자를 처리하는 서버의 멀티스레드 환경에서도 충돌이 날 염려는 없다.

이렇게 트랜잭션 동기화 기법을 사용하면 파라미터를 통해 일일이 Connection 오브젝트를 전달할 필요가 없어진다.
### 트랜잭션 동기화 적용
문제는 멀티스레드 환경에서도 안전한 트랜잭션 동기화 방법을 구현하는 일이 간단하지 않은데, 다행이도 스프링은 JdbcTemplate과 더불어 이런 트랜잭션 동기화 기능을 지원하는 간단한 유틸리티 메소드를 제공하고 있다.

* 트랜잭션 동기화 방식을 적용한 UserService

```java
private DataSource dataSource;

public void setDataSource(DataSource dataSource) {  //  Connection을 생성할 때 사용할 DataSource를 DI받는다.
    this.dataSource = dataSource;
}

public void upgradeLevels() throws Exception {  //  
    TransactionSymchronizationManager.initSynchronization();    //  트랜잭션 동기화 관리자를 이용해 동기화 작업을 초기화
    Connection c = DataSourceUtils.getConnection(dataSource);   //  DB 커넥션을 생성하고 트랜잭션을 시작. 이후의 DAO 작업은 모두 여기서 시작한 트랜잭션 안에서 진행됨.
    c.setAutoCommit(false);     //  트랜잭션의 시작 선언
    //  DB 커넥션 생성과 동기화를 함께 해주는 유틸리티 메소드

    try {
        List<User> users = userDao.getAll();
        for (User user : users) {
            if (canUpgradeLevel(user)) {
                upgradeLevel(user);
            }
        }
        c.commit();     //  정상적으로 작업을 마치면 트랜잭션 커밋
    } catch (Exception e) {
        c.rollback();   //  예외가 발생하면 롤백
        throw e;
    } finally {
        DataSourceUtils.releaseConnection(c, dataSource);   //  스프링 유틸리티 메소드를 이용해 DB 커넥션을 안전하게 닫는다.
        TransactionSynchronizationManager.unbindResource(this.dataSource);  //  동기화 작업 종료
        TransactionSynchronizationManager.clearSynchronization();           //  동기화 작업 정리
    }
}
```

UserService에서 DB 커넥션을 직접 다룰 때 DataSource가 필요하므로 DataSource 빈에 대한 DI 설정을 해둬야 한다.<br>
스프링이 제공하는 트랜잭션 동기화 관리 클래스는 TransactionSynchronizationManager다. 그리고 DataSourceUtils에서 제공하는 getConnection() 메소드를 통해 DB 커넥션을 생성한다. DataSource에서 Connection을 직접 가져오지 않고, `스프링이 제공하는 유틸리티 메소드를 쓰는 이유는 이 DataSourceUtils의 getConnection() 메소드는 Connection 오브젝트를 생성해줄 뿐만 아니라 트랜잭션 동기화에 사용하도록 저장소에 바인딩해주기 때문`이다.<br>
작업을 정상적으로 마치면 트랜잭션을 커밋해주고 스프링 유틸리티 메소드의 도움을 받아 커넥션을 닫고 트랜잭션 동기화를 마치도록 요청하면 된다. 만약 예외가 발생하면 트랜잭션을 롤백해준다. 이때도 DB 커넥션을 닫는 것과 동기화 작업 중단은 동일하게 진행해야 한다.
## 트랜잭션 서비스 추상화
이로써 책임과 성격에 따라 데이터 액세스 부분과 비즈니스 로직을 잘 분리, 유지할 수 있게 만들었다.
### 기술과 환경에 종속되는 트랜잭션 경계설정 코드
하지만, 새로운 문제가 발생한다. 이 사용자 관리 모듈을 구매해서 사용하기로 한 G 사에서 들어온 새로운 요구사항 때문이다.
### 트랜잭션 API의 의존관계 문제화 해결책
### 스프링의 트랜잭션 서비스 추상화
### 트랜잭션 기술 설정의 분리
## 서비스 추상화와 단일 책임 원칙
### 수직, 수평 계층구조와 의존관계
### 단일 책임 원칙
### 단일 책임 원칙의 장점

