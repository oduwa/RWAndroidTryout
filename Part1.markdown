# Custom watch faces for Android Wear

In this tutorial you'll learn how to take advantage of the Android Wear APIs to create your own Android Wear watch face.

Here's a sneak peek of what you'll be building throughout the rest of this tutorial:

![](Images/time_date.png)

Does that whet your appetite? Of course it does - time to get right to it!

## Getting started

I'm going to assume you already have [Android Studio](http://developer.android.com/tools/studio/index.html) installed.

### Creating the project

Open Android Studio and select `Start a new Android Studio project` from the Quick Start menu:

![](Images/quick_start.png)

Name the app `WatcFhace`, set an appropriate company domain and project location, and then click Next. On the Target Android Devices dialog, make sure you check both Phone and Tablet and Wear, set the Minimum SDK to `API 12: Android 5.0 (Lollipop)` for both, and click Next:

![](Images/target_devices.png)

On the subsequent Add an activity to Mobile dialog, select Add No Activity since we don't require an activity, and click Next:

![](Images/add_activity_mobile.png)

Then do the same on the Add an activity to Wear dialog, and click Finish:

![](Images/add_activity_waer.png)

Now we've created the project, it's time to create an emulator suitable for testing Android Wear watch faces.

### Creating an Android Were virtual device

Select `Tools\Android\AVD Manager` and in the window that appears click Create Virtual Device...:

![](Images/AVD.png)

In the Select Hardware dialog, choose Wear from the category list, then Android Wear Square from the hardware list, and then click Next:

![](Images/hardware.png)

When the System Image dialog appears, select the x86 Lollipop image and click Next:

![](Images/system_image.png)

On the next and final dialog, check Use Host GPU, which allows the emulator to use our machines GPU and therefore provide a much better experience. Click Finish:

![](Images/wear_device.png)

Next, select the new Android Wear virtual device in the Android Virtual Device manager and click the green Play button in the Actions section to launch it:

![](Images/AVD_after.png)

After a couple of minutes - you probably _do_ have time to go make a brew - you'll should see the following:

![](Images/emulator.png)

As this is the first time we've launched the emulator follow the on-screen instructions to complete the on-boarding process.

And with that, let's get coding!

## Building a watch face service

Android Wear watch faces are implemented as services and are packaged inside a wearable app. When a user installs an Android app that contains a wearable app with a watch face, the watch face becomes available in the Android Wear companion app on the phone and in the watch face picker on the wearable. When the user selects the watch face, the wearable device shows the watch face and invokes its service callback methods as required.

When our watch face is active, Android Wear invokes the methods in its service when the time changes, or when other important events occur like switching to ambient mode or receiving a new notification. Your service implementation can then redraw the watch face on the screen using the updated time and any other relevant data.

To implement a watch face, you need to extend both **CanvasWatchFaceService** and **CanvasWatchFaceService.Engine**, and then  override the necessary callback methods in **CanvasWatchFaceService.Engine**. Both of these classes are included in the Wearable Support Library.

Right-click on `wear\java\your-package-name.watchface` in the project navigator and choose `New\Java Class`. Name the class WatchFaceService and click OK.

Replace the existing **WatchFaceService** class definition with the following:

    public class WatchFaceService extends CanvasWatchFaceService {

      @Override
      public Engine onCreateEngine() {
        return new Engine()
      }

      private class Enigne extends CanvasWatchFaceService.Engine {

      }
    }

Here you've updated your class definition so it extends **CanvasWatchFaceService**, which is the entry point for Android Wear watch faces, and also implemented **onCreateEngine**, which simply returns an instance of your implementation of **CanvasWatchFaceService.Engine**.

Before we can build and run our new watch face, there's a bit of housekeeping we need to take care of before it'll even show up on the watch.

## Updating the manifest

Right-click `wear\manifests\AndroidManifest.xml` in the project navigator to open it, and add the following just above the application tag:

    <uses-permission android:name="com.google.android.PROVIDE_BACKGROUND" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />

Here we're declaring the two permissions required to run the watch face.

Next, add the following _inside_ the application tag:

    <!-- 1 -->
    <service
      android:name=".WatchFaceService"
      android:label="@string/app_name"
      android:permission="android.permission.BIND_WALLPAPER">
      <!-- 2 -->
      <meta-data
        android:name="android.service.wallpaper"
        android:resource="xml/watch_face" />
      <!-- 3 -->
      <meta-data
        android:name="com.google.android.wearable.watchface.preview"
        android:resource="drawable/preview" />
      <meta-data
        android:name="com.google.android.wearable.watchface.preview_circular"
        android:resource="drawable/preview_circular" />
      <!-- 4 -->
      <intent-filter>
        <action android:name="android.service.wallpaper.WallpaperService" />
        <category android:name="com.google.android.wearable.watchface.category.WATCH_FACE" />
      </intent-filter>
    </service>

