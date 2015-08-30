# Custom watch faces for Android Wear

In this tutorial you'll learn how to take advantage of the Android Wear APIs to create your own Android Wear watch face.

The Android Wear API enables you to create custom watch faces for Wear devices. When a user installs a handheld app that contains a wearable app with watch faces, the watch faces become available in the Android Wear companion app on the handheld device and in the watch face picker on the wearable device. This tutorial will be focusing on implementing custom watch faces, packaged inside a wear app.

The two major things you're going to learn in this tutorial are: 

1. How to create a custom watch face that can be selected from the watch face picker.
2. How to design a custom watch face.

Here's a sneak peek of what you'll be building throughout the rest of this tutorial:

![](Images/time_date.png)

Does that whet your appetite? Of course it does - time to get right to it!

## Getting started

In this tutorial, you will be creating your custom watch face with Android Studio. Android Studio is a tool for making Android apps that is released and maintained by Google. If you don't already have it installed, [download it here](https://developer.android.com/sdk/index.html) and follow the instructions to install it. If you need more detailed instructions to install it, you should check out our [Android for Beginners Tutorial](http://www.raywenderlich.com/78574/android-tutorial-for-beginners-part-1).

### Creating the project

Open up Android Studio and you should be greeted with a window just like the one below. Select **Start a new Android Studio project** from the Quick Start menu:

![](Images/quick_start.png)

Name the app **WatchFace**, set the company domain and project location that you want. The domain name should already be filled in as **com.\<your-computer-username\>** but if you want to change it, just keep in mind that the general convention is **com.\<organization-name\>**. After this, click **Next**.

![](Images/create_project.png)

On the Target Android Devices dialog, make sure you check both **Phone and Tablet** and **Wear** and set the Minimum SDK to **API 21: Android 5.0 (Lollipop)** for both. After that, click **Next**:

![](Images/target_devices.png)

On the next dialog, select **Add No Activity** since you won't require an activity, and click **Next**:

![](Images/add_activity_mobile.png)

You will be presented with a Wear variant of the last dialog. Again, select **Add No Activity** and click **Next**: 

![](Images/add_activity_wear.png)

Now you've created the project, it's time to create an emulator suitable for testing Android Wear watch faces.


### Creating an Android Wear virtual device

Select **Tools\Android\AVD Manager** and in the window that appears click **Create Virtual Device**:

![](Images/AVD.png)

In the Select Hardware dialog, choose **Wear** from the category list, then **Android Wear Round** from the hardware list, and then click **Next**:

![](Images/hardware.png)

When the System Image dialog appears, select the **x86** Lollipop image and click **Next**:

![](Images/system_image.png)

On the next and final dialog, check **Use Host GPU**. What does it do? The short story is that doing this lets the emulator use your computer's graphics card which makes for a much smoother experience. The slightly longer story is that this makes it so that when a program inside the emulator uses OpenGL for graphics operations, the work goes out to your real GPU, and the result goes back into the emulator, instead of emulating a GPU (which is very slow). The result is a significant speed-up, because most view and canvas drawing uses OpenGL in Android ≥ 4, even in non-graphics apps. After that, click **Finish**:

![](Images/wear_device.png)

Next, select the new Android Wear virtual device you just created in the Android Virtual Device manager and click the **green play button** in the Actions section to launch it:

![](Images/AVD_after.png)

