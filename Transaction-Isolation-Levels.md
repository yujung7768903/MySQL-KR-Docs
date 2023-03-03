# 15.7.2.1 Transaction Isolation Levels
트랜잭션 격리는 데이터베이스 처리의 기초 중 하나입니다. 격리는 ACID의 I에 해당합니다. 격리 수준은 여러 트랜잭션이 동시에 변경하고 쿼리를 수행할 때 결과의 성능과 신뢰성, 일관성 및 재현성 사이의 균형을 미세하게 조정하는 설정입니다.

InnoDB는 SQL:1992 표준에 묘사된 총 4개의 트랜잭션 격리 수준(READ UNCOMMITTED, READ COMMITED, REPEATABLE READ, SERIALIZABLE)을 모두 제공합니다. InnoDB의 기본 격리 수준은 REPEATABLE READ입니다.

사용자는 SET TRANSACTION 문을 사용하여 싱글 세션 또는 이후 모든 연결에 대한 격리수준을 변경할 수 있습니다. 모든 연결에 대한 서버의 기본 격리 수준을 변경하기 위해서는 명령줄이나 옵션 파일에 --transaction-isolation 옵션을 사용하세요. 격리 수준 및 설정 구문에 대한 자세한 내용은 [13.3.7 "SET TRANSACTION Statement" 참고](https://dev.mysql.com/doc/refman/8.0/en/set-transaction.html)

<!-- InnoDB supports each of the transaction isolation levels described here using different locking strategies. You can enforce a high degree of consistency with the default REPEATABLE READ level, for operations on crucial data where ACID compliance is important. Or you can relax the consistency rules with READ COMMITTED or even READ UNCOMMITTED, in situations such as bulk reporting where precise consistency and repeatable results are less important than minimizing the amount of overhead for locking. SERIALIZABLE enforces even stricter rules than REPEATABLE READ, and is used mainly in specialized situations, such as with XA transactions and for troubleshooting issues with concurrency and deadlocks.

InnoDB는 다른 종류의 [locking](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_locking) 전략을 사용하여 묘사된 각각의 격리수준을 지원합니다. 당신은 ACID 준수가 중요한 데이터에 대해서는 기본 레벨인 REPEATABLE READ 를 사용하여 높은 정도의 일관성을 보장할 수 있습니다.

InnoDB는 서로 다른 잠금 전략을 사용하여 여기에 설명된 각각의 트랜잭션 격리 레벨을 지원합니다. 산 준수가 중요한 중요한 데이터에 대한 작업에 대해 기본 반복 읽기 수준으로 높은 수준의 일관성을 적용할 수 있습니다. 또는 대량 보고와 같이 정확한 일관성과 반복 가능한 결과가 잠금에 대한 오버헤드를 최소화하는 것보다 덜 중요한 상황에서 READ COMMITED 또는 READ UNCOMITED를 사용하여 일관성 규칙을 완화할 수 있습니다. 직렬화는 반복 가능한 읽기보다 더 엄격한 규칙을 적용하며, XA 트랜잭션과 같은 특수한 상황에서 동시성 및 교착 상태 문제를 해결하는 데 주로 사용됩니다. -->


다음 리스트는 MySQL이 다양한 트랜잭션 레벨을 어떻게 지원하는지 나타낸다. 목록은 가장 흔하게 사용되는 것부터 적게 사용되는 것까지 사용 되는 수준에 따라 정렬됩니다.

* REPEATABLE READ       
    InnoDB 에서 기본 격리 레벨에 해당합니다. [Consistent reads](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_consistent_read)는 첫번째 읽기 시점에 저장된 [스냅샷](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_snapshot)을 읽습니다. 즉, 동일한 트랜잭션 안에서 몇 가지 기본(nonlocking) `SELECT` 문을 실행하는 경우, 이러한 `SELECT` 문은 서로에 대해서도 일치합니다. [15.7.2.3 “Consistent Nonlocking Reads” 참고](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html)
    <br>[locking read](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_locking_read) (`FOR UPDATE` 또는 `FOR SHARE` 가 활용된 `SELECT`), `UPDATE` 그리고 `DELETE` 문의 경우, locking은 그 구문이 고유한 검색 조건을 가진 고유한 인덱스를 사용하는지 또는 범위 타입의 검색 조건을 사용하는지에 다라 달라집니다.
    <br>
    
    * 고유한 검색 조건을 가진 고유한 인덱스의 경우, InnoDB는 검색하는 인덱스 레코드만 잠그고, 이전의 [gap](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_gap)은 잠그지 않습니다.
  
    * 다른 검색 조건의 경우, InnoDB는 해당 범위에서 다루는 gap에 대해 다른 세션에 의한 삽입을 막기 위해 [gap lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_gap_lock) 또는 [next-key lock]을 사용하여 스캔한 인덱스 범위를 잠급니다. gap lock 또는 [next-key lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_next_key_lock)에 대한 더 많은 정보를 원할 경우, [15.7.1 "InnoDB Locking"](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html) 참고
    <br>

* READ COMMITTED        
    같은 트랜잭션에 있더라도 각각의 Consistent read는 그들만의 새로운 스냅샷을 세팅하고 읽습니다. consistent reads에 대한 정보를 위해서는 [15.7.2.3 “Consistent Nonlocking Reads”](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html) 참고.
    <br>[locking read](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_locking_read) (`FOR UPDATE` 또는 `FOR SHARE` 가 활용된 `SELECT`), `UPDATE` 그리고 `DELETE` 문의 경우, InnoDB는 인덱스 레코드만 잠그고, 이전의 gap은 잠그지 않으므로 잠금된 레코드 다음에 새로운 레코드를 자유롭게 삽입할 수 있습니다. Gap locking은 외래키 제약 조건 확인과 중복 키 확인을 위해서만 사용됩니다.
    <br>gap locking이 비활성화되기 때문에, 다른 세션에서 갭에 새로운 행을 추가할 수 있으므로 phantom row 문제가 발생할 수 있습니다. [15.7.4 "Phantom Rows" 참고](https://dev.mysql.com/doc/refman/8.0/en/innodb-next-key-locking.html)
    <br>READ COMMITED 격리 수준에서는 row-based binary logging(행 기반의 이진 로그)만 지원됩니다. 당신이 READ COMMITED를 binlog_format=MIXED와 함께 사용한다면, 서버는 자동으로 row-based logging를 사용할 것입니다.
    <br>READM COMMITED는 다음과 같은 추가적인 효과가 있습니다:
    <br>

    * UPDATE 또는 DELETE 문의 경우, InnoDB는 업데이트나 삭제할 행에 대해서만 잠금을 유지합니다. 일치하는 않는 행에 대한 Record locks은 MySQL이 WHERE 조건을 평가한 이후에 해제됩니다. 이는 deadlock에 대한 가는성을 크게 줄이지만, 여전히 발생할 수 있습니다.

    * UPDATE 문의 경우, 행이 이미 잠금되었다면, InnoDB는 MySQL이 행이 UPDATE의 WHERE 조건과 일치하는지에 대해 판단할 수 있도록 "semi-consistent" read를 수행하고 가장 최신의 커밋된 버전을 반환합니다. 행이 일치한다면 (업데이트 해야함), MySQL은 그 행을 다시 읽고, 이번에는 InnoDB가 잠그거나 잠그는 것을 기다립니다.

    <br>이 테이블을 가지고, 다음의 예시를 고려해보십시오:
    
    ```sql
    CREATE TABLE t (a INT NOT NULL, b INT) ENGINE = InnoDB;
    INSERT INTO t VALUES (1,2),(2,3),(3,2),(4,3),(5,2);
    COMMIT;
    ```
    이러한 경우에는, 테이블에 인덱스가 존재하지 않으므로, searches와 index scan는 record locking을 위해 인덱스 컬럼이 아닌 hidden clustered index를 사용합니다. [15.6.2.1 "Clustered and Secondary Indexes" 참고](https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html)

    <br>첫 번째 세션이 다음의 구문을 사용하여 UPDATE문을 수행한다고 가정해보십시오:

    ```sql
    # Session A
    START TRANSACTION;
    UPDATE t SET b = 5 WHERE b = 3;
    ```
    두 번째 세션이 첫 번째 세션에 이어 다음 구문을 실행하여 UPDATE문을 수행한다고 가정해보십시오:

    ```sql
    # Session B
    UPDATE t SET b = 4 WHERE b = 2;
    ```

    InnoDB는 각각의 UPDATE문을 수행하면서, 각각의 행에 대한 exclusive lock을 획득한 후 수정할지 결정합니다. InnoDB가 행을 수정하지 않으면, lock을 해제합니다. 그렇지 않으면 InnoDB는 트랜잭션이 끝날때까지 lock을 유지합니다. 이는 다음과 같이 트랜잭션 처리에 영향을 줍니다.

    <br>기본 REPEATABLE READ 격리 수준을 사용할 때, 첫 번째 UPDATE는 읽은 각각의 행에 대한 x-lock을 획득하고 어떤 것도 해제하지 않습니다:

    ```
    x-lock(1,2); x-lock 유지
    x-lock(2,3); update(2,3) to (2,5); x-lock 유지
    x-lock(3,2); x-lock 유지
    x-lock(4,3); update(4,3) to (4,5); x-lock 유지
    x-lock(5,2); x-lock 유지
    ```

    두 번째 UPDATE는 lock을 얻는 즉시 차단하고(첫 번째 update가 모든 행에 대한 lock을 유지하고 있기 때문에), 첫 번째 UPDATE가 커밋 또는 롤백 될 때까지 진행하지 않습니다.

    ```
    x-lock(1,2); 첫 번째 UPDATE가 커밋 또는 롤백될 때까지 차단 후 대기
    ```

    READ COMMITTED가 대신 사용된다면, 첫 번째 UPDATE는 읽은 각각의 행에 대해 x-lock을 얻고 읽은 후 수정되지 않는 행에 대해서는 lock을 해제합니다.

    ```
    x-lock(1,2); (1,2) lock 해제
    x-lock(2,3); update(2,3) to (2,5); x-lock 유지
    x-lock(3,2); (3,2) lock 해제
    x-lock(4,3); update(4,3) to (4,5); x-lock 유지
    x-lock(5,2); (5,2) lock 해제
    ```
    두 번째 UPDATE 문의 경우, InnoDB는 MySQL이 행이 UPDATE의 WHERE 조건과 일치하는지에 대해 판단할 수 있도록 "semi-consistent" read를 수행하고 가장 최신의 커밋된 버전을 반환합니다:

    ```
    x-lock(1,2); update(1,2) to (1,4); x-lock 유지
    x-lock(2,3); (2,3) lock 해제
    x-lock(3,2); update(3,2) to (3,4); x-lock 유지
    x-lock(4,3); (4,3) lock 해제
    x-lock(5,2); update(5,2) to (5,4); x-lock 유지
    ```

    그러나, WHERE 조건이 인덱스 컬럼을 포함하고, InnoDB가 인덱스를 사용하는 경우, record lock을 획득하고 유지할 때 인덱스 컬럼만 고려됩니다. 아래의 예시에서 첫 번째 UPDATE 문은 b가 2인 각각의 행에 대해 x-lock을 얻고 유지합니다. 두 번째 UPDATE도 b 컬럼으로 정의된 인덱스를 사용하므로 동일한 행에 대해 x-lock을 획득하려고 시도할 때 차단합니다.

    ```sql
    CREATE TABLE t (a INT NOT NULL, b INT, c INT, INDEX (b)) ENGINE = InnoDB;
    INSERT INTO t VALUES (1,2,3),(2,2,4);
    COMMIT;

    # Session A
    START TRANSACTION;
    UPDATE t SET b = 3 WHERE b = 2 AND c = 3;

    # Session B
    UPDATE t SET b = 4 WHERE b = 2 AND c = 4;
    ```
    REAM COMMITTED 격리 수준은 시작 시 설정하거나 런타임에 변경할 수 있습니다. 런타임 시에는 모든 세션에 대해 전역적으로 설정하거나 세션별로 개별적으로 설정할 수 있습니다.

<br>

* READ UNCOMMITTED      
SELECT 구문은 nonlocking 환경에서 수행되지만, 이전 버전의 행이 사용될 수 있습니다. 따라서 해당 격리 수준을 사용하면 읽기가 일관되지 않습니다. 이것은 dirty read 라고도 불립니다. 그렇지 않을 때 이 격리 수준은 READ COMMITED 와 비슷하게 작동합니다.

<br>

* SERIALIZABLE      
이 레벨은 REPEATABLE READ 와 비슷하지만, autocommit이 비활성화 되어 있다면, InnoDB는 기본 SELECT 문을 SELECT ... FOR SHARE 로 변경합니다. autocommit이 활성화된 경우, SELECT는 자체 트랜잭션입니다. 따라서 읽기 전용으로 알려져있고, consistent(nonlocking) read로 수행되면 직렬화될 수 있으며 다른 트랜잭션에 대해 차단할 필요가 없습니다. (다른 트랜잭션이 선택된 행에 대해 수정할 때 일반 SELECT를 강제로 차단하려면, autocommit을 비활성화 하십시오.)

> Note
MySQL 8.0.22를 기준으로 , MySQL grant 테이블에서 데이터를 읽지만(join list 또는 서브쿼리를 통해) 이를 수정하지 않는 DML 연산은 격리 수준에 관계없이 MySQL grant table에 대한 read lock을 획득하지 않는다. [Grant Table Concurrency 참고](https://dev.mysql.com/doc/refman/8.0/en/grant-tables.html#grant-tables-concurrency)