Here's the play-by-play of what's happening above:

1. We declare the service, specifying the class and necessary permissions.
2. We declare the xml resource for the watch face, which we'll add in a moment.
3. We declare the preview images to be used by the Android Wear companion app and the watch face picker on the watch itself. Again, we'll add these shortly.
4. We declare what the service can do by way of an intent filter.

Right-click on `wear\res` in the project navigator and choose `New\Android resource directory`. In the subsequent dialog change Resource type to xml and click OK:

![](Images/xml_folder.png)

This creates a new resource folder in which to store xml files.

Right-click on the new `wear\res\xml` folder in the project navigator and choose `New\XML resource file`. Name the file watch_face and click OK:

![](Images/watch_face_xml_file.png)

Replace the contents of the new xml file with the following:

    <?xml version="1.0" encoding="UTF-8"?>
    <wallpaper />

Here we're simply declaring the necessary resource required by Android Wear for watch faces.

Finally, we need to add the preview images. Usually we would want these would be actual screenshots of our watch face, but for the sake of brevity I've prepared some ahead of time that we can use. Download [this zip](Preview-Images.zip) file and unarchive it. Select the two files in Finder and then right-click and choose Copy 2 Items. Head back to Android Studio, right-click on `wear\res\drawable` and choose Paste. Click OK in the confirmation dialog.

