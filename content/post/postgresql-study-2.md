+++
date = '2025-11-10T07:53:17+08:00'
draft = true
title = 'PostgreSQL 中创建俩中唯一约束条件记录'
tags = ['PostgreSQL','数据库']
categories = ['学习']
+++

最近在做一个项目的时候遇到了常见的场景：用户注册功能，注册时需要保证`username`的唯一性，还有要保证已经激活状态下`email`的唯一性，如果放到以前，我直接就用代码逻辑去控制了，但是最近在重新学习PostgreSQL，而且既然数据库已经有唯一约束这种特性，为什么我们还需要自己用代码去实现呢？除非你的项目是超大集群，否则如果是一个小型项目，开发人员也不多，这里只有我自己，那么充分利用数据库的这些特性是最佳选择，因此上述场景我就使用了数据库的俩中唯一约束。

## 普通唯一约束

普通唯一约束只要在创建字段时添加`UNIQUE`就可以，如果字段已经添加了，想更新为唯一约束，则像下面这样：

```sql
alter users
add constraint users_username_unique unique (username);
```

添加上述约束后，如果插入重复的`username`则会抛出错误，在SpringBoot+JPA场景下可以这样获取错误以及对应的约束信息

```java
private void insertUser(UserCreateRequest userBody) {
      try {
          // 首先进行用户数据的合法性校验
          UserValidator.validateUser(userBody.username(), userBody.email(), userBody.password(), userBody.nickname());
          // 校验通过后插入数据库
          User user = new User();
          String encodePassword = PasswordEncoderFactories.createDelegatingPasswordEncoder().encode(userBody.password());
          user.setPassword(encodePassword);
          user.setNickname(userBody.nickname());
          user.setEmail(userBody.email());
          user.setUsername(userBody.username());
          // 查询普通用户角色
          Optional<Role> roleOptional = roleRepository.findByName("USER");
          roleOptional.ifPresent(role -> user.setRoles(Set.of(role)));
          userRepository.save(user);
      } catch (Exception e) {
          Throwable cause = e.getCause();
          if (cause instanceof org.hibernate.exception.ConstraintViolationException cve) {
              if (USERS_USERNAME_UNIQUE.equals(cve.getConstraintName())) {
                  throw new ValidationException(ErrorCode.CONSTRAINT_USERNAME);
              }
          }
          throw e;
      }
  }
```

## 条件唯一约束

对于 email 的唯一性校验的前提是它已经被激活，因此需要使用条件唯一约束，像下面这样：

```sql
CREATE UNIQUE INDEX users_email_verified_unique
ON users(email)
WHERE email_verified = true;
```

对应的在 SpringBoot 异常捕获中增加下面的即可

```java
if (cause instanceof org.hibernate.exception.ConstraintViolationException cve) {
	  if (USERS_USERNAME_UNIQUE.equals(cve.getConstraintName())) {
	      throw new ValidationException(ErrorCode.CONSTRAINT_USERNAME);
	  }
	  if (USERS_EMAIL_VERIFIED_UNIQUE.equals(cve.getConstraintName())) {
	  throw new ValidationException(ErrorCode.CONSTRAINT_EMAIL);
	  }
}
```
