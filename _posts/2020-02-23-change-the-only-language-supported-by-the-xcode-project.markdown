---
layout: post
title: Change the Only Language Supported by the Xcode Project
---

Have you ever saw some apps that do not support English but declare they are supporting Engish on the product page of the App Store?

The most probable reason I think is that they wanted to change the only language supported, but didn't know how to do, so they just added the language they wanted to support as a new supported language.

Let's see what we should do to change the only language supported.

## Background

So what's the default setting of localization when we create a brand new Xcode project?

- **Base** with All localized files
- **English (Development Language)** with no localized file

We can confirm it in the project navigator by selecting the project (not a target) and click Info.

## Problem

You may notice that we cannot delete **English** even we add another language first. And since it's the **Development Language**, all localized files of **Base** will finally belong to **English**. That's why some apps that do not support English but declare they are supporting Engish on the product page of the App Store.

## An Inappropriate Solution

Let's do not use **Base** and move all localized files to the language we want since **Base** is the reason.

It works since there are no localized files for **English** in the project.

But the **English (Development Language)** is still there. So this may solve the issue on the product page of the App Store, but it still is an inappropriate solution.

## Solution

Just change the **Development Language** to the language we want.

Open `project.pbxproj` file in our project's `.xcodeproj` package, search for **region**.

``` text
developmentRegion = en;
knownRegions = (
    en,
    Base,
);
```

Then change the `en` (both in `developmentRegion` and `knownRegions`) to the language code[^1] we want.

``` text
developmentRegion = ja;
knownRegions = (
    ja,
    Base,
);
```

That's it, check the info tag again, the only language supported by the Xcode project has been changed to the language we want.

### XcodeGen

XcodeGen uses the `.lproj` folders to determine `knownRegions`, so all we need to do is adding one line to your `options`.

``` yml
options:
  developmentLanguage: ja
```

***

[^1]: If you don't know the code of the language you want, you can add the language from the info tab then the code will appear in `knownRegions`.
