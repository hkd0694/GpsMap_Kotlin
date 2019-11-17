# Kotlin Study (6/9) - 2019/11/07

Kotlin을 공부하기 위해 간단한 앱부터 복잡한 앱까지 만들어 봄으로써 Kotlin에 대해 하기!

총 9개의 앱 중 여섯 번째 앱

프로젝트명 : GpsMap

기능

* GPS로 현재 위치 정보를 얻어 지도에 표시한다.
* 주기적으로 현재 위치를 갱신하며 선을 그린다.
  

핵심 구성 요소

* Google Maps Activity : 지도를 표시하는 기본 템플릿이다.
  
* FusedLocationProviderClient : 현재 위치 정보를 얻는 클래스이다.


라이브러리 설정
* Anko : 인텐트 다이얼로그, 고르 등을 구현하는 데 도움이 되는 라이브러리
  
* play-services-maps : 구글 지도 라이브러리
  
* play-services-location : 위치 정보 라이브러리


## 위치 정보 얻기

주기적으로 위치 정보를 얻으려면 다음 순서대로 구현을 해야 한다.

1. 매니페스트에 위치 권한 추가
2. onResume() 메서드에서의 위치 정보 요청
3. 위치 정보 갱신 콜백 정의
4. onPause() 메서드에서의 위치 정보 요청 중지
   

>### 위치 권한 확인

구글 플레이 서비스를 최신 버전으로 업데이트해야 위치 서비스에 연결이 된다. 위치 서비스에 연결된 앱은 FusedLoactionProviderClient 클래스의 requestLocationUpdates() 메서드를 호출하여 위치 정보를 요청할 수 있다.

```kotlin

requestLocationUpdates(locationRequest : LocationRequest,
 locationCallback: LocationCallback, looper: Looper)
```

* locationRequest : 위치 요청 객체이다.
  
* locationCallback : 위치를 갱신되면 호출되는 콜백이다.
  
* looper : 특정 루퍼 클래스를 지정한다. 특별한 경우가 아니라면 null을 지정한다.



``` kotlin
val cursor = contentResolver.query(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
            null, //가여올 항목 배열
            null, //조건
            null, //조건
            MediaStore.Images.ImageColumns.DATE_TAKEN + " DESC") //찍은 날짜 내림차순
```

그리고 Manifest에 권한을 추가해야 되는데, 안드로이드 6.0 버전부터는 모든 앱이 외부에서 리소스 또는 정보를 사용하는 경우 앱에게 사용자에게 권한을 요청해야 한다. 매니페스트에 권한을 나열하고 앱을 실행 중에 사용자에게 각 권한을 승인 받으면 된다.

>### 위치 정보 요청

```kotlin

override fun onResume() {
        super.onResume()
        addLocationListener()
    }

    @SuppressLint("MissingPermission")
    private fun addLocationListener(){
        fusedLocationProviderClient.requestLocationUpdates(locationRequest,locationCallback, null)
    }

```

requestLocationUpdates() 메서드에서 총 3개의 인자에 대해서 설명을 하자면 우선 첫번째는 LocationRequest 객체로 우치 정보를 요청하는 시간 주기를 설정하는 개체이다.

```kotlin
locationRequest = LocationRequest()
        locationRequest.priority = LocationRequest.PRIORITY_HIGH_ACCURACY
        // 업데이터 인터벌
        // 위치 정보가 없을 때는 업데이트 안 함
        // 상황에 따라 짧아질 수 있음, 정확하지 않음
        // 다른 앱에서 짧은 인터벌로 위치 정보를 요청하면 짧아질 수 있음
        locationRequest.interval = 10000
        //정확함. 이것보다 짧은 업데이터는 하지 않음
        locationRequest.fastestInterval = 5000

```

두번째 인자는 LocationCallback 객체로 최근 현재 위치에 대한 Location 객체를 얻을 수 있다. 

```kotlin

inner class MyLocationCallBack : LocationCallback(){
        override fun onLocationResult(locationResult: LocationResult?) {
            super.onLocationResult(locationResult)

            val location = locationResult?.lastLocation
        }
}

```

>### 위치 정보 삭제

onResume()메서드에서 위치 정보를 요청해서 앱이 동작 중일 때만 위치 정보를 갱신한다면, 삭제해야 할 경우에는 앱이 동작하지 않을 때 불리는 onPause() 메서드에서 삭제를 하면 된다.

```kotlin

override fun onPause() {
        super.onPause()
        removeLocationListener()
    }

private fun removeLocationListener(){
    fusedLocationProviderClient.removeLocationUpdates(locationCallback)
}

```

>### 화면 유지하기

지도를 테스트 하거나 자동으로 꺼지면 테스트하기가 어렵기 때문에 화면 방향을 고정하고, 화면이 자동으로 꺼지지 않도록 코드를 추가해야 한다.

```kotlin

override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_maps)
        //화면이 꺼지지 않게 하기
        window.addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
        //세로 모드로 화면 고정
        requestedOrientation = ActivityInfo.SCREEN_ORIENTATION_PORTRAIT

        // SupportMapFragment를 가져와서 지도가 준비되면 알림을 받는다.
        //val mapFragment = supportFragmentManager
            //.findFragmentById(R.id.map) as SupportMapFragment
        //mapFragment.getMapAsync(this)

        //locationinit()
    }
```

## GpsMap_Kotlin을 통해 배운 것들

## run() 함수

run()함수는 익명 함수처럼 사용하는 방법과, 객체에서 호출하는 방법을 몯 제공한다. 익명 함수처럼 사용할 때는 블록의 결과로 반환한다. 블록안에 선언된 변수는 모두 임시로 사용되는 변수이다. 

사용 예제

```kotlin

val avg = run {
    val korean = 100
    val english = 80
    val math = 50

    (korean + english + math) / 3.0
}

```

GpsMaps_Kotlin에서 사용했을 경우

```kotlin

    location?.run {
        val latLng = LatLng(latitude,longitude)
        mMap.animateCamera(CameraUpdateFactory.newLatLngZoom(latLng, 17f))  
        Log.e("Start","위도 : $latitude, 경도 : $longitude")

        //PolyLine에 좌표 추가
        polylineOptions.add(latLng)

        //선 그리기
        mMap.addPolyline(polylineOptions)
    }

```

안전한 호출(?.)을 할 수 있어서 with()함수보다는 더 유용하게 사용 가능하다.


## Kotlin Study List

1. [BmiCalculator](https://github.com/hkd0694/BmiCalc_Kotlin)
2. [StopWatch](https://github.com/hkd0694/StopWat_Kotlin)
3. [MyWebBrowser](https://github.com/hkd0694/MyWeb_Kotlin)
4. [TiltSensor](https://github.com/hkd0694/TSens_Kotlin)
5. [MyGallery](https://github.com/hkd0694/MGallery_Kotlin)
6. [GpsMap](https://github.com/hkd0694/GpsMap_Kotlin)
7. [Flashlight](https://github.com/hkd0694/FLight_Kotlin)
8. [Xylophone](https://github.com/hkd0694/Xyloph_Kotlin)
9. [Todo 리스트](https://github.com/hkd0694/TodoList_Kotlin)