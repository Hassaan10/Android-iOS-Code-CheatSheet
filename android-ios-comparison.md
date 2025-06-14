
# iOS vs Android


## UIAppDelegate vs android.app.Application + ProcessLifecycleOwner

| **Lifecycle Phase**  | **iOS: UIAppDelegate Method** | **Android: Application + ProcessLifecycleOwner** | **Notes / Similarity**                                                                 |
|----------------------------|--------------------------------------------------|--------------------------------------------------------|-----------------------------------------------------------------------------------------|
| App Launch                 | `application(_:didFinishLaunchingWithOptions:)` | `Application.onCreate()`                               | Initialization of app components                                                       |
| App Enters Foreground      | `applicationWillEnterForeground(_:)`            | `ON_START` (ProcessLifecycleOwner)                     | Called when app comes to foreground                                                    |
| App Becomes Active         | `applicationDidBecomeActive(_:)`                | `ON_RESUME` (ProcessLifecycleOwner)                    | App becomes interactive                                                                |
| App Will Resign Active     | `applicationWillResignActive(_:)`               | `ON_PAUSE` (ProcessLifecycleOwner)                     | iOS informs before interruption (e.g., call)                                           |
| App Entered Background     | `applicationDidEnterBackground(_:)`             | `ON_STOP` (ProcessLifecycleOwner)                      | App fully moved to background                                                          |
| App Termination            | `applicationWillTerminate(_:)`                  | `onTerminate()` (called only in emulators/debug)       | Android’s `onTerminate()` is unreliable in production                                  |
| Low Memory Warning         | `applicationDidReceiveMemoryWarning(_:)`        | `onLowMemory()` / `onTrimMemory()`                     | Memory pressure notification                                                            |

> **Note:** ProcessLifecycleOwner is part of Android Jetpack and allows global lifecycle awareness across all activities.



## iOS vs Android App Lifecycle Code Comparison


### UIAppDelegate

```
import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    // Called when the app has finished launching
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        print("App launched")
        return true
    }

    // Called when the app becomes active (foreground)
    func applicationDidBecomeActive(_ application: UIApplication) {
        print("App became active")
    }

    // Called when the app is about to become inactive (e.g., incoming call)
    func applicationWillResignActive(_ application: UIApplication) {
        print("App will resign active")
    }

    // Called when the app enters the background
    func applicationDidEnterBackground(_ application: UIApplication) {
        print("App entered background")
    }

    // Called when the app enters the foreground (but not active yet)
    func applicationWillEnterForeground(_ application: UIApplication) {
        print("App will enter foreground")
    }

    // Called when the app is about to terminate
    func applicationWillTerminate(_ application: UIApplication) {
        print("App will terminate")
    }
}

```

### Android App + DefaultLifecycleObserver

```
import android.app.Application
import android.util.Log
import androidx.lifecycle.DefaultLifecycleObserver
import androidx.lifecycle.LifecycleOwner
import androidx.lifecycle.ProcessLifecycleOwner

class MyApplication : Application(), DefaultLifecycleObserver {

    override fun onCreate() {
        super.onCreate()
        Log.d("AppLifecycle", "App launched")

        // Register this Application class as a lifecycle observer
        ProcessLifecycleOwner.get().lifecycle.addObserver(this)
    }

    // Called when the app process is created and starts
    override fun onCreate(owner: LifecycleOwner) {
        Log.d("AppLifecycle", "ProcessLifecycleOwner created")
    }

    // Called when the app enters foreground (visible to user)
    override fun onStart(owner: LifecycleOwner) {
        Log.d("AppLifecycle", "App entered foreground")
    }

    // Called when the app becomes interactive
    override fun onResume(owner: LifecycleOwner) {
        Log.d("AppLifecycle", "App became interactive (resumed)")
    }

    // Called when the app is no longer interactive
    override fun onPause(owner: LifecycleOwner) {
        Log.d("AppLifecycle", "App paused")
    }

    // Called when the app goes to background (no UI visible)
    override fun onStop(owner: LifecycleOwner) {
        Log.d("AppLifecycle", "App entered background")
    }

    // Called when the lifecycle is being destroyed (rare for apps)
    override fun onDestroy(owner: LifecycleOwner) {
        Log.d("AppLifecycle", "App process being destroyed")
    }
}

```

