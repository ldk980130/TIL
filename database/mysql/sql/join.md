# Join의 종류
## JOIN이란

2개 이상의 table에 있는 데이터를 한 번에 조회

## INNER JOIN

```sql
SELECT * FROM table_A a JOIN table_B b on a.b_id = b.id;
```

- 두 테이블에서 join condition을 만족하는 tuple들로 결과 tuple을 만드는 join
    - join condition 연산자 - =, >, <, != 등
    - join condition에서 null 값을 가지는 tuple은 포함되지 않는다.
    - 두 테이블의 교집합
- 일반적으로 그냥 `JOIN`만 작성하면 `INNER`가 생략되어 있는 것이다.

## OUTTER JOIN

```sql
SELECT * FROM table_A a LEFT JOIN table_B b on a.b_id = b.id;
SELECT * FROM table_A a RIGHT JOIN table_B b on a.b_id = b.id;
SELECT * FROM table_A a FULL JOIN table_B b on a.b_id = b.id;
```

- 두 테이블에서 join condition을 만족하지 않는 tuple들로도 결과 tuple을 만드는 join
    - `left (outer) join` - table_A의 b_id가 null이더라도 table_A의 tuple은 모두 조회한다. (table_A는 모두 조회, table_B는 모두 조회 X)
    - `right (outer) join` - table_B의 tuple을 모두 조회하고 table_B의 id에 해당하는 table_A의 b_id가 없으면 그 자리는 null로 채운다. (table_A는 모두 조회 X, table_B는 모두 조회)
    - `full (outer) join` - table_A와 table_B가 join condition이 맞지 않아도 모두 조회된다.
        - MySQL에서는 지원하지 않음

> **equi join**
join condition에서 = 연산자를 사용하는 JOUIN
>

## NATURAL JOIN

```sql
SELECT * FROM table_A NATURAL JOIN table_B;
SELECT * FROM table_A NATURAL LEFT JOIN table_B;
SELECT * FROM table_A NATURAL OUTER table_B;
SELECT * FROM table_A NATURAL FULL table_B;
```

- 두 테이블에서 같은 이름을 가지는 모든 attribute pair에 대해서 equi join을 수행
    - table_A의 table_B에 대한 외래키가 b_id이고, table_B가 자신의 pk를 b_id라고 설정했다면 두 테이블의 b_id에 대해 equi join이 실행된다.
    - `SELECT * FROM table_A NATURAL JOIN table_B;` == `SELECT * FROM table_A a JOIN table_B b on a.b_id = b.b_id;`
- join condition을 따로 명시하지 않는다.

## CROSS JOIN

```sql
FROM table_A, table_B; -- 묵시적 cross join
FROM table_A CROSS JOIN table_B -- 명시적 cross join
```

- 두 테이블의 tuple pair로 만들 수 있는 모든 조합을 결과 tuple로 반환
    - table_A의 tuple 개수가 10개, table_B의 tuple 개수가 10개이면 모든 조합인 100개의 tuple이 반환된다.
    - where 절로 원하는 tuple만 반환할 수도 있겠지만 애초에 모둔 tuple 조합을 계산하기 때문에 성능상 좋지 않다.
- join condition이 없다
- MySQL에서는 cross join == inner join == join
    - cross join에 on 절로 조건을 명시하면 inner join으로 동작한다.
    - on 절 조건이 없다면 cross join으로 동작한다.

---

### 참고

[https://www.youtube.com/watch?v=E-khvKjjVv4](https://www.youtube.com/watch?v=E-khvKjjVv4)
