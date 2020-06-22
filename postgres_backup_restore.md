## PostgreSQL 데이터베이스의 `Backup`과 `Restore`

### `pg_dump`를 이용해서 디비 백업하기
pg_dump란 SQL 스크립트 파일 또는 tar 등의 파일형태로 데이터베이스를 추출합니다.

```
$ pg_dump --dbname=DBname --host=host --port=5432 --username=username --password --format=p --file=./mydb.sql --schema=mydb
```
* 옵션
  - dbname: 연결할 데이터베이스의 이름
  - host: 서버의 호스트 이름
  - port: 서버의 포트 번호
  - username: 데이터베이스에 접속할 유저 이름
  - password: 명령어 실행 후 user에 대한 password 입력
  - format: 출력 포맷
    * p: plain-text SQL 스크립트 파일 (default)
    * c: custom-format (pg_restore input format)
    * d: directory-format (pg_restore input format)
  - tar: tar-format
  - file: 출력 파일경로 지정
  - schema: 매치된 스키마만 덤프

### `psql`의 `f`옵션을 이용해서 디비 복원하기
#### User 생성
1. create role 
```
CREATE ROLE myuser WITH CREATEDB CREATEROLE LOGIN PASSWORD 'mypassword';
```

2. pg_hba.conf 파일 수정
```
$ sudo vi /etc/postgresql/12/main/pg_hba.conf
... local   all   all   peer ...
```
peer를 md5로 변경

3. conf 파일 reload
```
SELECT pg_reload_conf();
```

4. db 접속 확인
```
psql -d DBname -U myuser
```

#### 데이터베이스 restore
1. ssh로 파일 전송

```
$ scp (전송할 파일) (아이디@전송할 서버 주소):(저장될 서버의 경로)
```

2. psql 명령어 이용
```
psql -U myuser -d DBname < mydb.sql
```
