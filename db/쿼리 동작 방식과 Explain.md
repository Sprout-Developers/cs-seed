# 쿼리 동작 방식

![09610baf606b4854b273a8ffc452b009](https://user-images.githubusercontent.com/50267433/146646441-bdfcf900-7763-476e-b6c7-36e08849286b.png)
  
**쿼리 동작 방식**  
1. Query Caching 
2. Parsing 
3. Preprocessor
4. Optimization 
5. Handler(Storage Engine) 
     
**쿼리 동작 방식 설명 및 문맥**       
1. SQL 문장을 잘게 쪼개서 MySQL 서버가 이해할 수 있는 수준으로 분리
2. SQL의 파싱 정보(파스 트리)를 확인하면서 어떤 테이블부터 읽고 어떤 인덱스를 이용해 테이블을 읽을지 선택
3. 결정된 테이블의 읽기 순서나 선택된 인덱스를 이용해 스토리지 엔진으로부터 데이터를 가져옴 
   
## Query Caching        
   
**SQL문을 키값으로 실행 결과를 Value로 저장하여 캐싱하는 방식을 사용한다.**          
  
|Key|Value|
|---|------|
|SQL문|실행결과| 
  
* Key : SQL문
* Value : 실행 결과
    
         
**쿼리 캐싱 영역은 아래와 같은 작업들을 진행한다.(조회/등록)**    
* 요청된 쿼리 문장이 쿼리 캐시에 존재하는가?      
* 해당 사용자가 그 결과를 볼 수 있는 권한이 있는가?    
* 트랜잭션 내에서 실행된 쿼리인 경우 가시 범위 내에 있는 결과인가?  
* 호출 시점에 따라 결과가 달라지는 요소가 있는가? (랜덤/현재값)     
* 캐시가 만들어지고 난 이후 해당 데이터가 다른 사용자에 의해 변경되지 않았는가  
* 쿼리에 의해 만들어진 결과가 캐시하기에 너무 크지 않은가?  

## Parsing 
    
* 사용자로부터 요청된 SQL을 잘게 쪼개서 서버가 이해할 수 있는 수준으로 분리한다.            
* 이 단계에서 `SQL Parse Tree`가 만들어지는데    
  MySQL 서버는 SQL문장 그자체가 아니라 **SQL Parse Tree를 이용해 쿼리를 실행한다.**               
  
## Preprocessor
  
해당 쿼리가 문법적으로 올바른지 검증하며 부적합하다면 처리를 중단한다.         
만약, 쿼리 작업이 일괄 처리일 경우, 일괄 처리 전체를 취소한다.       

## Optimization 

SQL Parse Tree를 참조하면서, 아래와 같은 작업을 처리한다.   
   
**최적화 작업**
* 쿼리 분석: Where 절의 검색 조건인지 Join 조건인지 판단한다.      
* 인덱스 선택 : 각 테이블에 사용된 조건과 인덱스 통계 정보를 이용해 사용할 인덱스를 결정한다.    
* 조인 처리 : 여러 테이블의 조인이 있는 경우 어떤 순서로 테이블을 읽을지 결정한다.  

**쉽게 표현**
* 어떤 쿼리를 사용할까?      
* 조인이 있다면 어느 순서부터 조인할까?      
* 인덱스는 어느 인덱스가 가장 빠를까?   

`최적화`라는 이름에 맞게 실제 쿼리를 실행하기전에 최적화된 실행 계획을 세우는 것을 의미한다.   
이렇게 최적화된 실행 계획을 세우는 것이 Optimization 영역이고      
최적화된 요소를 선택하는 것이 바로 Query Optimizer 이다. 
     
그리고 MySQL의 `EXPLAIN`이라는 키워드는 딱 여기까지만 실행하는 것을 의미한다.      

### Optimizer 종류 
#### 비용 기반 최적화  
          
쿼리를 처리하기 위한 여러가지 가능한 방법을 만들고               
`각 단위 작업의 비용 정보`와 `대상 테이블의 예측된 통계 정보`를 이용해 **각 실행 계획별 비용을 산출한다.**          
**이렇게 산출된 각 실행 방법별로 최소 비용이 소요되는 처리 방식을 선택해 최종 쿼리를 실행하게 된다.**         
   
#### 규칙 기반 최적화 
       
과거, 각 테이블이나 인덱스의 통계 정보가 거의 없고,         
상대적으로 느린 CPU 연산 탓에 계산 비용 과정이 부담스러웠기 때문에 **사용되었던 최적화 방법이었다.**       
현재는 거의 대부분의 RDBMS가 비용 기반의 옵티마이저를 채택하고 있으며 RDBMS도 마찬가지이다.     

#### 통계 정보  
MySQL 통계 정보는 `대략의 레코드 건수`와, `인덱스의 유니크한 값의 개수` 정도가 전부다.             
그러나 사용자가 알아채지 못할만큼 순간 순간 자동으로 변경되기 때문에 상당히 동적인 편이다.        
        
하지만, 레코드 건수가 많지 않으면 통계 정보가 상당히 부정확한 경우가 많으므로     
ANALYZE 명령을 이용해 강제적으로 통계 정보를 갱신해야할 때도 있다.      
            
**MySQL 옵티마이저**는 비용 기반으로,    
**어떤 실행 계획으로 쿼리를 실행했을 때 비용이 얼마나 발생하는지를 계산하여 비용이 가장 적은 것을 택한다.**      
어디까지나 추정 값이므로 정확한 비용은 실행전까지 정확히 알 수 없다.    
        
## Handler(Storage Engine)   

* MySQL 실행 엔진의 요청에 따라 데이터를 디스크로 저장하고, 디스크로부터 읽어오는 역할을 담당한다.     
* MySQL 엔진에서는 스토리지 엔진으로부터 받은 레코드를 조인하거나 정렬하는 작업을 수행한다.   
    
# Explain  
   
Explain은 MySQL 서버가 어떠한 쿼리를 실행할 것인가.        
즉, **실행 계획이 무엇인지 알고 싶을 때 사용하는 기본적인 명령어이다.**       

|id|select_type|table|partitions|type|possible_keys|key|key_len|ref|rows|filtered|Extra|
|-|------------|-----|----------|----|-------------|---|-------|---|----|--------|-----|
|1|SIMPLE|m|NULL|range|PRIMARY|PRIMARY|8|NULL|3|100.00|Using where|
|1|SIMPLE|o|NULL|ref|FKpktxw…|FKpktxw…|8|sample.m.id|90|100.00|NULL|
|1|SIMPLE|t|NULL|eq_ref|PRIMARY|PRIMARY|8|sample.o.transaction_id|1|100.00|NULL|

## Table 

* 접근하고 있는 테이블에 대해 표시한다(alias로 보여줌)   

## id 

* select에 붙은 번호,   

## select_type 

* 대부분의 쿼리에서 SIMPLE 값이 출력되고, 서브쿼리나 UNION을 사용하면 바뀐다.  

## partitions 
  
* 테이블이 파티셔닝 되어있는 경우에 파티션 목록이 출력된다.(없으면 null)      

## type 
* 접근 방식을 표현하며, 테이블에서 어떻게 행 데이터를 가져올 것인가를 나타낸다.      
* 대상 테이블로의 접근이 효율적일지 여부를 판단해주는 아주 중요한 항목이다.        

|type|설명|
|----|---|
|ALL|전체 행 스캔, 테이블의 데이터 전체에 접근한다.|
|const|기본키 또는 고유키에 의한 lookup(등가비교)<br> 조인이 아닌 가장 외부의 테이블에 접근 하는 방식<br>결과는 항상 1인 행<br>단 기본 키, 고유 키를 사용하고 있으므로 범위 검색으로 지정하는 경우 const가 되지 않는다<br>| 
|index|인덱스 스캔, 테이블의 특정 인덱스의 전체 엔트리에 접근한다.|  
|ref|고유키가 아닌, 인덱스에 대한 등가비교 여러 개 행에 접근할 가능성이 있다.|  
|ref_or_null|ref와 마찬가지로 인덱스 접근 시 맨 앞에 저장되어 있는 NULL의 엔트리를 검색한다.|  
|eq_ref|조인이 내부 테이블로 접근할 때 기본키 또는 공유 키에 의한 loockup이 일어난다.<br>const와 비슷하지만 조인의 내부 테이블에 접근한다는 점이 다르다.|   
|range|인덱스 특점 범위의 행에 접근한다.|      
|fulltext|fulltext 인덱스를 사용한 검색|          
|system|테이블에 1행밖에 없는 경우의 특수한 접근 방식|   
|index_merge|여러개 인스턴스를 사용해 행을 가져오고 그 결과를 통합한다|     
|unique_subquery|IN 서브쿼리 접근에서 기본 키 또는 고유 키를 사용한다.<br>이 방식은 쓸데 없는 오베헤드를 줄여 상당히 빠르다.|       
|index_subquery|unique_sunquery 와 거의 비슷하지만, 고유한 인덱스를 사용하지 않는점이 다르다.<br>이 접근 방식도 상당히 빠르다.|               
   
## possible_keys 
* 이용 가능성이 있는 인덱스의 목록 

## key 
* 이용 가능성이 있는 인덱스의 목록 중, 실제로 옵티마이저가 선택한 인덱스     
* null 인 경우는, 행 데이터를 가져오기 위해 인덱스를 사용할 수 없다는 뜻이다.    
    
## key_len 
* 선택된 인덱스의 길의를 나타낸다(인덱스가 너무 긴 것도 비효율적이므로)   

## rows    
* 해당 접근 방식을 사용해서 몇 행을 가져왔는가를 표시한다.        
* 최초에 접근하는 테이블에 대해서 쿼리 전체에 의해 접근하는 행의 수     
* 그 이후에 테이블에 대헤서는 1행의 조인으로 평균 몇 행에 접근했는가를 표시한다.      
* 단, 어디까지나 통계 값으로 계산한 값이므로 실제 행 수와 반드시 일치하지 않는다.        

# filtered   
* 행 데이터를 가져와 WHERE 구의 검색 조건이 적용되면 몇행이 남는지를 표시한다.      
* 이 값도 통계 값 바탕으로 계산한 값이므로 현실의 값과 반드시 일치하지 않는다.      
  
# extra   
* 옵티마이저가 동작하는데 대해서 우리에게 알려주는 힌트다.     
* 이 필드는 EXPLAIN 을 사용해 옵티마이저의 행동을 파악할 때 아주 중요하다.     

|Extra|설명|
|-----|---|
|Using where|접근 방식을 설명한 것으로, 테이블에서 행을 가져온 후 추가적으로 검색조건을 적용해 행의 범위를 축소한 것을 표시한다.|
|Using index|테이블에는 접근하지 않고 인덱스에서만 접근해서 쿼티를 해결하는 것을 의미한다. 커버링 인덱스로 처리됨 index only scan이라고도 부른다|
|Using index for group-by|Using index와 유사하지만 GROUP BY가 포함되어 있는 쿼리를 커버링 인덱스로 해결할 수 있음을 나타낸다|
|Using filesort|ORDER BY 인덱스로 해결하지 못하고, filesort(MySQL의 quick sort)로 행을 정렬한 것을 나타낸다.|
|Using temporary|암묵적으로 임시 테이블이 생성된 것을 표시한다.|
|Using where with pushed|엔진 컨디션 pushdown 최적화가 일어난 것을 표시한다. 현재는 NDB만 유효|
|Using index condition|인덱스 컨디션 pushdown(ICP) 최적화가 일어났음을 표시한다.<br>ICP는 멀티 칼럼 인덱스에서 왼쪽부터 순서대로 칼럼을 지정하지 않는 경우에도 인덱스를 이용하는 실행 계획이다.|
|Using MRR|멀티 레인지 리드(MRR) 최적화가 사용되었음을 표시한다.|
|Using join buffer(Block Nested Loop)|조인에 적절한 인덱스가 없어 조인 버퍼를 이용했음을 표시한다.|
|Using join buffer(Batched Key Access)|Batched Key Access(BKAJ) 알고리즘을 위한 조인 버퍼를 사용했음을 표시한다.|
	
# JSON 형식 

```sql
explain format = json 
```
위와 같이 쿼리를 추가하면, 쿼리 결과를 JSON으로 받아볼 수 있다.    
    
```sql
explain format = json 
select m.*, o.*, t.* from member  m
inner join orders o on m.id = o.member_id
inner join transaction t on o.transaction_id = t.id
where m.id in (1, 2, 33)
```
```json
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "449.03"
    },
    "nested_loop": [
      {
        "table": {
          "table_name": "m",
          "access_type": "range",
          "possible_keys": [
            "PRIMARY"
          ],
          "key": "PRIMARY",
          "used_key_parts": [
            "id"
          ],
          "key_length": "8",
          "rows_examined_per_scan": 3,
          "rows_produced_per_join": 3,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "3.61",
            "eval_cost": "0.60",
            "prefix_cost": "4.21",
            "data_read_per_join": "6K"
          },
          "used_columns": [
            "id",
            "email",
            "name"
          ],
          "attached_condition": "(`sample`.`m`.`id` in (1,2,33))"
        }
      },
      {
        "table": {
          "table_name": "o",
          "access_type": "ref",
          "possible_keys": [
            "FKpktxwhj3x9m4gth5ff6bkqgeb",
            "FKrylnffj7sn97iepyqadlfnsg0"
          ],
          "key": "FKpktxwhj3x9m4gth5ff6bkqgeb",
          "used_key_parts": [
            "member_id"
          ],
          "key_length": "8",
          "ref": [
            "sample.m.id"
          ],
          "rows_examined_per_scan": 90,
          "rows_produced_per_join": 272,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "63.00",
            "eval_cost": "54.55",
            "prefix_cost": "121.76",
            "data_read_per_join": "279K"
          },
          "used_columns": [
            "id",
            "order_number",
            "member_id",
            "transaction_id"
          ]
        }
      },
      {
        "table": {
          "table_name": "t",
          "access_type": "eq_ref",
          "possible_keys": [
            "PRIMARY"
          ],
          "key": "PRIMARY",
          "used_key_parts": [
            "id"
          ],
          "key_length": "8",
          "ref": [
            "sample.o.transaction_id"
          ],
          "rows_examined_per_scan": 1,
          "rows_produced_per_join": 272,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "272.73",
            "eval_cost": "54.55",
            "prefix_cost": "449.03",
            "data_read_per_join": "820K"
          },
          "used_columns": [
            "id",
            "code",
            "partner_transaction_id",
            "payment_method_type"
          ]
        }
      }
    ]
  }
}
```

# WorkBench Visual Explain 

![image](https://user-images.githubusercontent.com/50267433/146504381-6951e694-283e-4762-92e1-5c4bd7bb0f51.png)

|색상|오브젝트 내 텍스트|EXPLAIN type|
|---|-------------|------------|
||Single row: constant|const|
||Unique Key Lookup|eq_ref|
||Non-Unique Key Lookup|ref|
||Fulltext Index Search|fulltext|
||Index Range Scan|range|
||Full Index Scan|index|
||Full Table Scan|ALL|


