<div align="center">


# Camera Kit for Android


</div>

Camera Kit brings the power of AR platform to your websites and mobile apps on iOS and Android. It has never been easier to create and deliver scalable, multi-platform AR experiences to meet your customers, wherever they are.

<p align="center">
 <img src="https://github.com/user-attachments/assets/c7a6e407-ee76-4dd6-b174-4ac03f641131" width="9%" alt="distort" />
 <img src="https://github.com/user-attachments/assets/8285ec1d-8b3a-4d1d-a7d2-db62b16d7ee3" width="9%" alt="hair_simulation" /> 
 <img src="https://github.com/user-attachments/assets/8530eb66-567c-4432-958d-15285d50d6cb" width="9%" alt="chane_physics" />
 <img src="https://github.com/user-attachments/assets/4af949f9-9426-413d-8011-0292278106ea" width="9%" alt="try_on" />
 <img src="https://github.com/user-attachments/assets/b79dff9b-34cd-4949-8c8f-fa46399d5351" width="9%" alt="3d_hand_tracking" /> 
 <img src="https://github.com/user-attachments/assets/ff32ab27-e48d-4aed-aa1a-8f46726e5b0b" width="9%" alt="wrist_wear_try_on" />
 <img src="https://github.com/user-attachments/assets/dff811af-b7b4-4e86-be28-d149b4860e5b" width="9%" alt="eye_wear_try_on" />
 <img src="https://github.com/user-attachments/assets/6005c5ed-ad31-45c6-8fad-90a388724ec0" width="9%" alt="true_size_object" />   
 <img src="https://github.com/user-attachments/assets/2ed8522c-280a-4694-bc3e-79ce450fb0a0" width="9%" aly="vfx">
 <img src="https://github.com/user-attachments/assets/c4097e49-855f-4c94-8a35-2753f7bcbd83" width="9%" alt="landmarkers" />
</p>

## Features

### AR Capabilities
- Face Effects
- Body / Face / Hand Tracking
- World Tracking
- Background Segmentation
- Location AR

### Android SDK
- Integrate with Camera Kit com.snap.camerakit.Session, which allows to maintain full control over session configuration, management, and lifecycle
- Fetch and display your lenses
- Capture media
- Leverage Reference UI modules to quickly build Camera Kit based experiences
- Supports Android 5.0+ and SDK 21+

## Integration Steps
2. Integrate Camera Kit SDK into your Android application

### Configuration

All of the Camera Kit artifacts are published under a single version and it is possible to pick and choose the dependencies necessary for your specific project:

```groovy
    implementation "com.snap.camerakit:camerakit:$cameraKitVersion"
    implementation "com.snap.camerakit:lenses-bundle:$cameraKitVersion"
    implementation "com.snap.camerakit:support-camerax:$cameraKitVersion"
```

In order for Camera Kit to be able to communicate with remote services to get content such as lenses, app needs to provide Camera Kit its unique "API token", The easiest way to do this is to define the token within the app's [AndroidManifest.xml](./Samples/camerakit-sample-basic/src/main/AndroidManifest.xml):

```xml
<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
     
        <meta-data android:name="com.snap.camerakit.api.token" android:value="REPLACE-THIS-WITH-YOUR-OWN-APP-SPECIFIC-VALUE" />

</application>
```

Camera Kit is built targeting Java8 bytecode which requires enabling Java8 compatibility (desugar) support via Android Gradle Plugin (AGP) `compileOptions` for your app:

```groovy
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

*For more information, see build configuration in `camerakit-sample-full` [build.gradle](./Samples/camerakit-sample-full/build.gradle).*

## Usage

### Initializing Image Processor and Preview

```kotlin
    import com.snap.camerakit.support.camerax.CameraXImageProcessorSource

    var imageProcessorSource = CameraXImageProcessorSource(
            context = this, lifecycleOwner = this
    )

    imageProcessorSource.startPreview(true) // true = front camera , false = back
```

### Initializing Camera Kit Session

```kotlin
    var cameraKitSession = Session(context = this) {
        imageProcessorSource(imageProcessorSource)
        attachTo(findViewById(R.id.camera_kit_stub))
    }
```

### Applying AR Lens

```kotlin
    cameraKitSession.apply {
        lenses.repository.observe(
            LensesComponent.Repository.QueryCriteria.ById(LENS_ID, LENS_GROUP_ID)
        ) { result ->
            result.whenHasFirst { requestedLens ->
                lenses.processor.apply(requestedLens)
            }
        }
    }
```

### Lifecycle

`Session` instance is typically shared within a single Android application, service or activity lifecycle scope as `Session` is costly in terms of memory and cpu resources it requires to operate. Once done with a `Session`, It is **essential** to dispose it using `Session#close` method which releases all the acquired resources in Camera Kit safe manner. 

```kotlin
    override fun onDestroy() {
        cameraKitSession.close()
        super.onDestroy()
    }
```

### Samples



- [`camerakit-sample-full`](./Samples/camerakit-sample-full) contains a fully functioning camera capture with lenses and preview flow.


## Development




## Troubleshooting

The following is a list of common issues and suggestions on how to troubleshoot them when integrating Camera Kit into your own app.

### Camera preview is black

- Check that your device is supported by Camera Kit using `Sessions#supported` method. The minimum OpenGLES version that Camera Kit supports is 3.0.
- Check that a camera based `Source<ImageProcessor>` such as `CameraXImageProcessorSource` is provided to the `Session.Builder`. If you cannot provide an implementation of `Source<ImageProcessor>` then make sure to connect a `SurfaceTexture` based input to the current `Session.processor`.
- If no `ViewStub` is provided to the `Session.Builder` then Camera Kit does not attempt to render any views such as lenses carousel as well as camera preview. To see camera preview without any other Camera Kit views, a `TextureView`, `SurfaceTexture` or `Surface` based output must be connected to the current `Session.processor`.
- Compare versions of dependencies of your app to the Camera Kit sample apps. If dependency versions differ, for example the `camerakit-sample-full` uses `androidx.constraintlayout:constraintlayout:1.1.3` while your app uses `androidx.constraintlayout:constraintlayout:2.0.0`, it is possible that code ported from Camera Kit sample to your app may not work as expected.

### Nothing works as expected

- Attach debugger to your app, enable Java exception breakpoints and build a `Session` while checking that there are no unexpected exceptions with stacktraces related to Camera Kit.
- Attach debugger to your app, pause all threads and export their state into a text file - check that there are no deadlocked threads related to Camera Kit.
