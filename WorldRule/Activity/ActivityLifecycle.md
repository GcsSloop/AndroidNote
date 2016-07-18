# Activity生命周期

## 典型情况下的生命周期

<table border="2" width="85%" align="center" frame="hsides" rules="rows">
     <colgroup align="left" span="3" />
     <colgroup align="left" />
     <colgroup align="center" />
     <colgroup align="center" />

     <thead>
     <tr><th colspan="3">Method</th> <th>Description</th> <th>Killable?</th> <th>Next</th></tr>
     </thead>

     <tbody>
     <tr><td colspan="3" align="left" border="0"><code><a href="https://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)">onCreate()</a></code></td>
         <td>Called when the activity is first created.
             This is where you should do all of your normal static set up:
             create views, bind data to lists, etc.  This method also
             provides you with a Bundle containing the activity's previously
             frozen state, if there was one.
             <p>Always followed by <code>onStart()</code>.</td>
         <td align="center">No</td>
         <td align="center"><code>onStart()</code></td>
     </tr>

     <tr><td rowspan="5" style="border-left: none; border-right: none;">&nbsp;&nbsp;&nbsp;&nbsp;</td>
         <td colspan="2" align="left" border="0"><code><a href="https://developer.android.com/reference/android/app/Activity.html#onRestart()">onRestart()</a></code></td>
         <td>Called after your activity has been stopped, prior to it being
             started again.
             <p>Always followed by <code>onStart()</code></td>
         <td align="center">No</td>
         <td align="center"><code>onStart()</code></td>
     </tr>

     <tr><td colspan="2" align="left" border="0"><code><a href="https://developer.android.com/reference/android/app/Activity.html#onStart()">onStart()</a></code></td>
         <td>Called when the activity is becoming visible to the user.
             <p>Followed by <code>onResume()</code> if the activity comes
             to the foreground, or <code>onStop()</code> if it becomes hidden.</td>
         <td align="center">No</td>
         <td align="center"><code>onResume()</code> or <code>onStop()</code></td>
     </tr>

     <tr><td rowspan="2" style="border-left: none;">&nbsp;&nbsp;&nbsp;&nbsp;</td>
         <td align="left" border="0"><code><a href="https://developer.android.com/reference/android/app/Activity.html#onResume()">onResume()</a></code></td>
         <td>Called when the activity will start
             interacting with the user.  At this point your activity is at
             the top of the activity stack, with user input going to it.
             <p>Always followed by <code>onPause()</code>.</td>
         <td align="center">No</td>
         <td align="center"><code>onPause()</code></td>
     </tr>

     <tr><td align="left" border="0"><code><a href="https://developer.android.com/reference/android/app/Activity.html#onPause()">onPause()</a></code></td>
         <td>Called when the system is about to start resuming a previous
             activity.  This is typically used to commit unsaved changes to
             persistent data, stop animations and other things that may be consuming
             CPU, etc.  Implementations of this method must be very quick because
             the next activity will not be resumed until this method returns.
             <p>Followed by either <code>onResume()</code> if the activity
             returns back to the front, or <code>onStop()</code> if it becomes
             invisible to the user.</td>
         <td align="center"><font color="#800000"><strong>Pre-<code><a href="https://developer.android.com/reference/android/os/Build.VERSION_CODES.html#HONEYCOMB">HONEYCOMB</a></code></strong></font></td>
         <td align="center"><code>onResume()</code> or<br>
                 <code>onStop()</code></td>
     </tr>

     <tr><td colspan="2" align="left" border="0"><code><a href="https://developer.android.com/reference/android/app/Activity.html#onStop()">onStop()</a></code></td>
         <td>Called when the activity is no longer visible to the user, because
             another activity has been resumed and is covering this one.  This
             may happen either because a new activity is being started, an existing
             one is being brought in front of this one, or this one is being
             destroyed.
             <p>Followed by either <code>onRestart()</code> if
             this activity is coming back to interact with the user, or
             <code>onDestroy()</code> if this activity is going away.</td>
         <td align="center"><font color="#800000"><strong>Yes</strong></font></td>
         <td align="center"><code>onRestart()</code> or<br>
                 <code>onDestroy()</code></td>
     </tr>

     <tr><td colspan="3" align="left" border="0"><code><a href="https://developer.android.com/reference/android/app/Activity.html#onDestroy()">onDestroy()</a></code></td>
         <td>The final call you receive before your
             activity is destroyed.  This can happen either because the
             activity is finishing (someone called <code><a href="https://developer.android.com/reference/android/app/Activity.html#finish()">finish()</a></code> on
             it, or because the system is temporarily destroying this
             instance of the activity to save space.  You can distinguish
             between these two scenarios with the <code><a href="https://developer.android.com/reference/android/app/Activity.html#isFinishing()">isFinishing()</a></code> method.</td>
         <td align="center"><font color="#800000"><strong>Yes</strong></font></td>
         <td align="center"><em>nothing</em></td>
     </tr>
     </tbody></table>
