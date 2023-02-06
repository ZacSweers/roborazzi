# Roborazzi

**Make JVM Android integration test visible**

## How to use

### Generate gif automatically

```kotlin
@Test
fun captureRoboGifSample() {
  onView(ViewMatchers.isRoot())
    .captureRoboGif("build/test.gif") {
      // launch
      launch(MainActivity::class.java)
      // move to next page
      onView(withId(R.id.button_first))
        .perform(click())
      // back
      pressBack()
      // move to next page
      onView(withId(R.id.button_first))
        .perform(click())
    }
}
```

<img width="443" alt="image" src=https://user-images.githubusercontent.com/1386930/215935187-0b4579cb-dffa-4fcd-a633-a2b665e6910c.gif >

### Generate gif automatically with Test Rule

With the JUnit Test rule, you do not need to name the gif image, and if you prefer, you can output the gif image only when the test fails.

This test outputs this file.

`build/outputs/roborazzi/com.github.takahirom.roborazzi.sample.RuleTestWithOnlyFail_captureRoboGifSampleFail.gif`

```kotlin
@RunWith(AndroidJUnit4::class)
class RuleTestWithOnlyFail {
  @get:Rule val roborazziRule = RoborazziRule(
    captureRoot = onView(isRoot()),
    captureOnlyFail = true
  )
  
  @Test
  fun captureRoboGifSampleFail() {
    // launch
    launch(MainActivity::class.java)
    // move to next page
    onView(withId(R.id.button_first))
      .perform(click())
    // should fail because the button does not exist
    // Due to failure, the gif image will be saved in the outputs folder.
    onView(withId(R.id.button_first))
      .perform(click())
  }
}
```

### Compose Support

Test target

```kotlin
@Composable
fun SampleComposableFunction() {
  var count by remember { mutableStateOf(0) }
  Column(
    Modifier
      .size(300.dp)
  ) {
    Box(
      Modifier
        .testTag("MyComposeButton")
        .size(50.dp)
        .clickable {
          count++
        }
    )
    (0..count).forEach {
      Box(
        Modifier
          .size(30.dp)
      )
    }
  }
}
```

Test (Just add RoborazziRule)

```kotlin
@RunWith(AndroidJUnit4::class)
class ComposeTest {
  @get:Rule
  val composeTestRule = createAndroidComposeRule<ComponentActivity>()

  @get:Rule
  val roborazziRule = RoborazziRule(composeTestRule, composeTestRule.onRoot())

  @Test
  fun composable() {
    composeTestRule.setContent {
      SampleComposableFunction()
    }
    (0 until 5).forEach { _ ->
      composeTestRule
        .onNodeWithTag("MyComposeButton")
        .performClick()
    }
  }
}
```

Result

<img width="443" alt="image" src=https://user-images.githubusercontent.com/1386930/216755124-08f7e443-e98e-4b65-b41f-baeb999dc6c1.gif >



### Manually take a screenshot

```kotlin
@Test
fun captureRoboImageSample() {
  // launch
  launch(MainActivity::class.java)
  
  // screen level image
  onView(ViewMatchers.isRoot())
    .captureRoboImage("build/first_screen.png")

  // compose image
  composeTestRule.onNodeWithTag("MyComposeButton")
    .onParent()
    .captureRoboImage("build/compose.png")

  // small component image
  onView(withId(R.id.button_first))
    .captureRoboImage("build/button.png")

  // move to next page
  onView(withId(R.id.button_first))
    .perform(click())

  onView(ViewMatchers.isRoot())
    .captureRoboImage("build/second_screen.png")
}
```

<img width="443" alt="image" src="https://user-images.githubusercontent.com/1386930/215309972-8a2098be-91b7-45a5-82a2-647b89e1f6f8.png">
<img width="486" alt="image" src="https://user-images.githubusercontent.com/1386930/215248859-03a4f66e-3c42-42d8-863a-4cfbc3090b3f.png">


### Large project example


[From DroidKaigi 2022 app](https://github.com/DroidKaigi/conference-app-2022)

<img src=https://user-images.githubusercontent.com/1386930/215334118-ae1de2e0-0748-44f3-a735-4cf03b856767.png width=400 />

## Why

Whenever you test with Robolectric, you feel like you are writing tests blindfolded because you cannot see the layout.  
This tool makes the layout visible and provides the necessary information for debugging.

## Why test with JVM instead of testing on Android?

Because when testing on a device, it is easy for the test to fail due to the device environment, animations, etc., and ultimately, if the test fails, it will not be fixed.

## Why not Paparazzi?

Paparazzi is a great tool to see the actual display in the JVM.  
Paparazzi relies on LayoutLib, Android Studio's layout drawing tool, which is incompatible with Robolectric. This is because they both mock the Android framework.  
Without Robolectric, you can't write tests that actually click on components and run them with Hilt tests.

## Download

It is available on maven central.

```kotlin
// Core functions
testImplementation("io.github.takahirom.roborazzi:roborazzi:0.1.0")
// JUnit rules
testImplementation("io.github.takahirom.roborazzi:roborazzi-junit-rule:0.1.0")
```
