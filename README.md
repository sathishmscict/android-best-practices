# Best practices in Android development


## Summary

#### Use Gradle and its recommended project structure
#### Put passwords and sensitive data in gradle.properties
#### Don't write your own HTTP client, use Retrofit
#### Don't parse json, let the gson converter with Retrofit do it
#### Keep Activity class as small as possible.
#### Don't write Apicalls or Async methods in Activities
#### Use Picasso for image loading, dont write async task for that. 
#### Layout XMLs are code, organize them well
#### Use styles to avoid duplicate attributes in layout XMLs
#### Use multiple style files to avoid a single huge one
#### Keep your colors.xml short and DRY, just define the palette
#### Also keep dimens.xml DRY, define generic constants
#### Do not make a deep hierarchy of ViewGroups
#### Use Genymotion as your emulator
#### Always use ProGuard or DexGuard
#### Use SharedPreferences for simple persistence, otherwise ContentProviders


----------

### Android SDK

Place your [Android SDK](https://developer.android.com/sdk/installing/index.html?pkg=tools) somewhere in your home directory or some other application-independent location. Some IDEs include the SDK when installed, and may place it under the same directory as the IDE. This can be bad when you need to upgrade (or reinstall) the IDE, or when changing IDEs. Also avoid putting the SDK in another system-level directory that might need sudo permissions, if your IDE is running under your user and not under root.

### Build system

Your default option should be [Gradle](http://tools.android.com/tech-docs/new-build-system). With Gradle, it's simple to:

- Build different flavours or variants of your app
- Make simple script-like tasks
- Manage and download dependencies
- Customize keystores
- And more

Android's Gradle plugin is also being actively developed by Google as the new standard build system.

### Project structure


```
new-structure
├─ library-foobar
├─ app
│  ├─ libs
│  ├─ src
│  │  ├─ androidTest
│  │  │  └─ java
│  │  │     └─ com/noman/project
│  │  └─ main
│  │     ├─ java
│  │     │  └─ com/noman/project
│  │     ├─ res
│  │     └─ AndroidManifest.xml
│  ├─ build.gradle
│  └─ proguard-rules.pro
├─ build.gradle
└─ settings.gradle
```


Having a top-level `app` is useful to distinguish your app from other library projects (e.g., `library-foobar`) that will be referenced in your app. The `settings.gradle` then keeps references to these library projects, which `app/build.gradle` can reference to.

**Prefer Maven dependency resolution instead of importing jar files.** If you explicitly include jar files in your project, they will be of some specific frozen version, such as `2.1.1`. Downloading jars and handling updates is cumbersome, this is a problem that Maven solves properly, and is also encouraged in Android Gradle builds. For example:

```groovy
dependencies {
    compile 'com.squareup.okhttp:okhttp:2.2.0'
    compile 'com.squareup.okhttp:okhttp-urlconnection:2.2.0'
}
```    

**Avoid Maven dynamic dependency resolution**
Avoid the use of dynamically versioned, such as `2.1.+` as this may result in different and unstable builds or subtle, untracked differences in behavior between builds. The use of static versions such as `2.1.1` helps create a more stable, predictable and repeatable development environment.

### IDEs and text editors

**Use whatever editor, but it must play nicely with the project structure.** Editors are a personal choice, and it's your responsibility to get your editor functioning according to the project structure and build system.

The most recommended IDE at the moment is [Android Studio](https://developer.android.com/sdk/installing/studio.html), because it is developed by Google, is closest to Gradle, uses the new project structure by default, is finally in stable stage, and is tailored for Android development.


### Libraries

**[Retrofit](http://square.github.io/retrofit/)** Retrofit is one of the most popular HTTP Client Library for Android as a result of its simplicity and its great performance compare to the others.

Retrofit turns your HTTP API into a Java interface.

```java
public interface GitHubService {
  @GET("/users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```

The `Retrofit` class generates an implementation of the GitHubService interface.

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

Each `Call` from the created `GitHubService` can make a synchronous or asynchronous HTTP request to the remote webserver.

```java
Call<List<Repo>> repos = service.listRepos("octocat");
```


**[Picasso](http://square.github.io/picasso/)**  aims to be fast and simple to use—often requiring only one line of code.

```java
Picasso.with(context).load("http://example.com/logo.png").into(imageView);
```
And that’s it! You can use this for downloading a single image or inside of an adapter’s getView method to download many. Picasso automatically handles caching, recycling, and displaying the final bitmap into the target view.


### Activities and Fragments

**Naming.** Name activities and fragment properly, so they won't look like other classes but should display by name that its an `Activity` or `Fragment`.
E.g `Dashboard.Java` that looks like a class name, name it like `DashboardActivity.java` and for fragments name them like `UserFragment.java`. 
Few points need to keep in mind using Activities and Fragments.

- Avoid using [nested fragments](https://developer.android.com/about/versions/android-4.2.html#NestedFragments) extensively, because [matryoshka bugs](http://delyan.me/android-s-matryoshka-problem/) can occur. Use nested fragments only when it makes sense (for instance, fragments in a horizontally-sliding ViewPager inside a screen-like fragment) or if it's a well-informed decision.
- Avoid putting too much code in activities. Whenever possible, keep them as lightweight containers, existing in your application primarily for the lifecycle and other important Android-interfacing APIs. Prefer single-fragment activities instead of plain activities - put UI code into the activity's fragment. This makes it reusable in case you need to change it to reside in a tabbed layout, or in a multi-fragment tablet screen. Avoid having an activity without a corresponding fragment, unless you are making an informed decision.
- Don't abuse Android-level APIs such as heavily relying on Intent for your app's internal workings. You could affect the Android OS or other applications, creating bugs or lag. For instance, it is known that if your app uses Intents for internal communication between your packages, you might incur multi-second lag on user experience if the app was opened just after OS boot.


### Java packages architecture

Java architectures for Android applications can be roughly approximated in [Model-View-Controller](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller). In Android, [Fragment and Activity are actually controller classes](http://www.informit.com/articles/article.aspx?p=2126865). On the other hand, they are explicity part of the user interface, hence are also views.

For this reason, it is hard to classify fragments (or activities) as strictly controllers or views. It's better to let them stay in their own `fragments` package. Activities can stay on the top-level package as long as you follow the advice of the previous section. If you are planning to have more than 2 or 3 activities, then make also an `activities` package.

Otherwise, the architecture can look like a typical MVC, with a `models` package containing POJOs to be populated through the JSON parser with API responses, and a `views` package containing your custom Views, notifications, action bar views, widgets, etc. Adapters are a gray matter, living between data and views. However, they typically need to export some View via `getView()`, so you can include the `adapters` subpackage inside `views`.

Some controller classes are application-wide and close to the Android system. These can live in a `managers` package. Miscellaneous data processing classes, such as "DateUtils", stay in the `utils` package. Classes that are responsible for interacting with the backend stay in the `network` package.

All in all, ordered from the closest-to-backend to the closest-to-the-user:

```
com.noman.project
├─ network
├─ models
├─ managers
├─ utils
├─ fragments
└─ views
   ├─ adapters
   ├─ actionbar
   ├─ widgets
   └─ notifications
```

### Resources

**Naming.** Follow the convention of prefixing the type, as in `type_foo_bar.xml`. Examples: `fragment_contact_details.xml`, `view_primary_button.xml`, `activity_main.xml`.

**Organizing layout XMLs.** If you're unsure how to format a layout XML, the following convention may help.

- One attribute per line, indented by 4 spaces
- `android:id` as the first attribute always
- `android:layout_****` attributes at the top
- `style` attribute at the bottom
- Tag closer `/>` on its own line, to facilitate ordering and adding attributes.
- Rather than hard coding `android:text`, consider using [Designtime attributes](http://tools.android.com/tips/layout-designtime-attributes) available for Android Studio.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >

    <TextView
        android:id="@+id/name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:text="@string/name"
        style="@style/FancyText"
        />

    <include layout="@layout/reusable_part" />

</LinearLayout>
```

As a rule of thumb, attributes `android:layout_****` should be defined in the layout XML, while other attributes `android:****` should stay in a style XML. This rule has exceptions, but in general works fine. The idea is to keep only layout (positioning, margin, sizing) and content attributes in the layout files, while keeping all appearance details (colors, padding, font) in styles files.

The exceptions are:

- `android:id` should obviously be in the layout files
- `android:orientation` for a `LinearLayout` normally makes more sense in layout files
- `android:text` should be in layout files because it defines content
- Sometimes it will make sense to make a generic style defining `android:layout_width` and `android:layout_height` but by default these should appear in the layout files

**Use styles.** Almost every project needs to properly use styles, because it is very common to have a repeated appearance for a view. At least you should have a common style for most text content in the application, for example:

```xml
<style name="ContentText">
    <item name="android:textSize">@dimen/font_normal</item>
    <item name="android:textColor">@color/basic_black</item>
</style>
```

Applied to TextViews:

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/price"
    style="@style/ContentText"
    />
```

You probably will need to do the same for buttons, but don't stop there yet. Go beyond and move a group of related and repeated `android:****` attributes to a common style.

**Split a large style file into other files.** You don't need to have a single `styles.xml` file. Android SDK supports other files out of the box, there is nothing magical about the name `styles`, what matters are the XML tags `<style>` inside the file. Hence you can have files `styles.xml`, `styles_home.xml`, `styles_item_details.xml`, `styles_forms.xml`. Unlike resource directory names which carry some meaning for the build system, filenames in `res/values` can be arbitrary.

**`colors.xml` is a color palette.** There should be nothing else in your `colors.xml` than just a mapping from a color name to an RGBA value. Do not use it to define RGBA values for different types of buttons.

*Don't do this:*

```xml
<resources>
    <color name="button_foreground">#FFFFFF</color>
    <color name="button_background">#2A91BD</color>
    <color name="comment_background_inactive">#5F5F5F</color>
    <color name="comment_background_active">#939393</color>
    <color name="comment_foreground">#FFFFFF</color>
    <color name="comment_foreground_important">#FF9D2F</color>
    ...
    <color name="comment_shadow">#323232</color>
```

You can easily start repeating RGBA values in this format, and that makes it complicated to change a basic color if needed. Also, those definitions are related to some context, like "button" or "comment", and should live in a button style, not in `colors.xml`.

Instead, do this:

```xml
<resources>

    <!-- grayscale -->
    <color name="white"     >#FFFFFF</color>
    <color name="gray_light">#DBDBDB</color>
    <color name="gray"      >#939393</color>
    <color name="gray_dark" >#5F5F5F</color>
    <color name="black"     >#323232</color>

    <!-- basic colors -->
    <color name="green">#27D34D</color>
    <color name="blue">#2A91BD</color>
    <color name="orange">#FF9D2F</color>
    <color name="red">#FF432F</color>

</resources>
```

Ask for this palette from the designer of the application. The names do not need to be color names as "green", "blue", etc. Names such as "brand_primary", "brand_secondary", "brand_negative" are totally acceptable as well. Formatting colors as such will make it easy to change or refactor colors, and also will make it explicit how many different colors are being used. Normally for a aesthetic UI, it is important to reduce the variety of colors being used.

**Treat dimens.xml like colors.xml.** You should also define a "palette" of typical spacing and font sizes, for basically the same purposes as for colors. A good example of a dimens file:

```xml
<resources>

    <!-- font sizes -->
    <dimen name="font_larger">22sp</dimen>
    <dimen name="font_large">18sp</dimen>
    <dimen name="font_normal">15sp</dimen>
    <dimen name="font_small">12sp</dimen>

    <!-- typical spacing between two views -->
    <dimen name="spacing_huge">40dp</dimen>
    <dimen name="spacing_large">24dp</dimen>
    <dimen name="spacing_normal">14dp</dimen>
    <dimen name="spacing_small">10dp</dimen>
    <dimen name="spacing_tiny">4dp</dimen>

    <!-- typical sizes of views -->
    <dimen name="button_height_tall">60dp</dimen>
    <dimen name="button_height_normal">40dp</dimen>
    <dimen name="button_height_short">32dp</dimen>

</resources>
```

You should use the `spacing_****` dimensions for layouting, in margins and paddings, instead of hard-coded values, much like strings are normally treated. This will give a consistent look-and-feel, while making it easier to organize and change styles and layouts.

**strings.xml**

Name your strings with keys that resemble namespaces, and don't be afraid of repeating a value for two or more keys. Languages are complex, so namespaces are necessary to bring context and break ambiguity.

**Bad**
```xml
<string name="network_error">Network error</string>
<string name="call_failed">Call failed</string>
<string name="map_failed">Map loading failed</string>
```

**Good**
```xml
<string name="error.message.network">Network error</string>
<string name="error.message.call">Call failed</string>
<string name="error.message.map">Map loading failed</string>
```

Don't write string values in all uppercase. Stick to normal text conventions (e.g., capitalize first character). If you need to display the string in all caps, then do that using for instance the attribute [`textAllCaps`](http://developer.android.com/reference/android/widget/TextView.html#attr_android:textAllCaps) on a TextView.

**Bad**
```xml
<string name="error.message.call">CALL FAILED</string>
```

**Good**
```xml
<string name="error.message.call">Call failed</string>
```

**Avoid a deep hierarchy of views.** Sometimes you might be tempted to just add yet another LinearLayout, to be able to accomplish an arrangement of views. This kind of situation may occur:

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >

    <RelativeLayout
        ...
        >

        <LinearLayout
            ...
            >

            <LinearLayout
                ...
                >

                <LinearLayout
                    ...
                    >
                </LinearLayout>

            </LinearLayout>

        </LinearLayout>

    </RelativeLayout>

</LinearLayout>
```

Even if you don't witness this explicitly in a layout file, it might end up happening if you are inflating (in Java) views into other views.

A couple of problems may occur. You might experience performance problems, because there are is a complex UI tree that the processor needs to handle. Another more serious issue is a possibility of [StackOverflowError](http://stackoverflow.com/questions/2762924/java-lang-stackoverflow-error-suspected-too-many-views).

Therefore, try to keep your views hierarchy as flat as possible: learn how to use [RelativeLayout](https://developer.android.com/guide/topics/ui/layout/relative.html), how to [optimize your layouts](http://developer.android.com/training/improving-layouts/optimizing-layout.html) and to use the [`<merge>` tag](http://stackoverflow.com/questions/8834898/what-is-the-purpose-of-androids-merge-tag-in-xml-layouts).

**Beware of problems related to WebViews.** When you must display a web page, for instance for a news article, avoid doing client-side processing to clean the HTML, rather ask for a "*pure*" HTML from the backend programmers. [WebViews can also leak memory](http://stackoverflow.com/questions/3130654/memory-leak-in-webview) when they keep a reference to their Activity, instead of being bound to the ApplicationContext. Avoid using a WebView for simple texts or buttons, prefer TextViews or Buttons.


### Emulators

If you are developing Android apps as a profession, buy a license for the [Genymotion emulator](http://www.genymotion.com/). Genymotion emulators run at a faster frames/sec rate than typical AVD emulators. They have tools for demoing your app, emulating network connection quality, GPS positions, etc. They are also ideal for connected tests. You have access to many (not all) different devices, so the cost of a Genymotion license is actually much cheaper than buying multiple real devices.

Caveats are: Genymotion emulators don't ship all Google services such as Google Play Store and Maps. You might also need to test Samsung-specific APIs, so it's necessary to have a real Samsung device.

### Proguard configuration

[ProGuard](http://proguard.sourceforge.net/) is normally used on Android projects to shrink and obfuscate the packaged code.

Whether you are using ProGuard or not depends on your project configuration. Usually you would configure gradle to use ProGuard when building a release apk.

```groovy
buildTypes {
    debug {
        minifyEnabled false
    }
    release {
        signingConfig signingConfigs.release
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

In order to determine which code has to be preserved and which code can be discarded or obfuscated, you have to specify one or more entry points to your code. These entry points are typically classes with main methods, applets, midlets, activities, etc.
Android framework uses a default configuration which can be found from `SDK_HOME/tools/proguard/proguard-android.txt`. Using the above configuration, custom project-specific ProGuard rules, as defined in `my-project/app/proguard-rules.pro`, will be appended to the default configuration.

A common problem related to ProGuard is to see the application crashing on startup with `ClassNotFoundException` or `NoSuchFieldException` or similar, even though the build command (i.e. `assembleRelease`) succeeded without warnings.
This means one out of two things:

1. ProGuard has removed the class, enum, method, field or annotation, considering it's not required.
2. ProGuard has obfuscated (renamed) the class, enum or field name, but it's being used indirectly by its original name, i.e. through Java reflection.

Check `app/build/outputs/proguard/release/usage.txt` to see if the object in question has been removed.
Check `app/build/outputs/proguard/release/mapping.txt` to see if the object in question has been obfuscated.

In order to prevent ProGuard from *stripping away* needed classes or class members, add a `keep` options to your ProGuard config:
```
-keep class com.noman.project.MyClass { *; }
```

To prevent ProGuard from *obfuscating* classes or class members, add a `keepnames`:
```
-keepnames class com.noman.project.MyClass { *; }
```

Check [this template's ProGuard config](https://guides.codepath.com/android/Configuring-ProGuard) for some examples.
Read more at [Proguard](http://proguard.sourceforge.net/#manual/examples.html) for examples.

**Early on in your project, make a release build** to check whether ProGuard rules are correctly keeping whatever is important. Also whenever you include new libraries, make a release build and test the apk on a device. Don't wait until your app is finally version "1.0" to make a release build, you might get several unpleasant surprises and a short time to fix them.

**Tip.** Save the `mapping.txt` file for every release that you publish to your users. By retaining a copy of the `mapping.txt` file for each release build, you ensure that you can debug a problem if a user encounters a bug and submits an obfuscated stack trace.

**DexGuard**. If you need hard-core tools for optimizing, and specially obfuscating release code, consider [DexGuard](http://www.saikoa.com/dexguard), a commercial software made by the same team that built ProGuard. It can also easily split Dex files to solve the 65k methods limitation.

### Data storage


#### SharedPreferences

If you only need to persist simple flags and your application runs in a single process SharedPreferences is probably enough for you. It is a good default option.

There are two reasons why you might not want to use SharedPreferences:

* *Performance*: Your data is complex or there is a lot of it
* *Multiple processes accessing the data*: You have widgets or remote services that run in their own processes and require synchronized data


#### ContentProviders

In the case SharedPreferences is not enough for you, you should use the platform standard ContentProviders, which are fast and process safe.

The single problem with ContentProviders is the amount of boilerplate code that is needed to set them up, as well as low quality tutorials. It is possible, however, to generate the ContentProvider by using a library such as [Schematic](https://github.com/SimonVT/schematic), which significantly reduces the effort.

You still need to write some parsing code yourself to read the data objects from the Sqlite columns and vice versa. It is possible to serialize the data objects, for instance with Gson, and only persist the resulting string. In this way you lose in performance but on the other hand you do not need to declare a column for all the fields of the data class.


#### Using an ORM

**[SugerORM](https://android-arsenal.com/)** Keep the database clean. Not need to write queries by yourself and handling `SqliteHelper` classes. Let the ORM do the job. 

#### License 

[Futurice Oy](http://futurice.com/) Creative Commons Attribution 4.0 International (CC BY 4.0)
Modified by Noman Rafique