> ℹ️ Note: `onDestroy()` in Android is rarely called in production unless the process is being shut down in a debug environment.

<br>
<br>

## iOS ViewController vs Android Activity Lifecycle Comparison

This document compares the lifecycles of iOS `UIViewController` and Android `Activity`, showing where they roughly map to each other.

| **Lifecycle Stage**   | **iOS – UIViewController**             | **Android – Activity**              | **Description** |
|-----------------------|-----------------------------------------|-------------------------------------|------------------|
| App Launch            | `AppDelegate.application(_:didFinishLaunchingWithOptions:)` | `Application.onCreate()` | When the app process starts |
| **View Creation**     | `viewDidLoad()`                        | `onCreate()`                        | Called once when the view/activity is created |
| **View Appearing**    | `viewWillAppear(_:)`                   | `onStart()`                         | Called when view/activity is about to become visible |
| **View Visible**      | `viewDidAppear(_:)`                    | `onResume()`                        | Called when view/activity is visible and interactive |
| **View Losing Focus** | `viewWillDisappear(_:)`                | `onPause()`                         | Called when view/activity is losing focus (partially hidden, another view on top) |
| **View Hidden**       | `viewDidDisappear(_:)`                 | `onStop()`                          | Called when view/activity is no longer visible |
| **View Destruction**  | `deinit` (controller deallocated)      | `onDestroy()`                       | Called when view/activity is destroyed |
| **Memory Warning**    | `didReceiveMemoryWarning()` (deprecated) | No direct equivalent (handled differently in Android) | Called when system is low on memory |

<br>
<br>

## UI Components: iOS (UIKit) vs Android XML

| **UI Element / Concept**       | **iOS (UIKit)**               | **Android (View System)**         | **Notes / Similarity**                                                                  |
|-------------------------------|-------------------------------|-----------------------------------|------------------------------------------------------------------------------------------|
| Basic View                    | `UIView`                      | `View`                            | Base class for all UI elements                                                           |
| Button                        | `UIButton`                    | `Button`                          | Standard tappable UI button                                                              |
| Label / Text Display          | `UILabel`                     | `TextView`                        | For displaying static text                                                               |
| Editable Text Input           | `UITextField` (single-line)<br>`UITextView` (multi-line) | `EditText` (supports both)        | Android’s `EditText` is highly flexible                                                  |
| Image Display                 | `UIImageView`                 | `ImageView`                       | Used to display images                                                                  |
| Container / Layout            | `UIStackView`, `UIView`       | `LinearLayout`, `RelativeLayout`, `ConstraintLayout` | iOS uses nested views or `AutoLayout`; Android has XML layout containers                 |
| List View                     | `UITableView`, `UICollectionView` | `RecyclerView`, `ListView`       | Both provide scrollable lists with reusable cells                                        |
| Scroll View                   | `UIScrollView`                | `ScrollView`, `NestedScrollView` | Enable scrolling of contained views                                                      |
| Navigation Controller         | `UINavigationController`      | `NavController` (Jetpack)         | Used for managing navigation stacks                                                      |
| Tab Bar                       | `UITabBarController`          | `BottomNavigationView`, `TabLayout` | Both provide tabbed navigation UI                                                        |
| Alert Dialog / Modal          | `UIAlertController`           | `AlertDialog`, `DialogFragment`   | Used to show alerts and actions                                                          |
| Picker / Spinner              | `UIPickerView`, `UIDatePicker`| `Spinner`, `DatePicker`, `TimePicker` | Used to select values from a list                                                        |
| Toggle Switch                | `UISwitch`                    | `Switch`                          | For on/off toggle controls                                                               |
| Checkbox                      | *(No native)* Custom via `UIButton` | `CheckBox`                        | iOS doesn’t have a native checkbox, uses custom views                                    |
| Progress Indicator            | `UIActivityIndicatorView`, `UIProgressView` | `ProgressBar`                    | Shows loading or progress                                                               |
| Gesture Recognition           | `UIGestureRecognizer`         | `GestureDetector`, `TouchListener`| Both platforms support touch and gesture handling                                        |
| View Controller / Screen Unit | `UIViewController`            | `Activity`, `Fragment`            | Fundamental units of screen and behavior                                                 |

