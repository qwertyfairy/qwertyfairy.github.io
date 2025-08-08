---
layout: post
category: works
---


Android의 위젯들은 모두 View를 상속 받고 있다. 이 View에는 onTouchEvent가 구현되어있는데 사용자를 통해 입력 받은 터치 이벤트가 여기서 제어된다. 기본적으로 onTouchEvent는 각 View의 상태에 맞춰 터치 이벤트가 처리되도록 구현되어있다.  

Zoom in / out 기능을 구현하고자 하는 FluidVideoView은 FramLayout의 ViewGruop을 상속 받은 형태로 단순 출력 용 View이기에 onTouchEvent에서 MotionEvent.ACTION_DOWN이 후 더 이상의 이벤트를 전달 받을 수 없었다. 상호 작용이 필요 없는 View는 ACTION_DOWN이후에 처리를 전부 무시하기 때문이다. 

현재 View에서 사용자 이벤트를 모두 받기 위해 상호작용 되는 View라는 설정이 필요했고 isClickable을 true로 변경해 FluidVideoView의 onTouchEvent에서 모든 이벤트를 수신할 수 있도록 했다.  

```kotlin
..
isClickable = turu
..
```
<br>
Android에서는 GestureDetector를 사용해 사용자의 동작을 감지 할 수 있다. 간단히 핀치 줌(Pinch Zoom)동작으로 Zoom in/out 기능을 제공하고자 했으며 해당 동작은 ScaleGestureDetector로 감지할 수 있다. ScaleGestureDetector 클래스에서 사용자의 Scale동작을 판단해주는데 이때 호출될  ScaleGestureDetector.OnScaleGestureListener를 구현해 주어야 한다. 


### **ScaleGestureDetector.OnScaleGestureListener**
- public boolean onScaleBegin(@NonNull ScaleGestureDetector detector)<br>
Scale 이벤트가 시작될 때 호출. false인 경우 onScale, onScaleEnd가 호출되지 않는다.
- public boolean onScale(@NonNull ScaleGestureDetector detector)<br>
Scale 이벤트가 진행 중일 때 호출. 반환값으로 Scale 처리 여부를 결정한다.
- public void onScaleEnd(@NonNull ScaleGestureDetector detector)<br>
Scale 이벤트가 끝났을 때
<br>

<br>
사용자가 핀치 줌 할 때 영상의 Scale이 즉시 반영되기 때문에 onScale의 반환 값은 true로 선언했다. 
이 값을 false로 선언하면 onScaleEnd에 최종 scaleFactor값이 넘어오게 된다. 

![image](/public/img/video_zoom_sample00.gif)

<br>
Scale은 영상(Video) View에 영향을 받는데, Youtube처럼 현재 리사이즈 모드(꽉찬 화면, 화면 맞춤, 화면 자름, 원본화면)에 해당하는 오차 허용 범위 만큼 Scale 상태가 되면 사용자에게 하이라이트로 알리고 해당 모드로 원복 하는 기능을 제공하고자 했다.때문에 Scale시 영향을 받는 영상과 하이라이트 View정보가 필요했고 이 두 가지를 지닌 FluidVideoView가 필요하다 판단해 ScaleGestureListenr생성시 파라미터로 전달했다.

```kotlin
class VideoViewScaleGestureListener(
    private val videoView: FluidVideoView,
): ScaleGestureDetector.OnScaleGestureListener { ... }
```
<br>
실제 Scale을 처리하는 onScale()는 다음과 같이 구현했다. 매개변수로 전달되는 detector의 확대/축소값scaleFactor을 currentScaleFactor로 누적 시켜 영상 View에 Scale값을 반영했다. 확대/축소의 범위는 기존 기획을 유지했다. (최소 0.5배 최대 3.5배 확대 가능)

<br>
실시간으로 현재 Scale되는 영상이 리사이즈 모드에 해당하는 영상 크기인지 확인하고 하이라이트 표시를 위해 이것을 판단하는 FluidVideoView의 detectResetScaleScopeHighlight()도 호출해주었다. 해당 함수는 단순  하이라이트 노출/비노출 처리만 수행한다. 

