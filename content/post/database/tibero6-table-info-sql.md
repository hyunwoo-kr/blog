---
title: "Tibero6 테이블 정보 조회 SQL"
date: 2026-01-20
categories:
- database
tags:
- tibero6
- sql
- database
- schema
keywords:
- Tibero6
- 테이블 정보 조회
- 스키마 조회
- ALL_TAB_COLUMNS
author: "hyunwoo"
draft: false
---

특정 테이블의 스키마 정보(컬럼명, 코멘트, 데이터 유형, 길이, NULL 조건, 디폴트 값)를 조회하는 SQL문입니다.

<!--more-->

## 조회 SQL

```sql
SELECT
    A.OWNER AS "스키마명",
    A.TABLE_NAME AS "테이블명",
    A.COLUMN_NAME AS "컬럼명",
    B.COMMENTS AS "코멘트",
    A.DATA_TYPE AS "데이터 유형",
    CASE
        WHEN A.DATA_TYPE IN ('VARCHAR', 'VARCHAR2', 'CHAR', 'NVARCHAR', 'NVARCHAR2', 'NCHAR')
            THEN TO_CHAR(A.DATA_LENGTH)
        WHEN A.DATA_TYPE = 'NUMBER' AND A.DATA_PRECISION IS NOT NULL
            THEN A.DATA_PRECISION || ',' || NVL(A.DATA_SCALE, 0)
        ELSE TO_CHAR(A.DATA_LENGTH)
    END AS "길이",
    A.NULLABLE AS "NULL 조건",
    A.DATA_DEFAULT AS "디폴트 값"
FROM
    ALL_TAB_COLUMNS A
LEFT JOIN
    ALL_COL_COMMENTS B
    ON A.OWNER = B.OWNER
    AND A.TABLE_NAME = B.TABLE_NAME
    AND A.COLUMN_NAME = B.COLUMN_NAME
WHERE
    A.OWNER = '스키마명'
    AND A.TABLE_NAME = '테이블명'
ORDER BY
    A.COLUMN_ID;
```

## 사용 방법

`'스키마명'`과 `'테이블명'`을 실제 조회할 값으로 변경해서 사용합니다.

## 출력 컬럼 설명

| 컬럼 | 설명 |
|------|------|
| 스키마명 | 테이블이 속한 스키마(소유자) |
| 테이블명 | 테이블 이름 |
| 컬럼명 | 컬럼 이름 |
| 코멘트 | 컬럼에 대한 설명 |
| 데이터 유형 | VARCHAR2, NUMBER, DATE 등 |
| 길이 | 문자열은 길이, NUMBER는 정밀도,스케일 |
| NULL 조건 | Y(NULL 허용) / N(NOT NULL) |
| 디폴트 값 | 기본값 설정 |

## 참고사항

- NUMBER 타입의 경우 `정밀도,스케일` 형태로 표시됩니다. 예: `10,2`는 NUMBER(10,2)를 의미합니다.
- `DATA_DEFAULT` 컬럼은 LONG 타입이므로 `TRIM()` 등의 함수를 직접 사용할 수 없습니다.