<br>
<br>

## SwiftUI vs Jetpack Compose – Declarative UI Comparison

| **Concept / Component**        | **SwiftUI (iOS)**             | **Jetpack Compose (Android)**      | **Notes / Similarity**                                                                 |
|-------------------------------|-------------------------------|------------------------------------|-----------------------------------------------------------------------------------------|
| Basic Text                    | `Text("Hello")`               | `Text("Hello")`                    | Displays static text — syntax is nearly identical                                       |
| Button                        | `Button(action:) {}`          | `Button(onClick = {}) {}`          | Declarative buttons with actions                                                        |
| Image                         | `Image("photo")`              | `Image(painter = ...)`             | Renders an image asset                                                                 |
| Text Input                    | `TextField(...)`              | `TextField(...)`                   | Two-way data binding supported                                                          |
| Secure Text Input             | `SecureField(...)`            | `TextField(..., visualTransformation)` | For passwords — Compose requires a transformation                                     |
| Toggle Switch                 | `Toggle(isOn: $state)`        | `Switch(checked = state)`          | Controlled component with reactive state                                                |
| Checkbox                      | Custom view or `Toggle` style | `Checkbox(checked = state)`        | Compose has a native Checkbox; SwiftUI does not                                         |
| Progress Indicator            | `ProgressView()`              | `CircularProgressIndicator()`      | Both have built-in progress indicators                                                  |
| List View                     | `List(items) {}`              | `LazyColumn { items(...) }`        | Scrollable lists, lazily rendered                                                       |
| Grid View                     | `LazyVGrid`, `LazyHGrid`      | `LazyVerticalGrid`, `LazyHorizontalGrid` | Lazy grids available in both frameworks                                          |
| Scroll View                   | `ScrollView {}`               | `Column/Row + verticalScroll()`    | Scrolling containers                                                                    |
| Navigation                   | `NavigationStack` / `NavigationLink` | `NavHost` / `rememberNavController()` | Declarative navigation stacks                                                      |
| Tab View                      | `TabView {}`                  | `BottomNavigation()` + `Scaffold()`| Tabbed navigation, often wrapped in Scaffold in Compose                                 |
| Modal / Sheet                 | `.sheet(...)`                 | `ModalBottomSheetLayout` / `Dialog`| Used to show modal content                                                              |
| Alert                         | `.alert(...)`                 | `AlertDialog(...)`                 | Modal alerts with buttons                                                              |
| Animation                     | `.animation(...)`             | `animate*AsState`, `AnimatedVisibility`, etc. | Both support rich animations                                                 |
| State Management              | `@State`, `@Binding`, `@ObservedObject` | `remember`, `mutableStateOf`, `StateFlow` | Both use reactive state and composition                                             |
| Theming / Styling             | `.foregroundColor`, `.font()` | `Modifier.background`, `Modifier.padding` | Modifiers are similar but platform-specific                                    |
| Lifecycle Awareness           | `@Environment(\.scenePhase)`  | `LaunchedEffect`, `DisposableEffect`, `LifecycleEventObserver` | Different lifecycle hooks                                                  |

> **Note:** Both frameworks are declarative, reactive, and support composition of UI components based on state.


# SwiftUI vs Jetpack Compose – Similar Methods & Functional Concepts

