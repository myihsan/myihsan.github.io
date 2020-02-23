---
layout: post
title: Xcode Version Management
---

We use Xcode to develop for Apple platforms and there are many versions (includes beta versions) of Xcode to choice for different purposes. So we need to manage the versions of Xcode and it's not that easy.

Let's take about how [Xcode::Install](https://github.com/xcpretty/xcode-install) can help us to manage the versions of Xcode.

## The Ways Without Xcode::Install

### App Store

It may be the most common place to download Xcode, but we cannot keep more than one version. So we have to turn off Automatic Updates for App Store to "manage" the version.

### [Downloads for Apple Developers](https://developer.apple.com/download/more/)

We can download the version we want from it and there is no need to turn off Automatic Updates for App Store. However, if we want to keep more than one version of Xcode, we need to change to the name of Xcode according to its version like `Xcode-11.3.1.app`.
The download may fail and need to retry from the beginning if we download by the browser, so we may rely on something like `wget --load-cookies` to download and resume the download from the point it failed.

## Xcode::Install

[Xcode::Install](https://github.com/xcpretty/xcode-install) use the way of  [Downloads for Apple Developers](https://developer.apple.com/download/more/) and handle all trivial things for us.

### Preparation

#### Install [Xcode::Install](https://github.com/xcpretty/xcode-install)

``` sh
gem install xcode-install
```

#### Setup Apple ID

[Xcode::Install](https://github.com/xcpretty/xcode-install) needs our Apple ID to access [Downloads for Apple Developers](https://developer.apple.com/download/more/).

Add our Apple ID to our shell configuration file.

``` sh
export XCODE_INSTALL_USER="name@example.com"
```

Then [Xcode::Install](https://github.com/xcpretty/xcode-install) will use this Apple ID when it needs without asking an Apple ID every time.

### Usage

#### List of Available Xcode Versions

``` sh
xcversion list
```

This list is cached. To update the list of available versions, run:

``` sh
xcversion update
```

#### Install Xcode

``` sh
xcversion install "11.3.1"
```

This line will install Xcode to `/Applications/Xcode-11.3.1.app`, link `/Applications/Xcode.app` to it and do `xcode-select --switch`.

We also can only do the installation with the option `—-no-switch`.

#### Command Line Tools for Xcode

Install Command Line Tools for Xcode

``` sh
xcversion install-cli-tools
```

Show the version of selected Command Line Tools for Xcode

``` sh
xcversion selected
```

Select the version of Command Line Tools for Xcode ( `xcode-select --switch` )

``` sh
xcversion select 11.3
```

Select the version of Command Line Tools for Xcode and link the `Xcode.app` to `Xcode-11.3.app`.

``` sh
xcversion select 11.3 —-symlink
```

#### Note

I have encountered some issues when I have an Xcode installed from the App Store. I thought it may be caused by the symlink, so I try to install with `--no-switch`, but it didn't work.
If you are in the same situation, trying to remove the Xcode installed from the App Store may help.

***

What do you think?

Thanks to [xcode-install](https://github.com/xcpretty/xcode-install), it's easy to manage the versions of Xcode. And we may also use it in some scripts or CI since it's a command-line tool.

## References

- [如何管理 Xcode 版本才不會害到自己跟團隊](https://13h.tw/2019/11/01/manage-xcode-versions.html)
