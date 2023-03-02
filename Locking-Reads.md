# 15.7.2.4 Locking Reads
<!-- If you query data and then insert or update related data within the same transaction, the regular SELECT statement does not give enough protection. Other transactions can update or delete the same rows you just queried. InnoDB supports two types of locking reads that offer extra safety: -->

* SELECT ... FOR SHARE
읽고있는 모든 행에 shared mode lock을 세팅한다. 다른 세션은 행을 읽을 수 있지만, 당신의 트랜잭션이 커밋될 때까지 수정할 수는 없습니다. 이러한 행 중 하나가 아직 커밋되지 않은 다른 트랜잭션에 의해 변경된 경우 쿼리는 해당 트랜잭션이 끝날 때까지 기다린 다음 최신 값을 사용합니다.

> Note
`SELECT ... FOR SHARE` 는 `SELECT ... LOCK IN SHARE MODE` 를 대체하지만, 이전 버전과의 호환성을 위해 `LOCK IN SHARE MODE`는 존재합니다. 두 개의 구문은 동일합니다. 하지만, `FOR SHARE` 은 `OF table_name`, `NOWAIT` 그리고 `SKIP LOCKED` 옵션을 지원합니다.. [NOWAIT 및 SKIP LOCKED 을 활용한 Locking Read Concurrency](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html#innodb-locking-reads-nowait-skip-locked) 참고.

