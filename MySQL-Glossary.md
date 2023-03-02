# MySQL 용어집
원본: https://dev.mysql.com/doc/refman/8.0/en/glossary.html

## C
* consistent read
동시에 실행되는 다른 트랜잭션에 의해 수행된 변경 사항과 상관없이, 특정 시점을 기준으로 쿼리 결과를 표시하기 위해 스냅샷을 사용하는 읽기 연산. 쿼리된 데이터가 다른 트랜잭션에 의해 변경된 경우, 원본 데이터는 undo log의 내용을 기반으로 재구성된다. 이 기술은 다른 트랜잭션이 끝날 때까지 트랜잭션을 기다리게 함으로써 동시성을 감소시킬 수 있는 일부 locking 이슈를 방지한다.
<br>`REAPEATABLE READ` 격리 수준에서, 스냅샷은 첫 번째 읽기 연산이 수행될 때에 기반합니다. `READ COMMITTED` 격리 수준에서, 스냅샷은 각각의 consistent read 연산 시간마다 재설정됩니다.
<br>Consistent read는 InnoDB가 `READ COMMITED` 와 `REAPETABLE READ` 격리 수준에서 `SELECT`문을 수행할 때 기본 모드이다. consistent read는 접근하는 테이블에 어떠한 lock도 세팅하기 않기 때문에, consistent read가 수행되는 동안 다른 세션에서 해당 테이블을 자유롭게 수정 가능하다.
<br>적용 가능한 격리 수준에 대한 세부사항은 [15.7.2.3 "Consistent Nonlocking Reads”](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html)를 참조.
<br>참고: [concurrency](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_concurrency), [isolation level](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_isolation_level), [locking](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_locking), [READ COMMITTED](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_read_committed), [REPEATABLE READ](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_repeatable_read), [snapshot](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_snapshot), [transaction](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_transaction), [undo log](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_undo_log).

## G
* gap
InnoDB 인덱스 데이터 구조에서 새 값을 삽입할 수 있는 위치입니다. 당신이 `SELECT ... FOR UPDATE`와 같은 연산으로 행의 집합을 잠그면, InnoDB는 인덱스의 실제 값 뿐만 아니라 gap에도 적용되는 lock을 생성할 수 있습니다. 예를 들어, 수정을 위해 10보다 큰 모든 값을 선택한다면, gap lock은 다른 트랜잭션이 10보다 큰 값을 삽입하는 것을 방지합니다. supremum record 와 infimun record는 현재 인덱스보다 크거나 작은 모든 값을 포함한 gap을 나타냅니다.
<br>참고: [concurrency](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_concurrency), [gap lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_gap_lock), [index](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_index), [infimum record](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_infimum_record), [isolation level](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_isolation_level), [supremum record](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_supremum_record).

## S
* snapshot
특정 시간의 데이터 표현으로, 다른 트랜잭션에 의해 변경이 커밋되더라도 동일하게 유지됩니다. Consistent read를 허용하기 위해 특정 격리 수준에서 사용됩니다.

참고: [commit](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_commit), [consistent read]([consistent read](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_consistent_read)), [isolation level](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_isolation_level), [transaction](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_transaction).



## U
* undo log
활성화된 트랜잭션에 의해 수정된 데이터의 복사본을 저장하는 영역입니다. 다른 트랜잭션에서 원본 데이터를 봐야한다면(consistent read 연산의 일부로), 이 공간에서 수정되지 않은 데이터가 반환됩니다.

MySQL 5.6과 5.7에서는 undo log가 undo tablespaces에 존재할 수 있도록 [innodb_undo_tablespaces](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_undo_tablespaces) 변수를 사용할 수 있으며, SSD와 같은 다른 저장 장치에 위치할 수 있다. MySQL 8.0에서 undo log는 MySQL이 초기화 될때 생성되는 두 개의 기본 undo tablespaces에 존재하고, 추가적인 undo tablespace는 CREATE UNDO TABLESPACE 구문을 통해 생성될 수 있다.

undo log는 insert undo buffer와 update undo buffer 두 개로 나뉩니다.

참고: [consistent read](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_consistent_read), [rollback segment](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_rollback_segment), [SSD](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_ssd), [system tablespace](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_system_tablespace), [transaction](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_transaction), [undo tablespace](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_undo_tablespace).
