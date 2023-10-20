# 6.2 테이블 압축
- 테이블 압축은 페이지 압축과 달리 운영체제나 하드웨어의 제약 없이 사용가능하다.
  - 그래도 몇 가지 아래와 단점이 존재한다
    - 버퍼 풀 공간 활용률이 낮음
    - 쿼리 처리 성능이 낮음
    - 빈번한 데이터 변경 시 압축률이 떨어짐

## 6.2.1 압축 테이블 생성
- 압축 테이블을 생성하려면 먼저 별도의 테이블 스페이스를 사용해야 한다. (innodb_file_per_table 시스템 변수 ON 상태)
- 테이블을 생성할 때 ROW_FORMAT=COMPRESSED 옵션 설정, KEY_BLOCK_SIZE 옵션을 이용해서 압축된 타깃의 사이즈를 명시한다.
  - KEY_BLOCK_SIZE는 2n(n>=2)으로만 설정할 수 있다.
  - 예를들어 현재 페이지가 16KB 라면 4KB 또는 8KB로만 설정할 수 있다.

```SQL
CREATE TABLE COMPRESSED_TABLE(
	C1 INT PRIMARY KEY,
)
ROW_FORMAT=COMPRESSED
KEY_BLOCK_SIZE=8;
```
- InnoDB 스토리지 엔진의 압축 절차 예시 (페이지 크기 = 16KB, KEY_BLOCK_SIZE = 8)
  - 16KB의 데이터 페이지를 압축
    - 압축된 결과가 8KB이하이면 그대로 디스크에 저장(압축 완료)
    - 압축된 결과가 8KB를 초과하면 원본 페이지를 스플릿해서 2개의 페이지에 8KB 씩 저장
  - 나뉜 페이지 각각에 대해 1번 단계를 반복 실행
- 중요한 것은 페이지의 압축 결과가 목표 크기(KEY_BLOCK_SIZE)보다 작거나 같을 때까지 반복해서 페이지를 스플릿하는 것이다.

## 6.2.2 KEY_BLOCK_SIZE 결정
- 테이블 압축에서 가장 중요한 부분이 압축된 결과가 어느 정도가 될지 예측해서 KEY_BLOCK_SIZE를 결정하는 것이다.
- 샘플 데이터를 저장해보고 적절한지 판단하는 것이 좋다.

```SQL
SELECT table_name, index_name, compress_ops, compress_ops_ok,
	(compress_ops-compress_ops_ok)/compress_ops * 100 as compress_failure_pct
FROM information_schema.INNODB_CMP_PER_INDEX;
```
- 압축 실패율이 높다고 해서 압축을 사용하지 말아야 하는것은 아니다.
- 반대로 압축 실패욜이 낮다고 해서 압축을 해야한다는 것도 아니다.
- 압축 알고리즘은 많은 CPU자원을 소모한다.

## 6.2.3 압축된 페이지의 버퍼 풀 적재 및 사용
- 압축된 테이블의 데이터 페이지를 버퍼 풀에 적재하면 압축된 상태와 압축이 해제된 상태 2개 버전을 관리한다.
- 그래서 InnoDB 스토리지 엔진은 압축된 그대로의 데이터 페이지를 관리하는 LRU, 압축이 해제된 페이지를 관리하는 Unzip_LRU 리스트를 별도로 관리한다.
- 압축된 테이블에 대해서는 버퍼 풀의 공간을 이중으로 사용함으로써 메모리를 낭비하는 효과가 있다.
  - 이를 관리하기위해 Adaptive 알고리즘을 사용한다.

## 6.2.4 테이블 압축 관련 설정
- 테이블 압축에 연관된 여러 시스템 변수가 있다. 이는 압축 실패율을 낮추기 위한 튜닝 포인트를 제공한다.
  - innodb_cmp_per_index_enabled
  - innodb_compression_level
  - innodb_compression_failure_threshold_pct, innodb_compression_pad_pct_max
  - innodb_log_compressed_pages