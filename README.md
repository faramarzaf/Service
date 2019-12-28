# Service   

<p align="center">
  <img src="https://clicksite.org/files2/14571f987/images/services.jpg" /> 
</p>

A *service* is a component that runs in the background to perform long-running operations without needing to interact with the user and it works even if application is destroyed.   


- Started  
A service is started when an application component, such as an activity, starts it by calling `startService()`. Once started, a service can run in the background indefinitely, even if the component that started it is destroyed.  
 	
- Bound  
A service is bound when an application component binds to it by calling `bindService()`. A bound service offers a client-server interface that allows components to interact with the service, send requests, get results, and even do so across processes with interprocess communication (IPC).


The *Service* base class defines various callback methods and the most important are given below. You don't need to implement all the callbacks methods.   

| Method  | Description |
| ------------- | ------------- |
|onStartCommand()| The system calls this method when another component, such as an activity, requests that the service be started, by calling `startService()`. If you implement this method, it is your responsibility to stop the service when its work is done, by calling `stopSelf()` or `stopService()` methods. |
|onBind()| The system calls this method when another component wants to bind with the service by calling bindService(). If you implement this method, you must provide an interface that clients use to communicate with the service, by returning an IBinder object. You must always implement this method, but if you don't want to allow binding, then you should return null. |
|onUnbind()| The system calls this method when all clients have disconnected from a particular interface published by the service.|
|onRebind()| The system calls this method when new clients have connected to the service, after it had previously been notified that all had disconnected in its onUnbind(Intent). |
|onCreate()| The system calls this method when the service is first created using onStartCommand() or onBind(). This call is required to perform one-time set-up. |
|onDestroy()| The system calls this method when the service is no longer used and is being destroyed. Your service should implement this to clean up any resources such as threads, registered listeners, receivers, etc. |

The following skeleton service demonstrates each of the life cycle methods:  

```java
public class HelloService extends Service {
   
   /** indicates how to behave if the service is killed */
   int mStartMode;
   
   /** interface for clients that bind */
   IBinder mBinder;     
   
   /** indicates whether onRebind should be used */
   boolean mAllowRebind;

   /** Called when the service is being created. */
   @Override
   public void onCreate() {
     
   }

   /** The service is starting, due to a call to startService() */
   @Override
   public int onStartCommand(Intent intent, int flags, int startId) {
      return mStartMode;
   }

   /** A client is binding to the service with bindService() */
   @Override
   public IBinder onBind(Intent intent) {
      return mBinder;
   }

   /** Called when all clients have unbound with unbindService() */
   @Override
   public boolean onUnbind(Intent intent) {
      return mAllowRebind;
   }

   /** Called when a client is binding to the service with bindService()*/
   @Override
   public void onRebind(Intent intent) {

   }

   /** Called when The service is no longer used and is being destroyed */
   @Override
   public void onDestroy() {

   }
}
```



**Service OR Thread?**  

- A service is simply a component that can run in the background, even when the user is not interacting with your application, so you should create a service only if that is what you need.

- If you must perform work outside of your main thread, but only while the user is interacting with your application, you should instead create a new thread. For example, if you want to play some music, but only while your activity is running, you might create a thread in `onCreate()`, start running it in `onStart()`, and stop it in `onStop()`. Also consider using `AsyncTask` or `HandlerThread` instead of the traditional Thread class.  

- Remember that if you do use a service, it still runs in your application's main thread by default, so you should still create a new thread within the service if it performs intensive or blocking operations.   


**IntentService**  
This is a subclass of Service that uses a worker thread to handle all of the start requests, one at a time. This is the best option if you don't require that your service handle multiple requests simultaneously. Implement `onHandleIntent()`, which receives the intent for each start request so that you can complete the background work.



**The IntentService class does the following:**  

- It creates a default worker thread that executes all of the intents that are delivered to `onStartCommand()`, separate from your application's main thread.  

- Creates a work queue that passes one intent at a time to your `onHandleIntent()` implementation, so you never have to worry about multi-threading.  

- Stops the service after all of the start requests are handled, so you never have to call `stopSelf()`.

- Provides a default implementation of `onBind()` that returns null.

- Provides a default implementation of `onStartCommand()` that sends the intent to the work queue and then to your `onHandleIntent()` implementation.  

- To complete the work that is provided by the client, implement `onHandleIntent()`. However, you also need to provide a small constructor for the service.  

Here's an example implementation of IntentService:  
```java
public class HelloIntentService extends IntentService {

  /**
   * A constructor is required, and must call the super <code><a href="/reference/android/app/IntentService.html#IntentService(java.lang.String)">IntentService(String)</a></code>
   * constructor with a name for the worker thread.
   */
  public HelloIntentService() {
      super("HelloIntentService");
  }

  /**
   * The IntentService calls this method from the default worker thread with
   * the intent that started the service. When this method returns, IntentService
   * stops the service, as appropriate.
   */
  @Override
  protected void onHandleIntent(Intent intent) {
      // Normally we would do some work here, like download a file.
      // For our sample, we just sleep for 5 seconds.
      try {
          Thread.sleep(5000);
      } catch (InterruptedException e) {
          // Restore interrupt status.
          Thread.currentThread().interrupt();
      }
  }
}
```





