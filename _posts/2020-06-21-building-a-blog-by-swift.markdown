---
layout: post
title: Building a Blog by Swift
---

Nowadays, we are not only able to build iOS and macOS applications but also to build web applications, command-line tools. But what about a blog?

Thanks to [Publish](https://github.com/JohnSundell/Publish)–a static site generator for Swift developers, we indeed can build a blog by Swift.

## Preparation

Install Publish's command-line tool.

```sh
$ git clone https://github.com/JohnSundell/Publish.git
$ cd Publish
$ make
```

Create a website project.

```sh
$ mkdir MyBlog
$ cd MyBlog
$ publish new
```

That's it. We can use this project as our blog by adding markdown files in the posts folder.

Run `Publish run` and access to http://localhost:8000 to see the result.

If your blog does not contain any code, you may skip the next section. But if your blog is about development like me, we have to support syntax highlighting for our blog.

## Syntax Highlighting

There are three plugins for Publish to support syntax highlighting:

- [Splash](https://github.com/JohnSundell/Publish/blob/master/Documentation/HowTo/SyntaxHighlighting/using-splash.md)–a native Swift syntax highlighter for Swift code
- [Pygments](https://github.com/JohnSundell/Publish/blob/master/Documentation/HowTo/SyntaxHighlighting/using-pygments.md)–a Python tool with support for over 500 languages
- [highlight.js](https://github.com/JohnSundell/Publish/blob/master/Documentation/HowTo/SyntaxHighlighting/using-highlight-js.md)–a JavaScript tool with support for over 180 languages

You can choose what you like, but the setup is almost the same for all plugins.

Here is the example to set up the highlight.js plugin.

### Using Plugin

Since our blog is a Swift package, we can add dependencies by editing `Package.swift` as usual.

```swift
let package = Package(
    ...

    dependencies: [
        .package(url: "https://github.com/alex-ross/highlightjspublishplugin", from: "1.0.0")
    ],
    targets: [
        .target(
            ...
            dependencies: [
                ...
                "HighlightJSPublishPlugin"
            ]
        )
    ]
    ...
)
```

Set the plugin when `publish`.

```swift
try MyBlog().publish(
    withTheme: .foundation,
    plugins: [.highlightJS()]
)
```

### Color Scheme

To add a color scheme, we have to define our theme to include custom CSS files since the default theme only contains one CSS file.

```swift
static func head<T: Website>(
    ...
    stylesheetPaths: [Path] = ["/styles.css"],
    ...
)
```

And the file is copied from Publish. If we add a `styles.css` file in the `Resources` fold, it will cause a runtime error since the `styles.css` is duplicated.

```swift
static var foundation: Self {
    Theme(
        htmlFactory: FoundationHTMLFactory(),
        resourcePaths: ["Resources/FoundationTheme/styles.css"]
    )
}
```

And the `FoundationHTMLFactory` is `private`, we cannot create our theme by `FoundationHTMLFactory`.

So if you are not familiar with front-end like me, the simplest way may be copy `FoundationHTMLFactory` to our code and modify it as we needed.

With our own `HTMLFactory` we can add any CSS files we want.

```swift
try MyBlog().publish(
    withTheme: .myTheme,
    plugins: [.highlightJS()]
)
```

We have to define styles according to the plugin we are using. Here are some [themes](https://github.com/highlightjs/highlight.js/tree/master/src/styles) for the highlight.js plugin.

It may be better to choose two themes since the foundation theme is supporting dark mode.

## Deployment

Since Publish is a static site generator, it's not hard to deploy it manually or create a custom deployment tool. But the simplest way to deploy a site developed by Publish is by using GitHub Pages.

### Github Pages

> GitHub Pages is a static site hosting service that takes HTML, CSS, and JavaScript files straight from a repository on GitHub

As the description above, it's suitable to host our blog. All we need to do is push everything in the `Output` folder to according to this [tutorial](https://pages.github.com/).

Fortunately, there is a `DeploymentMethod` that pushes everything in the `Output` folder to a repository for us.

```swift
try MyBlog().publish(
    withTheme: .myTheme,
    deployedUsing: .gitHub("user-name/repository-name", useSSH: true),
    plugins: [.highlightJS()]
)
```

When we are ready to deploy, just run `publish deploy`.

## Where to Go from Here

Today we learned how to build a blog by Publish. But Publish is not only for building a blog but for any static site, so we can try to extend our blog by adding some pages such as an about page.

Along with the development of our blog, we may find some common use cases. It's a great way to give back to the community by extracting them to plugins. Remember to add the `publish-plugin` [topic](https://help.github.com/en/github/administering-a-repository/classifying-your-repository-with-topics#adding-topics-to-your-repository).
