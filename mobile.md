# Run app on mobile

         using iOS as example

[Tutorial](https://www.meteor.com/tutorials/blaze/running-on-mobile)

[prerequisites](http://guide.meteor.com/mobile.html#installing-prerequisites)

### on an iOS Simulator

``` sh
meteor add-platform ios

meteor install-sdk ios

meteor-practice git:(master) ✗ meteor run ios
[[[[[ ~/Documents/projects/meteor/meteor-practice ]]]]]

=> Started proxy.                             
=> Started MongoDB.                           
=> Started your app.                                                               
                                              
=> App running at: http://localhost:3000/     
=> Errors executing Cordova commands:         
                                              
   While running Cordova app for platform iOS with options --emulator:
   Error: Command failed:                     
   /Users/tiansiyuan/Documents/projects/meteor/meteor-practice/.meteor/local/cordova-build/platforms/ios/cordova/run --emulator
   ENOENT, no such file or directory
   \'/Users/tiansiyuan/Library/Logs/CoreSimulator/AAFAAAD9-0BEC-4121-9F67-F79B3ABDF000/system.log\'
   at ChildProcess.exitCallback (/tools/utils/processes.js:151:23)
   at ChildProcess.emit (events.js:98:17)
   at Process.ChildProcess._handle.onexit (child_process.js:820:12)

/Users/tiansiyuan/.meteor/packages/meteor-tool/.1.3.2_2.uohqey++os.osx.x86_64+web.browser+web.cordova/mt-os.osx.x86_64/isopackets/cordova-support/npm/node_modules/meteor/promise/node_modules/meteor-promise/promise_server.js:116
      throw error;
            ^
ExitWithCode:1
➜  meteor-practice git:(master) ✗ meteor run ios
[[[[[ ~/Documents/projects/meteor/meteor-practice ]]]]]

=> Started proxy.                             
=> Started MongoDB.                           
=> Started your app.                          
                                              
=> App running at: http://localhost:3000/     
=> Started app on iOS Simulator.
```

### on an iOS device



``` sh
meteor-practice git:(master) ✗ meteor run ios-device
[[[[[ ~/Documents/projects/meteor/meteor-practice ]]]]]

=> Started proxy.                             
=> Started MongoDB.                           
                                              
WARNING: You are testing your app on a remote device. For the mobile app to be able to connect to the local server, make sure
         your device is on the same network, and that the network configuration allows clients to talk to each other (no client
         isolation).
                                              
Your project has been opened in Xcode so that you can run your app on an iOS device. For further instructions, visit this wiki
page: https://github.com/meteor/meteor/wiki/How-to-run-your-app-on-an-iOS-device
                                              
=> Started app on iOS Device.                 
=> Started your app.                          
                                              
=> App running at: http://localhost:3000/     
```

### Connecting to the server

A Meteor app should be able to connect to a server in order to load data and to enable hot code push, which automatically updates a running app when you make changes to its files. During development, this means the device and the computer you run meteor on will have to be part of the same WiFi network, and the network configuration shouldn’t prevent the device from reaching the server. 