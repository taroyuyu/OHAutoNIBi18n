# What's this

This class allows you to **automate the internationalisation** (i18n) of your interface (XIB files) **without any additional code in your application**. No more using multiple XIBs for various locales or having outlets just to translate the text of a label!

Simply use the keys of your Localizable.strings in the text of any element in your XIB
(title of an `UIButton`, text of a `UILabel`, …), and it will be automatically translated on the fly at runtime!

* You don't need to have one XIB file by language/locale any more!
* You don't need to have an `IBOutlet` to all of your XIB objects just to translate their text manually by code with `NSLocalizedString`!

> Note: strings starting with a digit won't be translated. This is a feature used to avoid useless translation of static numbers you put in your XIB (especially because you will probably change this "0.00" string in your XIB with some real value by code later)


# Installation

To use this, simply **add `OHAutoNIBi18n.h` and `OHAutoNIBi18n.m` in your project** and… that's all! `OHAutoNIBi18n` will be loaded automatically at runtime and translate your XIB on the fly without any additional line of code.

If you also want to use the `_T()`, `_Tf()` and `_Tfn()` macros, you can also add and `#import "OHL10nMacros.h"`.

Alternatively, you can install all of it using CocoaPods. Simply add `pod "OHAutoNIBi18n"` to your `Podfile`.


# Debugging your unlocalized strings

You can `#define OHAutoNIBi18n_DEBUG 1` if you want `OHAutoNIBi18n` to warn every time it encounters a string for which it does not find any translation in your `Localizable.strings` file. In that case, it will log a message in your console and add the `$` character around the text in your XIB at runtime so you can easily see it.

> Note that you can use strings starting with a "." in your XIB (like ".Lorem ipsum dolor sit amet" to avoid warnings on those strings when `OHAutoNIBi18n_DEBUG` is set. This is useful for strings you use only as "layout helpers" (dummy strings to help you see your label in your XIB, which is easier than an empty label to help you position it), like for a `UILabel` in a custom `UITableViewCell` for which you don't want those warnings, knowing that you will override its text by code anyway.


# How it works

This class uses method swizzling to intercept the calls to `awakeFromNib` and automatically localize any object created/extracted from a XIB file by the runtime. The method swizzling is done automatically when your application is loaded in memory, so you don't even have to add code to install this: the only presence of the OHAutoNIBi18n.m file in your project makes everything magic!


### Comparison with Xcode 4.5's "Base Localization"

Starting with Xcode 4.5, you can use the "Base Localisation" (see [Apple Tutorial: Internationalize your application](https://developer.apple.com/library/ios/referencelibrary/GettingStarted/RoadMapiOS/chapters/InternationalizeYourApp/InternationalizeYourApp/InternationalizeYourApp.html)).

But this process requires that you create a `*.strings` file for each of your XIB files, with the same name as the XIB file, and that you rely on Interface Builder's "Object IDs" which are fare from descriptive and human-readable.

This can become quite a pain:

* If you have a lot of XIB files, you will have to create as many `.strings` files, resulting in a lot of files
* With Base Localization, your `.strings` files will use abstract Object IDs, which are far from convenient to identify objects of your XIB (not even mentioning when you need to provide those `Localizable.strings` files to the person you pay to translate your application, how will s/he understand how to match those IDs with the actual UI?)
* This is also problematic for every generic term in your application that is used across multiple XIB, like some vocabulary or strings specific to your application domain and used in all of your XIB files. In that case, Base Localization requires that you repeat this term in every of your `.strings` files, whereas with `OHAutoNIBi18n` you can just translate it once in your `Localizable.strings`.

For all these reasons, Base Localization is just not such a great solution after all, whereas `OHAutoNIBi18n` does not have those quirks.

### Using it in a framework

If you have a framework containing storyboards and XIBs that needs to be auto-localized using `OHAutoNIBi18n`, you can embed `OHAutoNIBi18n` in your framework, alongside your storyboards, XIBs, and `Localizable.strings` files.

In that special case, because your localized strings won't be located in the main bundle of your app using the framework, but will be in the `Localizable.strings` file inside your framework bundle, you'll have to tell `OHAutoNIBi18n` to use your framework bundle instead of the main bundle. To do that, you'll have to call the following method as early as possible in your framework's lifecycle, especially before any of your framework's storyboard or XIB is loaded:

```objc
// Objective-C
NSBundle* fmkBundle = [NSBundle bundleForClass:SomeClassOfYourFmk.class];
[OHAutoNIBi18n setLocalizationBundle:fmkBundle tableName:nil];
```
```swift
// Swift
let fmkBundle = Bundle(for:SomeClassOfYourFmk.self)
OHAutoNIBi18n.setLocalizationBundle(fmkBundle, tableName:nil)
```

_Using `SomeClassOfYourFmk` as a class belonging to your framework to easily derive your framework bundle from it._

You can also set a different table to look up for localizations when auto-translating XIBs and Storyboards, providing a specific `tableName` here. If set to `nil`, it will use the default name `Localizable` instead.

# License

This code is distributed under the MIT License.
