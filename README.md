# 그날쿠폰 데이터베이스 / MySql :open_file_folder:

## __1. 설계__ :open_file_folder:

- 설계에 들어가기 앞서
    - 개발 과정에서 데이터베이스 변경이 있을 수 있으나 기획과 디자인에서 최대한 완성도가 있어야 개발 과정에서 시간을 아낄 수 있다. 기획, 다지인에서 바뀌어버리면 데이터베이스가 변경되어야 하고, 이에 따른 서버와 클라이언트가 모두 변경되어야 하기 때문에 시간이 배로 들어가게 된다. 혼자 개발할 경우 쉽게 결정하지 못하고 자꾸 바꿀 수 있기 때문에 더욱 초기에 확실하게 결정해 놓는 것이 중요하다
    - 쿼리문 숙지 : 단순히 insert, select, update, delete 기본 쿼리만 사용하면 비효율적인 상황이 발생한다. 구체적인 기능들을 익히도록 한다.
- 테이블 간의 관계 설정 : 서버, 클라이언트에서도 그렇지만 메인이 되는 테이블을 두고 code 테이블, 유동 데이터 테이블, 파생 데이터 테이블을 분리하거나 수평적 관계 설정
- 테이블 작성시 고려 사항
    - (1) code : 다른 테이블로 들어가는 일정하게 정해진 값 . 예) 카테고리, 메인 카테고리, 서브 카테고리
    - (2) 키 : 중심이 되는 데이터와 파생되는 데이터의 관계, 그 안에서 데이터를 조회하고 수정하기 위한 연결점 설정이 중요하다. 각 테이블의 관계에 따라 데이터 중복 저장 여부와 쿼리 속도에 영향을 미칠 수 있기 때문에 시간이 조금 걸리더라도 초기 설계에 신경 쓰도록 한다.
    - (3) 쿼리 칼럼 : 서비스의 중심이 되는 카테고리 분류와 이를 담당하는 칼럼을 먼저 정한다. dday_info_detail의 카테고리 참고.
    - (4) 유동 테이터 : 유동적인 데이터는 따로 테이블로 뺴준다. dday_info_detail description, fileimage 참고
    - (5) 데이터 유효성 : dday_info_detail의 status 참고
- 보안
    - ssl
- 미해결 : 화면 설계가 끝난 후 서버측에서 먼저 API를 넘겨주고 프론트에서 그 후에 API를 받아서 개발하는데, 문제는 어떻게 서버측에서 프론트가 사용할 디테일한 부분까지도 신경을 쓰고 데이터베이스 설계를 하는지이다. 물론 그렇게 하지 않으면 추후 변경해야 하는 것은 맞으나, 데이터베이스 설계를 마친 후 안드로이드를 개발하면서 무수히 데이터베이스를 바꿨던 것을 생각해 볼 때 아직 실력이 부족해서 한번에 설계를 끝내지 못하는 것인지 아니면 원래 협의를 통해서 중간에 설계를 변경해 나가는지 생각해 볼 필요 있음.
- 미해결 : 유동적인 데이터를 string으로 묶어서 처리하는 것이 좋은가 아니면 따로 테이블로 빼줘야 하는가

## __2. 구현__ :open_file_folder:

- code
    - 실제 code라는 테이블이 아닌 특정 혹은 각 테이블에서 사용할 리스트가 될만한 데이터를 저장해 둔다. 예를 들어 the_day에 해당하는 테이블과 칼럼을 미리 작성해 두면 여기저기서 반복하지 않고 참조를 하나로 통일할 수 있다.

- dday_info_detail
    - 쿠폰 데이터 테이블로 핵심 데이터를 담고 있다. 
    - no : 쿠폰 데이터를 구분해주는 핵심 키로 다른 테이블에서 접근할 수 있는 키가 된다. 중복 저장하지 않기 위해서는 이렇게 테이블 간 관계 설정과 키 설정이 필요함
    - the_day, main_category, sub_category : 초기에는 따로 상위 테이블을 만드려고 했으나 앱에서 the_day를 중심으로 데이터를 분류하며 각 업체가 the_day에 해당하는 쿠폰을 1개씩밖에 발행하지 않고 다른 the_day가 되면 같은 업체라 하더라고 다른 카테고리에서 다른 쿠폰으로 보여지기 때문에 따로 저장한다. 서비스의 핵심이 되는 카테고리 분류와 이를 담당하는 칼럼을 헷갈리지 않아야 한다.
    - description : 하나의 string으로 묶어서 splitd으로 나눠 사용하도록 설계는 했으나 데이터가 상당히 크고 무엇보다 개수가 유동적이기 때문에 따로 테이블로 뺴줘야 할 듯 하다. 
    - start_date, end_date : 데이터베이스에서 사용하는 시간과 서버, 클라이언트에서 사용하는 시간이 서로 다르고, date, datetime, timestamp 상호 호환이 되지 않는다. 때에 따라서 서버와 데이터베이스가 같은 이름의 datetime을 사용함에도 호환이 되지 않은 경우가 있음. 물론 시간은 서버를 기준으로 작성한다.
    - business_hour : 데어터가 크지 않고 예상 범위에서 string으로 묶어서 처리할 수 있기 떄문에 string으로 처리해도 상관 없을 듯 하지만 영업시간으로 쿼리를 한다거나 분류를 할 때 상당히 불편해 질 수 있을 듯 하다.
    - status : 쿠폰 데이터의 상태관리 칼럼. 간단하게 데이터의 유효성을 검증할 수 있다.

