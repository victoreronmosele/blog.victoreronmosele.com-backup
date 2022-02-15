## Mocking and Testing Firestore Operations | Part 1 (Documents and Collections)

# Introduction

This article examines  [Firestore](https://firebase.google.com/docs/firestore) operations with respect to Documents and Collections and demonstrates how to mock and test them.

The article was created using `Flutter Channel stable, 2.8.1`.

# Prerequisites

Add the following packages to your `pubspec.yaml` file and run `flutter pub get`.

```yaml
dependencies:
  ...
  cloud_firestore: ^3.1.4
  fake_cloud_firestore: ^1.2.0

dev_dependencies:
  ...
  test:
```
This adds `cloud_firestore` which is the Cloud Firestore package, `fake_cloud_firestore` which is the mock Firestore package and `test` which is the package used for unit tests.


# Problem Statement
We have a class `FirestoreService` which performs read and write operations to Firestore and we want to write a set of unit tests for this class.

The class is shown below:

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

class FirestoreService {
  const FirestoreService({required this.firestore});

  final FirebaseFirestore firestore;

  /// Collection Operations

  Future<DocumentReference<Map<String, dynamic>>> addToCollection(
      {required Map<String, dynamic> data,
      required String collectionPath}) async {
    return firestore.collection(collectionPath).add(data);
  }

  Future<QuerySnapshot<Map<String, dynamic>>> getFromCollection(
      {required String collectionPath}) {
    return firestore.collection(collectionPath).get();
  }

  Stream<QuerySnapshot<Map<String, dynamic>>> getSnapshotStreamFromCollection(
      {required String collectionPath}) {
    return firestore.collection(collectionPath).snapshots();
  }

  /// Document Operations

  Future<void> deleteDocumentFromCollection(
      {required String collectionPath, required String documentPath}) async {
    return firestore.collection(collectionPath).doc(documentPath).delete();
  }

  Future<DocumentSnapshot<Map<String, dynamic>>> getFromDocument(
      {required String collectionPath, required String documentPath}) {
    return firestore.collection(collectionPath).doc(documentPath).get();
  }

  Future<void> setDataOnDocument(
      {required Map<String, dynamic> data,
      required String collectionPath,
      required String documentPath}) {
    return firestore.collection(collectionPath).doc(documentPath).set(data);
  }

  Stream<DocumentSnapshot<Map<String, dynamic>>> getSnapshotStreamFromDocument(
      {required String collectionPath, required String documentPath}) {
    return firestore.collection(collectionPath).doc(documentPath).snapshots();
  }

  Future<void> updateDataOnDocument(
      {required Map<String, dynamic> data,
      required String collectionPath,
      required String documentPath}) {
    return firestore.collection(collectionPath).doc(documentPath).update(data);
  }
}
```

# Testing Approach
### Construction Injection

Since the class `FirestoreService` injects the `Firestore` object through its constructor, we can pass the `FakeFirebaseFirestore` object from the `fake_cloud_firestore` package and use that to test the methods in the class.

### Mocking Strategy

The `FakeFirebaseFirestore` object works like the `Firestore` object and the read and write operations that can be used to retrieve and add data to the `Firestore` object can also be used to retrieve and add data to the `FakeFirebaseFirestore` object. 

This helps us prepopulate the mock object right before we call our methods and helps confirm the expected state of the `Firestore` database.

### Database State Set Up
  - #### *Initial State*

  The initial state of the `FakeFirebaseFirestore` object can be set up by calling the regular write operations on the `CollectionReference` and `DocumentReference`.

  For instance, we want the initial state of the database to have an object `{answer: 42}` written to a document `lifeTheUniverseAndEverything` in collection `answers`, we write the code below to set it up:
```dart
 final FakeFirebaseFirestore fakeFirebaseFirestore = FakeFirebaseFirestore();

 const String collectionPath = 'answers';
 const String documentPath = 'lifeTheUniverseAndEverything';

 const Map<String, dynamic> data = {'answer': 42};

 await fakeFirebaseFirestore
        .collection(collectionPath)
        .doc(documentPath)
        .set(data);
```

  - #### *Final State*

  The final state of the `FakeFirebaseFirestore` object can be gotten by calling the regular read operations on the `CollectionReference` and `DocumentReference`.

  Now, to get the final state of the `FakeFirebaseFirestore` object from above, we write the code below:
```dart
 final DocumentSnapshot<Map<String, dynamic>> documentSnapshot =
        await fakeFirebaseFirestore
            .collection(collectionPath)
            .doc(documentPath)
            .get();
 final Map<String, dynamic> actualData = documentSnapshot.data()!;

 print(actualData); // {'answer': 42}
```

  - #### *Assertion*

  Once we have our data, our database's initial state set up and the final state retrieved, we can assert that the data in the database matches the expected data.

  Going by what we have above, we can write the assertion using the test package's `expect` method like below:

  ```dart
  expect(actualData, data);  /// Passes ✅
  ```

# Testing the FirestoreService class
We will test the `FirestoreService` class by following the approach described in the `Testing Approach` section above.

This section will be divided into three:
- Set-up (where the objects needed for the tests are created)
- Collection Operations Testing (where operations involving Collections are tested)
- Document Operations Testing (where operations involving Documents are tested)

### Set-up

Create a test file `firestore_service_test.dart` and add the following code to it:

```dart
import 'package:fake_cloud_firestore/fake_cloud_firestore.dart';
import 'package:test/test.dart';

void main() {
  group('FirestoreService', () {
    FakeFirebaseFirestore? fakeFirebaseFirestore;
    const Map<String, dynamic> data = {'data': '42'};

    setUp(() {
      fakeFirebaseFirestore = FakeFirebaseFirestore();
    });
  });
}
```

This creates the `FakeFirebaseFirestore` object and the data needed for every test we'll write for the `FirestoreService` class. All of the tests are put inside the "FirestoreService" group.

### Collection Operations Testing
  - #### *Map Equality*
  This section will be testing Collection operations and this means we will be asserting that the collection contains the data.
  Typically, that is done by using the [contains](https://api.flutter.dev/flutter/package-matcher_matcher/contains.html) matcher. 

  > Returns a matcher that matches if the match argument contains the expected value.
   
   This is how an assertion using the `contains` matcher would be written:

   ```dart
    final List<int> intList = [1, 2];
    final int intData = 2;

    expect(intList, contains(intData));  /// Passes ✅
    ```

   This passes but when it comes to Lists of Maps, it's a different case. 

   The assertion below fails:

   ```dart
   final List<Map<String, dynamic>> mapList = [
        {'data': 42}
   ];
   final mapData = {'data': 42};
   expect(mapList, contains(mapData)); /// Fails ❌
   ```

   This is because the `contains`  matcher for Lists is implemented this way:

   ```dart
   return item.contains(_expected);
   ```
   which in turn uses the `contains` method on List which is implemented like below:

   ```dart
   bool contains(Object? element) {
     for (E e in this) {
       if (e == element) return true;
     }
     return false;
   }
   ```

   The code above checks that the `e` object is the same object as the `element` object. And the `mapData` object won't be the same as the one contained in the List. That explains why the test fails.

- #### *mapEquals*
    In order to get around this, we can make use of the [mapEquals](https://api.flutter.dev/flutter/foundation/mapEquals.html) function that does the following:

   > Compares two maps for element-by-element equality.
   > 
   > Returns true if the maps are both null, or if they are both non-null, they have the same lengths and they contain the same keys associated with the same values. Returns false otherwise.

   We can check if any of the element in the list is equal to the `mapData` object by using the `mapEquals` function like this:

   ```dart
   final List<Map<String, dynamic>> mapList = [
           {'data': 42}
   ];
   final mapData = {'data': 42};
   expect(mapList.any((element) => mapEquals(element, mapData)), true); /// Passes ✅
   ```

   This test passes because the equality check using `mapEquals` compares the two Map objects by confirming that the keys and values are the same.

- #### *Custom Matcher*

   While using the check above would work for our tests, it's cleaner and more readable to follow the Flutter convention of using a Matcher to assert our test.

   We move the logic `mapList.any((element) => mapEquals(element, mapData))` into the Matcher like this below:

   ```dart
   class MapListContains extends Matcher {
     final Map<dynamic, dynamic> _expected;

     const MapListContains(this._expected);

     @override
     Description describe(Description description) {
       return description.add('contains ').addDescriptionOf(_expected);
     }

     @override
     bool matches(dynamic item, Map matchState) {
       if (item is List<Map>) {
         return item.any((element) => mapEquals(element, _expected));
       }
       return false;
     }
   }
   ```

   The `MapListContains` class extends the `Matcher` abstract class and overrides the `describe` method and the `matches` method.
   The assertion logic is implemented in the `matches` method in the `MapListContains` class. 

   So we can update the test to this below:

   ```dart
   final List<Map<String, dynamic>> mapList = [
           {'data': 42}
   ];
   final mapData = {'data': 42};
   expect(mapList, MapListContains(mapData));  /// Passes ✅
   ```

   And this test passes.

- #### *Set-up*
     
    Now, we move to the actual test set-up.

    We first create a group "Collection Operations" to hold tests for collection operations.
    
  ```dart
  import 'package:fake_cloud_firestore/fake_cloud_firestore.dart';
  import 'package:test/test.dart';

  void main() {
    group('FirestoreService', () {
      FakeFirebaseFirestore? fakeFirebaseFirestore;
      const Map<String, dynamic> data = {'data': '42'};

      setUp(() {
         fakeFirebaseFirestore = FakeFirebaseFirestore();
      });

      group('Collection Operations', () {

      });
    });
   }
  ```

   We, can add each test inside of the group.

- #### *Test For Adding To A Collection.* 
   ```dart 
   Future<DocumentReference<Map<String, dynamic>>> addToCollection(
      {required Map<String, dynamic> data,
       required String collectionPath}) async {
     return firestore.collection(collectionPath).add(data);
   }
   ```
   The `addToCollection` method above can be tested by:
   - Injecting the mock Firestore object, `fakeFirebaseFirestore`,  to the service.
   - Adding the data to the collection via the service's `addToCollection` method.
   - Retrieving the actual data list in the collection via `fakeFirebaseFirestore`.
   - Asserting that the actual data list contains the added data using the `MapListContains` matcher.

  This is shown below:

   ```dart
   test('addToCollection adds data to given collection', () async {
       final FirestoreService firestoreService =
              FirestoreService(firestore: fakeFirebaseFirestore!);
       const String collectionPath = 'collectionPath';

       await firestoreService.addToCollection(
              data: data, collectionPath: collectionPath);

       final List<Map<String, dynamic>> actualDataList =
              (await fakeFirebaseFirestore!.collection('collectionPath').get())
                  .docs
                  .map((e) => e.data())
                  .toList();

       expect(actualDataList, const MapListContains(data));  /// Passes ✅
   });
   ```

- #### *Test For Getting Data From A Collection.* 

   ```dart 
 Future<QuerySnapshot<Map<String, dynamic>>> getFromCollection() {
    return firestore.collection('collectionPath').get();
  }
   ```

   The `getFromCollection` method above can be tested by:
   - Injecting the mock Firestore object, `fakeFirebaseFirestore`,  to the service.
   - Adding the data to the collection via the `fakeFirebaseFirestore` object.
   - Retrieving the actual data list in the collection via the service's `getFromCollection` method.
   - Asserting that the data list contains the added data using the `MapListContains` matcher.

  This is shown below:

   ```dart
   test('getFromCollection gets data from a given collection', () async {
      final FirestoreService firestoreService =
              FirestoreService(firestore: fakeFirebaseFirestore!);
      const String collectionPath = 'collectionPath';

      await fakeFirebaseFirestore!.collection(collectionPath).add(data);

      final List<Map<String, dynamic>> dataList = (await firestoreService
                  .getFromCollection(collectionPath: collectionPath))
              .docs
              .map((e) => e.data())
              .toList();

      expect(dataList, const MapListContains(data)); /// Passes ✅
   });
   ```

- #### *Test For Getting A Snapshot Stream From A Collection.* 

   ```dart
Stream<QuerySnapshot<Map<String, dynamic>>> getSnapshotStreamFromCollection(
      {required String collectionPath}) {
    return firestore.collection(collectionPath).snapshots();
  }
   ```

   The `getSnapshotStreamFromCollection` method above can be tested by:
   - Injecting the mock Firestore object, `fakeFirebaseFirestore`,  to the service
   - Adding the data to the collection via the `fakeFirebaseFirestore` object.
   - Retrieving the expected snapshot stream from the `fakeFirebaseFirestore` object and the actual snapshot stream from the service's `getSnapshotStreamFromCollection` method.
   - Getting the data from the two snapshot streams and putting them into individual lists (the actual data list and the expected data list).
   - Asserting that the actual data list is equal to the expected data list.

  This is shown below:

   ```dart
   test( 'getSnapshotStreamFromCollection returns a stream of QuerySnaphot containing the data added', () async {
       final FirestoreService firestoreService =
              FirestoreService(firestore: fakeFirebaseFirestore!);
       const String collectionPath = 'collectionPath';

       final CollectionReference<Map<String, dynamic>> collectionReference =
              fakeFirebaseFirestore!.collection(collectionPath);
       await collectionReference.add(data);

       final Stream<QuerySnapshot<Map<String, dynamic>>>
              expectedSnapshotStream = collectionReference.snapshots();

       final actualSnapshotStream = firestoreService
              .getSnapshotStreamFromCollection(collectionPath: collectionPath);

       final QuerySnapshot<Map<String, dynamic>> expectedQuerySnapshot =
              await expectedSnapshotStream.first;
       final QuerySnapshot<Map<String, dynamic>> actualQuerySnapshot =
              await actualSnapshotStream.first;

       final List<Map<String, dynamic>> expectedDataList =
              expectedQuerySnapshot.docs.map((e) => e.data()).toList();
       final List<Map<String, dynamic>> actualDataList =
              actualQuerySnapshot.docs.map((e) => e.data()).toList();

       expect(actualDataList, expectedDataList); /// Passes ✅
   });
   ```

### Document Operations Testing

- #### *Test For Deleting A Document From A Collection.* 

   ```dart
  Future<void> deleteDocumentFromCollection(
      {required String collectionPath, required String documentPath}) async {
    return firestore.collection(collectionPath).doc(documentPath).delete();
  }
   ```

   The `deleteDocumentFromCollection` method above can be tested by:
   - Injecting the mock Firestore object, `fakeFirebaseFirestore`,  to the service.
   - Adding the data to the collection via the `fakeFirebaseFirestore` object.
   - Deleting the document at the collection via the service's `deleteDocumentFromCollection` method.
   - Getting the data at the document's path from the `fakeFirebaseFirestore`.
   - Asserting that the document does not exist.

  This is shown below:
   ```dart
 test('deleteDocumentFromCollection deletes a document from a given collection', () async {
        final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);
        const String collectionPath = 'collectionPath';

        final CollectionReference<Map<String, dynamic>> collectionReference =
            fakeFirebaseFirestore!.collection(collectionPath);

        final DocumentReference<Map<String, dynamic>> documentReference =
            await collectionReference.add(data);

        final String documentPath = documentReference.path;

        await firestoreService.deleteDocumentFromCollection(
            collectionPath: collectionPath, documentPath: documentPath);

        final DocumentSnapshot<Map<String, dynamic>> documentSnapshot =
            await collectionReference.doc(documentPath).get();

        expect(documentSnapshot.exists, false); /// Passes ✅
      });
   ```
- #### *Test For Getting A Document's Data.* 
   ```dart
 Future<DocumentSnapshot<Map<String, dynamic>>> getFromDocument(
      {required String collectionPath, required String documentPath}) {
    return firestore.collection(collectionPath).doc(documentPath).get();
  }
   ```
   The `getFromDocument` method above can be tested by: 
   - Injecting the mock Firestore object, `fakeFirebaseFirestore`,  to the service.
   - Setting data to the document via the `fakeFirebaseFirestore` object.
   - Retrieving the actual data in the document via the service's `getFromDocument` method.
   - Asserting that the data retrieved via the service is equal to the data set via `fakeFirebaseFirestore`. 

  This is shown below:
    ```dart
 test('getFromDocument gets data from a given document', () async {
        final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);

        const String collectionPath = 'collectionPath';
        const String documentPath = 'documentPath';

        final DocumentReference<Map<String, dynamic>> documentReference =
            fakeFirebaseFirestore!.collection(collectionPath).doc(documentPath);

        await documentReference.set(data);

        final DocumentSnapshot<Map<String, dynamic>> expectedDocumentSnapshot =
            await documentReference.get();

        final DocumentSnapshot<Map<String, dynamic>> actualDocumentSnapshot =
            await firestoreService.getFromDocument(
                collectionPath: collectionPath, documentPath: documentPath);

        final Map<String, dynamic>? expectedData =
            expectedDocumentSnapshot.data();
        final Map<String, dynamic>? actualData = actualDocumentSnapshot.data();

        expect(actualData, expectedData); /// Passes ✅
      });
   ```

- #### *Test For Setting Documents's Data.* 
   ```dart
  Future<void> setDataOnDocument(
      {required Map<String, dynamic> data,
      required String collectionPath,
      required String documentPath}) {
    return firestore.collection(collectionPath).doc(documentPath).set(data);
  }
   ```

   The `setDataOnDocument` method above can be tested by: 
   - Injecting the mock Firestore object, `fakeFirebaseFirestore`,  to the service.
   - Setting data to the document via the service's `setDataOnDocument` method.
   - Retrieving the actual data in the document via the `fakeFirebaseFirestore` object.
   - Asserting that the data set via the service is equal to the data retrieved via `fakeFirebaseFirestore`.

  This is shown below:
   ```dart
