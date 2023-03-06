---
title: "Hiding in Plain Sight with Steganography"
date: 2018-10-30
image: code.png
categories:
    - Technology
tags:
    - Linux
    - Steganography
---

*This article was originally written by me for UMass IT Techbytes Blog. Since then the blog has been discontinued. This article is republished here for archival purposes, the original can still be found [here](https://blogs.umass.edu/Techbytes/2018/10/30/hiding-in-plain-sight-with-steganography/).*

Steganography is the process of hiding one file inside another, most popularly, hiding a file within a picture. If you’re a fan of Mr. Robot you are likely already somewhat familiar with this.

Although hiding files inside pictures may seem hard, it is actually rather easy. All files at their core are just text, so to hide one file into another it is just a case of inserting the text value of one file into another.

Even though this possible on all platforms, it is easiest to accomplish on Linux (although the following commands will probably work on Mac OS as well).

There are many different ways to hide different types of files, however the easiest and most versatile method is to use zip archives.

Once you create your own zip archive we can then append it to the end of an image file, such as a png.

```bash
cat deathstarplans.zip >> r2d2.png
```

If you’re wondering what just happened, let me explain. `cat` prints out a file as text (`deathstarplans.zip` in this instance). Instead of printing to the terminal, `>>` tells your terminal to appends the text to the end of the specified file -> `r2d2.png`.

We could have also just done `>` however that would replace the text of the specified file, specifically the metadata of `r2d2.png` in this instance. This does work and it would still allow you to view the image… BUT `r2d2.png` would be easily recognized as containing a zip file and defeat the entire purpose.

Getting the file(s) out is also easy, simply run unzip `r2d2.png`. Unzip will throw a warning that “x extra bytes” are before the zip file, which you can ignore, basically just restates that we hid the zip in the png file. And so they files pop out.

So why zip? Tar tends to be more popular on Linux… however tar has a problem with this method. Tar does not parse through the file and get to the actual start of the archive whereas zip does so automatically. That isn’t to say its impossible to get tar to work, it simply would require some extra work (aka scripting). However there is another, more advanced way, `steghide`.

![](crypt.jpg)

Unlike zip, `steghide` does not come preinstalled on most Linux distros, but is in most default repositories, including for Arch and Ubuntu/Linux Mint.

*Install on Arch Linux*
```bash
sudo pacman -S steghide
```

*Install on Ubuntu/Linux Mint*
```bash
sudo apt install steghide
```

`steghide` does have its ups and downs. One upside is that it is a lot better at hiding and can easily hide any file type. It does so by using an advanced algorithm to hide it within the image (or audio) file without changing the look (or sound) of the file. This also means that without using `steghide` (or at least the same mathematical approach as `steghide`) it is very difficult to extract the hidden files from the image.

However there is big draw back: `steghide` only supports a limited amount of ‘cover’ files – `JPEG`, `BMP`, `WAV`, and `AU`. But since `JPEG` files are a common image type, it isn’t a large draw back and will not look out of place.

To hide the file the command would be:

```bash
steghide embed -cf clones.jpg -ef order66.pdf
```

At which point `steghide` will prompt you to enter a password. Keep in mind that if you lose the password you will likely never recover the embedded file.

To extract the file we can run:

```bash
steghide extract -sf clones.jpg
```

Assuming we use the correct password, the hidden file is revealed.

All that being said, both methods leave the ‘secret’ file untouched and only hide a copy. Assuming the goal is to hide the file, the files in the open need to be securely removed. shred is a good command which overwrites the file multiple times to make it as difficult to recover as possible.

```bash
shred -z order66.pdf
```

or to delete it automatically

```bash
shred -zu order66.pdf
```