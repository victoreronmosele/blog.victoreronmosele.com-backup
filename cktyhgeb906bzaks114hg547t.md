## Mocking callbacks in Dart/Flutter unit tests

This article is a guide on how to mock callbacks functions and verify that the callback functions are called in Dart/Flutter unit tests.

The article is divided into:

1. [Introduction](#introduction)
2. [Mocking The Callback](#mocking)
3. [Testing The Callback](#testing)

## <span id="introduction">1. Introduction</span>

Let's say we have a function `readDate` with the definition below:
```dart
void readData(void Function() onDataReadComplete) {
    print('Reading data');
    onDataReadComplete();
}
```
The function takes a callback `onDataReadComplete` as a lone  [positional parameter](https://dart.dev/guides/language/language-tour#parameters) and it calls `onDataReadComplete` after it prints ('Reading data').

Our task here is to write a unit test to confirm that `onDataReadComplete` is run whenever `readData` runs.

In order to do this we need to mock `onDataReadComplete` and confirm that it is actually called.

## <span id="mocking"> 2. Mocking The Callback</span>

In other to mock the callback, we will use the mock library [mockito](https://pub.dev/packages/mockito).

The first step is to add the `mockito` dependecy to our project by including it in the `pubspec.yaml` file. We will also add the [test](https://pub.dev/packages/test) library to the `pubspec.yaml` file.

```yaml
dev_dependencies:
  mockito:
  test:
```

Mockito has two ways to generate mock objects:

1.  [Code Generation](https://github.com/dart-lang/mockito/blob/master/NULL_SAFETY_README.md#code-generation) 
2.  [Manual Mock Implementation](https://github.com/dart-lang/mockito/blob/master/NULL_SAFETY_README.md#manual-mock-implementaion) 

For this article, we will use the manual mock implementation for simplicity.

A class can be mocked manually by creating a class that extends Mock and implements the class to be mocked. 

Below is a sample from the documentation:

```dart
class MockHttpServer extends Mock implements HttpServer {}
``` 

This means we need a class to represent our function so our mock class can implement it.

We will make use of Dart's  [Callable Classes](https://dart.dev/guides/language/language-tour#callable-classes).
 
>To allow an instance of your Dart class to be called like a function, implement the call() method.
>
> \- [Callable Classes](https://dart.dev/guides/language/language-tour#callable-classes)
 
This means we can represent our `void Function() onDataReadComplete` function as this below:

```dart
abstract class OnDataReadComplete {
    void call();
}
```

The class is an `abstract` class since we won't be providing an implementation of the `call` method
.

The `call` method must have the same definition as the function to be mocked. This is why it is a `void` method.

Now, we can create our mock class like this below:

```dart
class MockOnDataReadComplete extends Mock implements OnDataReadComplete {}
```

We can now, use `MockOnDataReadComplete` in our test.

## <span id="testing">3. Testing The Callback</span>

We will follow the following steps in testing the callback function:

1. Create the mock object `mockOnDataReadComplete` using `MockOnDataReadComplete`.
2. Run the `readData` function with `mockOnDataReadComplete` passed as a parameter.
3. Use the `verify` function to confirm that `mockOnDataReadComplete()` is called once. *Note that you need to call the function like this `mockOnDataReadComplete()` and not just pass the variable `mockOnDataReadComplete` to the verify function*

```dart
test('onDataReadComplete gets called whenever readData runs', () {
    //Creates the mock object
    final mockOnDataReadComplete = MockOnDataReadComplete();

    //Runs the function
    readData(mockOnDataReadComplete);

    //Confirm that the mock object is called
    verify(mockOnDataReadComplete()).called(1);
  });
```

Here is the full test file:

```dart
import 'package:mockito/mockito.dart';
import 'package:test/test.dart';

void readData(void Function() onDataReadComplete) {
  print('Reading data');
  onDataReadComplete();
}

abstract class OnDataReadComplete {
  void call();
}

class MockOnDataReadComplete extends Mock implements OnDataReadComplete {}

void main() {
  test('onDataReadComplete gets called whenever readData runs', () {
    //Creates the mock object
    final mockOnDataReadComplete = MockOnDataReadComplete();

    //Runs the function
    readData(mockOnDataReadComplete);

    //Confirm that the mock object is called
    verify(mockOnDataReadComplete()).called(1);
  });
}
```

And now we run our tests and it passes!

The complete project can be found on  [Github](https://github.com/victoreronmosele/callback-test-dart).


 

















