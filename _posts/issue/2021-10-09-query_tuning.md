---
date: 2021-10-09 01:40:40
layout: post
title: 회원 profile 중복 체크 query 개선 히스토리
subtitle: 회원 profile 중복 체크 query 개선 히스토리
description: 회원 profile 중복 체크 query 개선 히스토리
image: https://leejaedoo.github.io/assets/img/sql.jpg
optimized_image: https://leejaedoo.github.io/assets/img/sql.jpg
category: issue
tags:
- issue
- sql
- querydsl
paginate: true
comments: true
---
# 개요
client에서 회원 가입이나 회원 정보를 수정하기 위해 memberId, email, mobileNo, nickname 값의 중복 여부를 체크 하기 위한 api에서 database에 조회하는 기존 query 성능이 1.6초 정도 소요되는 현상이 발견되었다.<br>
해당 쿼리는 OLTP 쿼리가 1초 이상 소요되는 것은 매우 긴 시간이라는 DBA분의 지적이 있어 해당 쿼리를 개선하는 과정을 정리해보려 한다.

# 내용

## 회원 테이블 DDL

```sql
create table mb_member
(
    member_no                    int auto_increment primary key,
    mobile_no                    varchar(50) charset utf8          null,
    member_status                char(8) charset utf8              null comment '회원 상태',
    email                        varchar(100) charset utf8         null,
    nickname                     varchar(45)                       null,
    mall_no                      int                               not null,
    member_id                    varchar(50)                       null,
    join_ymdt                    datetime                          null
) comment '회원 정보';

create index INK10_MB_MEMBER
    on mb_member (mall_no, email);

create index MB_MEMBER_MALL_NO_JOIN_YMDT_IDX
    on mb_member (mall_no, join_ymdt);

create index ink4_mb_member
    on mb_member (mobile_no, mall_no);

create index mb_member_mall_no_idx
    on mb_member (mall_no);

create index member_id_idx
    on mb_member (member_id);
```

## 기존 쿼리
```sql
DBadmin@localhost:** 15:30:04>explain
    -> select *
    -> from mb_member member0_
    -> where (member0_.member_id='dlwoen9@naver.com' or member0_.email='dlwoen9@naver.com')
    -> and member0_.mall_no=22
    -> and member0_.member_status<>'MBST0004';
+----+-------------+----------+------------+------+--------------------------------------------------------------------------------------------------------------------+-----------------------+---------+-------+--------+----------+-------------+
| id | select_type | table    | partitions | type | possible_keys | key                   | key_len | ref   | rows   | filtered | Extra       |
+----+-------------+----------+------------+------+--------------------------------------------------------------------------------------------------------------------+-----------------------+---------+-------+--------+----------+-------------+
|  1 | SIMPLE      | member0_ | NULL       | ref  | member_id_idx,mb_member_mall_no_idx,MB_MEMBER_MALL_NO_JOIN_YMDT_IDX,MB_MEMBER_MALL_NO_BIRTHDAY_IDX,INK10_MB_MEMBER | mb_member_mall_no_idx | 4       | const | 197165 |    17.10 | Using where |
+----+-------------+----------+------------+------+--------------------------------------------------------------------------------------------------------------------+-----------------------+---------+-------+--------+----------+-------------+
```

explain으로 확인해보면 `mb_member_mall_no_idx` index만 타고 제대로 member_id와 email에 대해서 제대로 index가 타지 못하고 Full Scan이 이루어지고 있기 때문에 등록된 사용자가 증가하면 증가할 수록 처리 속도는 지연된다.<br>

## 코드 확인

