---
date: 2021-07-31T11:54:35+09:00
group: blog
image: /images/posts/spring-boot-490x257.png
tags: ["spring boot", "multi datasource", "data.sql"]
title: "멀티 데이터소스에서 data.sql 사용하기"
url: /2021/07/31/multi-datasource-data-sql-initialize
type: post
summary: "데이터소스를 여러개 사용하고 있는 경우에 각각의 데이터베이스에 data.sql 을 적용하는 방법을 살펴보았다."
---

# 개요

spring boot 프로젝트를 개발하는 도중 `data.sql` 을 사용해서 테스트에 필요한 기초 데이터를 초기화 하는 작업을 수행했다. 그런데 연결되는 데이터베이스가 늘어나서 데이터소스 설정을 추가한 뒤에 각각의 데이터베이스 마다 `data.sql` 을 사용하여 초기화 하려고 하자 테스트가 계속 실패했다. 그 원인을 찾아서 수정해보았다. 

# 멀티 데이터소스와 Data.sql 연결

## 싱글 데이터소스 연결

spring boot 프로젝트에서 여러개의 데이터베이스에 연결하는 경우가 있다. 나의 경우에는 신규 프로젝트를 시작하면서 제일 처음 사용자의 계정 데이터베이스에 연결하는 데이터소스를 설정하기 위해서 다음과 같이 코드를 작성하였다. 

다음은 *AccountDataSourceConfiguration* 이다.
```java
@Configuration
@EnableJpaRepositories(basePackages = {
"me.home.domain.account"
},
entityManagerFactoryRef = "accountEntityManagerFactory",
transactionManagerRef = "accountTransactionManager")
public class AccountDataSourceConfig {

    public static final String[] packages = new String[]{
            "me.home.domain.account"
    };

    private final JpaProperties jpaProperties;
    private final HibernateProperties hibernateProperties;

    public AccountDataSourceConfig(JpaProperties jpaProperties, HibernateProperties hibernateProperties) {
        this.jpaProperties = jpaProperties;
        this.hibernateProperties = hibernateProperties;
    }
    
    ....이하 생략...
```

이 상태에서 `data.sql` 을 작성한 뒤 테스트를 실행하면 바로 이 *AccountDataSource* 에 작성한 SQL 스크립트가 실행된다.

## 멀티 데이터소스 연결

위의 상황에서 추가적으로 비지니스 로직을 구현하면서 필요한 콘텐츠를 담아두는 데이터소스를 추가한다고 생각해보자. 

다음은 *ContentDataSourceConfiguration* 이다.

```java
@Configuration
@EnableJpaRepositories(basePackages = {
        "me.home.domain.post",
        "me.home.domain.comment"},
        entityManagerFactoryRef = "contentEntityManagerFactory",
        transactionManagerRef = "contentTransactionManager")
public class ContentDataSourceConfig {

    private final JpaProperties jpaProperties;
    private final HibernateProperties hibernateProperties;

    public ContentDataSourceConfig(JpaProperties jpaProperties, HibernateProperties hibernateProperties) {
        this.jpaProperties = jpaProperties;
        this.hibernateProperties = hibernateProperties;
    }
    
    ...이하 생략...
```

이 다음에는 content data source 에서 실행하고자 하는 `data.sql` 스크립트 내용을 추가로 작성해야하는데 이를 구분해서 적용하는 방법이 매뉴얼에는 안내되어 있지 않다. (매뉴얼에는 *While we do not recommend using multiple data source initialization technologies* 라고 하며 멀티 데이터소스에서 초기화 스크립트를 실행하는 것을 권장하지 않는다고 나와있다.) 이 상태에서 테스트를 실행하면 IllegalArgumentException 이 발생하거나 DataSourceInitializationConfiguration 에서 정상적으로 처리되지 않는다. 
따라서 다음의 순서대로 적용해보았다.

## DataSourceInitializer 역할 확인

`org.springframework.boot.autoconfigure.jdbc.DataSourceInitializer` 파일을 확인해보면 `initSchema()` 메서드에서 `runScripts` 메서드를 호출하는 부분을 찾을 수 있는데 이부분이 바로 `data.sql` 을 실행하는 부분이다. 

```java
List<Resource> scripts = getScripts("spring.datasource.data", this.properties.getData(), "data");
if (!scripts.isEmpty()) {
    if (!isEnabled()) {
        logger.debug("Initialization disabled (not running data scripts)");
        return;
    }
    String username = this.properties.getDataUsername();
    String password = this.properties.getDataPassword();
    runScripts(scripts, username, password);
}
```

따라서 각각의 datasource configuration 에 다음과 같이 bean 을 등록해주는 코드를 추가해주었다. 

## Bean 등록

```java
@Bean
@Profile("local") // 테스트에서만 runScript 에 필요한 sql 스크립트이기 때문에 'local' profile 에서만 bean 을 등록한다.
public DataSourceInitializer dataSourceInitializer(@Qualifier("accountDataSource") DataSource datasource) {
    ResourceDatabasePopulator resourceDatabasePopulator = new ResourceDatabasePopulator();
    resourceDatabasePopulator.addScript(new ClassPathResource("account-data.sql"));
    // 여기에서는 data.sql 을 사용하지 않고 account-data.sql 파일을 사용하였다.

    DataSourceInitializer dataSourceInitializer = new DataSourceInitializer();
    dataSourceInitializer.setDataSource(datasource);
    dataSourceInitializer.setDatabasePopulator(resourceDatabasePopulator);
    return dataSourceInitializer;
}
```

그리고 `application.yml` 파일에 다음과 같이 설정해주었다.

```yaml

  datasource.account:
    jdbcUrl: jdbc:h2:mem:testdb
    driverClassName: org.h2.Driver
    username: sa
    password:
    maximumPoolSize: 20
    poolName: account-hikari
    data: account-data.sql

  datasource.content:
    jdbcUrl: jdbc:h2:mem:testdb
    driverClassName: org.h2.Driver
    username: sa
    password:
    maximumPoolSize: 20
    poolName: content-hikari
    data: content-data.sql

```

## 정리

나의 경우에는 테스트에서 필요한 기초 데이터를 입력하기 위한 `data.sql` 파일을 사용하고 있었는데, 멀티 데이터소스 구조로 변경되면서 각각의 데이터베이스에 분리된 별도의 data.sql 을 실행할 필요가 있었다. 따라서 DataSourceInitializer Bean 을 직접 추가하므로써 분리된 각각의 `account-data.sql`, `content-data.sql` 이 실행될 수 있도록 하였다.  

## 참고

1. DataSource Configuration은 이름 순서대로 Component Scan 하므로 DataSource Configuration 파일의 이름이 중요할 수 있다. 
2. `@Primary` 어노테이션은 `Data.sql` 을 실행하는데 아무런 관계가 없다.
3. application.yml 을 어떤게 작성하여 datasource bean 을 등록하느냐에 따라서 위의 코드는 달라질 수 있다.

# 참고자료

- [공식 매뉴얼](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.data-initialization.using-hibernate)
- [스택오버플러오](https://stackoverflow.com/questions/24508223/multiple-sql-import-files-in-spring-boot)
