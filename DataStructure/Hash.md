# 📚 해시(Hash)

</br>

## Hash 란?
- 해시란 단방향 암호화 기법인 `해시함수`를 이용하여 생성된 고정된 길이의 비트열이다.
- 변경 전 원래 데이터 값을 키(key), 변경 후 데이터의 값을 해시값(hash value), 키와 값으로 변환되는 과정을 해싱(Hasing)이라고 한다.
- 해시함수를 사용한 키와 데이터 값을 함께 저장하는 자료구조를 `해시테이블`, `해시맵`이라고 한다.
- 자바에서 해시테이블과 해시맵은 다르다.

</br></br>

## HashTable
- key, value 로 데이터를 저장하는 자료구조
- `synchronized` 키워드를 사용하여 스레드 안정성을 보장한다.
- key 에 null 을 허용하지 않는다.

</br></br>

## HashMap
- key, value 로 데이터를 저장하는 자료구조
- key 에 null 을 허용한다. (중복을 허용하지 않기 때문에 하나만 가능)
- 동기화를 하지 않기 때문에 HashTable 보다 빠르다.

</br></br>

## ConcurrentHashMap
- 동기화를 제공하는 HashMap
- HashTable 은 Collection 이 나오기 전부터 존재한 구형버전이기 때문에 HashMap에 비해 느려 ConcurrentHashMap 을 사용한다.
