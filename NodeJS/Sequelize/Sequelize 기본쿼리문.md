# Sequelize 기본 쿼리문

<img width="939" alt="스크린샷 2020-11-15 오전 1 21 18" src="https://user-images.githubusercontent.com/45676906/99151792-ddc2e800-26e0-11eb-8253-00ed24988768.png">

시퀄라이즈는 ORM(Object-relational-Mapping)이다. ORM은 자바스크립트 객체와 데이터베이스의 릴레이션을 매핑해주는 도구이다.
따라서 직접 쿼리를 작성하지 않아도 Sequelize가 자동으로 생성해서 적용해준다. 실제 쿼리와 시퀄라이즈 문법을 비교해보면서 알아보자.

<br>

## 1. `Sequelize create`

```javascript
const { User } = require('./model/index.js');

const user = await User.create({
   email: "Gyuuny@naver.com",
   password: "password",
   userName: "Gyunny",
   salt: "salt",
});
```

## `SQL INSERT Query`

```sql
INSERT INTO users (email, password, userName, salt) VALUES("Gyunny@naver.com", "password", "Gyunny", "salt");
```

`Sequelize create`는 `SQL 쿼리`로 `insert`하는 것이다. 시퀄라이즈 문법과 SQL 문법의 차이를 비교하면서 보자.

<br>

## 2. `Sequelize findOne`

```javascript
const { User } = require('./model/index.js');

const user = await User.findOne({
   where: {
    name: 'Gyunny'
   }
});
```

## `SQL findOne Query`

```sql
SELECT `id`, `email`, `userName` FROM `User` AS `User` WHERE `User`.`userName` = 'Gyunny' LIMIT 1;;
```

`findOne`은 말 그대로 where 옵션에 맞는 하나를 찾아오는 것이다. SQL WHERE과 쓰임새는 같고 원하는 row만 꺼내올 수 있는 기능이다. 
그리고 마지막에 `LIMIT 1` 옵션이 붙는다. 

<br>

## `Sequelize findAll`

```javascript
const { User } = require('./model/index.js');

const user = await User.findALl({
    attribute: ['email', 'userName'],
    where: {
        address: '경기도 군포시'
    }
});
```

## `SQL findAll Query`

```sql
SELECT email, userName FROM users WHERE address = '경기도 군포시';
```

`attribute` 옵션으로 원하는 컬럼값만 뽑아오는 것이 가능하다.

<br>

### 속성은 중첩 배열을 사용하여 이름을 바꿀 수 있다. 

```javascript
User.findAll({
  attributes: ['foo', ['bar', 'baz'], 'qux']
});
```

```sql
SELECT foo, bar AS baz, qux FROM users
```

<br>

### `sequelize.fn`은 `집계 함수`를 사용하는 경우 사용할 수 있다. 

```javascript
const sequelize = require('sequelize');

const praiseCount = await user.findAll({
   attributes: [[sequelize.fn('COUNT', 'id'), 'likeCount']],
   include: [{
     model: praise,
     as: 'praiser',
   }]
})
```

```sql
SELECT `users`.`id`, COUNT('id') AS `likeCount`, `praiser`.`id` AS `praiser.id`, `praiser`.`daily_praise` AS `praiser.daily_praise`, `praiser`.`mission_praise` AS `praiser.mission_praise`, `praiser`.`createdAt` AS `praiser.createdAt`, `praiser`.`updatedAt` AS `praiser.updatedAt`, `praiser->isDo`.`is_do` AS `praiser.isDo.is_do`, `praiser->isDo`.`userId` AS `praiser.isDo.userId`, `praiser->isDo`.`praiseId` AS `praiser.isDo.praiseId` 
FROM `users` AS `users` LEFT OUTER JOIN ( `isDo` AS `praiser->isDo` INNER JOIN `praise` AS `praiser` ON `praiser`.`id` = `praiser->isDo`.`praiseId`) 
ON `users`.`id` = `praiser->isDo`.`userId`;
```

<br>

### `Count` 예시1

```javascript
const sequelize = require('sequelize');

const praiseCount = await user.findAll({
   attributes: [[sequelize.fn('COUNT', 'id'), 'likeCount']]
})
```
```sql
SELECT COUNT('id') AS `likeCount` FROM `users` AS `users`;
```

