# 데이터베이스(database)

Laravel 의 데이터베이스를 사용하면서 다중 연결 설정을 통한 부한분산 처리와 Query 처리할 때 추가적인 처리가 가능한 구조를 제공합니다. xe.php 에서 database 설정을 통해 여러개의 connection 을 설정 할 수 있으며 패키지별로 다른 connection을 사용해 부하분산을 처리할 수 있습니다.

XE3 에서는 default, document, user 세개 connection 을 정의하고 있으며 laravel database connection 설정 및 xe database 설정을 통해 부하분산을 처리할 수 있습니다.

ProxyManager 는 Query 처리할 때 추가적인 처리를 지원하기 위한 인터페이스를 제공하며 DynamicField 는 Database 의 ProxyInterface 를 이용해 구현되었습니다.

Illuminate\Database 에서 제공하는 인터페이스에 따라 클래스를 구성했으며 `XeDB` 파사드는 `DB` 파사드의 거의 모든 메소드를 지원합니다. Laravel Database 문서를 숙지하고 아래 XE3 Database 에서 추가된 내용을 참고해서 사용하시기 바랍니다.

## 클래스 구조

### VirtualConnection
여러개의 논리적인 데이터베이스 연결을 사용하여 다중 커넥션 사용을 가능하도록 하고 서버에 트래픽이 증가할 경우 패키지별 데이터베이스 연결 설정을 변경하여 부하분산을 빠르게 처리할 수 있도록 합니다. 또한 하나의 논리적인 데이터베이스 연결에 여러개의 데이터베이스 연결 설정을 할 수 있도록 하고 Query 처리할 때 랜덤하게 커넥션을 사용하도록 구현하여 부하분산을 처리합니다.
물리적으로 다른 여러개의 커넥션에 대해서 트랜젝션을 사용할 수 있습니다. 여러개의 커넥션에서 각각 발생하는 트랜젝션을 관리하여 하나의 커넥션에서 처리되는 것과 같이 동작합니다. 
다이나믹 쿼리를 이용해서 ProxyManager 에 등록된 Proxy 들을 사용할 수 있습니다. 다이나믹 쿼리는 데이터베이스 CRUD 처리 시 발생하여 쿼리를 조작할 수 있도록 인터페이스를 제공합니다. ProxyInterface 의 인터페이스를 QueryBuilder 에서 각 메소드에서 필요한 인터페이스를 사용합니다.

### DatabaseCoupler
Illuminate\Database 와 Xpressengine\Database 를 연결시킵니다. XE3 의 데이터베이스는 VirtualConnection 이라는 가상의 연결을 바탕으로 처리됩니다.

### DatabaseHandler
DatabaseHandler 는 Laravel의 DatabaseManager(\Illuminate\Database\DatabaseManager)를 대체합니다. 

### ProxyManager

### TransactionHandler
가상연결로 처리되는 하나 이상의 데이터베이스 연결에 대해서 트랜젝션 을 처리합니다. 
DB1, DB2 에 insert 처리 할 때 물리적으로 다른 두개의 데이터베이스에 트랜젝션이 처리될 수 있도록 지원합니다.

### DynamicQuery

### DynamicBuilder


## 설정
config/xe.php 의 `database` 설정에 가상연결 설정이 정의되어 있습니다.
```php
'database' => [
        'default' => [
            'slave' => ['default'],
        ],
        'document' => [
            'slave' => ['default'],
        ],
        'user' => [
            'slave' => ['default'],
        ],
    ],
```

부하분산을 위해 slave 데이터베이스를 추가할 수 있습니다.
config/database.php 에서 데이터베이스 연결을 추가하고 추가한 연결을 slave 에 추가해 주면 됩니다.

문서와 회원의 부하분산을 처리하기 위해서 document, user 의 데이터베이스 연결 정보를 변경하면 조회에 대한 부하를 분산 시킬 수 있습니다.


## 쿼리 실행
> [기본적인 데이터베이스 사용](http://xpressengine.github.io/laravel-korean-docs/docs/5.0/database/) 의 내용을 숙지하시기 바랍니다.

기본적인 구조는 Laravel 의 쿼리 실행과 같습니다. 다만 XE3 에서 추가된 VirtualConnection 을 이용한 DynamicQuery 사용에 대한 부분만 익히면 됩니다.

`XeDB` 파사드를 사용해서 Database 객체에 접근할 수 있습니다.

### 일반적인 쿼리 실행

```php
$results = XeDB::select('select * from users where id = ?', [1]);
```
기본 연결 설정된 데이터베이스 연결을 이용해 질의합니다.
일반적인 쿼리 실행은 Laravel과 같습니다.


## 데이터베이스 트랜잭션
트랜잭션은 아래와 같이 사용합니다.
```php
XeDB::beginTransaction();
... query ...
XeDB::commit(); // or XeDB::rollback();
```
XE3 데이터베이스는 여러개의 연결에 대해서 트랜잭션을 처리합니다. 전체 커넥션 정보를 참고하고 있는 TransactionHandler 는 트랜잭션이 시작될 때 연결되어 있는 모든 가상연결에 트랙잭션을 시작하고 또한 새로 맺는 연결도 트랜잭션이 시작되도록 처리합니다.



## 쿼리빌더
> [쿼리빌더](http://xpressengine.github.io/laravel-korean-docs/docs/5.0/queries/) 의 내용을 숙지하시기 바랍니다.

XE3 의 데이터베이스를 사용하기 위해서 Laravel `DB` 파사드를 대신해서 `XeDB`를 사용하면 됩니다.
`XeDB`는 Laravel 에서 제공하는 `table()` 방식과 `dynamic()` 두가지 방식의 처리를 지원합니다.
