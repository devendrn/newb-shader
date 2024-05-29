+++
title = "Customization"
description = "Learn how to customize Newb X Legacy"
+++

<div style="text-align: center;">

[Android](#android) | [Windows](#windows)

</div>

> Customizing Newb X Legacy is not as simple as just editing a text file. You can ask for help on our [Discord server](https://discord.gg/newb-community-844591537430069279) if you face issues following this guide.

# Android
---

Requires around 200Mb data to download necessary apps and files.

Install **[Termux](https://f-droid.org/repo/com.termux_118.apk)** for building shader.  
Install **[NMM File Manager](https://play.google.com/store/apps/details?id=in.mfile)** for editing (You can also use Acode instead).

### Getting source code

Open Termux. Then copy-paste and run the following commands:

```
pkg install openjdk-17 git zip
```
It will ask for confirmation, type `y` to confirm.

Now clone the source code:
```
git clone https://github.com/devendrn/newb-x-mcbe
```

Go inside directory and run setup:
```
cd newb-x-mcbe
./setup.sh
```

### Accessing Termux storage

Launch NMM File Manager and open sidebar.

Click `+` icon and choose External Storage.

Select Termux app in sidebar and accept permission.

You can now see a new storage entry in sidebar from which you can access Termux files.

### Building shader

> **Tip**: Double tap `↹` to autcomplete commands in Termux.


In Termux, go inside newb-x-mcbe folder:
```
cd newb-x-mcbe
```

Now build all materials (takes more time) by running:
```
./build.sh
```

To build only some materials, run:
```
./build.sh -m RenderChunk Sky
```
The output will be in `build/android` folder.

Available parameters for the build.sh:

| Option | Parameter description |
| :-: | :- |
| -p | Target platforms (Android, iOS) |
| -m | Materials to compile (if unspecified, builds all material files) |
| -t | Number of threads to use for compilation (default is CPU core count) |

### Editing config

Open NMM, go inside Termux home and navigate to `newb-x-mcbe/include/newb/config.h`.

Open the file and make your changes. After making a change, build the shader and test it to make sure it works.


# Windows
---

Install [Git](https://git-scm.com/download/win).

### Getting the source code

Download [newb-x-mcbe-source.zip](https://github.com/devendrn/newb-x-mcbe/archive/refs/heads/main.zip) and then extract contents into a folder.

Go inside the extracted folder and open command prompt there.

Run `.\setup.bat`. This will download MBT, shaderc, and material data required to build shader.

### Building shader

Go inside newb-x-mcbe folder and open command prompt.

To build all shader files, do:

```
.\build.bat
```

When editing config, you generally want to save build time by only building shader files you need:

```
.\build.bat -m RenderChunk Sky
```

This will only build terrain and sky materials. The output will be in `build\Windows` folder.

Available parameters for the build.bat:

| Option | Parameter description |
| :-: | :- |
| -p | Target platforms (Windows, Android, iOS, Merged) |
| -m | Materials to compile (if unspecified, builds all material files) |
| -t | Number of threads to use for compilation (default is CPU core count) |

### Editing config

To edit config, open `include/newb/config.h` and make your changes.

After making a change, build the shader and test it to make sure it works.