After a short while, (unless you're running a version of Android Studio < 1.0, in which case you probably have time to go make a brew), you'll see the following:

![](Images/emulator.png)

If this is the first time you've launched the emulator, follow the onscreen instructions to complete the on-boarding process.

And with that, let's get coding!

## Building a Watch Face Service

Android Wear watch faces are implemented as services and are packaged inside a wearable app.

A Service is basically an application component that runs in the background and can exist outside of the application's lifetime and interact with other applications. For the more advanced readers, it is important to note that while they run in the background away from the user, they still run on the main thread.

When a user installs an Android app that contains a wearable app with a watch face, the watch face becomes available in the Android Wear companion app on the phone and in the watch face picker on the wearable. When the user selects the watch face, the wearable device shows the watch face and invokes callback methods provided in the Service as required (such as when the time changes, or when other important events occur like switching to ambient mode or receiving a new notification). Your service implementation can then redraw the watch face on the screen using the updated time and any other relevant data.

To implement a watch face, you need to extend both **CanvasWatchFaceService** and **CanvasWatchFaceService.Engine**, and then override the necessary callback methods in **CanvasWatchFaceService.Engine**. Both of these classes are included in the Wearable Support Library.

Right-click on **wear\java\your-package-name.watchface** in the project navigator and choose **New\Java Class**. Name the class **CustomWatchFaceService** and click **OK**.

At the top of the file, right beneath the package declaration, replace the import statements with the following:

    import android.graphics.Canvas;
    import android.graphics.Rect;
    import android.support.wearable.watchface.CanvasWatchFaceService;
    import android.support.wearable.watchface.WatchFaceStyle;
    import android.view.SurfaceHolder;
    
Those imports contain classes we will use later in the tutorial, so its better to get them out of the way now.

Now, replace the existing **CustomWatchFaceService** class definition with the following:

    public class CustomWatchFaceService extends CanvasWatchFaceService {

      @Override
      public Engine onCreateEngine() {
        return new Engine();
      }

      private class Engine extends CanvasWatchFaceService.Engine {

      }
    }

Here you've updated your class definition so it extends `CanvasWatchFaceService`, which is the entry point for Android Wear watch faces. You also implemented `onCreateEngine()`, which simply returns an instance of your implementation of `CanvasWatchFaceService.Engine`.

Before you can build and run your new watch face, there's a bit of housekeeping you need to take care of before it'll even show up on the watch.

## Updating the Manifest

First, you must declare the necessary resource required by Android Wear for watch faces. Right-click on **wear\res** in the project navigator and choose **New\Android resource directory**. In the subsequent dialog, change **Resource type** to xml and click **OK**:

![](Images/xml_folder.png)

This creates a new resource folder in which xml files will be stored.

Right-click on the new **wear\res\xml** folder in the project navigator and choose **New\XML resource file**. Name the file **watch_face** and click **OK**:

![](Images/watch_face_xml_file.png)

Replace the contents of the new xml file with the following:

    <?xml version="1.0" encoding="UTF-8"?>
    <wallpaper />

Next, you need to add the preview images. These are the images that will be displayed when a user is looking through different watch faces to select from. Usually, you would want these to be actual screenshots of your watch face, but for the sake of brevity, I've prepared some ahead of time that you can use. Download [this zip](Preview-Images.zip) file and unarchive it. Copy the two files. Head back to Android Studio, right-click on **wear\res\drawable** and choose **Paste**. Click **OK** in the confirmation dialog.

Finally, its time to set up your manifest. Open **wear\manifests\AndroidManifest.xml** in the project editor (by double-clicking it) and add the following just above the application tag:

    <uses-permission android:name="com.google.android.PROVIDE_BACKGROUND" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />

Here we're declaring the two permissions required to run the watch face.

Next, add the following _inside_ the application tag:

    <!-- 1 -->
    <service
      android:name=".CustomWatchFaceService"
      android:label="@string/app_name"
      android:permission="android.permission.BIND_WALLPAPER">
      
      <!-- 2 -->
      <meta-data
        android:name="android.service.wallpaper"
        android:resource="@xml/watch_face" />
        
      <!-- 3 -->
      <meta-data
        android:name="com.google.android.wearable.watchface.preview"
        android:resource="@drawable/preview" />
        
      <meta-data
        android:name="com.google.android.wearable.watchface.preview_circular"
        android:resource="@drawable/preview_circular" />
        
      <!-- 4 -->
      <intent-filter>
        <action android:name="android.service.wallpaper.WallpaperService" />
        <category android:name="com.google.android.wearable.watchface.category.WATCH_FACE" />
      </intent-filter>
    </service>

Here's the play-by-play of what's happening above:

1. We declare the service, specifying the class and necessary permissions.
2. We declare the xml resource for the watch face, which you added earlier.
3. We declare the preview images to be used by the Android Wear companion app and the watch face picker on the watch itself (which you also added earlier).
4. We declare what the service can do by way of an intent filter. If you want to learn more about Intents and what this means, you can try our [Android Intents tutorial](http://www.raywenderlich.com/103044/android-intents-tutorial).

And thats it, you're good to go! Select **Run\Run...** and then choose **wear** in the pop-up dialog:

![](Images/run.png)

In the Edit configuration dialog select **Do not launch Activity** in the Activity section, and then click **Run**:

![](Images/run_config.png)

You only have to do this once. Future runs of the watch face will remember the configuration you've chosen here.

In the subsequent Choose Device dialog, select the Android Wear emulator that's already running - the one you set up and launched earlier - and click **OK**:

![](Images/choose_device.png)

Once the watch face has been installed, click and hold on the watch face in the emulator to invoke the watch face picker, click and drag left to find your watch face:

![](Images/picker.png)

Tap the preview to choose the watch face. You'll be presented with a black watch face, as shown below:

![](Images/initial_launch.png)

It might not look much as this point, but you've laid all the foundations necessary for building your watch face, and now you can concentrate on the actual coding.

Time to get something on that screen!

## Starting the Engine

As mentioned earlier, `Engine` subclasses `CanvasWatchFaceService.Engine`, which is what draws the watch face on the canvas, and also contains a series of useful callbacks.

To begin, you need to define your watch face style, and you do this in the `onCreate()` method of `CanvasWatchFaceService.Engine`. In defining the watch face style, you can customise how interface elements such as the battery indicator are drawn over the watch face, or how the cards behave in both normal and ambient modes. In order to define the watch face style we call `setWatchFaceStyle()`.

Add the following inside **Engine**:

    @Override
    public void onCreate(SurfaceHolder holder) {
      setWatchFaceStyle(new WatchFaceStyle.Builder(CustomWatchFaceService.this)
        .setCardPeekMode(WatchFaceStyle.PEEK_MODE_SHORT)
        .setAmbientPeekMode(WatchFaceStyle.AMBIENT_PEEK_MODE_HIDDEN)
        .setBackgroundVisibility(WatchFaceStyle.BACKGROUND_VISIBILITY_INTERRUPTIVE)
        .setShowSystemUiTime(false).build());
    }

Here you're defining how four different areas of the watch face should behave:

* You set the card peek mode to `PEEK_MODE_SHORT` which means cards will only show a single line of content.
* You set the ambient peek mode to `AMBIENT_PEEK_MODE_HIDDEN` meaning cards won't be displayed when the watch face enters ambient mode.
* You set the background visibility to `BACKGROUND_VISIBILITY_INTERRUPTIVE` which means the background of the card should be shown briefly, but only if that card represents an interruptive notification.
* And finally, you declare that you don't want the system time to be displayed because you're going to draw it manually.

[This page](http://developer.android.com/reference/android/support/wearable/watchface/WatchFaceStyle.Builder.html) provides a list of methods you can use to define a watch face style.

With the watch face style defined, you now need to implement the necessary callbacks. But first, let's take a look at a selection of the callbacks available to get a better idea of how watch faces behave:

* **onDraw(Canvas canvas, Rect bounds)**: This is called every time the watch face is invalidated. Think `drawRect(_:)` and `setNeedsDisplay()` on iOS. Here we'll draw the watch face using the provided Canvas and Rect, the latter defining the face's bounds.
* `onTimeTick()`: This is called periodically in ambient mode to update the time shown by the watch face. This method is called at least once per minute.
* **onVisibilityChanged(boolean visible)**: This is called to inform you of the watch face becoming visible or hidden. If you decide to override this method, you must call super.onVisibilityChanged(visible) as the very first statement in your override.
* **onAmbientModeChanged(boolean inAmbientMode)**: This is called when the device enters or exits ambient mode. Google recommends that the watch face should switch to a black and white display while in ambient mode, and if the watch face displays seconds, it should be hidden in ambient mode.
* **onPeekCardPositionUpdate(Rect rect)**: This is called when the first peeking card positions itself on the screen. This is where the watch face can change its appearance depending on where the card is on the screen. This callback doesn't provide information about all movements of the card, only about its location when it's peeking at the bottom and allowing the watch face to be exposed.

There are some other callbacks that may be of interest, so I highly recommend you [check out the docs](http://developer.android.com/reference/android/support/wearable/watchface/WatchFaceService.Engine.html).

With the theory out of the way, add the following to **Engine**, just below **onCreate()**:

    @Override
    public void onTimeTick() {
      super.onTimeTick();
      invalidate();
    }

    @Override
    public void onDraw(Canvas canvas, Rect bounds) {
      super(canvas, bounds);
    }

In `onTimeTick()`, we simply invalidate the display so the watch face is redrawn. We'll flesh out `onDraw()` shortly, but for now, it simply calls through to it's super implementation.

## Drawing the Time

To keep our code clean and readable, we'll create a separate class that'll be responsible for actually performing the drawing, and will be invoked from within `onDraw()` so that it happens at the correct time. We'll also pass the canvas and bounds to this new class.

Right-click on **wear\java\your-package-name.watchface** in the project navigator and choose **New\Java Class**. Name the class **WatchFace** and click **OK**.

Now let's add the supporting resources and values that will be used in your brand new `WatchFace` class. Right-click on **wear\res\values** in the project navigator and choose **New\Values resource file**. In the subsequent dialog, name the file **colors** and click **OK**.

Add the following to the new file, _inside_ the resources tag:

    <color name="watch_face_time_color">#FFFFFF</color>
    <color name="watch_face_fill_color">#00796B</color>

Here, you're simply defining the colours to be used as the fill colour for the watch face background and the text colour for the time.

Right-click again on **wear\res\values** in the project navigator and choose **New\Values resource file**. Name this file **dimensions** and click **OK**.

Add the following to the new file, _inside_ the resources tag:

    <dimen name="time_size">46dp</dimen>

Here you're defining the size of the text used to the display the time. We're abstracting these values out of the code so they're easily changeable and can be reused across different classes; it's a good practice to follow in Android development.

Its time to get back to coding! Open the **WatchFace** class you created a while ago.

Replace the imports beneath the package declaration with the following:

    import android.content.Context;
    import android.content.res.Resources;
    import android.graphics.Canvas;
    import android.graphics.Paint;
    import android.graphics.Rect;
    import android.text.format.Time;

Add the following inside the definition of **WatchFace**:

    // 1
    private static final String TIME_FORMAT = "%02d:%02d";
    private final Paint mTimePaint;
    private final int mBackgroundColor;

    // 2
    WatchFace(Paint timePaint, int backgroundColor) {
      this.mTimePaint = timePaint;
      this.mBackgroundColor = backgroundColor;
    }

    // 3
    public static WatchFace newInstance(Context context) {
      Resources resources = context.getResources();

      // 4
      Paint timePaint = new Paint();
      timePaint.setColor(resources.getColor(R.color.watch_face_time_color));
      timePaint.setTextSize(resources.getDimension(R.dimen.time_size));
      timePaint.setAntiAlias(true);

      return new WatchFace(timePaint, resources.getColor(R.color.color_watch_face_fill));
    }

    // 5
    public void draw(Canvas canvas, Rect bounds) {

    }

Here's a breakdown of the code above:

1. We define some private variables that are required for drawing the watch face, including an instance of Paint that will be used to draw the text. `TIME_FORMAT` is a special string that is used to specify the format the time will be displayed in. In this case, if either the hour or minute value has less than 2 digits, pad it with a zero.
2. We implement the default initialiser for the new class.
3. We implement a convenience initialiser that's used to do any necessary setup before calling the default initialiser.
4. We create an instance of Paint, and set the colour, text size, and whether or not the text should be drawn using anti-aliasing. In the simplest sense, anti-aliasing refers to graphics techniques used to improve the quality of raster images.
5. This is where the actual drawing will take place, and it'll be called from within our `onDraw()` in `Engine`. You will be fleshing it out shortly.

Now it's time to actually put something onscreen. Add the following inside **draw()**:

    // 1
    Time time = new Time();
    time.setToNow();
    
    // 2
    canvas.drawColor(mBackgroundColor);
    
    // 3
    float centerX = bounds.exactCenterX();
    float centerY = bounds.exactCenterY();
    Rect boundingBox = new Rect();
    
    // 4
    String timeText = String.format(TIME_FORMAT, time.hour, time.minute);
    float timeCenterX = centerX - (mTimePaint.measureText(timeText) / 2.0f);
    mTimePaint.getTextBounds(mTimeText, 0, mTimeText.length(), boundingBox);
    float timeCenterY = centerY + (boundingBox.height() / 2.0f);
    canvas.drawText(timeText, timeCenterX, timeCenterY, mTimePaint);

Here's the play-by-play of what's happening:

1. Grab a reference to the current time. You should be using the `Time` class imported from `android.text.format.Time` _NOT_ `java.util.date` or `java.sql.date`.
2. Fill the canvas with the background fill colour you defined just a moment ago.
3. Get the center point of the watch face, and declare an instance of Rect that we'll use to measure the dimensions of the text to draw.
4. Format the current time, work out the position at which is should be drawn, and then draw it!

There's just one more thing you need to do before you'll finally see something onscreen, and that's update `Engine` to use your new drawing class.

Double-click on **CustomWatchFaceService** in the project navigator to open it, and then add the following just above **onCreate()** in Engine:

    private WatchFace mWatchFace;

Here, you're just declaring a variable that'll hold a reference to your new drawing class. Now initialise this variable by adding the following to the bottom of **onCreate()**:

    mWatchFace = WatchFace.newInstance(CustomWatchFaceService.this);

Finally, add the following to the bottom of **onDraw()**:

    mWatchFace.draw(canvas, bounds);

Here you're simply calling through to the drawing method on your new class, passing the provided canvas and rect instances.

Build and run by clicking **Run\Run 'wear'**, and then when prompted select the Android Wear emulator that's already running and click **OK**. Once the watch face has been installed you should see something like this in the emulator:

![](Images/time.png)

Nice work! You've successfully created your first Android Wear watch face. But why stop here?! There's more stuff you can add - like the date!

## Drawing the Date

The watch face is pretty useful as it is, after all it does display the time. However, it'd be _even_ more useful if it also showed the date, and that's what you'll add now. You'll update the code that handles the drawing to render the date just above the time, in a lighter colour.

Open **wear\res\values\colors.xml** and add the following, just below the existing resources:

    <color name="watch_face_date_color">#009688</color>

This defines the light green colour that we'll use as the text color for the date. The colours used in this watch face were picked using [Material Palette](http://www.materialpalette.com), which is a great site for generating Material Design compliant colour palettes.

Next, open **wear\res\values\dimensions.xml** and add the following after the existing resources:

    <dimen name="date_size">20dp</dimen>

Just like before, here you're defining the size of the text used to display the date.

Open **WatchFace** and add another format string, this time for the date:

    private static final String DATE_FORMAT = "%02d.%02d.%d";

And just below that, declare another instance of Paint that can be used for _painting_ the date:

    private final Paint mDatePaint;

Next, replace the existing implementation of **WatchFace** with this one:

    WatchFace(Paint timePaint, Paint datePaint, int backgroundColor) {
      this.mTimePaint = timePaint;
      this.mDatePaint = datePaint;
      this.mBackgroundColor = backgroundColor;
    }

Here we've updated the method signature so that it accepts a second Paint parameter, and then you assign that to the `mDatePaint` variable you created in the previous step.

Now add the following just above the return statement in **newInstance()**:

    Paint datePaint = new Paint();
    datePaint.setColor(resources.getColor(R.color.watch_face_date_color));
    datePaint.setTextSize(context.getResources().getDimension(R.dimen.date_size));
    datePaint.setAntiAlias(true);

Just like we did for the time, here you're creating an instance of Paint which will be used for drawing the date. You then set the color, text size, and whether or not the text should be drawn using anti-aliasing.

Next, update the return statement in **newInstance()** so that it uses the new method signature of the default initialiser, passing it the new Paint object:

    return new WatchFace(timePaint, datePaint, resources.getColor(R.color.watch_face_fill));

Finally, add the following to the very bottom of **draw()**, which will actually draw the date:

    String dateText = String.format(DATE_FORMAT, time.monthDay, time.month + 1, time.year);
    float dateCenterX = centerX - (mDatePaint.measureText(dateText) / 2.0f);
    float dateCenterY = timeCenterY - (boundingBox.height() + 10.0f);
    canvas.drawText(dateText, dateCenterX, dateCenterY, mDatePaint);

Here we format the date using both the format string you just added and the existing instance of Time, work out the position it should be drawn, which in this case is just above the time, and then draw it! Also, it is worth noting that you have to add 1 to the month because months are indexed from 0 (so January is 0 and December is 11).

Build and run by clicking **Run\Run 'wear'**, and then when prompted, select the Android Wear emulator that's already running and click **OK**. Once the updated watch face has been installed you should see something similar to the following in the emulator:

![](Images/time_date.png)

The watch face now displays both the time and date, in different colours, using different font sizes. Pretty sharp!

![](Images/victory.gif)

## Where to Go From Here?

You can [download the finished version of the project here](http://finished.project.link).

There are still some other cool things you can do with Android Wear watch faces. Google outlines some of these things in their [developer website here](https://developer.android.com/training/wearables/watch-faces/index.html). As a challenge, see if you can extend your watch face so that it makes a funny sound whenever you tap on it. You can use the information on the Google developer page mentioned earlier.

Also, if you've acquired a taste for Android wearables that will not be "worn down"(see what I did there :]), you can have a look at our [Google Glass tutorial](http://www.raywenderlich.com/92840/google-glass-app-tutorial).

We hope you enjoyed this tutorial, and if you have any questions or comments, please join the forum discussion below!
