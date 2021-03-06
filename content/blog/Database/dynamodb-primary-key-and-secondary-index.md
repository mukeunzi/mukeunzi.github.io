---
title: 'DynamoDB의 기본키와 보조 인덱스'
date: 2021-04-17 23:55:13
category: 'Database'
draft: false
---

## 테이블 구성 요소

DynamoDB와 MySQL의 테이블 구성요소를 비교한 것입니다.

| DynamoDB | MySQL|
|:---:|:---:|
| Table | Table |
| Row | Item |
| Column | Attribute |

## 기본키 Primary key

기본키는 각 테이블에서 item들의 고유 식별자입니다. 기본키는 String, Number, Binary로 구성할 수 있습니다.

DynamoDB는 2가지 기본키를 지원합니다.

### 1. partition key - 하나의 속성으로 구성된 기본키

하나의 attribute로 기본키를 지정할 수 있습니다. SQL에서 하나의 컬럼을 기본키로 지정하는 것과 동일합니다.

기본키를 partition key라고 부르는 이유는 DynamoDB의 데이터 저장 방식과 연관되어 있습니다. DynamoDB에서는 partition key에 따라 데이터가 저장되는 파티션의 위치(물리적인 저장소)가 결정됩니다. 

같은 테이블의 데이터도 partition key가 다르면 다른 파티션에 저장됩니다. 기본키가 partition key 하나일 경우에는 모든 데이터가 다른 파티션에 저장됩니다. 

파티션의 위치는 partition key값으로 내부 해시 함수를 이용해 결정됩니다.

### 2. partition key, sort key - 두가지 속성으로 구성된 복합키

두개의 속성으로 기본키를 지정할 수 있습니다. 단어 그대로 partition key는 저장소의 위치를, sort key는 동일한 partition key 값을 정렬하는 데 사용됩니다. 

partition key가 동일한 값은 sort key를 기준으로 정렬되어 파티션에 저장됩니다. 복합키로 구성된 테이블에서는 partition key는 같을 수 있지만 sort key는 같을 수 없습니다.

## 보조 인덱스 - Secondary Index

DynamoDB에서 기본키를 사용하면 데이터를 빠르게 조회할 수 있습니다. 

하지만 기본키가 아닌 속성을 쿼리하면 데이터가 많아질수록 성능이 저하됩니다. 따라서 키가 아닌 속성의 쿼리 성능을 높이기 위해서는 보조 인덱스를 사용해야 합니다.

<br>
모든 보조 인덱스는 하나의 테이블과 연결되어 있으며 이 테이블을 기본 테이블이라고 합니다. 

인덱스를 생성할 때 인덱스 이름과 기본키(partition key, sort key), 프로젝션 속성을 지정할 수 있습니다.

- 프로젝션 속성이란 기본 테이블에서 보조 인덱스로 복사되는 속성을 뜻합니다.
  
<br>
보조 인덱스는 DynamoDB에서 자동으로 관리됩니다. 기본 테이블에서 데이터가 추가/삭제 되면 인덱스에도 동일하게 변경사항이 반영됩니다.

<br>
DynamoDB는 2가지의 인덱스를 지원합니다.

### Global Secondary Index(GSI)

GSI는 기본 테이블의 키와 다른 속성으로 기본키를 지정할 수 있습니다. 기본키는 단일 기본키(partiton key)이거나 복합 기본키(partition key, sort key)일 수 있습니다. 

기본 테이블과는 다르게 키 값이 고유하지 않아도 됩니다.

GSI는 테이블당 최대 20개의 인덱스를 생성할 수 있습니다. 인덱스 크기는 제한이 없으며 테이블이 생성된 이후에도 인덱스를 추가하거나 삭제할 수 있습니다.

다른 테이블의 속성도 프로젝션 속성에 추가할 수 있습니다. 따라서 모든 파티션의 전체 테이블을 쿼리할 수 있습니다.

하지만 기본테이블의 속성일지라도 프로젝션으로 지정하지 않은 속성들은 쿼리할 수 없습니다.

### Local Secondary index(LSI)

LSI는 기본 테이블과 partition key는 동일하지만 sort key가 다른 인덱스입니다. 기본키는 반드시 복합키여야 합니다.

LSI는 테이블당 최대 5개의 인덱스를 생성할 수 있습니다. 

partition key 값마다 인덱싱된 항목의 크기는 10GB이하여야 하고, 테이블이 생성된 이후에는 인덱를 추가하거나 삭제할 수 없습니다.

LSI를 사용하면 파티션 값에 지정된 파티션을 단일로 쿼리할 수 있습니다.

프로젝션하지 않은 속성도 테이블에서 자동으로 가져오기 때문에 프로젝션하지 않은 속성도 쿼리할 수 있습니다.

### 프로젝션 속성 3가지
프로젝션을 지정할 수 있는 속성은 3가지가 있습니다.

- KEYS_ONLY : 기본 테이블의 기본키(partition key, sort key)와 인덱스의 키 값으로만 구성합니다. 인덱스의 크기를 최소화할 수 있는 방법입니다.

- INCLUDE : KEYS_ONLY에 추가로 다른 속성을 지정하여 인덱스를 구성하는 방법입니다.

- ALL : 기본 테이블의 모든 속성을 추가하는 방법입니다. 모든 테이블의 데이터가 복사되기 때문에 보조 인덱스의 크기가 최대화 됩니다.

<br>
일반적으로 GSI에서 지원할 수 없는 경우(strong consistency)를 제외하고는 LSI보다 GSI를 사용해야 합니다. 

보조 인덱스는 스토리지와 프로비저닝된 처리량을 사용하므로 인덱스를 최소한 적은 수로, 작게 유지해야 합니다. 인덱스가 작을수록 쿼리 성능이 좋아지기 때문에 자주 쿼리하지 않는 속성에는 인덱스를 지정하지 않는 것이 좋습니다. 

## 참고 문서
- [AWS DynamoDB 공식문서](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html)
