---
title: UI Test 사용기
---

# UI Test ?

클라이언트는 테스트가 까다롭다. 수많은 UI와 그에 따른 복잡한 인터렉션들이 혼합되어 있기 때문이다. 처음에 유닛 테스트란걸 알게 된 후 TDD로 개발을 하게 되었는데, 처음에 뷰를 어떻게 테스트해야 할 지 정말 막막했다. 그래서 처음에는 비즈니스 로직만 테스트 코드를 작성하다가 후에 MVP 패턴을 적용해 뷰, 뷰로직, 비즈니스 로직들을 테스트 할 수 있게 되었다. 하지만 그럼에도 실제로 QA 를 하면 정말 예상하지 못한 인터렉션에서 앱이 비정상으로 작동하는 것을 보며 낙심하게 되었다. 특히나 안드로이드는 기종이 매우 다양하기에 모든 폰을 수동으로 테스트하는 것은 정말 고된 일이었다.

UI 테스트에는 이런 장점이 있다
+ 좀 더 사용자 위주의 테스트를 통해 사용자가 겪을 수 있는 테스트를 해볼 수 있다.
+ 전문적인 QA 가 없는 환경에서 자동화 테스트는 시간을 크게 절약할 수 있다.
+ 테스트 속도가 빠르다
+ 가상의 클라우드 테스트로 많은 다양한 기종을 테스트 할 수 있다.

# 어떤 종류가 있나

찾아본 것은

1. 에스프레소
2. UIAutomator
3. Robolectric
4. Appium
5. Calabash
6. 등등..

실제로 써 본것은

1. Calabash
2. Appium
3. UIAutomator 2

내가 직접 써 본 3가지 UI Testing framework를 훑어보겠다

# Appium
> https://appium.io/

위에 찾아본 UI Testing framework 중에 가장 기능이 풍부하다.

+ 가장 눈에 띄는 것은 하나의 테스트 코드로 Android, iOS, FirefoxOS 와 같이 다른 플랫폼을 모두 지원한다
+ 지원하는 언어가 상당히 많다. Ruby, Javascript, Python, Java 등등..
+ 설치가 비교적 쉽다
+ AWS Device Farm 에서 Appium 을 지원한다

## 간단한 사용 방법

### Install

1. Android SDK
+ Java

+ Node : Appium 환경 설정
```
$ brew install node
$ npm install -g appium
$ npm install wd
```

+ Ruby : 스크립트 개발 언어
```
$ brew install ruby
```

+ Selenium-Cucumber : BDD를 지원하는 고수준 테스팅 툴
```
$ gem install selenium-cucumber
$ ls
features/
```

### 터미널 환경에서 시작

디렉토리 구조

+ my_first.feature : 사용자 스토리(비지니스 요구사항)와 요구사항을 테스트하는 다수의 테스트 시나리오와 각 시나리오를 구성하는 다수의 테스트 스텝을 가지는 파일이다.
+ step_definition 폴더 : my_first.feature에서 시나리오를 구성하는 테스트 스텝은 미리 정의된 구현 코드를 가지는 스텝일 수도 있고, 정의되지 않아 개발자가 구현을 만들어줘야 하는 스텝일 수도 있다. 정의되지 않은 스텝일 경우 스텝에 대한 구현 코드 파일을 가지는 폴더이다 .
+ support 폴더 : 테스트 진행 시 로드되는 파일들로, 테스트 코드를 작성할 때 개발자가 별도로 만든 루비 파일들을 사용하고자 할 때, 혹은 각 테스트 전 후에 공통적인 코드를 추가하고자 할 때 이 폴더 내의 파일에 코드를 추가한다. 
+ actual_images, expected_images, Image_difference, screenshots 폴더 : 테스트 도중 캡쳐한 스크린 샷이나 테스트 할 때 사용하는 이미지 등을 가지는 폴더이다 .

### feature 스크립트 수정

+ 한 스크립트에는 한 시나리오를 가지고 검증을 위한 테스트 스텝 있다
+ Given : 검증 전 준비 단계, 어떤 상태 등을 나타낸다
+ When : 사용자가 특정 액션을 취했음을 나타낸다
+ Then : 액션에 대한 결과가 어떠해야 함을 나타낸다

```
Feature: Login feature
  Scenario: As a valid user I can log into my app
    When I enter "gudnam@switcher.co.kr" into input field having id "io.gudnam.appiumtest:id/email"
    When I enter "1234512345" into input field having id "io.gudnam.appiumtest:id/password"
    When I click on element having id "io.gudnam.appiumtest:id/email_sign_in_button"
    Then element having id "io.gudnam.appiumtest:id/tv_result" should have text as "Logined"
```