| **Purpose**                      | **SwiftUI Method / Concept**             | **Jetpack Compose Method / Concept**       | **Notes / Similarity**                                                        |
|----------------------------------|------------------------------------------|--------------------------------------------|-------------------------------------------------------------------------------|
| Define a screen / UI             | `var body: some View`                    | `@Composable fun MyScreen()`               | Entry point for defining a UI                                                |
| Display text                     | `Text("Hello")`                          | `Text("Hello")`                            | Basic static text                                                             |
| Add a button                     | `Button(action:) { Text("Click") }`      | `Button(onClick = {}) { Text("Click") }`   | Both use closures to define action and content                               |
| Image rendering                  | `Image("photo")`                         | `Image(painter = ...)`                     | Loads image from resources or asset                                          |
| Scrollable content               | `ScrollView {}`                          | `Column(modifier = Modifier.verticalScroll())` | Both enable vertical scrolling                                          |
| List rendering                   | `List(items) { item in ... }`            | `LazyColumn { items(items) { ... } }`      | Lazy loading for lists                                                       |
| Navigation                      | `NavigationStack`, `NavigationLink`      | `NavController`, `NavHost`, `navigate()`   | Stack-based navigation in both platforms                                     |
| Alert dialogs                    | `.alert(isPresented:) { Alert(...) }`    | `AlertDialog(...)`                         | Declarative alert presentation                                               |
| Show modal / sheet               | `.sheet(isPresented:) {}`                | `ModalBottomSheetLayout {}` / `Dialog {}`  | Modals and sheets supported in both                                          |
| Show progress indicator          | `ProgressView()`                         | `CircularProgressIndicator()`              | Indeterminate progress UI                                                    |
| Vertical spacing                 | `.padding(.vertical, 8)`                 | `Modifier.padding(vertical = 8.dp)`        | Styling via modifiers                                                        |
| Set background color             | `.background(Color.red)`                 | `Modifier.background(Color.Red)`           | Both use modifiers                                                           |
| Center content                   | `HStack { Spacer(); Text(); Spacer() }`  | `Box(contentAlignment = Alignment.Center)` | Different patterns for centering                                             |
| Manage local state              | `@State var count = 0`                   | `var count by remember { mutableStateOf(0) }` | Reactive state bindings                                                  |
| Observe external state           | `@ObservedObject` / `@EnvironmentObject` | `StateFlow`, `LiveData`, `collectAsState()`| Integration with external state sources                                      |
| Handle side effects              | `.onAppear {}`                           | `LaunchedEffect(key1)`                     | Trigger code on UI composition / lifecycle                                   |
| Handle lifecycle changes         | `@Environment(\.scenePhase)`             | `LifecycleEventObserver` / `DisposableEffect` | Lifecycle hooks for app state transitions                                |
| Animation                        | `.animation(.easeInOut)`                 | `animate*AsState()`, `AnimatedVisibility()` | Built-in support for transitions and effects                                |


<br>
<br>


## TextField

### SwiftUI

```
import SwiftUI

struct ContentView: View {
    @State private var name: String = ""

    var body: some View {
        VStack {
            Text("Hello, \(name)")
            TextField("Enter your name", text: $name)
                .textFieldStyle(RoundedBorderTextFieldStyle())
                .padding()
        }
    }
}
```

<br>

### Jetpack Compose

```
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun MyTextFieldScreen() {
    var name by remember { mutableStateOf("") }

    Column(modifier = Modifier.padding(16.dp)) {
        Text("Hello, $name")
        TextField(
            value = name,
            onValueChange = { name = it },
            label = { Text("Enter your name") },
            modifier = Modifier.fillMaxWidth()
        )
    }
}
```


<br>
<br>

## Secure Text