- dday_info_photo_url
    - dday_info_detail의 사진 url 테이블
    - dday_info_detail의 no을 parnt_no값으로 갖는다

- dday_info_version
    - 서버 데이터를 로컬 데이터베이스에 받아서 사용할 경우 서버 데이터와 로컬 버전을 확인하기 위한 테이블
    - version

- favorite_info
    - 즐겨찾기 테이블
    - dday_info_detail의 no을 parnt_no값으로 갖는다

- member_info
    - 유저 테이블
    - 비밀번호로 로그인하는 유저의 경우 절대 비밀번호를 그대로 저장하지 않는다.

- request_info
    - 쿠폰 요청 테이블

- wrong_info
    - 잠롯된 정보 테이블


## __3. 데이터__ :open_file_folder:

#### (1) Excel to MySql / 대량 데이터 삽입

#### (2) 데이터 크롤링


## __4. MySql 쿼리__ :open_file_folder:

디테일한 기능을 익히면 중복처리, 쿼리속도 향상 등 데이터 관리를 효율적으로 할 수 있다.

- 한 개 데이터 삽입
    - 데이터 추가시 중복처리 꼭 해주자
    ```javaScript
    INSERT INTO tablename (member_id, parent_no, reg_date) VALUES(?,?,?)
    ```

- 여러개 데이터 삽입
    - 한 개 데이터 삽입을 for 문으로 여러번 호출하는 것보다 쿼리문을 사용하는 것이 빠르다
    ```javaScript
    INSERT INTO tablename (member_id, parent_no, reg_date) VALUES (values);
    ```

- 한 개 데이터 업데이트
    ```javaScript
    UPDATE tablename SET member_id = ? WHERE member_id = ?;
    ```

- 여러개 데이터 업데이트 / 다중 조건문
    ```javaScript
    UPDATE tablename SET favorite_count = IF (isnull(favorite_count), 1, favorite_count+1) WHERE no IN(values);
    ```

- 한 개 데이터 삭제
    ```javaScript
    DELETE FROM tablename WHERE member_id = ?;
    ```

- 여러개 데이터 삭제 / 다중 조건문
    ```javaScript
    DELETE FROM tablename WHERE member_id = ? AND parent_no IN (values);
    ```

- favorite_count가 null이면 1 아니면 +1
    ```javaScript
    UPDATE tablename SET favorite_count = IF (isnull(favorite_count), 1, favorite_count+1) WHERE no = ?;
    ```

- 여러개를 한꺼번에 업데이트, 삭제, select 할 경우 
    - distict, group by는 insert 할 때 사용함
    - function() 작성

- DB에서 현재 위치 기반 r km 반경 내 데이터 쿼리
    ```javaScript
    function select(qs, callback){
        var user_lat = parseFloat(qs.latitude);
        var user_long = parseFloat(qs.longitude);
        var radius = qs.radius; // 결과값이 km인 값을 params로 전달해줘야 함
        var distance = '(6371*(acos(cos(radians(?)) * cos(radians(latitude)) * cos(radians(longitude)-radians(?))'
                        +'+ sin(radians(?)) * sin(radians(latitude)))))';
        var query = 'SELECT *, '+distance+' AS distance FROM '+tablename+' HAVING distance <= '+radius+' ORDER BY distance';   
        var values = [user_lat, user_long, user_lat];
        db.executeByValues(query, values, callback);
    }
    ```

## __5. DB 호스팅__ :open_file_folder:

#### (1) DB 호스팅

#### (2) CLI

    호스팅 서버를 구입할 때 mysql이 설치되어 있기도 하고 따로 설치해야 하기도 한다. 이번 호스팅은 mysql이 설치되어 있음. centOS mysql-v5.6.35

- 리눅스 명령어
    - 실행중인 mysql 확인 : ps -ef | grep mysqld
    - 경로 확인 : which mysql
    - 설치 확인 : rpm -q mysqlg
