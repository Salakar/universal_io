# Introduction
A cross-platform _dart:io_ that works in VM/Flutter, browsers, and elsewhere.

Licensed under the [Apache License 2.0](LICENSE).
Much of the source code was adopted from the original 'dart:io' in [Dart SDK](https://github.com/dart-lang/sdk),
which was licensed under a BSD-style license (license terms of Dart SDK can be found in the source
code files that were derived from Dart SDK).

## Issues
  * Found issues? Report them at the [Github issue tracker](https://github.com/terrier989/dart-universal_io/issues).
  * Have a fix? [Create a pull request](https://github.com/terrier989/dart-universal_io/pull/new/master)!

## Similar packages
  * [universal_html](https://pub.dev/packages/universal_html) (cross-platform _dart:html_)
  * [node_io](https://pub.dev/packages/node_io)

# Getting started
## 1.Add a dependency
In `pubspec.yaml`:
```yaml
dependencies:
  universal_io: '>=0.5.0 <2.0.0'
```

## 2. Choose a driver (optional)
### VM/Flutter?
  * Library "package:universal_io/io.dart" will automatically export _dart:io_ for you.

### Browser?
  * _BrowserIODriver_ is automatically used when you use _Dart2js_ / _devc_. This is possible with
    "conditional imports" feature of Dart.
  * The driver implements _HttpClient_ (with restrictions) and a few other features.
    If you need features such as sockets or unrestricted HTTP connections, choose one of the options
    below.

### Chrome OS App?
  * [universal_io_driver_chrome_os](https://github.com/terrier989/dart-universal_io_driver_chrome_os)

### Node.JS? Google Cloud Functions?
  * [universal_io_driver_node](https://github.com/terrier989/dart-universal_io_driver_node)

### GRPC messaging?
  * [universal_io_driver_grpc](https://github.com/terrier989/dart-universal_io_driver_grpc)

## 3. Use

```dart
import 'package:universal_io/io.dart';

void main() async {
  // Use 'dart:io' HttpClient API.
  final httpClient = new HttpClient();
  final request = await httpClient.getUrl(Uri.parse("http://google.com"));
  final response = await request.close();
}

```

# Manual
## Default driver behavior
### HTTP client
In browser, HTTP client is implemented using _dart:html_ _HttpRequest_, which uses 
[XmlHttpRequest](https://developer.mozilla.org/en/docs/Web/API/XMLHttpRequest).

Unlike HTTP client in the standard _dart:io_, the browser implementation sends HTTP request only
after _httpRequest.close()_ has been called.

If a cross-origin request fails, error message contains a detailed description how to fix
possible issues like missing cross-origin headers. The error messages look like:

    BrowserHttpClient received an error from XMLHttpRequest (which doesn't tell
    reason for the error).
    
    HTTP method:   PUT
    URL:           http://destination.com
    Origin:        http://source.com
    
    Cross-origin request!
    CORS credentials mode' is disabled (cookies will NOT be supported).
    
    If the URL is correct and the server actually responded, did the response
    include the following required CORS headers?
      * Access-Control-Allow-Origin: http://source.com
        * Wildcard '*' is also acceptable.
      * Access-Control-Allow-Methods: PUT

### Platform
  * In browser, variables are determined by browser APIs such as _navigator.userAgent_.
  * Elsewhere (e.g. Node.JS), appears like Linux environment.

### Files
  * Any attempt to use these APIs will throw _UnimplementedError_.

### Socket classes and HttpServer
  * Any attempt to use these APIs will throw _UnimplementedError_.

## Writing your own driver?
```dart
import 'package:universal_io/io.dart';
import 'package:universal_io/driver.dart';
import 'package:universal_io/driver_base.dart';

void main() {
  IODriver.zoneLocal.freezeDefault(const ExampleDriver());
  
  // Now the APIs will use your driver.
  final file = new File();
  print(file is ExampleFile);
}

class ExampleDriver extends BaseDriver {
  const ExampleDriver() : super(fileSystemDriver:const MyFileSystemDriver());
}

class ExampleFileSystemDriver extends BaseFileSystemDriver {
  const ExampleFileSystemDriver();

  @override
  File newFile(String path) {
    return new ExampleFile();
  }
}

class ExampleFile extends BaseFile {
  // ...
}
```