```java
    public List<Member> getDuplicatedMembers(String memberId, String email, String mobileNo, int mallNo,
                                             String nickName) {
        BooleanBuilder builder = new BooleanBuilder();
        if (!Strings.isNullOrEmpty(memberId)) {
            builder.or(member.memberId.eq(StringUtils.lowerCase(memberId)));
        }
        if (!Strings.isNullOrEmpty(email)) {
            builder.or(member.email.eq(StringUtils.lowerCase(email)));
        }
        if (!Strings.isNullOrEmpty(mobileNo)) {
            builder.or(member.mobileNo.eq(mobileNo));
        }
        if (!Strings.isNullOrEmpty(nickName)) {
            builder.or(member.nickname.eq(nickName));
        }

        return from(member).where(builder)
                           // .where(member.memberStatus.notIn(Arrays.asList(MemberStatus.NOT_REJOINABLE_EXPELLED,
                           // MemberStatus.NOT_REJOINABLE_LEAVE)))
                           .where(member.mall.mallNo.eq(mallNo))
                           .where(member.memberStatus.ne(MemberStatus.EXPELLED))
                           .fetch();
    }
```

QueryDSL을 활용하여 조건절에 memberId, email, mobileNo, nickname 컬럼을 or 조건을 통해 조회하는 query다.<br>
or조건으로 이어진 query이기 때문에 full scan 탈 수 밖에 없었을 것이다.

## 쿼리 개선

```java
    public List<Member> getDuplicatedMembers(String memberId, String email, String mobileNo, int mallNo,
                                             String nickName) {
        List<Member> members = new ArrayList<>();
        if (!Strings.isNullOrEmpty(memberId)) {
            members.addAll(getMemberIdBuilder(memberId, mallNo));
            if (CollectionUtils.isNotEmpty(members)) {
                return members;
            }
        }
        if (!Strings.isNullOrEmpty(email)) {
            members.addAll(getEmailBuilder(email, mallNo));
            if (CollectionUtils.isNotEmpty(members)) {
                return members;
            }
        }
        if (!Strings.isNullOrEmpty(mobileNo)) {
            members.addAll(getMobileNoBuilder(email, mallNo));
            if (CollectionUtils.isNotEmpty(members)) {
                return members;
            }
        }
        if (!Strings.isNullOrEmpty(nickName)) {
            members.addAll(getNickNameBuilder(email, mallNo));
            if (CollectionUtils.isNotEmpty(members)) {
                return members;
            }
        }

        return members;
    }

    private List<Member> getNickNameBuilder(String nickName, int mallNo) {
        BooleanExpression predicate = member.nickname.eq(nickName);
        return getDuplicatedMembers(new BooleanBuilder(predicate), mallNo);
    }

    private List<Member> getMobileNoBuilder(String mobileNo, int mallNo) {
        BooleanExpression predicate = member.mobileNo.eq(mobileNo);
        return getDuplicatedMembers(new BooleanBuilder(predicate), mallNo);
    }

    private List<Member> getEmailBuilder(String email, int mallNo) {
        BooleanExpression predicate = member.email.eq(StringUtils.lowerCase(email));
        return getDuplicatedMembers(new BooleanBuilder(predicate), mallNo);
    }

    private List<Member> getMemberIdBuilder(String memberId, int mallNo) {
        BooleanExpression predicate = member.memberId.eq(StringUtils.lowerCase(memberId));
        return getDuplicatedMembers(new BooleanBuilder(predicate), mallNo);
    }

    private List<Member> getDuplicatedMembers(BooleanBuilder builder, int mallNo) {
        return from(member).where(builder)
                           .where(member.mall.mallNo.eq(mallNo))
                           .where(member.memberStatus.ne(MemberStatus.EXPELLED))
                           .fetch();
    }
```

# 해결
해결 방법으로는

1. or 조건절들을 다 쪼개서 개별 query로 조회
2. union 으로 처리

가 있는데 여기선 1번을 활용하여 기본 where 조건절이었던 mallNo와 함께 복합 index 형태로 조회되도록 함으로써 개선할 수 있었다.<br>
다 쪼갠 로직을 더 깔끔하게 구현해볼 수 있는 방법은 더 고민해봐야 할 것 같다.
