# Lifecycle (생명주기)


## Activity Lifecycle

### onCreate()
- 액티비티가 생성될 때 실행됩니다.

### onStart()
- 여기부터 사용자에게 Activity 가 표시됩니다.

### onResume()
- 사용자와 본격적으로 앱과 상호작용할 수 있습니다.
- 특정 상황이 발생하지 않으면 계속 onResume 상태에 머뭅니다.

### onPause()
- 해당 Activity 가 foreground process 를 떠나있을 때 호출됩니다.  
  (상황에 따라 Activity 가 여전히 사용자에게 보이는 경우가 있음)
- UI 관련 리소스와 작업을 완전히 해제할 경우는 onPause 를 사용하지 않는 것이 좋습니다.
- 일부 Activity 가 보이는 상황에서 다시 Activity 가 포커스를 찾았을 때,

### onStop()
- 기존 Activity 가 아예 화면에서 벗어나면 호출됩니다.
- Activity 객체가 메모리 안에 머물게 되고, 시스템은 각 View 객체의 현재 상태를 기록합니다.
  (그래서 홈버튼으로 화면을 벗어났다가 다시 돌아와도 UI 데이터들이 그대로 남아 있음)
- 필요하지 않은 리소스를 해제하거나 CPU 를 비교적 많이 소모하는 종료 작업을 실행합니다.

### onDestroy()
- 두가지 경우로 Activity 가 소멸되기 직전에 호출됩니다.
  - Activity 가 종료되는 경우
  - 화면 회전과 같은 이유로 시스템이 일시적으로 Activity 를 소멸시키는 경우
- 리소스 해제가 있으면 이곳에서 처리합니다.

### onRestart()
- 유저의 액션에 따라 해당 Activity 가 홤녀에서 벗어났다가 다시 불러왔을 때 호출됩니다.
- `onStop -> onRestart -> onStart` 순서대로 호출됩니다.

### 특정 상황에서의 lifecycle
#### [1] `A` Activity 실행
1. `A` onCreate
2. `A` onStart
3. `A` onResume

#### [2] `A` -> `B` Activity 이동
1. [1]에서 시작
2. `A` onPause
3. `B` onCreate
4. `B` onStart
5. `B` onResume
6. `A` onPause
7. `A` onSaveInstanceState

#### [3] `B`->`A` Activity 뒤로가기
1. [2]에서 시작
2. `B` onPause
3. `A` onRestart
4. `A` onStart
5. `A` onResume
6. `B` onStop
7. `B` onDestroy
