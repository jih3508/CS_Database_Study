# 7.2  keyring_file 플러그인 설치
- keyring_file 플러그인은 커뮤니티 에디션에서 사용 가능한 플러그인이다.
- 테이블스페이스 키를 암호화하기 위한 마스터 키를 디스크의 파일로 관리한다.
  - 이때 마스터 키는 평문으로 디스크에 저장된다.
- 마스터 키 초기화
  - ALTER INSTANCE ROTATE INNODB MASTER KEY;