## Mocking and Testing The File System In Dart/Flutter Unit Tests

This article will describe how to mock and test the file system in Dart/Flutter for tests and in particular unit tests.

This article assumes that the reader is familiar with Flutter development and the Dart language.

The article is divided into:

1. [Introduction](#introduction)
2. [Using the dart:io library](#dartio)
3. [Using the file library](#file)
4. [Conclusion](#conclusion)

## <span id="introduction">1. Introduction</span>

We want to create a function `createHelloWorldFile` that creates a file `hello_world.dart` at a location `generated/hello_world.dart`. 

We also want to unit test this function to ensure that the file is created at the specific location when the function runs.

We'll look at two different ways of doing this:

- [Using the dart:io library](#dartio)
- [Using the file library](#file)


## <span id="dartio">2. Using the dart:io library</span>

### Writing the Function 

This implementation is to make use of the [dart:io](https://api.dart.dev/dart-io/dart-io-library.html) library and its [File](https://api.dart.dev/stable/dart-io/File-class.html) object to create the file.

We first create the `File` object with the desired location passed as a parameter and call `.createSync` to create the file. 

`recursive:true` is needed to ensure that any non-existing path is created first before creating the file.

Here is the code for the function:

```dart
import 'dart:io';

void createHelloWorldFile() {
  final file = File('generated/hello_world.dart');
  file.createSync(recursive: true);
}
```

### Calling the Function 

We can call the function like this:

```dart
import 'dart:io';

void main(List<String> arguments) {
  createHelloWorldFile();
}
```

### Testing the Function

The first step is to add the test library to `dev_dependencies` in the `pubspec.yaml` file like this:

```yaml
dev_dependencies:
  test:
```

The next step is to write the test for the function like this:

```dart
import 'dart:io';
import 'package:test/test.dart';

void main() {
  test('createHelloWorldFile creates a file at generated/hello_world.dart', () {
    createHelloWorldFile();
    final file = File('generated/hello_world.dart');
    expect(file.existsSync(), true);
  });
}
``` 

The test code above uses the [.existsSync](https://api.dart.dev/stable/2.14.3/dart-io/FileSystemEntity/existsSync.html) property on the created `File` object to confirm if it does exist.

The test passes when it is run.

### The Problem with the Test

Running this test passes but the problem with this test is that it actually creates the `generated/hello_world.dart` file. 

This generated file is not needed for the actual test so it would be better to use a mock file system so we can write/run as many tests as we want without worrying about side effects.

## <span id="file">3. Using the file library</span>

### Writing the Function 

To solve the problem with the extra file generated, we will use the [file](https://pub.dev/packages/file) library which gives the ability to create custom file systems and in-memory file systems for easy unit testing of code that works with file systems.

The first step is to add the `file` library to `dependecies` in our `pubspec.yaml` file like this:

```yaml
dependencies:
  file:
```

The next step is to update our `createHelloWorldFile` function to this:

```dart
import 'package:test/test.dart';
import 'package:file/file.dart';

void createHelloWorldFile(FileSystem fileSystem) {
  final file = fileSystem.file('generated/hello_world.dart');
  file.createSync(recursive: true);
}
```

In the code above, we inject the fileSystem into the function `createHelloWorldFile` and use that to create the file.

### Calling the Function 

To call the function, we pass a [LocalFileSystem](https://pub.dev/documentation/file/latest/local/LocalFileSystem-class.html) object into the `createHelloWorldFile` function.

>LocalFileSystem class 

>A wrapper implementation around dart:io's implementation.

>Since this implementation of the FileSystem interface delegates to dart:io, [it] is not suitable for use in the browser.

```dart
import 'package:file/local.dart';

void main(List<String> arguments) {
  final localFileSystem = LocalFileSystem();
  createHelloWorldFile(localFileSystem);
}
```

### Testing the Function

In order to test the function, we can pass a [MemoryFileSystem](https://pub.dev/documentation/file/latest/memory/MemoryFileSystem-class.html) object into the `createHelloWorldFile`.

> MemoryFileSystem class 

> An implementation of FileSystem that exists entirely in memory with an internal representation loosely based on the Filesystem Hierarchy Standard.

> MemoryFileSystem is suitable for mocking and tests, as well as for caching or staging before writing or reading to a live system.

```dart
import 'package:file/memory.dart';
import 'package:test/test.dart';

void main() {
  test('createHelloWorldFile creates a file at generated/hello_world.dart', () {
    final memoryFileSystem = MemoryFileSystem();
    createHelloWorldFile(memoryFileSystem);

    final file = memoryFileSystem.file('generated/hello_world.dart');

    expect(file.existsSync(), true);
  });
}
```

Running this test passes and it does not create any extra files.

## <span id="conclusion">4. Conclusion</span>

The [file](https://api.dart.dev/stable/dart-io/File-class.html) library provides an easy way to test operations involving files with the use of the `MemoryFileSystem` which is an in-memory file system that does not write to the actual file system. 


The complete project can be found on [Github](https://github.com/victoreronmosele/file-system-test-dart).









