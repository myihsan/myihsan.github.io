---
layout: post
title: Build a Command-Line Tool With Swift
---

With Swift Package Manager, we can build a command-line tool with Swift effortlessly.

Today, let's build a command-line tool to help us capitalize title from scratch.

## Creating an Executable Package

``` sh
mkdir TitleCapitalizer
cd TitleCapitalizer
swift package init --type executable
```

With the command above, Swift Package Manager will create all we need for a command-line tool including a `.gitignore` file.

We can build and run it right away.

``` sh
swift build # We can skip this command since `swift run` will do it for us
swift run
```

The output will be `Hello, world!` as the `main.swift` file created by Swift Package Manager.

``` swift
print("Hello, world!")
```

## Build the Title Capitalizer

Let's build our title capitalizer with APA style capitalization rules.

> 1. Capitalize the first word of the title/heading and of any subtitle/subheading;
> 2. Capitalize all “major” words (nouns, verbs, adjectives, adverbs, and pronouns) in the title/heading, including the second part of hyphenated major words (e.g., Self-Report not Self-report); and
> 3. Capitalize all words of four letters or more.

I am not sure if the rules above are enough, but let's build our title capitalizer according to the rules as an example.

``` swift
import NaturalLanguage

let text = "build a command-line tool with Swift"

var capitalized = ""

let tagger = NLTagger(tagSchemes: [.lexicalClass])
let tags: [NLTag] = [.noun, .verb, .adjective, .adverb, .pronoun]
tagger.string = text
tagger.enumerateTags(in: text.startIndex..<text.endIndex, unit: .word, scheme: .lexicalClass) { (tag, tokenRange) -> Bool in
    var token = String(text[tokenRange])
    if token.count >= 4 || tag.flatMap({ tags.contains($0) }) == true { // 3. or 2.
        // A string extension from
        // https://www.hackingwithswift.com/example-code/strings/how-to-capitalize-the-first-letter-of-a-string
        token.capitalizeFirstLetter()
    }
    capitalized.append(token)
    return true
}
capitalized.capitalizeFirstLetter() // 1.

print(capitalized)
```

Since `NaturalLanguage` is only available in macOS 10.14 or newer, we have to set the platforms in `Package.swift` file as blow.

``` swift
let package = Package(
    name: "TitleCapitalizer",
    platforms: [
        .macOS(.v10_14),
    ],
    ...
)
```

`swift run` again, we will get the result—"Build a Command-Line Tool With Swift".

## Get Input

Let's get the input instead of hard coding it.

``` swift
// The first arugment will be the path to the executable file
let text = CommandLine.arguments.dropFirst().first
```

We also have to validate there is a text. If there is not, print an error message and exit with a failure.

``` swift
guard let text = CommandLine.arguments.dropFirst().first else {
    print("No string to capitalize.")
    exit(EXIT_FAILURE)
}
```

Let's test it.

``` sh
$ swift run
No string to capitalize.

# We have run TitleCapitalizer specifically
# Otherwise `swift run` will try to run a executable product named "build a command-line tool with Swift"
$ swift run TitleCapitalizer "build a command-line tool with Swift"
Build a Command-Line Tool With Swift
```

It works great!

## Swift Argument Parser

Let's assume we want to support other capitalization styles, so we want to run our tool like below.

``` sh
$ swift run TitleCapitalizer "build a command-line tool with Swift"
Build a Command-Line Tool With Swift

$ swift run TitleCapitalizer --style apa "build a command-line tool with Swift"
Build a Command-Line Tool With Swift

$ swift run TitleCapitalizer -s chicago "build a command-line tool with Swift"
Build a Command-Line Tool with Swift
```

It's tedious to complete this need by getting arguments directly from `CommandLine.arguments`. But there is a package that can help us out—[Swift Argument Parser](https://github.com/apple/swift-argument-parser).

### Add Swift Argument Parser as a Dependency

Add Swift Argument Parser to the dependencies of our package in `Package.swift` file.

``` swift
let package = Package(
    ...
    dependencies: [
        .package(url: "https://github.com/apple/swift-argument-parser", from: "0.0.4"),
    ],
    ...
)
```

Also add it to the dependencies of our target, so that we can improve it as ArgumentParser in our source file.

``` swift
.target(
    name: "TitleCapitalizer",
    dependencies: [
        .product(name: "ArgumentParser", package: "swift-argument-parser"),
]),
```

### Refactor Using ArgumentParser

It's simple to use `ArgumentParser`. All we need to do is to implement `ParsableCommand` and wrap all arguments with corresponding property wrapper.

``` swift
import ArgumentParser
import NaturalLanguage

struct TitleCapitalizer: ParsableCommand {
    enum Style: String, ExpressibleByArgument {
        case apa, chicago
    }

    @Option(name: .shortAndLong, default: .apa, help: "The style used to capitalize.")
    var style: Style

    @Argument(help: "The text to capitalize.")
    var text: String

    func run() throws {
        // Capitalize the text here according to the style
    }
}

TitleCapitalizer.main()
```

With the code above, we are supporting the option to specify the style to capitalize.

And thanks to Swift Argument Parser, we got a help page automatically.

``` sh
$ swift run TitleCapitalizer --help
USAGE: title-capitalizer [--style <style>] <text>

ARGUMENTS:
  <text>                  The text to capitalize.

OPTIONS:
  -s, --style <style>     The style used to capitalize. (default: apa)
  -h, --help              Show help information.
```

## Where to Go From Here

### Use Everywhere

Run the following command to be able to run our command-line tool everywhere.

``` sh
export PATH=$PATH:/path/to/package/.build/debug
```

### Share by [Mint 🌱](https://github.com/yonaskolb/Mint)

Add an `executable` to the package's `products` list.

``` swift
let package = Package(
    ...
    products: [
        .executable(name: "TitleCapitalizer", targets: ["TitleCapitalizer"]),
    ],
    ...
)
```

Release the package to GitHub.

That's it. The user can install your command-line tool by:

``` sh
mint install github_name/repo_name
```

You may also consider adding your package to the `README.md` file of Mint.

## References

- [Usage - Swift Package Manager Project](https://github.com/apple/swift-package-manager/blob/master/Documentation/Usage.md#creating-an-executable-package)
- [Title Case and Sentence Case Capitalization in APA Style](https://blog.apastyle.org/apastyle/2012/03/title-case-and-sentence-case-capitalization-in-apa-style.html)
- [Swift Argument Parser](https://github.com/apple/swift-argument-parser)
- [Mint 🌱](https://github.com/yonaskolb/Mint)