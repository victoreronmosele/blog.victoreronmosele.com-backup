## Mocking and Testing Shared Preferences in Flutter Unit Tests

# Introduction
This article demonstrates how to unit-test  [Shared Preferences](https://pub.dev/packages/shared_preferences) in Flutter projects.

This article assumes that the reader is familiar with Flutter development and the Dart language. 

The code in the demo was created using `Flutter Channel stable, 2.5.3`.


# Problem Statement
We have a `SharedPreferencesUtil` class that makes use of the `SharedPreferences` object to store and retrieve an integer `count`. Our task is to unit-test the methods of this class.

The class is shown below:

```dart
import 'package:flutter/foundation.dart';
import 'package:shared_preferences/shared_preferences.dart';

class SharedPreferencesUtil {
  SharedPreferencesUtil({required this.sharedPreferences});

  final SharedPreferences sharedPreferences;

  @visibleForTesting
  static const String countKey = 'counter';

  int getCount() {
    return sharedPreferences.getInt(countKey) ?? 0;
  }

  Future<bool> setCount(int count) async {
    return sharedPreferences.setInt(countKey, count);
  }
}
```

# Testing the methods

In the `SharedPreferencesUtil` class above, the `SharedPreferences` object is passed through the constructor and it is used in the following methods:
- `getCount()`:  To return the value of the count integer stored in `SharedPreferences` with the `countKey` key and return 0 if the value in `SharedPreferences` is `null`
- `setCount()`: To set the value of count int with the `countKey` key in `SharedPreferences`

In order to write a test, we need to mock the `SharedPreferences` object that we pass into the `SharedPreferencesUtil` class.

This is done using the [SharedPreferences.setMockInitialValues()](https://pub.dev/documentation/shared_preferences/latest/shared_preferences/SharedPreferences/setMockInitialValues.html) static method.

## setMockInitialValues()


> 
Initializes the shared preferences with mock values for testing.
>
[Source](https://pub.dev/documentation/shared_preferences/latest/shared_preferences/SharedPreferences/setMockInitialValues.html) 

This method is used to mock the initial state of the `SharedPreferences` object used in tests. It takes a parameter of type `Map<String, Object>` for which the key is used to identify the stored `SharedPreferences` object and the value is the actual object stored in `SharedPreferences`.

For instance:

If we want our `SharedPreferences` object in our test to store the number 42 with the key `counter`, we do it this way:

```dart
SharedPreferences.setMockInitialValues({'counter': 42});
```


## Testing the getCounter() method


- *Test that the `getCount()` method returns 0 if there is no value stored in `SharedPreferences`.*

    We first call the `SharedPreferences.setMockInitialValues()` and pass it `{}` (empty `Map` object) method to make sure nothing is stored in `SharedPreferences`. We, then, pass it into the `SharedPreferencesUtil` class. 

    We, then, can call the `getCount()` method and compare it with the expected value of 0 using the  [expect()](https://pub.dev/documentation/test_api/latest/expect/expect.html)  method from the test library.


```
import 'package:shared_preferences/shared_preferences.dart';
import 'package:shared_preferences_unit_tests_flutter/shared_preferences_util.dart';
import 'package:test/test.dart';

main() {
  group(SharedPreferencesUtil, () {
    group('getCounter', () {
      test('returns 0 if no value is stored in SharedPreferences', () async {
        SharedPreferences.setMockInitialValues({});

        final SharedPreferences sharedPreferences =
            await SharedPreferences.getInstance();

        final SharedPreferencesUtil sharedPreferencesUtil =
            SharedPreferencesUtil(sharedPreferences: sharedPreferences);

        const expectedCounter = 0;
        final actualCounter = sharedPreferencesUtil.getCount();

        expect(expectedCounter, actualCounter);
      });
    });
  });
}
```

- *Test that the `getCount()` method returns the correct value stored in `SharedPreferences`.*

    We first call `SharedPreferences.setMockInitialValues()` and pass it `{SharedPreferencesUtil.countKey: counterValue}` where `SharedPreferencesUtil.counterKey` is the key and `counterValue` contains the value we want to store. We, then, pass it into the `SharedPreferencesUtil` class. 

    We, then, can call the `getCount()` method and compare it with the expected value of 0 using the `expect()` method.

```dart
import 'package:shared_preferences/shared_preferences.dart';
import 'package:shared_preferences_unit_tests_flutter/shared_preferences_util.dart';
import 'package:test/test.dart';

main() {
  group(SharedPreferencesUtil, () {
    group('getCounter', () {
      test('returns 0 if no value is stored in SharedPreferences', () async {
       ...
      });

      test('returns the correct value stored in SharedPreferences', () async {
        const counterValue = 42;
        SharedPreferences.setMockInitialValues(
            {SharedPreferencesUtil.countKey: counterValue});
        final SharedPreferences sharedPreferences =
            await SharedPreferences.getInstance();

        final SharedPreferencesUtil sharedPreferencesUtil =
            SharedPreferencesUtil(sharedPreferences: sharedPreferences);

        const expectedCount = counterValue;
        final actualCounter = sharedPreferencesUtil.getCount();
        expect(expectedCount, actualCounter);
      });
    });
  });
}
```
## Testing the setCounter() method

- *Test that the `setCount()` method stores the correct value in `SharedPreferences`.*

    We first call the `SharedPreferences.setMockInitialValues()` and pass it `{}` (empty `Map` object) method to make sure nothing is stored in `SharedPreferences`. We, then, pass it into the `SharedPreferencesUtil` class. 

    We, then, can call the `SharedPreferences.getInt()` method, pass the `SharedPreferencesUtil.countKey` key to it to get the value in `SharedPreferences`  compare it with the expected value in the `countValue` variable.

```dart
import 'package:shared_preferences/shared_preferences.dart';
import 'package:shared_preferences_unit_tests_flutter/shared_preferences_util.dart';
import 'package:test/test.dart';

main() {
  group(SharedPreferencesUtil, () {
    group('getCounter', () {
      test('returns 0 if no value is stored in SharedPreferences', () async {
        ...
      });

      test('returns the correct value stored in SharedPreferences', () async {
        ...
      });
    });

    group('setCounter', () {
      test('stores the correct count value in SharedPreferences', () async {
        SharedPreferences.setMockInitialValues({});
        final SharedPreferences sharedPreferences =
            await SharedPreferences.getInstance();

        final SharedPreferencesUtil sharedPreferencesUtil =
            SharedPreferencesUtil(sharedPreferences: sharedPreferences);
        const countValue = 42;

        sharedPreferencesUtil.setCount(countValue);
        const expectedCounter = countValue;
        final actualCounter =
            sharedPreferences.getInt(SharedPreferencesUtil.countKey);

        expect(expectedCounter, actualCounter);
      });
    });
  });
}
```

## Complete Test File
The content of the entire test file is shown below:

```dart
import 'package:shared_preferences/shared_preferences.dart';
import 'package:shared_preferences_unit_tests_flutter/shared_preferences_util.dart';
import 'package:test/test.dart';

main() {
  group(SharedPreferencesUtil, () {
    group('getCounter', () {
      test('returns 0 if no value is stored in SharedPreferences', () async {
        SharedPreferences.setMockInitialValues({});

        final SharedPreferences sharedPreferences =
            await SharedPreferences.getInstance();

        final SharedPreferencesUtil sharedPreferencesUtil =
            SharedPreferencesUtil(sharedPreferences: sharedPreferences);

        const expectedCounter = 0;
        final actualCounter = sharedPreferencesUtil.getCount();

        expect(expectedCounter, actualCounter);
      });

      test('returns the correct value stored in SharedPreferences', () async {
        const counterValue = 42;
        SharedPreferences.setMockInitialValues(
            {SharedPreferencesUtil.countKey: counterValue});
        final SharedPreferences sharedPreferences =
            await SharedPreferences.getInstance();

        final SharedPreferencesUtil sharedPreferencesUtil =
            SharedPreferencesUtil(sharedPreferences: sharedPreferences);

        const expectedCount = counterValue;
        final actualCounter = sharedPreferencesUtil.getCount();
        expect(expectedCount, actualCounter);
      });
    });

    group('setCounter', () {
      test('stores the correct count value in SharedPreferences', () async {
        SharedPreferences.setMockInitialValues({});
        final SharedPreferences sharedPreferences =
            await SharedPreferences.getInstance();

        final SharedPreferencesUtil sharedPreferencesUtil =
            SharedPreferencesUtil(sharedPreferences: sharedPreferences);
        const countValue = 42;

        sharedPreferencesUtil.setCount(countValue);
        const expectedCounter = countValue;
        final actualCounter =
            sharedPreferences.getInt(SharedPreferencesUtil.countKey);

        expect(expectedCounter, actualCounter);
      });
    });
  });
}
```

# Conclusion

Mocking the `SharedPreferences` object in tests makes use of the `setMockInitialValues` method to initialize the state. 

The complete code can be found on [GitHub](https://github.com/victoreronmosele/shared-preferences-unit-test-flutter).
