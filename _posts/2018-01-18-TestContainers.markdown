---
layout: post
title:  "TestContainers - the best thing since sliced bread ;)"
date:   2018-01-18 20:52:29 -0700
categories: jekyll update
---
# TestContainers - the best thing since sliced bread ;)

From https://www.testcontainers.org/

```
TestContainers is a Java library that supports JUnit tests, providing lightweight, throwaway instances of common databases,
Selenium web browsers, or anything else that can run in a Docker container.
```

### Who is using TestContainers?
ZeroTurnaround - Testing of the Java Agents, micro-services, Selenium browser automation
Zipkin - MySQL and Cassandra testing
Apache Gora - CouchDB testing
Apache James - LDAP and Cassandra integration testing
StreamSets - LDAP, MySQL Vault, MongoDB, Redis integration testing
Playtika - Kafka, Couchbase, MariaDB, Redis, Neo4j, Aerospike, MemSQL
JetBrains - Testing of the TeamCity plugin for HashiCorp Vault
Plumbr - Integration testing of data processing pipeline micro-services

## Why use testContainers??
The thing that comes to mind most often while writing and designing a service is how it interacts with
the data store that backs it, or the messaging tier that it consumes or produces messages to.
No service works in isolation, so in order to test that your service works you need to test these 
other system.
Now traditionally it has been hard to rely on these discrete systems, you can spin them up in a common 
environment but the state can be corrupted by one or more factors, there could be tests that could be stepping 
on each other's feet etc etc.
According to the traditional test pyramid
![consul](/assets/traditional-testing-pyramid-31.png) 

The use of testcontainers would be to write integration test, but maybe also write some integrated tests.
For e.g lets say to bring up certain other services which may be useful to write some integrated tests against.

But IMO this should be done on a case by case basis,
Integration tests however should be written and are a good case to use Testcontainers.

## How to use them

We will take the  [spring petclinic kotlin](https://github.com/spring-petclinic/spring-petclinic-kotlin), to demonstrate the use of testcontainers.
The petclinic project obviously makes great use of h2 and @DataJpaTest to write some integration tests, but most people don't use h2 in production, 
they probably use postgres, mysql or oracle.
So we can write an integration tests against Mysql, pretty much using everything that has been already wired up
The property ```
spring.datasource.initialization-mode=ALWAYS ``` makes initialization of the DDL and seed data DML super easy.

All that left is to start TestContainers in the beforeClass method and you are set.Also you need to swap
out the HikariConfig with one you have.

```
 @TestConfiguration
    class TestConfig {
        @Bean
        @Throws(SQLException::class)
        fun dataSource(): DataSource {
            var hikariConfig = HikariConfig()
            val portMapping = VisitPetRepositoryContainersTest.Companion.mysqlSQLContainer.getMappedPort(3306)
            hikariConfig.jdbcUrl = "jdbc:mysql://localhost:${portMapping}/test"
            hikariConfig.username = "root"
            hikariConfig.password = "test"
            hikariConfig.isAllowPoolSuspension = true

            val ds = HikariDataSource(hikariConfig);
            return ds
        }
    }


    companion object {
        var mysqlSQLContainer: KMysqlContainer = KMysqlContainer("mysql:5.7.8")

        @ClassRule
        @JvmField
        val resource: ExternalResource = object : ExternalResource() {


            override fun before() {
                println("ClassRule Before")
                this@Companion.mysqlSQLContainer.start()
            }


            override fun after() {
                println("ClassRule After")
            }
        }
    }



    class KMysqlContainer(imageName: String?) : MySQLContainer<KMysqlContainer>(imageName)

``` 

I had to wrap the MySQLContainer TestContainer class in KMysqlContainer because the way MySqlContainer uses Generics doesnt agree with 
Kotlin too well.TestContainers depends on construction of raw types and pattern like class C<SELF extends C<SELF>>. Unfortunately Kotlin, and probably other JVM languages don't like it
 - see https://youtrack.jetbrains.com/issue/KT-17186
 
 The above is an example of Specialized containers.
 
 [code for example above](https://github.com/arnabmitra/spring-petclinic-kotlin) 
 
 ### Specialized containers
 The specialized containers in TestContainers are created by extending the GenericContainer class. 
 Out of the box, one can use a containerized instance of a MySQL, PostgreSQL or Oracle database to test your data access layer code for complete compatibility.

```
 class KPostgresContainer(imageName: String) : PostgreSQLContainer<KPostgresContainer>(imageName)

 var postgresSQLContainer: KPostgresContainer = KPostgresContainer("postgres:9.5.9")
                .withDatabaseName("foo")
                .withUsername("bar")
                .withPassword("baz")
                .withExposedPorts(5432).waitingFor(LogMessageWaitStrategy()
                .withRegEx(".*database system is ready to accept connections.*\\s")
                .withTimes(2)
                .withStartupTimeout(Duration.of(60, SECONDS)))

```
### GenericContainer
With TestContainers you will notice the GenericContainer class being used quite frequently:

```
class KPostgresContainer(imageName: String) : PostgreSQLContainer<KPostgresContainer>(imageName)

 val genericContainer: KGenericContainer = KGenericContainer("some-image")
                        .withEnv("FOO_BAR", "baz")
                        .withNetworkMode("host")


```

### Custom containers
By extending GenericContainer it is possible to create custom container types. 
This is quite convenient if we need to encapsulate the related services and logic.


### Conclusion
TestContainers are a good idea with bringing isolation to you integration testing.
I think it can be a good uses case for using in integrated tests also.