Re-open AndroidManifest.xml and we should see that all the resources that were previously red (meaning the corresponding files didn't exist) are now green.

And as green means _go_, hit Run\Run... and then choose wear in the popup dialog:

![](Images/run.png)

In the Edit configuration dialog select Do not launch Activity in the Activity section, and then click Run:

![](Images/run_config.png)

We only have to do this once. Future runs of the watch face will remember the configuration we've chosen here.

In the subsequent Choose device dialog, select the Android Wear emulator that's already running - the one we set up and launched earlier - and click OK:

![](Images/choose_device.png)

Once the watch face has been installed, click-and-hold on the watch face in the emulator to invoke the watch face picker, and scroll left to find our watch face:

![](Images/picker.png)

Tap the preview to choose the watch face. We'll be presented with a black watch face, as shown here:

![](Images/initial_launch.png)

It might not look much as this point, but we've laid all the foundations necessary for building your watch face, and now we can concentrate on the actual coding.

Time to get something on that screen!

## Starting the engine

As I mentioned earlier, Engine implements **CanvasWatchFaceService.Engine**, which is what draws the watch face on the canvas, and also contains a series of useful callbacks.

To begin we need to define our watch face style, and we do this in **onCreate**, which is declared in **CanvasWatchFaceService.Engine**. In defining the watch face style, we can customise how interface elements such as the battery indicator are drawn over the watch face, or how the cards behave in both normal and ambient modes. In order to define the watch face style we call **setWatchFaceStyle**.

Add the following inside **Engine**:

    @Override
    public void onCreate(SurfaceHolder holder) {
      setWatchFaceStyle(new WatchFaceStyle.Builder(WatchFaceService.this)
        .setCardPeekMode(WatchFaceStyle.PEEK_MODE_SHORT)
        .setAmbientPeekMode(WatchFaceStyle.AMBIENT_PEEK_MODE_HIDDEN)
        .setBackgroundVisibility(WatchFaceStyle.BACKGROUND_VISIBILITY_INTERRUPTIVE)
        .setShowSystemUiTime(false));
    }

Here we're defining how four different areas of your watch face should behave:

* We set the card peek mode to **PEEK_MODE_SHORT** which means cards will only show a single line of content.
* We set the ambient peek mode to **AMBIENT_PEEK_MODE_HIDDEN** meaning cards won't be displayed when the watch face enters ambient mode.
* We set the background visibility to **BACKGROUND_VISIBILITY_INTERRUPTIVE** which means the background of the card should be shown briefly, but only if that card represents an interruptive notification.
* And finally, we declare that we don't want the system time to be displayed because we're going to draw it manually.

With the watch face style defined, we now need to implement the necessary callbacks. But first, let's take a look at a selection of the callbacks available to get a better idea of how watch faces behave:

* **onDraw(Canvas canvas, Rect bounds)**: This is called every time the watch face is invalidated, think `drawRect(_:)` and `setNeedsDisplay()` on iOS. Here we'll draw the watch face using the provided Canvas and Rect, the latter defining the face's bounds.
* **onTimeTick()**: This is called periodically in ambient mode to update the time shown by the watch face. This method is called at least once per minute.
* **onVisibilityChanged(boolean visible)**: This is called to inform you of the watch face becoming visible or hidden. If we decide to override this method, you must call super.onVisibilityChanged(visible) as the very first statement in your override.
* **onAmbientModeChanged(boolean inAmbientMode)**: This is called when the device enters or exits ambient mode. Google recommends that the watch face should switch to a black and white display while in ambient mode, and if the watch face displays seconds, they should be hidden in ambient mode.
* **onPeekCardPositionUpdate(Rect rect)**: This is called when the first, peeking card positions itself on the screen. This is where the watch face can change its appearance depending on where the card is on the screen. This callback doesn't provide information about all movements of the card, only about its location when it's peeking at the bottom and allowing the watch face to be exposed.

There are some other callbacks defined in **CanvasWatchFaceService.Engine** that may be of interest, so I highly recommend you check out the docs.

With the theory out of the way, add the following to **Engine**, just below **onCreate**:

    @Override
    public void onTimeTick() {
      super.onTimeTick();
      invalidate()
    }

    @Override
    public void onDraw(Canvas canvas, Rect bounds) {

    }

In **onTimeTick** we simply invalidate the display so the watch face is redrawn. We'll flesh out **onDraw** shortly, but for now it simply calls through to it's super implementation.

## Drawing time

To keep our code clean and readable, we'll create a separate class that'll be responsible for actually performing the drawing, and which will be invoked from within **onDraw** so that it happens at the correct time. We'll also pass the canvas and bounds to this new class.

Right-click on `wear\java\your-package-name.watchface` in the project navigator and choose `New\Java Class`. Name the class WatchFace and click OK.

Add the following inside the definition of **WatchFace**:

    // 1
    private static final String TIME_FORMAT = "02d:02d";
    private final Paint timePaint;
    private final int backgroundColor;

    // 2
    WatchFace(Paint timePaint, int backgroundColor) {
      this.timePaint = timePaint;
      this.backgroundColor = backgroundColor;
    }

    // 3
    public static WatchFace newInstance(Context context) {
      Resources resources = context.getResources();

      // 4
      Paint timePaint = new Paint();
      timePaint.setColor(resources.getColor(R.color.watch_face_time));
      timePaint.setTextSize(resources.getDimension(R.dimen.time_size));
      timePaint.setAntiAlias(true);

      new WatchFace(timePaint, resources.getColor(R.color.watch_face_fill));
    }

    // 5
    public void draw(Canvas canvas, Rect bounds) {

    }

Here's a breakdown of the code above:

1. We define some private variables that are required for drawing the watch face, including an instance of Paint that will be used to draw the text.
2. We implement the default initialiser for the new class.
3. We implement a convenience initialiser that's used to do any necessary setup before calling the default initialiser.
4. We create an instance of Paint, and set the colour, text size, and whether or not the text should be drawn using anti-aliasing.
5. This is where the actual drawing will take place, and it'll be called from within our **onDraw** in **Engine**. We'll flesh this out shortly.

At this point you've probably noticed that any call using resources is currently red, indicating the corresponding resource is missing. We'll fix that now, starting with the colours.

Right-click on `wear\res\values` in the project navigator and choose `New\Values` resource file. In the subsequent dialog name the file colours and click OK.

Add the following to the new file, _inside_ the resources tag:

    <colour name="watch_face_time">#FFFFFF</color>
    <color name="watch_face_fill">#00796B</color>

Here we're simply defining the colours to be used as the fill for the watch face background and the text colour for the time.

Right-click again on `wear\res\values` in the project navigator and choose `New\Values` resource file. Name this file dimensions and click OK.

Again, add the following to the new file, _inside_ the resources tag:

    <dimen name="time_size">46</dimen>

Here we're defining the size of the text used to the display the time. We're abstracting these values out of the code so they're easily changeable, and can be reused across different classes; it's a good practice to follow in Android development.

Re-open WatchFace.java and we should see that all the resources that were previously red are now green.

Now it's time to actually put something on-screen. Add the following inside **draw**:

    // 1
    Time time = new Time();
    time.setToNow();
    // 2
    canvas.drawColor(backgroundColor);
    // 3
    float centerX = bounds.exactCenterX();
    float centerY = bounds.exactCenterY();
    Rect boundingBox = new Rect();
    // 4
    String timeText = String.format(TIME_FORMAT, time.hour, time.minute);
    float timeCenterX = centerX - (timePaint.measureText(timeText) / 2.0f);
    timePaint.getTextBounds(timeText, 0, timeText.length(), boundingBox);
    float timeCenterY = centerY + (boundingBox.height() / 2.0f);
    canvas.drawText(timeText, timeCenterX, timeCenterY, timePaint);

Here's the play-by-play of what's happening:

1. Grab a reference to the current time.
2. Fill the canvas with the background fill colour you defined just a moment ago.
3. Get the center point of the watch face, and declare an instance of Rect that we'll use to measure the dimensions of the text to draw.
4. Format the current time, work out the position at which is should be drawn, and then draw it!

There's just one more thing we need to do before we'll finally see something on-screen, and that's update **Engine** to use our new drawing class.

Double-click on WatchFaceService.java in the project navigator to open it, and then add the following just above **onCreate** in Engine:

    private WatchFace watchFace;

Here we're just declaring a variable that'll hold a reference to your new drawing class. Now initialise this variable by adding the following to the bottom of **onCreate**:

    watchFace = WatchFace.newInstance(WatchFaceService);

Finally, add the following to the bottom of **onDraw**:

    watchFace.draw(canvas, bounds);

Here we're simply calling through to the drawing method on you new class, passing the provided canvas and rect instances.

Build and run by clicking `Run\Run 'wear'`, and then when prompted select the Android Wear emulator that's already running and click OK. Once the watch face has been installed we should see the following in the emulator:

![](Images/time.png)

Nice work! We've successfully created our first Android Wear watch face...but, we're not going to stop there. There's one more thing we need to add - the date!

## Drawing the date

The watch face is pretty useful as it is, after all it does display the time. However, it'd be _even_ more useful if it also showed the date, and that's what we'll add now. We'll update the code that handles the drawing to render the date just above the time, in a lighter colour.

Open `wear\res\values\colors.xml` and add the following, just below the existing resources:

    <color name="watch_face_date">009688</color>

This defines the light green color that we'll use as the text color for the date. The colours used in this watch face were picked using [Material Palette](http://www.materialpalette.com), which is great site for generating Material Design compliant color palettes.

Next, open `wear\res\values\dimensions.xml` and add the following after the existing resources:

    <dimen name="date_size">20dp</dimen>

Just like before, here we're defining the size of the text used to the display the date.

Open WatchFace.java and add another format string, this time for the date:

    private static final String DATE_FORMAT = "%02d.%02d.%d";

And just below that, declare another instance of Paint that can be used for _painting_ the date:

    private final Paint datePaint;

Next, replace the existing implementation of **WatchFace** with this one:

    WatchFace(Paint timePaint, Paint datePaint, int backgroundColor) {
      this.timePaint = timePaint;
      this.backgroundColor = backgroundColor;
    }

Here we've updated the method signature so that it accepts a second Paint parameter, and then you assign that to the datePaint variable you created in the previous step.

Now add the following just above the return statement in **newInstance**:

    Paint datePaint = new Paint();
    datePaint.setColor(resources.getColor(R.color.watch_face_date));
    datePaint.setTextSize(context.getResources().getDimension(R.dimen.date_size));
    datePaint.setAntiAlias(true)

Just like we did for the time, here you're creating an instance of Paint which will be used for drawing the date, and set the color, text size, and whether or not the text should be drawn using anti-aliasing.

Next, update the return statement in **newInstance** so that is uses the new method signature of the default initialiser, passing the new Paint object:

    return new WatchFace(timePaint, datePaint, resources.getColor(R.color.watch_face_fill));

Finally, add the following to the very bottom of **draw**, which will actually draw the date:

    String dateText = String.format(DATE_FORMAT, time.monthDay, time.month + 1, time.year);
    float dateCenterX = centerX - (datePaint.measureText(dateText) / 2.0f);
    float dateCenterY = timeCenterY - (boundingBox.height() + 10.0f);
    canvas.drawText(dateText, dateCenterX, dateCenterY, datePaint);

Here we format the date using both the format string you just added and the existing instance of Time, work out the position it should be drawn, which in this case is just above the time, and then draw it!

Build and run by clicking `Run\Run 'wear'`, and then when prompted select the Android Wear emulator that's already running and click OK. Once the updated watch face has been installed you should see something similar to the following in the emulator:

![](Images/time_date.png)

The watch face now displays both the time and date, in different colours, and using different font sizes. Pretty sharp!

![](Images/victory.gif)