### Appium 실행

Appium Test Server 실행
```
$ appium
```

Selenium-Cucumber로 생성한 경로로 이동
```
$ cd [APPIUM_PROJECT]
$ ls
features/
```
Cucumber 명령어로 Platform 과 App 경로 인자로 실행
```
$ cucumber PLATFORM=android APP_PATH=app-debug.apk
```
결과
```
Feature: Login feature

  Scenario: As a valid user I can log into my app                                                   # features/my_first.feature:3
    When I enter "gudnam@switcher.co.kr" into input field having id "io.gudnam.appiumtest:id/email" # selenium-cucumber-3.1.5/lib/selenium-cucumber/input_steps.rb:4
    When I enter "1234512345" into input field having id "io.gudnam.appiumtest:id/password"         # selenium-cucumber-3.1.5/lib/selenium-cucumber/input_steps.rb:4
    When I click on element having id "io.gudnam.appiumtest:id/email_sign_in_button"                # selenium-cucumber-3.1.5/lib/selenium-cucumber/click_elements_steps.rb:3
    Then element having id "io.gudnam.appiumtest:id/tv_result" should have text as "Logined"        # selenium-cucumber-3.1.5/lib/selenium-cucumber/assertion_steps.rb:13

1 scenario (1 passed)
4 steps (4 passed)
0m14.212s
```

## 하지만 탈락
선택하지 않은 이유는 환경 설정할 Dependency 가 꽤 있는 편이다. 그리고 Appium 사용 방법이 터미널 말고도 GUI 버전, JUnit, TestNG, Python 등 여러개가 있는데, AWS Device Farm 에서 Appium JUnit, TestNG, Python 을 지원한다. Python 은 모르겠지만 JUnit, TestNG 를 사용하려면 이클립스나 IntelliJ 같은 다른 IDE 에서 Maven 프로젝트를 또 만들어 그곳에 테스트 코드를 작성해야 한다. Android Studio와 같이 말이다. 난 터미널만 해보긴 했지만 UI Test 환경에 익숙하지도 않은 내가 위에 환경을 모두 같이 할 자신이 없었다.

# Calabash

Appium 과 비슷하지만 Android, iOS 같은 다른 플랫폼에 테스트 코드를 적용하려면 한번 더 빌드해야 하는 수고로움이 있다. 언어는 Ruby 하나. 설치도 대부분 Appium과 비슷하다. 클라우드 테스트로는 AWS Device Farm, Xamarin Cloud Device 가 있다.

## 간단한 사용 방법

### 필요한 것

Ruby

