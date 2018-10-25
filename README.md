# Introduction
I got myself a cheap CCTV system from eBay (also widely available on Amazon), it came by the name of "Floureon", but from what I'm aware of, it's one of those chinese systems that has many different brands but pretty much run the same operating system and GUI. One of the main features of the system is the ability to use XMEye for viewing the cameras remotely. So if your DVR supports XMEye, or has [this interface](https://4.bp.blogspot.com/-08BoIYMlnZo/VvtKIsdqzgI/AAAAAAAABQ8/IwZcXdd5BpMIDI8Dxqqj7bk3CkOMq1hag/s1600/20160330_101201.jpg), then it should be supported.

Overall they run some sort of Linux OS, some DVR's come with features enabled like SSH, but the Floureon one I had didn't seem to have SSH enabled. I wanted to explore the device and try to make modifications to it, pretty much a Custom Firmware. I could possibily change the colours of menus, change the boot logo, and perhaps enable SSH.

In this, I'll be showing you how to grab the firmware, make modifications to it (in this case, change the boot logo), repackage it up and update it onto the DVR. (It took me about 2/3 days of work to figure it all out)

**Requirements:**
* 7Zip (so you'll need Windows...)
* Linux (I used Linux Subsystem for Windows)


# 1. Finding The Correct Firmware
As I mentioned above, they can all have different brand names but they end up using the same interface and operating system, so all you need is the "model" (i guess) number which can be found in **Info > Version**

Take note of the 8 digit number after the R11 on "System", mine was **00000117**, now using [this site](https://www.burglaryalarmsystem.com/technology-news/china-dvr-nvr-firmware-download.html), find the 8 digit number and download the corrosponding firwmare.

It'll take you to a chinese site, and then you'll have your hands on a .bin file

# 2. Verifying The Correct Firmware
Before making any modifcations to it, the best thing to do is run the firmware on the DVR to make sure it takes it all fine.

You can do this in a few ways, such as using a USB (put the .bin on the root) and the menu interface, or by using [this tool](https://www.unifore.net/analog-surveillance/ip-camera-dvr-nvr-tool-download-device-manager.html) to push the firmware to the DVR over ethernet.

Just make sure the firmware runs through fine, so we can verify that it's the correct update.

# 3. Start extracting the firmware
Using 7Zip, open up the .bin

You'll find 4 **.cramfs.img** and an **InstallDesc**, I'll be modifying the boot logo, therefore extract the **logo-x.cramfs.img** file.

Make a folder somewhere, and again using 7Zip, extract the contents of the .img into there. You should end up with */folderyoucreated/h264dvr.jpg* (If you're also modifying the boot logo)

# 4. Modifying the firmware
Now that you have extracted all the files from that specific image, modify them how you like. The boot logo is 800x800 (iirc) so use photoshop to make your own boot logo and overwrite the image you extracted with your own. (Keep in mind to keep the file size low, I never really looked into how big the files can be, compress the jpg if needed)

Once done modifying the boot logo, you should still have the image in */folderyoucreated/h264dvr.jpg*

# 5. Repackaging the firmware
When repacking the .img files, you have to be sure that you're doing it perfectly, as if the image isn't built right, the DVR won't take the update.

This is where you'll need to run Linux, as I mentioned in the intro, I used (Ubuntu) Linux Subsystem for Windows 10.

Firstly, change directory to the location of your */folderyoucreated/*
    cd /blah/blah/folderyoucreated

Now we are going to make a cramfs image out of the folder's contents
    -USAGE-
    mkfs.cramfs -v [folderwithcontents] [nameofimagetomake]
    
    -EXAMPLE-
    mkfs.cramfs -v folderyoucreated/ new_image.img
    
We now have an image called new_image.img, but we now need to use uboot-tools (install with sudo apt-get install u-boot-tools) to create a new image that will work on the DVR

    -USAGE-
    mkimage -A ARM -O linux -C gzip -n linux -a 0x00E80000 -e 0x00EC0000 -d [originalimage] [nameofimagetomake]
    
    -EXAMPLE-
    mkimage -A ARM -O linux -C gzip -n linux -a 0x00E80000 -e 0x00EC0000 -d new_image.img new_image2.img
   
We should now have a finalised image called **new_image2.img** which you can rename back to whichever original image you modified, therefore I can change the name of this image to **logo-x.cramfs.img**

With this finished .img, you can get back onto 7Zip to open the original .bin firmware, and overwrite the old **logo-x.cramfs.img** with your new modified image.

# 5.5 Extra Note

By default, the firmware includes logo-x.cramfs.img but it doesn't actually update it to the DVR, as the InstallDesc doesn't tell it to update. You need to add the line

    {
      "Command": "Burn",
      "FileName": "logo-x.cramfs.img"
    }
Within the InstallDesc under **UpgradeCommand** so the DVR will update the logo. Again this is only needed if you are updating the logo, and doesn't matter for the others (user, romfs, custom) as the InstallDesc already tells the DVR to update those.

# 6. Push Firmware to DVR
Now you can get your new .bin firmware, and push this to the DVR and if you repackaged it correctly, it will update successfully!
