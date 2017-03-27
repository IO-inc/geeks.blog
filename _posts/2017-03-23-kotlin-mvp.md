---
layout: post
categories: Study
tags: [Kotlin, DI]
author: Gudnam
title: 코틀린에서 MVP 패턴 사용해보자!
---
<div class="message"> 
MVC, MVVM, MVP 등의 View logic 패턴중에 MVP 를 선택한 이유는 View & Logic 분리가 쉽기도 하지만 테스트 코드를 깔끔하게 작성할 수 있기 때문이다. 
</div> 

# 개념 간단히 살펴보기

## MVP ?
![mvp](/public/img/mvp_architecture.png)

	VIEW
		오직 화면 구성에 관련된 기능만 있다
		이곳에서 로직은 왠만하면 피하라!
		모든 이벤트는 Presenter 에게 전가하기

	PRESENTER	
		원하는 화면을 결정해 View 에게 전달한다 (View 와 1:1 관계)
		View 를 보여주기 위한 Logic 은 이곳에서!
		화면을 테스트하고 싶다면 이곳에 테스트 코드를 작성하라!

	MODEL
		화면에 보여지는 정보가 있는 곳
		복잡한 Business Logic 담당
		예를 들어 DB, 서버 연동 등등에서 데이터 가져올 때 사용
		
# Kotlin & MVP 구현해보기

## 개발을 위한 환경설정
> Kotlin plugink

### Kotlin
1. Install kotlin plugin
![install_kotlin](/public/img/install_kotlin.png)

2. 빌드 스크립트
{% highlight js %}
// build.gradle
buildscript {
  ext.kotlin_version = '1.0.4'
  repositories {
    jcenter() 
  }
  dependencies {
    ...
    classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
  }
}
{% endhighlight %}

{% highlight js %}
// app/build.gradle
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'

android {
  ...
}

dependencies {
  compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
  ...
}
{% endhighlight %}

## 간단한 고객 정보 조회 앱을 개발해보자

### MVP 

* View
{% highlight js %}
// Create View
interface PresentationView

---

// Create MainView
class MainView : PresentationView {
  fun showUserName(name: String)
}
{% endhighlight %}

* Model 
{% highlight js %}
// Create Model : 사용자 정보
data class UserInfo(val id: Int, val name: String)

---

// Create Repository : UserInfo 를 관리
class UserInfoRepositoryImpl : UserInfoRepository {

    fun getList() : List<UserInfo> {
        var list = ArrayList<UserInfo>()
        list.add(UserInfo(0, "gudnam"))
        list.add(UserInfo(1, "nada"))
        return list
    }

    override fun getUserInfoList(): List<UserInfo> {
        return getList()
    }

    override fun getUserInfo(id: Int): UserInfo? {
        var userInfo: UserInfo? = getList().filter { it.id == id }[0]

        return userInfo
    }
}
{% endhighlight %}
{% highlight js %}
// Create Event : Model 을 반환
class UserInfoEvent(val userInfo: UserInfo) : Event

{% endhighlight %}
{% highlight js %}
// Create Interactor : Data 를 받아 Presenter 에 넘겨줌
class FindUserInfoInteractor() : Interactor {
    var id: Int = 0

    override fun setOnCallback(callback: ((Event)->Unit)) {
        var repository = UserInfoRepositoryImpl()
        val userInfo = repository.getUserInfo(id)
        userInfo?.let { callback.invoke(UserInfoEvent(it)) }
    }
}

{% endhighlight %}

* Presenter
{% highlight js %}
// Create MainPresenter
class MainPresenter(
        override val view: MainView,
        val interactor: FindUserInfoInteractor) : Presenter<MainView> {

    fun onResume() {
        interactor.id = 0
        interactor.setOnCallback {
            var event = it as UserInfoEvent
            view.showUserName(event.userInfo.name)
        }
    }
}
{% endhighlight %}

* Activity
{% highlight js %}
// Create MainActivity : MainView 상속!
class MainActivity : BaseActivity(), MainView {

    override val layoutResource = R.layout.activity_main
    lateinit var presenter: MainPresenter

    val tv_name by lazy { findViewById(R.id.tv_name) as TextView }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        presenter = MainPresenter(this, FindUserInfoInteractor())
    }

    override fun onResume() {
        super.onResume()
        presenter.onResume()
    }

    override fun showUserName(name: String) {
        tv_name.text = name
    }
}
{% endhighlight %}

Activity(View)에서 특정 이벤트 발생 시 이를 Presenter 에게 알려준다. Presenter 에게 요청을 할 때 한가지 중요한 점은 Presenter 는 자신이 주체적으로 행동하는 객체가 아니라는 점이다. 예를 들어 Presenter.initialize() 는 Presenter 가 능동적으로 초기화 하는 것 같이 보인다. 하지만 Presenter 의 역할은 **View의 요청을 받고 요청에 대한 처리를 하는** 역할이기 때문에 View 에 종속되는게 바람직하다. 그렇다면 initialize 대신 메소드 이름은 onCreate, onCreateView, onResume 같은 Activity 에서 발생한 메소드명으로 하는게 좋겠다. 사소한 것이라 생각할 수 있지만 이런 생각의 차이가 객체간의 Coupling 으로 이어질 수 있겠다.
