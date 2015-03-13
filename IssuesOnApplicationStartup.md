# Introduction #
This is adapted from the bug report kindly sent in by Darko Grozdanovski in [Issue 14](https://code.google.com/p/icesoap/issues/detail?id=14). Basically, within Android AsyncTasks have to be generated on the UI thread, and somehow this isn't always guaranteed even though it should be - see [the Android issue page](http://code.google.com/p/android/issues/detail?id=20915) for more details on the bug - at the time of writing it's still marked "for future release". As far as I can tell it seems to affect 2.2 and 2.3, but I'm not sure what else it affects.

# Details #

In your class extending Application, in the onCreate method, include this code:

```
   Class.forName("android.os.AsyncTask");
```

e.g.

```
public class App extends Application {

   ...

   @Override
   public void onCreate() {
      super.onCreate();

      try {
         // A bug in the async task, happens if the async task is initialized for the first time from a non-ui thread
         Class.forName("android.os.AsyncTask");
      } catch (Exception e) {
         Logger.d(TAG, "", e);
      }
   }

   ...

}
```