- A. MySql 접속
    - mysql
- B. 사용자 생성
    - 사용자 데이터베이스 접속 : use mysql
    - 사용자 생성
        - 원격지 접속 %을 쓸 때 ''넣어줘야 함
        - create user 아이디@'localhost' identified by '비밀번호'
        - create user 아이디@'%' identified by '비밀번호';
        - insert into user (Host, User, Password) VALUES('localhost', '' , '');
        - insert into user (Host, User, Password) VALUES('%', '' , '');
    - 사용자 검색 : select user, host, password from user;
    - 사용자 삭제 : drop user 아이디@localhost;
- C. 권한설정
    - root 계정으로 접속 : -u, -p 할 필요 없이 mysql로 접속한다. 먼저는 ssh로 접속할 때 아이디와 헷갈리지 않아야 한다. 그리고 원격지이지만 원격 서버의 로컬서버이기 때문에 로컬호스트이고, mysql에 접속하는 아이디가 루트가 되는 것이다. 참고로 초기 루트에는 mysql 데이터베이스를 다룰 수 있는 권한이 잆어서 함부로 삭제하면 단순히 사용자를 검색할 수도 없고, 당연히 추가로 권할 설정도 할 수 없다. 모든 사용자 생성, 삭제, 업데이트, 검색과 권한 설정을 담당한다. 
    - 권한 설정 : GRANT ALL PRIVILEGES ON *.* TO 아이디@localhost||%. 
        - identified by '비밀번호'로 할 경우 비밀번호 변경됨
        - DB명.테이블 : 유저에게 특정 DB의 특정 테이블 권한 부여
        - DB명.* : 유저에게 특정 DB의 모든 테이블 권한 부여
        - *.* : 유저에게 모든 DB의 모든 테이블 권한 부여
    - flush privileges : 적용
- D. 확인
	- show grants
	- show grants for 아이디@localhost
- E. 권한 해제
    - revoke all on *.* from 아이디

- #. root 밖에서 권한 설정하기 : root가 권한을 가지고 있다면 필요 없는데, user를 관리하다가 root의 권한까지 삭제해 버림. mysql 테이블을 관리할 수 있는 권한이 없기 때문에 외부에서 다시 설정해줘야 한다.
    - A. 현재 실행중인 mysqld 프로세스 종료
        - killall mysqld, mysqld_safe
        - systemctl stop mysqld & : 센토스7부터는 init.d/mysql에서 mysqld start, stop 하지 않는군
    - B. 권한 없이 사용 가능한 테이블로 접근
        - mysqld_safe --skip-grant-tables : 권한 없이 테이블 사용 가능하도록 한다. 다만 이 테이블에서는 GRANT 명령어를 사용할 수 없다.
    - 권한 수정
        - &(백그라운드)로 실행하지 않았다면 다른 터미널 창에서 mysql 실행
        ```mysql
        update mysql.user 
        set Select_priv='Y',Insert_priv='Y',Update_priv='Y',Delete_priv='Y',Create_priv='Y',Drop_priv='Y',
            Reload_priv='Y',Shutdown_priv='Y',Process_priv='Y',File_priv='Y',Grant_priv='Y',References_priv='Y',
            Index_priv='Y',Alter_priv='Y',Show_db_priv='Y',Super_priv='Y',Create_tmp_table_priv='Y',Lock_tables_priv='Y',
            Execute_priv='Y',Repl_slave_priv='Y',Repl_client_priv='Y',Create_view_priv='Y',Show_view_priv='Y',
            Create_routine_priv='Y',Alter_routine_priv='Y',Create_user_priv='Y',Event_priv='Y',Trigger_priv='Y' 
        where User='root';
        ```
        - FLUSH PRIVILEGES
    - mysqld_safe --skip-grant-tables 프로세스 종료
        - killall mysql
    - mysqld 
        - systemctl start mysqld

#### (3) 워크벤치 연결

- standard TCP/IP over SSH
- SSH Hostname : 제공받은 IP
- SSH Username : 호스팅 업체에서 제공받은 아이디(root)
- SSH Password : 호스팅 업체에서 제공받은 비밀번호(*******)
- SSH Key File : 무시
- MySQL Hostname : 127.0.0.1 임은 위에서 호스팅서버로 접속했고 그 컴퓨터 안에서 찾아가는 것이기 때문에 로컬서버로 접속하는 것
- MySQL Server Port : 3306인데 호스팅 서버에서 inbound 포트로 3306을 열어줘야 함. 참고로 3306은 호스팅 서버에 mysql을 설치하면서 자동으로 해 주는듯. 확인해 봐야 한다
- Username : ''
- Password : ''