<br>

### 다른 `Count` 사용법

```javascript
const praiseCount1 = await user.count({
  include: [{
    model: praise,
    as: 'praiser',
  }],
})
```
```sql
SELECT count(`users`.`id`) AS `count` 
FROM `users` AS `users` 
LEFT OUTER JOIN ( `isDo` AS `praiser->isDo` INNER JOIN `praise` AS `praiser` ON `praiser`.`id` = `praiser->isDo`.`praiseId`) 
ON `users`.`id` = `praiser->isDo`.`userId`;
```

위의 경우는 `users`테이블과 `praise`테이블은 `M대N 관계`라서 `매핑테이블`까지 이용해서 `JOIN`을 하는 상황이다. 
여기서 봐야할 점은 `테이블이름.count({})`를 사용할 수 있다는 것을 알면 될 것 같다.

<br>

### 하나 더 `Count` 예시

```javascript
const praiseCount = await User.count({});
```
```sql
SELECT count(*) AS `count` FROM `User` AS `User`;
```


<br>

### Sequelize에서 `max`, `min`, `sum` 사용법 

### `max` 사용법

```javascript
await User.max('id'); 
// SELECT max(`id`) AS `max` FROM `User` AS `User`;

await User.max('id', { where: { id: { [Op.lt]: 2 } } });
// SELECT max(`id`) AS `max` FROM `User` AS `User` WHERE `User`.`id` < 2;
```

<br>


<br>

### `exclude`로 속성을 제거하기

```javascript
User.findAll({
  attributes: { exclude: ['password', 'salt'] }
});
```

```sql
SELECT email, userName FROM users;
```

<br>

### `where 절`에서 `And`와 `OR` 사용하기


### `AND` 사용법

```javascript
User.findAll({
  where: {
    authorId: 12,
    status: 'active'
  }
});
```
```sql
SELECT * FROM post WHERE authorId = 12 AND status = 'active';
```

<br>

```javascript
User.findAll({
  attribute: ['name', 'age'],
  where: {
    married: 1,
    age: { [Op.gt]: 30 },
  },
})
```
```sql
SELECT name, age FROM users WHERE married = 1 AND age > 30;
```

<br>

```javascript
const { Op } = require("sequelize");
Post.findAll({
  where: {
    [Op.and]: [
      { authorId: 12 },
      { status: 'active' }
    ]
  }
});
```
```sql
SELECT * FROM post WHERE authorId = 12 AND status = 'active';
```

이렇게 AND를 사용하는 방법은 다양하다. 

<br>

### `OR` 사용하기

```javascript
const { Op } = require("sequelize");
Post.findAll({
  where: {
    [Op.or]: [
      { authorId: 12 },
      { authorId: 13 }
    ]
  }
});
```
```sql
SELECT * FROM post WHERE authorId = 12 OR authorId = 13;
```

<br>

```javascript
const { Op } = require("sequelize");
Post.destroy({
  where: {
    authorId: {
      [Op.or]: [12, 13]
    }
  }
});
```
```sql
DELETE FROM post WHERE authorId = 12 OR authorId = 13;
```

이렇게 OR를 사용하는 방법도 다양하게 있다. 

<br>

## `Sequelize에서 연산자 사용법`