test('setDataOnDocument sets data on a given document', () async {
        final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);

        const String collectionPath = 'collectionPath';
        const String documentPath = 'documentPath';

        await firestoreService.setDataOnDocument(
            data: data,
            collectionPath: collectionPath,
            documentPath: documentPath);

        final DocumentReference<Map<String, dynamic>> documentReference =
            fakeFirebaseFirestore!.collection(collectionPath).doc(documentPath);

        final DocumentSnapshot<Map<String, dynamic>> actualDocumentSnapshot =
            await documentReference.get();
        final Map<String, dynamic>? actualData = actualDocumentSnapshot.data();

        const Map<String, dynamic> expectedData = data;

        expect(actualData, expectedData); /// Passes ✅
      });
   ```

- #### *Test For Getting A Snapshot Stream From A Document.* 
   ```dart
  Stream<DocumentSnapshot<Map<String, dynamic>>> getSnapshotStreamFromDocument(
      {required String collectionPath, required String documentPath}) {
    return firestore.collection(collectionPath).doc(documentPath).snapshots();
  }
   ```
   The `getSnapshotStreamFromDocument` method above can be tested by: 
   - Injecting the mock Firestore object, `fakeFirebaseFirestore`,  to the service.
   - Setting the data to the document via the `fakeFirebaseFirestore` object.
   - Retrieving the expected snapshot stream from the `fakeFirebaseFirestore` object and the actual snapshot stream from the service's `getSnapshotStreamFromDocument` method.
   - Getting the data from the two snapshot streams.
   - Asserting that the actual data via the service is equal to the expected data via `fakeFirebaseFirestore`.

  This is shown below:
   ```dart