```kotlin
  ...
  companion object {
	  private const val MIN_SCALE_FACTOR = 0.5f // 최소 축소 범위
	  private const val MAX_SCALE_FACOTR = 3.5f // 최대 확대 범위
  }
  
  private var currentScaleFactor = 1.0f // Scale 값
  
  override fun onScale(detector: ScaleGestureDetector): Boolean {
        currentScaleFactor *= detector.scaleFactor
        currentScaleFactor = currentScaleFactor.coerceIn(MIN_SCALE_FACTOR, MAX_SCALE_FACTOR)
        videoView.setVideoContentScale(currentScaleFactor) // 영상 View의 Scale 반영
        videoView.detectResetScaleScopeHighlight() // Highlight 확인
        return true 
    
    }
```
<br>
Scale이벤트가 끝났을때 호출되는 onScaleEnd()는 다음과 같이 구현했다. Scale 이벤트가 끝나면 하이라이트를 표시할 필요가 없기에 비노출 하도록 하였고 현재 Scale된 영상의 사이즈가 원복 영역에 해당하는지 확인하고 원복하는 처리를 수행하는 FluidVideoView의 resetVideoContentScaleIfInResetScope()를 호출해 주었다. 해당 함수는 수행 결과 값을 반환 한다. 수행이 되었다면 ture를 반환하므로, 이때 currentScaleFactor도 초기화 한다.

```kotlin
  override fun onScaleEnd(detector: ScaleGestureDetector) {
        videoView.setScaleHighlightScopeViewVisibility(View.GONE)
        val isVideoScaleReset = videoView.resetVideoContentScaleIfInResetScope()
        if(isVideoScaleReset){
            currentScaleFactor = 1.0f
        }
  }
```
<br>
ScaleGestureDetector.OnScaleGestureListener를 구현한 전체적인 코드는 다음과 같다. 

```kotlin
class VideoViewScaleGestureListener(
    private val videoView: FluidVideoView,
): ScaleGestureDetector.OnScaleGestureListener {

    companion object{
        private const val  MIN_SCALE_FACTOR = 0.5f
        private const val  MAX_SCALE_FACTOR = 3.5f
    }

    private var currentScaleFactor = 1.0f
    
    override fun onScaleBegin(detector: ScaleGestureDetector): Boolean {
        return true
    }
    
    override fun onScale(detector: ScaleGestureDetector): Boolean {
        currentScaleFactor *= detector.scaleFactor
        currentScaleFactor = currentScaleFactor.coerceIn(MIN_SCALE_FACTOR, MAX_SCALE_FACTOR)
        videoView.setVideoContentScale(currentScaleFactor)
        videoView.detectResetScaleScopeHighlight()
        return true
    }

    override fun onScaleEnd(detector: ScaleGestureDetector) {
        videoView.setScaleHighlightScopeViewVisibility(View.GONE)
        val isVideoScaleReset = videoView.resetVideoContentScaleIfInResetScope()
        if(isVideoScaleReset){
            currentScaleFactor = 1.0f
        }
    }
}
```
<br>
이제 사용자의 제스처를 감지하기 위해 사용자의 터치 이벤트를 ScaleGestureDetetor로 전달 해야 한다.<br> FluidVideoView의 onTouchEvent를 override하여 사용자 이벤트를 전달해준다.

```kotlin
    // FluidVideoView.kt 
    ..
    private val scaleListener = VideoViewScaleGestureListener(this)
    private val scaleDetector = ScaleGestureDetector(context, scaleListener)

    override fun onTouchEvent(event: MotionEvent?): Boolean {
        event?.let {
            scaleDetector.onTouchEvent(event)
        }
        return super.onTouchEvent(event)
    }
    ..
```
<br>

## 결과 화면

![image](/public/img/video_zoom_sample01.gif)
