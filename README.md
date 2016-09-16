# AcceptanceMark

AcceptanceMark is a tool for generating Acceptance Tests in Xcode, inspired by [Fitnesse](http://fitnesse.org/).

[![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](http://mit-license.org)
[![Language](http://img.shields.io/badge/language-swift-orange.svg?style=flat)](https://developer.apple.com/swift)
[![Build](https://img.shields.io/travis/bizz84/AcceptanceMark.svg?style=flat)](https://travis-ci.org/bizz84/AcceptanceMark)
[![Issues](https://img.shields.io/github/issues/bizz84/AcceptanceMark.svg?style=flat)](https://github.com/bizz84/AcceptanceMark/issues)
[![Twitter](https://img.shields.io/badge/twitter-@biz84-blue.svg?maxAge=2592000)](http://twitter.com/biz84)

### Fitnesse advantages

* Easy to write business rules in tabular form in Markdown files.
* All shareholders can write Fitnesse tests.
* Convenient Test Report.

### Fitnesse disadvantages

* Does not integrate well with XCTest.
* Requires to run a separate server.
* Difficult to configure and run locally / on CI.

### The solution: AcceptanceMark

AcceptanceMark is the ideal tool to write Fitnesse-style acceptance tests that integrate seamlessly with XCTest:

* Write your tests inputs and expected values in markdown tables.
* AcceptanceMark generates XCTest test classes with **strong-typed input/outputs**. 
* Write test runners to evaluate the system under test with the given inputs.
* Run the chosen test target (Unit Tests supported, UI Tests **will** be supported) and get a test report.

## How does this work?

#### Write your own test sets, like so:

```
image-tests.md

## Image Loading

| name:String   || loaded:Bool  |
| ------------- || ------------ |
| available.png || true         |
| missing.png   || false        |
```

#### Run **amtool** manually or as an Xcode pre-compilation phase:

```
amtool -i image-tests.md
```

This generates an `XCTestCase` test class:

```swift
/*
 * File Auto-Generated by AcceptanceMark - DO NOT EDIT
 * input file: ImageTests.md
 * generated file: ImageTests_ImageLoadingTests.swift
 *
 * -- Test Specification -- 
 *
 * ## Image Loading
 * | name:String   || loaded:Bool  |
 * | ------------- || ------------ |
 * | available.png || true         |
 * | missing.png   || false        |
 */

//// Don't forget to create a test runner: 
//
//class ImageTests_ImageLoadingRunner: ImageTests_ImageLoadingRunnable {
//
//	func run(input: ImageTests_ImageLoadingInput) throws -> ImageTests_ImageLoadingOutput {
//		return ImageTests_ImageLoadingOutput(<#parameters#>)
//	}
//}

import XCTest

struct ImageTests_ImageLoadingInput {
	let name: String
}

struct ImageTests_ImageLoadingOutput: Equatable {
	let loaded: Bool
}

protocol ImageTests_ImageLoadingRunnable {
	func run(input: ImageTests_ImageLoadingInput) throws -> ImageTests_ImageLoadingOutput
}
class ImageTests_ImageLoadingTests: XCTestCase {

	var testRunner: ImageTests_ImageLoadingRunnable!

	override func setUp() {
		// MARK: Implement the ImageTests_ImageLoadingRunner() class!
		testRunner = ImageTests_ImageLoadingRunner()
	}

	func testImageLoading_row1() {
		let input = ImageTests_ImageLoadingInput(name: "available.png")
		let expected = ImageTests_ImageLoadingOutput(loaded: true)
		let result = try! testRunner.run(input: input)
		XCTAssertEqual(expected, result)
	}

	func testImageLoading_row2() {
		let input = ImageTests_ImageLoadingInput(name: "missing.png")
		let expected = ImageTests_ImageLoadingOutput(loaded: false)
		let result = try! testRunner.run(input: input)
		XCTAssertEqual(expected, result)
	}

}

func == (lhs: ImageTests_ImageLoadingOutput, rhs: ImageTests_ImageLoadingOutput) -> Bool {
	return
		lhs.loaded == rhs.loaded
}
```

#### Write your test runner:

```swift
// User generated file. Put your test runner implementation here.
class ImageTests_ImageLoadingTestRunner: ImageTests_ImageLoadingTestRunnable {

    func run(input: ImageTests_ImageLoadingInput) throws -> ImageTests_ImageLoadingResult {
        // Your business logic here
        return ImageTests_ImageLoadingResult(loaded: true)
    }
}
```

#### Add your generated test classes and test runners to your Xcode test target and run the tests.

### Notes

* Note the functional style of the test runner. It is simply a method that takes a stronly-typed input value, and returns a strongly-typed output value. **No state, no side effects**.

* `XCTestCase` sublasses can specify a `setUp()` method to configure an initial state that is shared across all unit tests. This is deliberately not supported with  **AcceptanceMark** test runners, and **state-less tests are preferred and encouraged** instead.

## Compilation / Installation

* AcceptanceMark includes **amtool**, a command line tool used to generate unit tests or UI tests.

* Xcode 8 is required as **amtool** is written in Swift 3. 

To compile **amtool**, clone this repo and run the script:

```
git clone https://github.com/bizz84/AcceptanceMark
cd AcceptanceMark
./scripts/build-amtool.sh
```

Once the build finishes, **amtool** can be found at this location:

```
./build/Release/amtool
```

For convenience, **amtool** can then be copied to a folder in your `$PATH`.

In the future, **amtool** may be distributed with a package manager of choice similar to **[rubygem](https://rubygems.org/)** or **[homebrew](http://brew.sh/)**. 

## amtool command line options

```
amtool -i <input-file.md> [-l swift2|swift3]
```

* Use `-i` to specify the input file 
* Use `-l` to specify the output language. Currently **Swift 2** and **Swift 3** are supported. **Objective-C** and other languages may be added in the future.

#### Not yet implemented
* Use `-v` to print the **amtool** version


## FAQ

* _**Q**: I want to have more than one table for each `.md` file. Is this possible?_
* **A**: Yes, as long as the file is structured as [ **Heading**, **Table**, **Heading**, **Table**, **...** ], **AcceptanceMark** will generate multiple swift test files named `<filename>_<heading>Tests.swift`. This way, each **test set** gets its own swift classes all in one file. Note that **heading names should be unique per-file**. Whitespaces and punctuation will be stripped from headings.

* _**Q**: I want to preload application data/state for each test in a table (this is done with builders in Fitnesse). Can I do that?_
* **A**: This is in the future roadmap. While the specification for this may change, one possible way of doing this is by allowing more than one table for each **heading**, with the convention that the **last** table represents the input/output set, while all previous tables represent data to be preloaded.
Until this is implemented, all preloading must be done directly in the test runner's `run()` method. Preloading example:

```
## Image Loading

// Preloaded data
| Country:String | Code:Bool |
| -------------- | --------- |
| United Kingdom | GB        |
| Italy          | IT        |
| Germany        | DE        |
 
// Test runner data
| name:String   || loaded:Bool  |
| ------------- || ------------ |
| available.png || true         |
| missing.png   || false        |
```

* _**Q**: I want to preload a JSON file for all tests running on a given table. Can I do that?_
* **A**: You could do that directly by adding the JSON loading code directly in the test runner's `run()` method. For extra configurability you could specify the JSON file name as an input parameter of your test set, and have your test runner load that file from the bundle.


## LICENSE

MIT License. See the [license file](LICENSE.md) for details.