test(
          'getSnapshotStreamFromDocument returns a stream of DocumentSnapshot containing the data set',
          () async {
        final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);

        const String collectionPath = 'collectionPath';
        const String documentPath = 'documentPath';

        final DocumentReference<Map<String, dynamic>> documentReference =
            fakeFirebaseFirestore!.collection(collectionPath).doc(documentPath);

        await documentReference.set(data);

        final Stream<DocumentSnapshot<Map<String, dynamic>>>
            expectedSnapshotStream = documentReference.snapshots();

        final Stream<DocumentSnapshot<Map<String, dynamic>>>
            actualSnapshotStream =
            firestoreService.getSnapshotStreamFromDocument(
                collectionPath: collectionPath, documentPath: documentPath);

        final DocumentSnapshot<Map<String, dynamic>> expectedDocumentSnapshot =
            await expectedSnapshotStream.first;
        final DocumentSnapshot<Map<String, dynamic>> actualDocumentSnapshot =
            await actualSnapshotStream.first;

        final Map<String, dynamic>? expectedData =
            expectedDocumentSnapshot.data();
        final Map<String, dynamic>? actualData = actualDocumentSnapshot.data();

        expect(actualData, expectedData); /// Passes ✅
      });
   ```

- #### *Test For Updating A Document's Data.* 
   ```dart
  Future<void> updateDataOnDocument(
      {required Map<String, dynamic> data,
      required String collectionPath,
      required String documentPath}) {
    return firestore.collection(collectionPath).doc(documentPath).update(data);
  }
   ```
   The `updateDataOnDocument` method above can be tested by: 
    - Injecting the mock Firestore object, `fakeFirebaseFirestore`,  to the service.
    - Setting data to the document via the `fakeFirebaseFirestore` object.
    - Updating the data set to the document via the service's `updateDataOnDocument` method.
    - Retrieving the data in the document via `fakeFirebaseStore`.
    - Asserting that the updated data set via the service is equal to the expected data retrieved via `fakeFirebaseFirestore`.

  This is shown below:

   ```dart
   test('updateDataOnDocument updates a given document\'s data', () async {
     final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);

     const String collectionPath = 'collectionPath';
     const String documentPath = 'documentPath';

     final DocumentReference<Map<String, dynamic>> documentReference =
            fakeFirebaseFirestore!.collection(collectionPath).doc(documentPath);

     await documentReference.set(data);

     final Map<String, dynamic> dataUpdate = {'data': '43'};

     await firestoreService.updateDataOnDocument(
            data: dataUpdate,
            collectionPath: collectionPath,
            documentPath: documentPath);

     final DocumentSnapshot<Map<String, dynamic>> actualDocumentSnapshot =
            await documentReference.get();

     final Map<String, dynamic>? actualData = actualDocumentSnapshot.data();

     final Map<String, dynamic> expectedData = dataUpdate;

     expect(actualData, expectedData); /// Passes ✅
   }); 
   ```

# Complete Test File
The content of the entire test file is shown below:

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:fake_cloud_firestore/fake_cloud_firestore.dart';
import 'package:firestore_unit_test_flutter/firestore_service.dart';
import 'package:flutter/foundation.dart';
import 'package:test/test.dart';

void main() {
  group('FirestoreService', () {
    FakeFirebaseFirestore? fakeFirebaseFirestore;
    const Map<String, dynamic> data = {'data': '42'};

    setUp(() {
      fakeFirebaseFirestore = FakeFirebaseFirestore();
    });

    group(
      'Collection Operations',
      () {
        test('addToCollection adds data to given collection', () async {
          final FirestoreService firestoreService =
              FirestoreService(firestore: fakeFirebaseFirestore!);
          const String collectionPath = 'collectionPath';

          await firestoreService.addToCollection(
              data: data, collectionPath: collectionPath);

          final List<Map<String, dynamic>> actualDataList =
              (await fakeFirebaseFirestore!.collection('collectionPath').get())
                  .docs
                  .map((e) => e.data())
                  .toList();

          expect(actualDataList, const MapListContains(data));
        });

        test('getFromCollection gets data from a given collection', () async {
          final FirestoreService firestoreService =
              FirestoreService(firestore: fakeFirebaseFirestore!);
          const String collectionPath = 'collectionPath';

          await fakeFirebaseFirestore!.collection(collectionPath).add(data);

          final List<Map<String, dynamic>> dataList = (await firestoreService
                  .getFromCollection(collectionPath: collectionPath))
              .docs
              .map((e) => e.data())
              .toList();

          expect(dataList, const MapListContains(data));
        });

        test(
            'getSnapshotStreamFromCollection returns a stream of QuerySnaphot containing the data added',
            () async {
          final FirestoreService firestoreService =
              FirestoreService(firestore: fakeFirebaseFirestore!);
          const String collectionPath = 'collectionPath';

          final CollectionReference<Map<String, dynamic>> collectionReference =
              fakeFirebaseFirestore!.collection(collectionPath);
          await collectionReference.add(data);

          final Stream<QuerySnapshot<Map<String, dynamic>>>
              expectedSnapshotStream = collectionReference.snapshots();

          final actualSnapshotStream = firestoreService
              .getSnapshotStreamFromCollection(collectionPath: collectionPath);

          final QuerySnapshot<Map<String, dynamic>> expectedQuerySnapshot =
              await expectedSnapshotStream.first;
          final QuerySnapshot<Map<String, dynamic>> actualQuerySnapshot =
              await actualSnapshotStream.first;

          final List<Map<String, dynamic>> expectedDataList =
              expectedQuerySnapshot.docs.map((e) => e.data()).toList();
          final List<Map<String, dynamic>> actualDataList =
              actualQuerySnapshot.docs.map((e) => e.data()).toList();

          expect(actualDataList, expectedDataList);
        });
      },
    );

    group('Document Operations', () {
      test(
          'deleteDocumentFromCollection deletes a document from a given collection',
          () async {
        final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);
        const String collectionPath = 'collectionPath';

        final CollectionReference<Map<String, dynamic>> collectionReference =
            fakeFirebaseFirestore!.collection(collectionPath);

        final DocumentReference<Map<String, dynamic>> documentReference =
            await collectionReference.add(data);

        final String documentPath = documentReference.path;

        await firestoreService.deleteDocumentFromCollection(
            collectionPath: collectionPath, documentPath: documentPath);

        final DocumentSnapshot<Map<String, dynamic>> documentSnapshot =
            await collectionReference.doc(documentPath).get();

        expect(documentSnapshot.exists, false);
      });

      test('getFromDocument gets data from a given document', () async {
        final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);

        const String collectionPath = 'collectionPath';
        const String documentPath = 'documentPath';

        final DocumentReference<Map<String, dynamic>> documentReference =
            fakeFirebaseFirestore!.collection(collectionPath).doc(documentPath);

        await documentReference.set(data);

        final DocumentSnapshot<Map<String, dynamic>> expectedDocumentSnapshot =
            await documentReference.get();

        final DocumentSnapshot<Map<String, dynamic>> actualDocumentSnapshot =
            await firestoreService.getFromDocument(
                collectionPath: collectionPath, documentPath: documentPath);

        final Map<String, dynamic>? expectedData =
            expectedDocumentSnapshot.data();
        final Map<String, dynamic>? actualData = actualDocumentSnapshot.data();

        expect(actualData, expectedData);
      });

      test('setDataOnDocument sets data on a given document', () async {
        final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);

        const String collectionPath = 'collectionPath';
        const String documentPath = 'documentPath';

        await firestoreService.setDataOnDocument(
            data: data,
            collectionPath: collectionPath,
            documentPath: documentPath);

        final DocumentReference<Map<String, dynamic>> documentReference =
            fakeFirebaseFirestore!.collection(collectionPath).doc(documentPath);

        final DocumentSnapshot<Map<String, dynamic>> actualDocumentSnapshot =
            await documentReference.get();
        final Map<String, dynamic>? actualData = actualDocumentSnapshot.data();

        const Map<String, dynamic> expectedData = data;

        expect(actualData, expectedData);
      });

      test(
          'getSnapshotStreamFromDocument returns a stream of DocumentSnapshot containing the data set',
          () async {
        final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);

        const String collectionPath = 'collectionPath';
        const String documentPath = 'documentPath';

        final DocumentReference<Map<String, dynamic>> documentReference =
            fakeFirebaseFirestore!.collection(collectionPath).doc(documentPath);

        await documentReference.set(data);

        final Stream<DocumentSnapshot<Map<String, dynamic>>>
            expectedSnapshotStream = documentReference.snapshots();

        final Stream<DocumentSnapshot<Map<String, dynamic>>>
            actualSnapshotStream =
            firestoreService.getSnapshotStreamFromDocument(
                collectionPath: collectionPath, documentPath: documentPath);

        final DocumentSnapshot<Map<String, dynamic>> expectedDocumentSnapshot =
            await expectedSnapshotStream.first;
        final DocumentSnapshot<Map<String, dynamic>> actualDocumentSnapshot =
            await actualSnapshotStream.first;

        final Map<String, dynamic>? expectedData =
            expectedDocumentSnapshot.data();
        final Map<String, dynamic>? actualData = actualDocumentSnapshot.data();

        expect(actualData, expectedData);
      });

      test('updateDataOnDocument updates a given document\'s data', () async {
        final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);

        const String collectionPath = 'collectionPath';
        const String documentPath = 'documentPath';

        final DocumentReference<Map<String, dynamic>> documentReference =
            fakeFirebaseFirestore!.collection(collectionPath).doc(documentPath);

        await documentReference.set(data);

        final Map<String, dynamic> dataUpdate = {'data': '43'};

        await firestoreService.updateDataOnDocument(
            data: dataUpdate,
            collectionPath: collectionPath,
            documentPath: documentPath);

        final DocumentSnapshot<Map<String, dynamic>> actualDocumentSnapshot =
            await documentReference.get();

        final Map<String, dynamic>? actualData = actualDocumentSnapshot.data();

        final Map<String, dynamic> expectedData = dataUpdate;

        expect(actualData, expectedData);
      });
    });
  });
}

class MapListContains extends Matcher {
  final Map<dynamic, dynamic> _expected;

  const MapListContains(this._expected);

  @override
  Description describe(Description description) {
    return description.add('contains ').addDescriptionOf(_expected);
  }

  @override
  bool matches(dynamic item, Map matchState) {
    if (item is List<Map>) {
      return item.any((element) => mapEquals(element, _expected));
    }
    return false;
  }
}
```

# Conclusion
This article demonstrates how to unit test Firebase Firestore operations involving Collections and Documents. It also shows how to use a custom Matcher in test assertions.

The complete code can be found on [GitHub](https://github.com/victoreronmosele/firestore-unit-test-flutter).