### Swift UI
```
struct SecureInputView: View {
    @State private var password = ""
    @State private var isSecure = true

    var body: some View {
        VStack {
            if isSecure {
                SecureField("Enter password", text: $password)
            } else {
                TextField("Enter password", text: $password)
            }

            Button(action: {
                isSecure.toggle()
            }) {
                Image(systemName: isSecure ? "eye.slash" : "eye")
                    .foregroundColor(.gray)
            }

            .padding()
        }
        .textFieldStyle(RoundedBorderTextFieldStyle())
        .padding()
    }
}
```

### Compose

```
import androidx.compose.foundation.layout.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Visibility
import androidx.compose.material.icons.filled.VisibilityOff
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.input.*
import androidx.compose.ui.unit.dp

@Composable
fun SecureInputField() {
    var password by remember { mutableStateOf("") }
    var isVisible by remember { mutableStateOf(false) }

    TextField(
        value = password,
        onValueChange = { password = it },
        label = { Text("Enter password") },
        visualTransformation = if (isVisible) VisualTransformation.None else PasswordVisualTransformation(),
        trailingIcon = {
            val icon = if (isVisible) Icons.Default.Visibility else Icons.Default.VisibilityOff
            IconButton(onClick = { isVisible = !isVisible }) {
                Icon(imageVector = icon, contentDescription = null)
            }
        },
        singleLine = true,
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    )
}
```



## Button

### SwifUI

```import SwiftUI

struct ContentView: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Text("Count: \(count)")
            Button("Click me") {
                count += 1
            }
            .padding()
            .background(Color.blue)
            .foregroundColor(.white)
            .cornerRadius(8)
        }
    }
}
```

### Jetpack Compose

```
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun MyButtonScreen() {
    var count by remember { mutableStateOf(0) }

    Column(modifier = Modifier.padding(16.dp)) {
        Text("Count: $count")
        Button(
            onClick = { count++ },
            modifier = Modifier.padding(top = 8.dp)
        ) {
            Text("Click me")
        }
    }
}
```

## Image

### SwiftUI

```
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            Image("example")
                .resizable()
                .scaledToFit()
                .frame(width: 100, height: 100)
                .clipShape(RoundedRectangle(cornerRadius: 10))
                .padding()
        }
    }
}
```

### Jetpack Compose

```
import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.unit.dp
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale

@Composable
fun MyImage() {
    Image(
        painter = painterResource(id = R.drawable.example),
        contentDescription = "Example",
        contentScale = ContentScale.Fit,
        modifier = Modifier
            .size(100.dp)
            .clip(RoundedCornerShape(10.dp))
            .padding(8.dp)
    )
}
```

## Network Image

### Swift UI

```
import SwiftUI

struct RemoteImageView: View {
    let imageURL = URL(string: "https://example.com/image.png")

    var body: some View {
        AsyncImage(url: imageURL) { phase in
            switch phase {
            case .empty:
                ProgressView() // Loading indicator
            case .success(let image):
                image
                    .resizable()
                    .scaledToFit()
                    .frame(width: 200, height: 200)
                    .cornerRadius(10)
            case .failure:
                Image(systemName: "photo")
                    .resizable()
                    .scaledToFit()
                    .foregroundColor(.gray)
                    .frame(width: 200, height: 200)
            @unknown default:
                EmptyView()
            }
        }
        .padding()
    }
}
```

### Compose

```
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.unit.dp
import coil.compose.AsyncImage
import coil.compose.AsyncImagePainter.State
import coil.compose.rememberAsyncImagePainter

@Composable
fun RemoteImageView() {
    AsyncImage(
        model = "https://example.com/image.png",
        contentDescription = "Example Image",
        modifier = Modifier
            .size(200.dp)
            .clip(RoundedCornerShape(10.dp))
            .padding(8.dp)
    )
}
```







# Dark Theme Handling – SwiftUI vs Jetpack Compose