- Ruby 2.3.1p112 (Xamarin Test Cloud 에서 해당 버전 사용)
- [Ruby 업데이트 하기](https://rorlab.gitbooks.io/railsguidebook/content/contents/rbenv.html)
  - Ruby -v 버전 안바뀌는 현상
    - rbenv shell 2.3.1
    - no such command 'shell'
    - [해결](https://github.com/rbenv/rbenv/issues/142)
      - $ export PATH="$HOME/.rbenv/bin:$PATH"
      - $ eval "$(rbenv init -)"

Calabash client libraries

- calabash-android 설치
  - $ gem install calabash-android

Android

- bash_profile 수정

```
$ vi ~/.bash_profile

...
export ANDROID_HOME=/Users/{USER}/Library/Android/sdk
...

:wq

$ source ~/.bash_profile
```

### 시작

Calabash 테스트를 위해 간단한 데모 만들어보기
```
$ mkdir CalabashDemo
$ cd CalabashDemo
$ calabash-android gen

----------Question----------
I'm about to create a subdirectory called features.
features will contain all your calabash tests.
Please hit return to confirm that's what you want.
---------------------------


----------Info----------
features subdirectory created.
---------------------------
$ ls
features/
```

Apk 파일 Resign
  - Resign 하는 이유는 Calabash 테스트 서버와 Application 이 동일한 키로 서명돼야 하기 때문이다

```
$ cp {ANDDROID_PROJECT_DIR}/app-demo.apk {DEMO_DIR}/.
$ ls
app-demo.apk  features/
$ calabash-android resign app-demo.apk
```

Feature 파일 수정 (Appium 에서 작성한 것과 동일)

```
Feature: Login feature
  Scenario: As a valid user I can log into my app
    When I enter "gudnam@switcher.co.kr" into input field having id "io.gudnam.appiumtest:id/email"
    When I enter "1234512345" into input field having id "io.gudnam.appiumtest:id/password"
    When I click on element having id "io.gudnam.appiumtest:id/email_sign_in_button"
    Then element having id "io.gudnam.appiumtest:id/tv_result" should have text as "Logined"
```

테스트 실행

```
$ calabash-android run app-demo.apk
```

## 탈락한 이유

Appium 과 역시 거의 동일하다. 꽤 높은 러닝커브

# UIAutomator2
> 선택

기능은 아무래도 Appium, Calabsh 보다는 많이 떨어지는 편이다. 그럼에도 UIAutomator 를 사용한 이유는 가장 쉬웠기 때문이다. Instrumentation 테스트이기 때문에 따로 환경 설정할 것도 없고 안드로이드 프로젝트 내에서 유닛 테스트 개발하듯이 개발하면 된다. 그리고 Appium, Calabsh 도 각각의 서버에서 앱을 테스트 할 때는 UI Automator 를 통해 테스트하기 떄문에 실제 UI Test 할 때 부족한 기능은 없어 보였다. 하나의 테스트 코드로 Android, iOS 두개 플랫폼을 지원 못한건 아쉽지만 Baby step 을 걷기로 했다.

사실 UIAutomator2 도 나온지 얼마 안됐고 많이 사용하지 않다보니 자료가 그렇게 많진 않다.

지원하는 클라우드 테스트는 AWS Device Farm

## 간단한 사용 방법

### Build.gradle 수정
```
defaultConfig {
     ...
     testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
 }
 ...
androidTestCompile 'com.android.support:support-annotations:25.3.1'
androidTestCompile 'com.android.support.test:runner:0.4.1'
androidTestCompile 'com.android.support.test:rules:0.4.1'
androidTestCompile 'com.android.support.test.uiautomator:uiautomator-v18:2.1.2'
```

### UI Test

Device 초기화 및 앱 실행
```
protected UiDevice device;
...
@Before
public void before() {
    device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation());
    assertNotNull(device);
    openApp();
}

private void openApp() {
    // 홈 화면으로 이동
    device.pressHome();

    // 앱 실행
    Context context = InstrumentationRegistry.getInstrumentation().getContext();
    Intent intent = context.getPackageManager().getLaunchIntentForPackage(BuildConfig.APPLICATION_ID);
    intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK);
    context.startActivity(intent);

    // 앱을 실행하는 동안 비동기적으로 로딩되는 UI 요소들을 LAUNCH_TIMEOUT 동안 기다려 동기화
    device.wait(Until.hasObject(By.pkg(BuildConfig.APPLICATION_ID).depth(0)), LAUNCH_TIMEOUT);
}
```

로그인 테스트
```
@Test
public void testActionSignInButton() {
    findByRes("email").setText("gudnam@switcher.co.kr");
    findByRes("password").setText("12341234");
    findButton(R.string.action_sign_in).click();
    device.wait(Until.hasObject(byRes("email")), DEFAULT_TIMEOUT);
    assertHas(byText(R.string.logined));
}
```

유틸
```
protected UiObject2 findByRes(String resourceIdString) {
    // resourceId가 표시된 요소를 찾는다.
    return findObject(byRes(resourceIdString));
}
protected BySelector byRes(String resourceIdString) {
    // resourceId가 표시된 요소를 나타내는 BySelector.
    return By.res(BuildConfig.APPLICATION_ID, resourceIdString);
}
protected UiObject2 findButton(int textResourceId) {
    // textResourceId가 표시된 버튼을 찾는다.
    return findObject(button(textResourceId));
}
protected BySelector button(int textResourceId) {
    // textResourceId가 표시된 버튼을 나타내는 BySelector.
    return By.text(string(textResourceId)).clickable(true);
}
protected UiObject2 findObject(BySelector selector) {
    device.wait(Until.hasObject(selector), DEFAULT_TIMEOUT);
    return device.findObject(selector);
}
```
# 마무리

UI Test를 통해 내가 노리는 것은 두가지이다. 하나는 더욱 안정성이 보장된 앱을 배포하는 것과 두번째는 회사에 없는 다양한 폰을 테스트하는 것이다. 이것 또한 완벽하다고 할 수는 없겠지만 그래도 앞으로의 고객들에게 더 나은 서비스가 제공될 것이라 생각된다.

다음에는 클라우드 테스트에 UIAutomator2를 적용해 여러 가상 디바이스를 테스트해 보겠다.

