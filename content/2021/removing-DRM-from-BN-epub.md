---
Title: Removing DRM from Barnes&Nobles .epub Books
Date: 2021-01-30
Keywords: DRM, Removing DRM from epub, Barnes&Noble, B&N epub DRB, DeDRM, Calibre
Version: Ubuntu, 20.04.1, Calibre, 5.10.1, Android Studio, 4.1.2, B&N Reader App .apk, 5.2.0.18, DeDRM Calibre Plugin, 5.10.1
---

Important
 ~ These instructions are intended to be used with books that you *legally* purchased from B&N and are trying to backup. Don't get me in trouble!

These instructions are adopted from Aric Renzo at [https://www.aricrenzo.com/2019-12-13-Liberate-Your-Nook-Ebooks/](https://www.aricrenzo.com/2019-12-13-Liberate-Your-Nook-Ebooks/) and are intended to fill in the gaps and/or clarify some steps for total noobs like myself. I am not smart enough to figure these out and all thanks and appreciation go to him. 

I’ve rearranged the order of operations from Aric to put the most difficult (IMO) steps in the beginning. This way, if you fail there is no sense in doing the other easier steps.

1. Install Android Studio and Android Studio SDK.
1. Spin up a virtual android device
1. Check if you have root access!
1. Install B&N eReader app on the virtual device
1. Log-in to the eReader and download your desired book
1. Go into root shell and pull out the book and encryption hash
1. Install Calibre
1. Install DeDRM plug-in for Calibre
1. Remove DRM from book

### Step 1. Install Android Studio and Android Studio SDK

1. [Download](https://developer.android.com/studio/) the .zip package for Android Studio for Linux. Save it somewhere like in your Downloads directory. At the time of writing I downloaded version 201.7042882.

![Android Studio Download at <https://developer.android.com/studio/>](/images/2021/DRM_01.png)

2. It’s generally a good practice to verify the checksum. Scroll down to the download area to find it. This was mine.

![Android Studio SHA-256 Checksum](/images/2021/DRM_02.png)

I usually just look at the last 4-5 digits to verify.

   - cd into directory where the download is
   - `sha256sum <filename>`
   - Compare the last 4-5 digits against the sha provided from the source

![](/images/2021/DRM_03.png)

3. You can follow the installation instructions here -- https://developer.android.com/studio/install#Linux.

 - Copy the .tar.gz file into `/usr/local/` directory
 - Uncompress it

Remember you will need root access to be messing around in `/usr/local`
![](/images/2021/DRM_04.png)

4. If you are running 64-bit linux you will need to install some 32-bit libraries to make it work well. At least according to the instructions.
```bash
sudo apt-get install libc6:i386 libncurses5:i386 libstdc++6:i386 lib32z1 libbz2-1.0:i386
```
5. How do you know if you are running 64-bit? Type `uname -i` and if you see `x86-64` you are.

6. Let's fire it up! Within `/usr/local/andoroid-studio/bin` type `./studio.sh`.

![](/images/2021/DRM_05.png)

### Step 2. Spin up a virtual android device

1. Start AVD Manager by clicking on the little phone icon in the far upper-right.

![Starting AVD Manager from Within Android Studio](/images/2021/DRM_06.png)

2. Next you need a virtual device. I am a total idiot in this area. I have no experience with Android development. But I can tell you that you must have root-access on the device you create, and in order to allow root access you must not use a Google Play image. So what does that mean? That means do not use an image that has the Google Play icon, like this **(A)**.
![Selecting an Image Withing AVD Manager](/images/2021/DRM_07.png)

- You can create your own image as others have done, but I just used the `Pixel_3a_API_30_x86` that was present when I started AVD Manager. 
- To start the image click the green "play" triangle **(B)**.

Warning
 ~ You a need a virtual device that is <u>not</u> a Google Play image so that you can access root. This is critical.

A phone will appear and after a couple seconds it will "boot up". Here's mine:

![Example of Virtual Device](/images/2021/DRM_08.png)

### Step 3. Check if you have root access
If you don't have root access you did something wrong and there is no point in continuing, so let's check now.

1. Open a terminal screen. When you install Android Studio it created an "Android" directory in your home folder. Go there and browser in `~/Android/Sdk/platform-tools`.

2. You should see a command named `adb`.

3. Type `./adb root` and you should see something like `restarting adbd as root`. If you do not, you are screwed. Go back and figure out what you did wrong. The primary source is that whole “Not a Google Play image thing”.

4. Now we can get inside the virtual device, `./adb shell`. Your cursor should be a pound "#" indicating you are root. 

5. If you are not root, you can't go into all the necessary directory folders.
![](/images/2021/DRM_15.png)

6. Keep this terminal around, we’ll need it later. 

### Step 4. Install B&N eReader app on the virtual device

1. In your virtual device, open the Chrome browser. Just navigate with your mouse.

2. Copy this url `https://apkpure.com/nook-read-ebooks-magazines/bn.ereader/versions` and paste into the Chrome browser.
- Left-click in the web address box
- Press and hold left mouse for a second, just like you would do on your mobile phone
- Select the "paste" option
- Press enter on your desktop keyboard
3. Download an .apk of the B&N Reader app 
- Again honesty... I don't know the difference of capabilities or restrictions between all the different versions. I can say that I used `V5.2.0.18` so you might as well do the same.

![Example Showing Downloading B&N Reader App .apk](/images/2021/DRM_09.png)

- I got an ad, so close the ad by pressing the "X". You typically get warnings when installing apps outside of the Google Store, so you will probably get a warning like this:

![Warning](/images/2021/DRM_10.png)

4. Install the app

![Install Confirmation](/images/2021/DRM_11.png)

5. Open the app

### Step 5. Log-in to the eReader and download your desired book

1. It will ask you to log-in with your username/password. I didn't take any screenshots of these steps, but it is typical app setup junk.

2. Then it may ask for credit card information if you don't already have it setup in your B&N account --like I did not. you can skip this section by scrolling to the bottom of the screen and hitting "Skip". 

3. Give it a minute and it will download your library. Somewhere it said you can also pull-up on the screen to make it refresh. Here's what mine looked like. It included the purchased book (upper left) and 5 other free books that I didn't ask for.

![B&N Reader Library](/images/2021/DRM_12.png)

4. Now you have to download your book onto your virtual device. I used this method on two different books, and for both all I did was open the book inside the device and the app downloaded the entire `.epub`. I instructions on the web about selecting `Download to SD Card` in the settings and such, but I didn't have to do that. 
5. OK, we should be done with the virtual device now. But don't shut it down because we need to go inside the shell and pull out the book you just downloaded.

### Step 6. Go into root shell and pull out the book and encryption hash

1. Go back to the terminal screen where you opened the shell.

2. Now we need to find that book file we just downloaded. I noticed that the B&N books appear to be in the format `123456789_epub.Vx.epub`. That is, a bunch of numbers +\ _epub+a version number+.epub extension.

3. I found the downloaded book at `/data/data/bn.ereader/files/B&N Downloads/Books/
![](/images/2021/DRM_17.png)
4. To copy a file from the virtual device to desktop you have to "pull" it from **back in the terminal** - not the ADB shell.
- Since you are not in the shell you cannot tab-complete the file dirs/filename so I suggest copy/pasting the directory path and then copy/pasting the file name.
- Also, don't forget to escape the "&" and space characters.
- See the example below.
![](/images/2021/DRM_18.png)
5. Now let's go get the encryption hash. It's stored in a database named `cchashdata.db`. Search for it inside the ADB shell but I would bet it's in the same directory as where you found the file = `./adb pull /data/data/bn.ereader/databases/cchashdata.db`
- Files will be in the current directory
![](/images/2021/DRM_19.png)
6. Find the hash and save it as a keyfile. 
- Type `sqlite3 cchashdata.db`
- This opens a SQLite shell. Look inside a specific table, type `select hash from cc_hash_data;`
- You'll get a HashKey, copy that INCLUDING THE TRAILING EQUAL SIGN. Here's mine, don't worry, I obfuscated a bunch of the letters.

![](/images/2021/DRM_20.png)

- Open a new file and save it somewhere with a .b64 extension, like "MyHash.b64".
- To exit the SQLite shell press Ctrl-Z
- To exit the ADB shell press Crtl-D

7. Note that this hashkey is good for all books purchased at B&N by that account. 

### Step 7. Install Calibre

1. This is pretty straight forward so I'll breeze through this part. I installed version 4.99.4 as that what was in the repository. At the time of writing version 5.10.1 was [available as binary install](https://calibre-ebook.com/download_linux). 
- Note that the plugin DeDRM version must be applicable for the correct Calibre version. DeDRM v6.x = Calibre 4.x and DeDRM v7.x = Calibre 5.x so keep that in mind and look at the release notes. 
2. Install the latest directly from Calibre
```{.bash}
sudo apt update && sudo apt install -y libfontconfig libgl1-mesa-glx
bash sudo -v && wget -nv -O- https://download.calibre-ebook.com/linux-installer.sh | sudo sh /dev/stdin
```
3. Install via Software Center

4. Install via apt-get
```{.bash}
sudo apt-get update
sudo apt-get install calibre
```
5. I chose the first option and installed directly from Calibre. So if you do something other than that "buyer beware".

### Step 8. Install DeDRM plug-in for Calibre

1. Open browser and go to DeDRM github page at <https://github.com/apprenticeharper/DeDRM_tools/releases>

2. Download the plugin .zip package. I chose to download the latest pre-release at the time (v7.0.3).  Just click the zip link in the "Assets" subsection and save. 
- You need the zip package named "DeDRM_plugin_x.x.x.zip"
- It appears that if you download a pre-release it is just that plugin folder, so don't unzip it.
- If you download a normal release it will say something like "DeDRM_tools_x.x.x.zip". In which case unzip it and you will find the "DeDRM_plugin.zip" inside
3. Open Calibre

4. Click the Preference icon in the tool bar

5. Scroll down to Advanced and click Plugins

![Plugin Location Within Calibre](/images/2021/DRM_21.png)

6. Click "Load plugin from file"

7. Select the `DeDRM_plugin.zip` zip package you saved. Click yes and ok to everything.

8. Back in the Plugins dialog, select "File type" and select "DeDRM", click the button "Customize plugin"

![DeDREM Plugin Customization Location](/images/2021/DRM_23.png)

9. Select "Barnes and Noble ebooks"

10. Select "Import Existing Keyfiles"

11. Browse and select the keyfile you saved, something like `MyHash.b64`

![Importing KeyFile](/images/2021/DRM_24.png)

### Step 9. Remove DRM from book

1. At this point, you can add your book that you downloaded from the virtual device. Remember in my example it was `9780133489293_epub.v2.epub`. WIihin Calibre select "Add books"

2. Browse to and select the epub. Select OK

3. If you double-click the book to open it and get an error message stating you have DRM then something went wrong.

4. You can go post a message to the [DeDRM forum](https://github.com/apprenticeharper/DeDRM_tools/issues/814) along with a log file. They are very helpful. [Here are instructions](https://github.com/apprenticeharper/DeDRM_tools/blob/master/FAQs.md#i-cannot-solve-my-problem-with-the-dedrm-plugin-and-now-i-need-to-post-a-log-how-do-i-do-that) on creating a debug log. 

### Conclusion

The goal of this post was to add clarifications to [Aric Renzo's excellent instructions](https://www.aricrenzo.com/2019-12-13-Liberate-Your-Nook-Ebooks/) based on what worked for me. These instructions are not for Windows but could probably be adjusted to make work. I'm positive there are other methods of making this work. If there are any mistakes to these instructions feel free to hit me up on twitter.