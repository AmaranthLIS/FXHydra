<p align="center"><img src="https://cloud.githubusercontent.com/assets/1567433/6684993/5971ef08-cc3a-11e4-984c-6769e4931497.png" height="100"/>

[![Cocoapods Compatible](https://img.shields.io/cocoapods/v/Nuke.svg)](https://img.shields.io/cocoapods/v/Nuke.svg)
[![Carthage Compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)

Advanced pure Swift framework for loading, caching, processing, displaying and preheating images. It uses latest advancements in iOS SDK and doesn't reinvent existing technologies.

Nuke is a [pipeline](#h_design) that loads images using multiple dependencies which can be injected in runtime.

```swift
Nuke.taskWithURL(URL) { 
    let image = $0.image 
}.resume()
```

1. [Getting Started](#h_getting_started)
2. [Usage](#h_usage)
3. [Design](#h_design)
4. [Installation](#installation)
5. [Requirements](#h_requirements)
6. [Contribution](#h_contribution)

## <a name="h_features"></a>Features

- Zero config, yet immense customization and flexibility
- Great performance even on outdated devices, asynchronous and thread safe
- Optional [Alamofire](https://github.com/Alamofire/Alamofire) integration
- Optional [FLAnimatedImage](https://github.com/Flipboard/FLAnimatedImage) integration

##### Loading
- Uses [NSURLSession](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSession_class/) with [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) support
- Uses a single data task for multiple equivalent requests
- [Intelligent preheating](https://github.com/kean/Nuke/wiki/Image-Preheating-Guide) of images close to the viewport
- Progress tracking using `NSProgress`

##### Caching
- Instead of reinventing a caching methodology it relies on HTTP cache as defined in [HTTP specification](https://tools.ietf.org/html/rfc7234) and caching implementation provided by [Foundation](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html)
- Caching is completely transparent to the client
- Two cache layers, including [top level memory cache](https://github.com/kean/Nuke/wiki/Image-Caching-Guide) for decompressed images

##### Decoding and Processing
- Apply image filters
- Background image decompression and scaling in a single step
- Scale large images (~6000x4000 px) and prepare them for display with ease
- Resize loaded images to [fit displayed size](https://developer.apple.com/library/ios/qa/qa1708/_index.html)

##### Advanced
- Image decoder composition
- Image filter composition
- Customize different parts of the framework using dependency injection

## <a name="h_getting_started"></a>Getting Started
- Download the latest [release](https://github.com/kean/Nuke/releases) version
- Experiment with Nuke APIs in a Swift playground
- Take a look at the demo project, it's easy to install with `pod try Nuke` command
- [Install using CocoaPods](#install_using_cocopods), import Nuke and enjoy!

## <a name="h_usage"></a>Usage

#### Zero Config

```swift
Nuke.taskWithURL(imageURL) {
    let image = $0.image
}.resume()
```

#### Adding Request Options

```swift
var request = ImageRequest(URL: imageURL)
request.targetSize = CGSize(width: 300.0, height: 400.0) // Set target size in pixels
request.contentMode = .AspectFill

Nuke.taskWithRequest(request) {
    let image = $0.image // Image is resized
}.resume()
```

#### Using Image Response

```swift
Nuke.taskWithRequest(request) { response in
    switch response { // Response is an enum with associated values
    case let .Success(image, info): 
        // Use image and inspect info
    case let .Failure(error): 
        // Handle error
    }
}.resume()
```

#### Using Image Task

```swift
let task = Nuke.taskWithURL(imageURL).resume()
let progress = task.progress // Track progress using NSProgress
let state = task.state // Track task state
task.completion { // Add completions, even for completed task
    let image $0.image
}
task.cancel() // Cancel image task
```

#### UICollectionView

```swift
func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCellWithReuseIdentifier(cellReuseID, forIndexPath: indexPath)

    let imageView: ImageView = <#view#>
    imageView.prepareForReuse()
    imageView.setImageWithURL(imageURL)

    return cell
}
```

Cancel image task as soon as the cell goes offscreen (optional):

```swift
func collectionView(collectionView: UICollectionView, didEndDisplayingCell cell: UICollectionViewCell, forItemAtIndexPath indexPath: NSIndexPath) {
    let imageView: ImageView = <#view#>
    imageView.prepareForReuse()
}
```

#### Applying Filters

```swift
let filter1: ImageProcessing = <#filter#>
let filter2: ImageProcessing = <#filter#>
let filterComposition = ImageProcessorComposition(processors: [filter1, filter2])

var request = ImageRequest(URL: <#image_url#>)
request.processor = filterComposition

Nuke.taskWithRequest(request) {
    // Filters are applied
    // Processed images are intelligently cached
    let image = $0.image
}.resume()
```

#### Composing Image Filters

```swift
let processor1: ImageProcessing = <#processor#>
let processor2: ImageProcessing = <#processor#>
let composition = ImageProcessorComposition(processors: [processor1, processor2])
```

#### Composing Image Decoders

```swift
let decoder1: ImageDecoding = <#decoder#>
let decoder2: ImageDecoding = <#decoder#>
let composition = ImageDecoderComposition(decoders: [decoder1, decoder2])
```

#### Preheating Images

```swift
let requests = [ImageRequest(URL: imageURL1), ImageRequest(URL: imageURL2)]
Nuke.startPreheatingImages(requests: requests)
Nuke.stopPreheatingImages(requests: requests)
```

#### Customizing Image Manager

```swift
let dataLoader: ImageDataLoading = <#data_loader#>
let decoder: ImageDecoding = <#decoder#>
let cache: ImageMemoryCaching = <#cache#>

let configuration = ImageManagerConfiguration(dataLoader: dataLoader, decoder: decoder, cache: cache)
let manager = ImageManager(configuration: configuration)

ImageManager.setShared(manager)
```

## <a name="h_design"></a>Design

<img src="https://cloud.githubusercontent.com/assets/1567433/9952711/971ae2ea-5de1-11e5-8670-6853d3fe18cd.png" width="66%"/>

|Protocol|Description|
|--------|-----------|
|`ImageManaging`|A high-level API for loading images|
|`ImageDataLoading`|Performs loading of image data (`NSData`)|
|`ImageDecoding`|Converts `NSData` to `UIImage` objects|
|`ImageProcessing`|Processes decoded images|
|`ImageMemoryCaching`|Stores processed images into memory cache|

## Installation<a name="installation"></a>

### CocoaPods

[CocoaPods](http://cocoapods.org) is a dependency manager for Cocoa projects. If you are not familiar with CocoaPods the best place to start would be [CocoaPods guides](http://guides.cocoapods.org). To install Nuke add a dependency in your Podfile:
```ruby
# Podfile
# platform :ios, '8.0'
# platform :watchos, '2.0'
# use_frameworks!
pod 'Nuke'
```

By default it will install these subspecs (if they are available for your platform):
- `Nuke/Core` - core classes
- `Nuke/UI` - UI components

There are two more optional subspec:
- `Nuke/Alamofire` - [Alamofire](https://github.com/Alamofire/Alamofire) integration
- `Nuke/GIF` - [FLAnimatedImage](https://github.com/Flipboard/FLAnimatedImage) integration

### Carthage

[Carthage](https://github.com/Carthage/Carthage) is a decentralized dependency manager for Cocoa projects. Nuke has a limitied Carthage support that doesn't feature [FLAnimatedImage](https://github.com/Flipboard/FLAnimatedImage) and [Alamofire](https://github.com/Alamofire/Alamofire) integration. To install Nuke add a dependency to your Cartfile:
```
github 'Nuke'
```

## <a name="h_requirements"></a>Requirements
- iOS 8.0+ / watchOS 2.0+
- Xcode 7.0+, Swift 2.0+

## <a name="h_contribution"></a>Contribution

- If you **need help**, use [Stack Overflow](http://stackoverflow.com/questions/tagged/iosnuke). (Tag 'iosnuke')
- If you'd like to **ask a general question**, use [Stack Overflow](http://stackoverflow.com/questions/tagged/iosnuke).
- If you **found a bug**, and can provide steps to reproduce it, open an issue.
- If you **have a feature request**, open an issue.
- If you **want to contribute**, branch of the `develop` branch and submit a pull request.

## Contacts

<a href="https://github.com/kean">
<img src="https://cloud.githubusercontent.com/assets/1567433/6521218/9c7e2502-c378-11e4-9431-c7255cf39577.png" height="44" hspace="2"/>
</a>
<a href="https://twitter.com/a_grebenyuk">
<img src="https://cloud.githubusercontent.com/assets/1567433/6521243/fb085da4-c378-11e4-973e-1eeeac4b5ba5.png" height="44" hspace="2"/>
</a>
<a href="https://www.linkedin.com/pub/alexander-grebenyuk/83/b43/3a0">
<img src="https://cloud.githubusercontent.com/assets/1567433/6521256/20247bc2-c379-11e4-8e9e-417123debb8c.png" height="44" hspace="2"/>
</a>

## License

Nuke is available under the MIT license. See the LICENSE file for more info.
