# oracle_subquery
서브쿼리 상관쿼리 비상관쿼리 


    select, insert, update, delete 구문안에 들어있는 또다른 select 문을 말한다.


    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- subquery 문제
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    ------------------------------------
    -- subquery(서브쿼리?)
    ------------------------------------
    -- select, insert, update, delete 구문안에 들어있는 또다른 select 문을 말한다.
    ------------------------------------
    -- subquery 사용시 주의점
    ------------------------------------
    -- select, update, delete 안의 서브쿼리는 꼭 ()로 묶는다.
    -- insert 안의 서브쿼리는 ()로 묶지 않는다.
    -- 경우에 따라 join 대신 subquery를 써도 같은 결과를 낼 수 있다.
    -- (이때 join 보다 subquery 의 부하가 더 많이 걸린다.)
    ------------------------------------
    -- subquery 종류
    ------------------------------------
    -- 비 상관 쿼리 => [서브쿼리]와 [외부쿼리]가 연관성이 없다. [서브 쿼리] 실행 후의 결과값을 가지고 메인쿼리 실행된다.
    -- 상관 쿼리(**어렵다) => [서브쿼리]와 [외부쿼리]가 연관성이 있다. [서브 쿼리]와 [외부 쿼리]가 서로 통신을 하면서 실행된다.

    ------------------------------------
    -- Q.89 최고/최저 연봉을 받는 직원을 검색하라.
    ------------------------------------
    select * from EMPLOYEE where SALARY = (select max(SALARY) from EMPLOYEE); -- 비상관쿼리다.
    select * from EMPLOYEE where SALARY = (select min(SALARY) from EMPLOYEE); -- 비상관쿼리다.
    ------------------------------------
    --아래처럼 하면 안된다.
    ------------------------------------
    -- select * from employee where salary = max(salary);
    ------------------------------------
    -- 그룹함수(min max sum avg count)는 select 절에만 나와야한다.
    -- max(salary) 코딩은 select 절에서만 나와야한다.
    -- * 즉 max 함수가 끌어안아서 나온 결과가 어떤 테이블 소속인지 모르면 안된다. *
    -- select 절 안에 나오면 from 절 뒤의 테이블 소속이라는 걸 알 수 있으니까 가능하다.
    -- where 절 안에 나오면 누구 컬럼인지 모르니까 안된다.
    -- 그룹 함수의 특징이다. 그룹함수(min, sum, avg, count)도 동일하다.

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.90 평균 연봉 이상을 받는 직원을 검색하라.
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    select * from EMPLOYEE where SALARY > (select avg(SALARY) from EMPLOYEE);

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.91 20번 부서에서 최고 연봉자 직원을 검색하라.
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    select * from EMPLOYEE where DEP_NO = 20 and SALARY = (select max(SALARY) from EMPLOYEE where DEP_NO=20);
    ----------------------
    -- 아래처럼 코딩하면 안된다.
    ----------------------
    select * from EMPLOYEE where SALARY = (select max(SALARY) from EMPLOYEE where DEP_NO=20);
    -- 모든 부서 안에 20번부서의 최대 연봉과 같은 사람 다나와가 된다. 그래서 틀린 답.

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.92 [직원명], [직급], [연봉], [전체연봉에서 차지하는 비율]을 검색하라.
    -- [전체연봉에서 차지하는 비율]은 소수점 버림 %로 표현하라.
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- floor : 소수점 버리는 함수
    select
    EMP_NAME "직원명"
    ,JIKUP "직급"
    ,SALARY "연봉"
    ,floor(SALARY/(select sum(SALARY) from EMPLOYEE)*100)||'%' "연봉비율"
    from EMPLOYEE;


    -- trunc 쓰게 되면 trunc(특정숫자, 0)
    select
    EMP_NAME "직원명"
    ,JIKUP "직급"
    ,SALARY "연봉"
    ,trunc(SALARY/(select sum(SALARY) from EMPLOYEE)*100,0)||'%' "연봉비율"
    from EMPLOYEE;

    -- 자리수 세자리 확보하고 출력 .(앞에 빌경우 0 으로 채운다.) -to_char(특정숫자,'999') -- <주의> 이건 반올림된다.
    select
    EMP_NAME "직원명"
    ,JIKUP "직급"
    ,SALARY "연봉"
    ,to_char(SALARY/(select sum(SALARY) from EMPLOYEE)*100,'999')||'%' "연봉비율"
    from EMPLOYEE;

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.93 10번부서 직원들이 관리하는 [고객번호], [고객명], [직원번호]를 검색하면?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    ----------------------------
    -- join 사용 답
    ----------------------------
    select
    c.CUS_NO,
    c.CUS_NAME,
    e.EMP_NO
    from CUSTOMER c , EMPLOYEE e
    where c.EMP_NO = e.EMP_NO and e.DEP_NO =10;
    ----------------------------
    -- subquery 사용 답
    ----------------------------
    -- 답 1
    ----------------------------
    -- in ( , , , ) 로 나열되는데 여러개 중 데이터가 하나이지만 in을 안쓰면 안에 데이터가 여러개이기 때문에 에러가 난다.
    select
    CUS_NO,
    CUS_NAME,
    EMP_NO
    from CUSTOMER
    where EMP_NO in (select EMP_NO from EMPLOYEE where DEP_NO=10);
    ----------------------------
    -- 답 2
    ----------------------------
    select
    CUS_NO,
    CUS_NAME,
    EMP_NO
    from CUSTOMER
    where EMP_NO = any (select EMP_NO from EMPLOYEE where DEP_NO=10);
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.94 직원명이 한국남과 직급이 동일한 직원을 검색하라
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    select * from EMPLOYEE where JIKUP=(select JIKUP from EMPLOYEE where EMP_NAME='한국남');

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.95 직원명이 무궁화와 직급이 동일한 직원을 검색하라
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- =이퀄 뒤에 나오는 데이터는 하나여야하는데 무궁화는 두명일 경우에 .. 에러가 난다.
    -- =이퀄을 빼는 in을 쓰면 된다.
    select * from EMPLOYEE where JIKUP in (select JIKUP from EMPLOYEE where EMP_NAME='무궁화');


    -------------------------------
    -- 상관 쿼리 문제 *
    -------------------------------
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.96 담당 고객이 2명 이상인 [직원번호], [직원명], [직급]을 검색하면?(***)
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- 테이블이 하나밖에 없는데 알리아스(별칭 e.~)를 쓴다면 상관쿼리라는 의미.
    -- 바깥족 테이블에 있는 쿼리를 안쪽에 쓰고있다면 상관쿼리다.
    select
    e.EMP_NO
    ,e.EMP_NAME
    ,e.JIKUP
    from EMPLOYEE e
    where (select count(*) from CUSTOMER c where e.EMP_NO = c.EMP_NO)>=2;

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.97 최고 연봉 직원의 [직원번호], [직원명], [부서명], [연봉]을 검색하면?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    --join 사용한 답
    select
    e.EMP_NO
    ,e.EMP_NAME
    ,d.DEP_NAME
    ,e.SALARY
    from EMPLOYEE e , DEPT d
    where d.DEP_NO = e.DEP_NO;
    --join 사용하지 않고 상관쿼리로 답
    select
    e.EMP_NO
    ,e.EMP_NAME

    from EMPLOYEE e
