# 📚 Android 4대 컴포넌트

</br>

## Activity
- 사용자와 상호작용을 담당하는 유저 인터페이스
- ex) 화면 구성

</br></br>

## Service
- 백그라운드에서 오래 실행되는 작업을 수행할 수 있는 어플리케이션 구성 요소
- 사용자 인터페이스를 제공하지 않음
- 포그라운드 (Foreground)
  - 사용자에게 잘 보이는 몇몇 작업 수행 (예를들어, 오디오 앱이면 오디오트랙을 재생할 때 사용)
  - 포그라운드 서비스는 알림을 표시해야 함
  - 사용자가 앱과 상호작용하지 않을 때도 계속 실행됨
- 백그라운드 (Background)
  - 사용자에게 직접 보이지 않는 작업 수행
  - API 26 이상을 대상으로 하면, 앱이 포그라운드에 있지 않을 때 시스템에서 백그라운드 서비스 실행에 대한 제한을 적용.
    - 즉시 실행해야 하는 작업 : Kotlin coroutine 사용
    - 지연된 작업 : WorkManager 사용
    - 정시에 실행해야 하는 작업 : AlarmManager
- 바인드 (Bind)
  - 바인딩된 서비스는 클라이언트-서버 인터페이스를 제공하여 구성 요소가 서비스와 상호작용하게 하며 결과를 받을 수 있음
- ex) 백그라운드 음악 재생

</br></br>

## BroadcastReceiver
- 각종 이벤트와 정보를 받아 핸들링하는 컴포넌트
- ex) 네트워크 변경, 문자 수신 등

</br></br>

## ContentProvider
- 다른 어플리케이션의 데이터를 제공하는데 사용되는 컴포넌트
- ex) 연락처 데이터 조회 등

</br></br>

---
### 참고 문헌
[Android Developer - 서비스 개요](https://developer.android.com/guide/components/services?hl=ko)  
[Android Developer - 브로드캐스트 개요](https://developer.android.com/guide/components/broadcasts?hl=ko)