```javascript
const { Op } = require("sequelize");

Post.findAll({
  where: {
    [Op.and]: [{ a: 5 }, { b: 6 }],            // (a = 5) AND (b = 6)
    [Op.or]: [{ a: 5 }, { b: 6 }],             // (a = 5) OR (b = 6)
    someAttribute: {
 
      [Op.eq]: 3,                              // = 3
      [Op.ne]: 20,                             // != 20
      [Op.is]: null,                           // IS NULL
      [Op.not]: true,                          // IS NOT TRUE
      [Op.or]: [5, 6],                         // (someAttribute = 5) OR (someAttribute = 6)
     
      [Op.col]: 'user.organization_id',        // = "user"."organization_id"

      // Number comparisons
      [Op.gt]: 6,                              // > 6
      [Op.gte]: 6,                             // >= 6
      [Op.lt]: 10,                             // < 10
      [Op.lte]: 10,                            // <= 10
      [Op.between]: [6, 10],                   // BETWEEN 6 AND 10
      [Op.notBetween]: [11, 15],               // NOT BETWEEN 11 AND 15

      // Other operators

      [Op.all]: sequelize.literal('SELECT 1'), // > ALL (SELECT 1)

      [Op.in]: [1, 2],                         // IN [1, 2]
      [Op.notIn]: [1, 2],                      // NOT IN [1, 2]

      [Op.like]: '%hat',                       // LIKE '%hat'
      [Op.notLike]: '%hat',                    // NOT LIKE '%hat'
      [Op.startsWith]: 'hat',                  // LIKE 'hat%'
      [Op.endsWith]: 'hat',                    // LIKE '%hat'
      [Op.substring]: 'hat',                   // LIKE '%hat%'
      [Op.notILike]: '%hat',                   // NOT ILIKE '%hat'  (PG only)
    }
  }
});
```

<br>


## `Like` 사용법

```javascript
await User.findAll({
  where: {
   userName: {
     [Op.like]: "%" + 'Gyunny' + "%",
    },
   },
});
```
```sql
SELECT `id`, `email`, `userName`, `password`, `salt` FROM `User` AS `User` WHERE `User`.`userName` LIKE '%Gyunny%';
```

<br>

## `Like` & `And` 사용

```javascript
await User.findAll({
   where: {
     userName: {
       [Op.like]: "%" + 'Gyunny' + "%",
     },
     id: 1
   },
});
```
```sql
SELECT `id`, `email`, `userName`, `password`, `salt` FROM `User` AS `User` WHERE `User`.`userName` LIKE '%Gyunny%' AND `User`.`id` = 1;
```


<br>

## 예제

```javascript
await User.findAll({
   where: {
     id: {
       [Op.lt]: 1000,
     }
   }
});
```
```sql
SELECT `id`, `email`, `userName`, `password`, `salt` FROM `User` AS `User` WHERE `User`.`id` < 1000;
```

<br>

## `IN` 연산자 사용법

```javascript
User.findAll({
  where: {
   id: [1,2,3] 
  }
});
```
```sql
SELECT `id`, `email`, `userName`, `password`, `salt` FROM `User` AS `User` WHERE `User`.`id` IN (1, 2, 3);
```

<br>

## `그룹화(GROUP BY)`

```javascript
User.findAll({ group: 'userName' });
```
```sql
SELECT `id`, `email`, `userName`, `password`, `salt` FROM `User` AS `User` GROUP BY `userName`;
```

<br>

## `Order 사용 법`

```javascript
const test = await User.findOne({
   order: [
      ['userName', 'DESC']
   ]
})
```
```sql
SELECT `id`, `email`, `userName`, `password`, `salt` FROM `User` AS `User` ORDER BY `User`.`userName` DESC LIMIT 1
```

<br>

## `Limit & paging 처리`

```javascript
User.findAll({ limit: 10 });
// SELECT `id`, `email`, `userName`, `password`, `salt` FROM `User` AS `User` LIMIT 10;

User.findAll({ offset: 8 });
// SELECT `id`, `email`, `userName`, `password`, `salt` FROM `User` AS `User` LIMIT 8, 10000000000000;

User.findAll({ offset: 5, limit: 5 });
// SELECT `id`, `email`, `userName`, `password`, `salt` FROM `User` AS `User` LIMIT 5, 5;
```

<br>

## `Sequelize Update(수정)`

```javascript
await User.update({ userName: "Gyunny" }, {
  where: {
    id: 2
  }
});
```
```sql
UPDATE users SET userName = "Gyunny" WHERE id = 2;
```

<br>

## `Sequelize delete(삭제)`

```javascript
// Delete everyone named "Jane"
await User.destroy({
  where: {
    id: 1
  }
});
```
```sql
DELETE FROM users WHERE id = 2;
```


<br>

# Reference

- [https://sequelize.org/master/manual/model-querying-basics.html#simple-insert-queries](https://sequelize.org/master/manual/model-querying-basics.html#simple-insert-queries)


