# Steps to run react-native-maps with user frameworks

## step 1. Instaling react-native-maps and react-native-photo-editoryarn

1. ```yarn add react-native-maps```
    1. [Enabling Google Maps](https://github.com/react-native-maps/react-native-maps/blob/master/docs/installation.md#enabling-google-maps): 
        1. In pod file put these line before "use_native_modules!".

            ```bash
            rn_maps_path = '../node_modules/react-native-maps'
            pod 'react-native-google-maps', :path => rn_maps_path
            ```

        1. put these line in your ```AppDelegate.m```
            1. ```#import <GoogleMaps/GoogleMaps.h>```

            1. add this line ```[GMSServices provideAPIKey:@"_YOUR_API_KEY_"];```to your function

            ```bash
            @implementation AppDelegate
            - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
            {
              [GMSServices provideAPIKey:@"_YOUR_API_KEY_"]; //Add this
              ...
            }
            ```

1. ```yarn add react-native-photo-editor```
    1. Put these line after use_native_modules as mentioned [here](https://github.com/prscX/react-native-photo-editor#rn61--rnpe-v1-).

    ```bash
    pod 'RNPhotoEditor', :path => '../node_modules/react-native-photo-editor/ios'
    use_frameworks! :linkage => :static
    pod 'iOSPhotoEditor', :git => 'https://github.com/prscX/photo-editor', :branch => 'master'
    
    post_install do |installer|
    installer.pods_project.targets.each do |target|
      if target.name.include?('iOSPhotoEditor')
        target.build_configurations.each do |config|
          config.build_settings['SWIFT_VERSION'] = '5'
        end
      end
    end
    end 
    
    $static_framework = ['FlipperKit', 'Flipper', 'Flipper-Folly',
    'CocoaAsyncSocket', 'ComponentKit', 'Flipper-DoubleConversion',
    'Flipper-Glog', 'Flipper-PeerTalk', 'Flipper-RSocket', 'Yoga', 'YogaKit',
    'CocoaLibEvent', 'OpenSSL-Universal', 'boost-for-react-native']
  
  
    pre_install do |installer|
    Pod::Installer::Xcode::TargetValidator.send(:define_method, :verify_no_static_framework_transitive_dependencies) {}
    installer.pod_targets.each do |pod|
        if $static_framework.include?(pod.name)
          def pod.build_type;
            Pod::BuildType.static_library
          end
        end
      end
    end
  


3. Other necessary changes in pod file

    1. As mentioned [here](https://fbflipper.com/docs/getting-started/react-native-ios/#react-native-063) add the following line to your post install script
    ```flipper_post_install(installer)```
    1. To avoid error ```'RCTConvert+AirMap.h' file not found``` add the following lines too in your post install script as mentioned [here](https://github.com/react-native-maps/react-native-maps/issues/3597#issuecomment-745026120):

    ```#!/bin/bash
      if target.name == 'react-native-google-maps'
        target.build_configurations.each do |config|
          config.build_settings['CLANG_ENABLE_MODULES'] = 'No'
        end
      end
      ```

4. Error:
    >  Cycle inside FBReactNativeSpec; building could produce unreliable results. This usually can be resolved by moving the shell script phase '[CP-User] Generate Specs' so that it runs before the build phase that depends on its outputs. Cycle details: → Target 'FBReactNativeSpec': Libtool /Users/../Library/Developer/Xcode/DerivedData/../Build/Products/Debug-iphonesimulator/FBReactNativeSpec/FBReactNativeSpec.framework/FBReactNativeSpec normal ○ Target 'FBReactNativeSpec' has compile command with input '/Users/../Documents/ReactNative/../ios/Pods/Target Support Files/FBReactNativeSpec/FBReactNativeSpec-dummy.m' ○ That command depends on command in Target 'FBReactNativeSpec': script phase “[CP-User] Generate Specs” ○ Target 'FBReactNativeSpec' has copy command from '/Users/../Documents/ReactNative/../node_modules/react-native/React/FBReactNativeSpec/FBReactNativeSpec/FBReactNativeSpec.h' to '/Users/../Library/Developer/Xcode/DerivedData/../Build/Products/Debug-iphonesimulator/FBReactNativeSpec/FBReactNativeSpec.framework/Headers/FBReactNativeSpec.h' ○ That command depends on command in Target 'FBReactNativeSpec': script phase “[CP-User] Generate Specs”```

    Solution: as mentioned [here](https://github.com/facebook/react-native/issues/31034#issuecomment-812564390) and [here](https://github.com/software-mansion/react-native-screens/issues/842#issuecomment-812543933), put these line on your pod file

    ```bash
    if (target.name&.eql?('FBReactNativeSpec'))
      target.build_phases.each do |build_phase|
        if (build_phase.respond_to?(:name) && build_phase.name.eql?('[CP-User] Generate Specs'))
          target.build_phases.move(build_phase, 0)
        end
      end
    end
    ```

5. Finally run ```cd ios/ && pod install && cd ..```

## step 2. Changes XCODE settings and native code

1. Goto target->build settings-> search for ```library search path``` and add the following line (with quotation) for both debug and release,
```"$(PODS_ROOT)/../../node_modules/react-native-maps/lib/ios/AirMaps"```
![Debug](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/1.%20librarySearchPathDebug.png)
![Release](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/2.%20librarySearchPathRelease.png)

1. Add AirMaps and AirGoogleMaps. [source](https://github.com/react-native-maps/react-native-maps/issues/3536#issuecomment-683563649)
    1. Right click on your project and click ```Add files to <projecjName>``` ![right click](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/3.%20rightclick%20on%20project%20name%20and%20select%20%22Add%20Files%20to%20%3CprojectName%3E.png)
    1. goto -> ```node_modules/react-native-maps/lib/ios/```
    1. Select both AirMaps and AirGoogleMaps
    1. Make sure the option ```create groups``` and your target is selected.![add AirMaps and AirGoogleMaps](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/4.%20select%20airmaps%20and%20airgooglemaps.png)
1. Add google-maps-ios-utils: According to these official [instruction](https://github.com/googlemaps/google-maps-ios-utils/blob/b721e95a500d0c9a4fd93738e83fc86c2a57ac89/Swift.md#integrating-with-swift-projects-which-use-use_frameworks-in-the-podfile)
    1. ```git clone https://github.com/googlemaps/google-maps-ios-utils master``` on ios folder . We need only the content of ```src``` folder. You can delete other files and folders
    1. On XCODE, Right click on a project and create a group named ```Google-Maps-iOS-Utils```. ![create a group name Google-Maps-iOS-Utils](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/10.%20creating%20a%20new%20group%20google-maps-iOS-utils.png)

    1. Right click on that folder and click ```Add files to ...```

    1. Select all the folders of ```ios/master/src``` directory of previously cloned directory of step(i).

    1. Make sure the the options is selected "create groups" and target is selected as your app target.
    ![Add all the content of ios/master/src folder](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/13.%20add%20src%20folder%20content%20to%20google-maps-ios-utils%20group.png)

    1. Delete the ```Google-Maps-iOS-Utils/Heatmap/HeatMapInterpolationPoint.swift file``` as mentioned [here](https://github.com/react-native-maps/react-native-maps/issues/3536#issuecomment-683563649). Select remove reference when deleting.

    1. Goto project -> build settings -> search for ```preprocessor macros``` and add the following for both Debug and Release.
        1. ```HAVE_GOOGLE_MAPS=1```
        1. ```HAVE_GOOGLE_MAPS_UTILS=1``` 
        ![add preprocessor macros have google maps and google masp utils](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/5.%20preprocessing%20macros%20havegooglemaps.png)

    1. run ```cd ios/ && pod intall && cd ..```

## Step 3

### If we want to run the project from xcode we will face a bunch of errors: lets solve those

#### Error:

>Multiple commands produce '/Users/../Library/Developer/Xcode/DerivedData/...app/LaunchScreen.storyboardc':
>
>1) Target '..' (project '..'): LinkStoryboards
>2) That command depends on command in Target '..' (project '..'): script phase “\[CP\] Copy Pods

#### Solution:

As mentioned "Multiple commands produce ... LaunchScreen.storyboardc", we can safely remove ```LaunchScreen.storyboard``` from build phases as it is already generated without us explicitly importing it.
 
 1. Goto ```Build Phases``` and look for ```Copy Bundle Resources```
 1. Delete ```LaunchScreen.storyboard```
 ![delete launchScreen.storyboard](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/6.%20delete%20launchscreen.storyboard.png)

*Caution: There is another solution to this. Setting the ``` Buil System``` to ```Legacy Build System```in ```File -> Workspace Settings```. But this leed to another problem later on. Error: ```library not found for -lCocoaAsyncSocket```. In legacy build system this problem occurs but not in new build system.*

#### Error: 'Google-Maps-iOs-Utils/GMUKMLParser.h' file not found
![problem with import statement](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/7.%20import%20statement%20problem.png)
#### Solution:

react-native-maps official guide [says](https://github.com/react-native-maps/react-native-maps/blob/master/docs/installation.md#build-issues-with-google-maps-ios-utils-ios) to follow this instructions on [```google-maps-ios-utils/Swift.md```](https://github.com/googlemaps/google-maps-ios-utils/blob/b721e95a500d0c9a4fd93738e83fc86c2a57ac89/Swift.md). However these instructions are vague. [Here](https://github.com/googlemaps/google-maps-ios-utils/issues/86#issuecomment-310500599) is a more clear instructions set:

1. We have cloned and added google-maps-ios-utils in step 2. If not, please follow these instructions on step 2(3).
1. Create Swift Bridging Header
    1. File > New > File > Header File
    1. Usually the name would be [Project Name]-Bridging-Header.h but you name it whatever you want. I am naming it just ```Header.h``` in this project as it is really a simple project.![new file](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/8.%20creating%20header%20file%201.png)![select file type](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/8.%20creating%20header%20file%202.png)![select location for header file](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/8.%20creating%20header%20file%203.png)![final header file location](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/8.%20creating%20header%20file%204.png)
    1. Goto target->Build Settings->search for ```Swift Compiler - General```
        1. In ```Objective-C Bridging Header``` put ```Header.h```.![linking the bridging header](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/9.%20objc%20bridging%20header.png)
1. Put these following line on the header file:

    ```bash
    #import "GMUMarkerClustering.h"
    #import "GMUGeoJSONParser.h"
    #import "GMUKMLParser.h"
    #import "GMUGeometryRenderer.h"
    ```

1. Put these following line in ```Google-Maps-iOS-Utils/Clustering/GMUMarkerClustering.h```

    ```bash
    #import "GMUCluster.h"
    #import "GMUClusterItem.h"
    #import "GMUClusterManager.h"
    #import "GMUDefaultClusterIconGenerator.h"
    #import "GMUDefaultClusterRenderer.h"
    #import "GMUGridBasedClusterAlgorithm.h"
    #import "GMUNonHierarchicalDistanceBasedAlgorithm.h"
    #import "GMUStaticCluster.h"

    #import "GQTPointQuadTree.h"
    ```

1. Change to import statement to ```#import "GMUHeatmapTileLayer.h"``` in ```AirGoogleMaps/AIRGoogleMapHeatmap.h``` file. ![changing the import statement in AIRGoogleMapHeatman.h](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/11.%20change%20the%20import%20statement%20on%20AirGoogleMapHeatmap.h.png)
1. Put these following line in ```AirGoogleMaps/AIRGoogleMap.m```

    ```bash
    #import "GMUKMLParser.h"
    #import "GMUPlacemark.h"
    #import "GMUPoint.h"
    #import "GMUGeometryRenderer.h"
    ```
![changing the import statement in AirGoogleMap.m](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/12.%20change%20the%20import%20statement%20in%20AIRGoogleMap.m.png)
### Error generated by add AirGoogleMaps, AirMaps, Google-Maps-iOS-Utils

#### Error: ld: 658 duplicate symbols for architecture x86_64

#### Solution

1. Delete ```Google-Maps-iOS-Utils``` from ```Pods/Pods``` pods section (select the option "Remove References"). Because we have manually added ```Google-Maps-iOS-Utils``` Manually.


![remove Google-Maps-iOS-Utils from Pods section](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/14.%20delete%20google-maps-ios-utils%20from%20pod%20section.png)

if we compile we'll get ```ld: 470 duplicate symbols for architecture x86_64```.

1. Delete ```react-native-maps``` and ```react-native-google-maps``` from ```Pods/Development Pods```
![remote react-native-maps and react-native-google-maps from Pods/Development Pods section](https://raw.githubusercontent.com/mrpmohiburrahman/react-native-maps-with-use-famework/main/assets/15.%20delete%20react-native-maps%20and%20react-native-google-maps%20from%20pods%20section.png)

Now the project will finally run.