| **Aspect**                      | **SwiftUI (iOS)**                                                                 | **Jetpack Compose (Android)**                                                                 |
|---------------------------------|------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| **Default Behavior**            | Follows system appearance by default                                              | Follows system theme by default                                                              |
| **Detect Dark Mode**            | `@Environment(\\.colorScheme) var colorScheme`                                    | `isSystemInDarkTheme()`                                                                      |
| **Conditionally Change UI**     | `if colorScheme == .dark { ... }`                                                 | `if (isSystemInDarkTheme()) { ... }`                                                         |
| **Set Dark Colors**             | Use `.foregroundColor(.white)` or `.preferredColorScheme(.dark)`                  | Define colors in `darkColors()` theme and use `MaterialTheme(colors = ...)`                 |
| **Force Dark Mode** (App-wide)  | `.preferredColorScheme(.dark)` on a View                                          | `android:forceDarkAllowed="true"` in XML or pass dark theme manually to `MaterialTheme`     |
| **Custom Theme Setup**          | Use `ColorScheme` in asset catalog or via `Color` extensions                       | Use `lightColors()` / `darkColors()` and pass to `MaterialTheme`                             |
| **Dynamic System Response**     | Auto-updates if using system defaults                                              | Auto-updates if using `MaterialTheme` and `isSystemInDarkTheme()`                            |



# Localization / Language Change – SwiftUI vs Jetpack Compose

| **Aspect**                      | **SwiftUI (iOS)**                                                                 | **Jetpack Compose (Android)**                                                              |
|----------------------------------|-----------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| **Localization Files**          | `.strings` files per language (e.g., `en.lproj/Localizable.strings`)             | `res/values/strings.xml`, `res/values-fr/strings.xml`, etc.                                |
| **Access Localized String**     | `Text("greeting")` → key in `Localizable.strings`                                | `stringResource(id = R.string.greeting)` or `context.getString(R.string.greeting)`         |
| **Set Language (System)**       | Controlled by iOS Settings app                                                   | Controlled by system settings                                                              |
| **Force Language (Programmatic)**| Not officially supported, but can override `Bundle.main`                         | Set a new `Locale` in `Configuration` and update `Context`                                 |
| **Preview in Simulator**        | Use `.environment(\.locale, Locale(identifier: "fr"))`                           | Use `Locale.setDefault(...)` or `AppCompatDelegate.setApplicationLocales(...)` (API 33+)   |
| **Change Locale at Runtime**    | Requires a workaround (e.g., reload UI with a new bundle)                        | Official support via `AppCompatDelegate.setApplicationLocales(...)`                        |
| **Dynamic Updates**             | ❌ Requires manual handling of view updates                                       | ✅ Supported on API 33+ or with libraries                                                   |


# Map Loading & Current Location – SwiftUI vs Jetpack Compose

| **Aspect**                  | **SwiftUI (iOS – MapKit)**                                                                 | **Jetpack Compose (Android – Google Maps)**                                                             |
|-----------------------------|----------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| **Import Required**         | `import MapKit`, `import CoreLocation`                                                      | `com.google.maps.android:maps-compose`, `androidx.compose.runtime`, `FusedLocationProviderClient`       |
| **Permissions**             | Add `NSLocationWhenInUseUsageDescription` in `Info.plist`                                  | Add location permission in `AndroidManifest.xml` and request at runtime                                 |
| **Map View Component**      | Use `Map(coordinateRegion:)` or wrap `MKMapView` in `UIViewRepresentable`                  | Use `GoogleMap()` composable from Maps Compose library                                                  |
| **Show User Location**      | Set `.showsUserLocation = true` on `MKMapView`                                              | Set `myLocationEnabled = true` and provide location updates via FusedLocationProvider                   |
| **Access User Location**    | Use `CLLocationManager` with delegate                                                       | Use `FusedLocationProviderClient.getLastLocation()` or `LocationCallback`                               |
| **Display Marker (Pin)**    | Add `Annotation` in `Map(coordinateRegion:annotationItems:)`                               | Use `Marker(state = MarkerState(position = LatLng(...)))`                                               |

