---
layout: post
title: Environment Variables for Xcode Build Configuration
---

It's a frequent need to change some values according to the build configuration—for example, the base URL.

We can fulfill this need by using the preprocessor macro as below.

``` swift
#if DEBUG
let baseURLString = "https://example.com/debug/"
#else
let baseURLString = "https://example.com/"
#endif
```

It works, but we have to write conditional compilation blocks and may have to set a preprocessor macro for each build configuration.

Let's talk about another way to fulfill this need.

## Add Build Settings

If you ever set the preprocessor macro to the build configuration, you may already notice that we can set different build settings for different build configurations.

So why not we just set our environment variables to the build settings?

![Build Settings](/assets/images/environment-variables-build-settings.png)

But how can we retrieve the value?

## Set the Value to the Information Property List File

By default, Xcode created an information property list file[^1] for us. We can see there are already some environment variables here, such as `$(PRODUCT_BUNDLE_IDENTIFIER)` which is a build setting of the default target.

Let's add a property here with the value we defined in the build settings.

``` xml
<key>BaseURL</key>
<string>$(BASE_URL)</string>
```

Xcode will set the value according to the build configuration.

### Debug

``` xml
<key>BaseURL</key>
<string>https://example.com/debug/</string>
```

### Release

``` xml
<key>BaseURL</key>
<string>https://example.com/</string>
```

## Retrieve the Value From the Information Property List File

``` swift
let baseURLString = Bundle.main.object(forInfoDictionaryKey: "BaseURL") as! String
```

It's the same as we get the version string.

With this approach, we don't have to write conditional compilation blocks anymore.

## Improvement

### Configuration Settings File

> This type of file can be edited outside of Xcode and integrates well with source control systems.

As the official description for the configuration settings (`.xcconfig`) file above, it may be a better place to set our environment variables rather than the `project.pbxproj` file—the actual place when we set them in the build settings.

Here is the [official document](https://help.apple.com/xcode/mac/current/#/deve97bde215) about adding a configuration settings file.

### Type-Safety

As we can see, the final step to get the environment variables has to use force casting. It's not type-safe.

#### Code Generation

We can improve this issue by code generation.

Here is an example using [SwiftGen](https://github.com/SwiftGen/SwiftGen).

Add below to the `swiftgen.yml` file.

``` yaml
plist:
  inputs: Info.plist
  outputs:
    templateName: runtime-swift4
    output: Info.swift
    params:
      enumName: Info
```

Then we can retrieve value without any human error.

``` swift
let baseURLString = Info.baseURL
```

You may notice that we are not entirely type-safe since we want a URL, but there is a String.

#### Decode the Property List File

Decoding the property list file may be the final answer for us to be type-safe.

Let's update our information property list file as below.

``` xml
<!-- To separate our environment variables from other properties -->
<key>Environment</key>
<dict>
    <key>BaseURL</key>
    <!-- The format that can be decoded as a URL according to the impelementation of URL -->
    <!-- https://github.com/apple/swift/blob/master/stdlib/public/Darwin/Foundation/URL.swift -->
    <dict>
        <key>relative</key>
        <string>$(BASE_URL)</string>
    </dict>
</dict>
```

Then create types for decoding.

``` swift
struct EnvironmentHolder: Decodable {
    let environment: Environment

    enum CodingKeys: String, CodingKey {
        case environment = "Environment"
    }
}

struct Environment: Decodable {
    let baseURL: URL

    enum CodingKeys: String, CodingKey {
        case baseURL = "BaseURL"
    }
}
```

Finally, decode our environment variables from the information property list file.

``` swift
final class EnvironmentDecoder {
    static func decode() -> Environment {
        let url = Bundle.main.url(forResource: "Info", withExtension: "plist")!
        let data = try! Data(contentsOf: url)
        let decoder = PropertyListDecoder()
        let environmentHolder = try! decoder.decode(EnvironmentHolder.self, from: data)
        return environmentHolder.environment
    }
}
```

With this approach, we can format our environment variables as we want and add custom validation thanks to the `Decodable` protocol. Although it's a little inconvenience than code generation.

It's right that we are still using some force operations, but we can simply add a unit test to ensure successful decoding.

## Summary

1. Add user-defined build settings
2. Set the value to the information property list file
3. Retrieve the value from the information property list file
4. Impove as needed

## References

- [Using Xcode Configuration (.xcconfig) to Manage Different Build Settings](https://www.appcoda.com/xcconfig-guide/)
- [Dev/Staging/Prod Configs in Xcode](https://medium.com/better-programming/how-to-create-development-staging-and-production-configs-in-xcode-ec58b2cc1df4)
- [Answer: Swift 4 Codable decode URL from String](https://stackoverflow.com/a/47918656?stw=2)

***

[^1]: Any other property list file will not work since Xcode will only replace the value according to the build configuration for the information property list file. The default file is located in the target folder and named `Info.plist`. If you have changed the file, you can find the path in `Info.plist file` from your the target's build settings.
