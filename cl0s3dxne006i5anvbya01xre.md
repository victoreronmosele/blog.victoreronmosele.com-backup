## Mocking and Testing Firestore Operations in Flutter Unit Tests | Part 2 (Transactions and Batched Writes)

# Introduction

Firestore offers the ability to perform atomic operations — ["series of database operations such that either all occurs, or nothing occurs"](https://en.wikipedia.org/wiki/Atomicity_(database_systems)) — in form of Transactions and Batched Writes.

> Transactions: a transaction is a set of read and write operations on one or more documents.
>
> Batched Writes: a batched write is a set of write operations on one or more documents.
>
> Source: [Firebase Documentation for Transactions and Batched Writes](https://firebase.google.com/docs/firestore/manage-data/transactions)

This article examines how to write unit tests for Firestore Transactions and Batched Writes in Flutter. 

We continue from the last article: [Mocking and Testing Firestore Operations in Flutter Unit Tests | Part 1 (Documents and Collections)](https://blog.victoreronmosele.com/mocking-firestore-flutter) where we looked at how to implement unit tests for Firestore operations involving Documents and Collections.

The article was created using `Flutter Channel stable, 2.8.1`.

# Prerequisites

The prerequisites are the same as in the previous article.

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
We extend our `FirestoreService` class from the previous article — which performed read and write operations to Firestore using Documents and Collections — to perform read and write operations to Firestore using Transactions and Batched Writes and we want to write a set of unit tests for this class.

The class is shown below (with the Document and Collection part removed for readability):
```dart
import 'package:cloud_firestore/cloud_firestore.dart';

class FirestoreService {
  const FirestoreService({required this.firestore});

  final FirebaseFirestore firestore;

  // Document and Collection operations removed for readability

  Future<void> runTransaction({
    required Map<String, dynamic> dataToUpdate,
    required Map<String, dynamic> dataToSet,
    required String collectionPath,
    required String documentPathToUpdate,
    required String documentPathToSetTo,
    required String documentPathToDelete,
  }) async {
    return firestore.runTransaction<void>((transaction) async {
      final DocumentReference documentReferenceToUpdate =
          firestore.collection(collectionPath).doc(documentPathToUpdate);
      final DocumentReference documentReferenceToSetTo =
          firestore.collection(collectionPath).doc(documentPathToSetTo);
      final DocumentReference documentReferenceToDelete =
          firestore.collection(collectionPath).doc(documentPathToDelete);

      final DocumentSnapshot documentSnapshotForUpdate =
          await transaction.get(documentReferenceToUpdate);

      final Map<String, dynamic> dataInDocumentPathToUpdate =
          documentSnapshotForUpdate.data() as Map<String, dynamic>;

      final Map<String, dynamic> updatedData = {
        ...dataInDocumentPathToUpdate,
        ...dataToUpdate
      };

      transaction.update(documentReferenceToUpdate, updatedData);

      transaction.set(documentReferenceToSetTo, dataToSet);

      transaction.delete(documentReferenceToDelete);
    });
  }

  Future<void> runBatchedWrite({
    required Map<String, dynamic> dataToSet,
    required Map<String, dynamic> dataToUpdate,
    required String collectionPath,
    required String documentPathToSetTo,
    required String documentPathToUpdate,
    required String documentPathToDelete,
  }) async {
    final WriteBatch writeBatch = firestore.batch();

    final CollectionReference collectionReference =
        firestore.collection(collectionPath);

    final DocumentReference documentReferenceToSetTo =
        collectionReference.doc(documentPathToSetTo);
    final DocumentReference documentReferenceToUpdate =
        collectionReference.doc(documentPathToUpdate);
    final DocumentReference documentReferenceToDelete =
        collectionReference.doc(documentPathToDelete);

    writeBatch.set(documentReferenceToSetTo, dataToSet);

    writeBatch.update(documentReferenceToUpdate, dataToUpdate);

    writeBatch.delete(documentReferenceToDelete);

    await writeBatch.commit();
  }
}
```

# Understanding The Methods

This section helps understand the `runTransaction` and the `runBatchedWrite` methods with regards to what is expected for unit testing them.

### runTransaction

```dart
 Future<void> runTransaction({
    required Map<String, dynamic> dataToUpdate,
    required Map<String, dynamic> dataToSet,
    required String collectionPath,
    required String documentPathToUpdate,
    required String documentPathToSetTo,
    required String documentPathToDelete,
  }) async {
    return firestore.runTransaction<void>((transaction) async {
      final DocumentReference documentReferenceToUpdate =
          firestore.collection(collectionPath).doc(documentPathToUpdate);
      final DocumentReference documentReferenceToSetTo =
          firestore.collection(collectionPath).doc(documentPathToSetTo);
      final DocumentReference documentReferenceToDelete =
          firestore.collection(collectionPath).doc(documentPathToDelete);

      final DocumentSnapshot documentSnapshotForUpdate =
          await transaction.get(documentReferenceToUpdate);

      final Map<String, dynamic> dataInDocumentPathToUpdate =
          documentSnapshotForUpdate.data() as Map<String, dynamic>;

      final Map<String, dynamic> updatedData = {
        ...dataInDocumentPathToUpdate,
        ...dataToUpdate
      };

      transaction.update(documentReferenceToUpdate, updatedData);

      transaction.set(documentReferenceToSetTo, dataToSet);

      transaction.delete(documentReferenceToDelete);
    });
  }
```

The `runTransaction` method demonstrates how to do transactions in Firestore. Since transactions allow you to perform read operations and writes operations, the `runTransaction` method has both read and write operations in it.

The major operations in this method are below:
- The transaction reads the data at `documentPathToUpdate` and merges it with `dataToUpdate` and updates the document there.
- The transaction sets `dataToSet` to the document at the `documentPathToSetTo`.
- The transaction deletes the document at `documentPathToDelete`.

### runBatchedWrite

 ```dart
  Future<void> runBatchedWrite({
    required Map<String, dynamic> dataToSet,
    required Map<String, dynamic> dataToUpdate,
    required String collectionPath,
    required String documentPathToSetTo,
    required String documentPathToUpdate,
    required String documentPathToDelete,
  }) async {
    final WriteBatch writeBatch = firestore.batch();

    final CollectionReference collectionReference =
        firestore.collection(collectionPath);

    final DocumentReference documentReferenceToSetTo =
        collectionReference.doc(documentPathToSetTo);
    final DocumentReference documentReferenceToUpdate =
        collectionReference.doc(documentPathToUpdate);
    final DocumentReference documentReferenceToDelete =
        collectionReference.doc(documentPathToDelete);

    writeBatch.set(documentReferenceToSetTo, dataToSet);

    writeBatch.update(documentReferenceToUpdate, dataToUpdate);

    writeBatch.delete(documentReferenceToDelete);

    await writeBatch.commit();
  }
```

The `runBatchedWrite` method demonstrates how to perform batched writes in Firestore. Since batched writes allow you to perform only write operations, the `runBatchedWrite` method contains only write operations.

The major operations in this method are below:
- The batched write updates the document at `documentPathToUpdate` with `dataToUpdate`.
- The batched write  sets `dataToSet` to the document at the `documentPathToSetTo`.
- The batched write deletes the document at `documentPathToDelete`.


# Testing Approach

Just as it was outlined in the [previous article](https://blog.victoreronmosele.com/mocking-firestore-flutter#heading-testing-approach), we do the following to test the methods:

- *Construction Injection*: 

   The `FakeFirebaseFirestore` object is passed into the `FirestoreService` instance to mock the `FirebaseFirestore` object.

- *Mocking Strategy & Database State Set Up*: 

   Data is added to specific document/collection paths in the `FakeFirebaseFirestore` object before carrying out the operation in the `FirestoreService` class. 

   Data is then retrieved from the `FakeFirebaseFirestore` object at specific paths to compare them with the expected values by asserting that they are the same.


The two methods above, individually, contain different read and/or write operations and those are the operations that will be tested.

The tests will be written in the same test file from the [previous article](https://blog.victoreronmosele.com/mocking-firestore-flutter#heading-testing-approach)

# Testing the Transactions and Batched Writes class

### Test For Running A Transaction

#### *runTransaction's Test*

The runTransaction method can be tested by:

- Injecting the mock Firestore object, `fakeFirebaseFirestore`, to the service.
- Prepopulating the document at `documentPathToUpdate` via the fakeFirebaseFirestore object with the `data` object.
- Prepopulating the document at `documentPathToDelete` via the fakeFirebaseFirestore object with the `data` object.
- Running the `runTransaction` method and passing `dataToUpdateWith` and `dataToSet` which will be used by the method to write to `documentPathToUpdate` and `documentPathToSetTo`.
- Retrieving the actual data contained in the `documentPathToUpdate`, `documentPathToSetTo` and `documentPathToDelete` document paths.
- Asserting that the data in `documentPathToUpdate` is equal to the expected updated data, `expectedUpdatedData` i.e `dataToUpdateWith` merged with `data`.
- Asserting that the data in `documentPathToSetTo` is equal to the expected set data i.e `dataToSet`.
- Asserting that the data in `documentPathToDelete` is null.

This is shown below:

```dart
 test('runTransaction runs the correct transaction operation', () async {
  final FirestoreService firestoreService =
      FirestoreService(firestore: fakeFirebaseFirestore!);

  const String collectionPath = 'collectionPath';
  const String documentPathToUpdate = 'documentPathToUpdate';
  const String documentPathToSetTo = 'documentPathToSetTo';
  const String documentPathToDelete = 'documentPathToDelete';

  final CollectionReference collectionReference =
      fakeFirebaseFirestore!.collection(collectionPath);

  final DocumentReference documentReferenceToUpdate =
      collectionReference.doc(documentPathToUpdate);
  final DocumentReference documentReferenceToSetTo =
      collectionReference.doc(documentPathToSetTo);
  final DocumentReference documentReferenceToDelete =
      collectionReference.doc(documentPathToDelete);

  documentReferenceToUpdate.set(data);
  documentReferenceToDelete.set(data);

  const Map<String, dynamic> dataToUpdateWith = {'updated_data': '43'};
  const Map<String, dynamic> dataToSet = {'data': 44};

  const Map<String, dynamic> expectedUpdatedData = {
    ...data,
    ...dataToUpdateWith
  };

  await firestoreService.runTransaction(
      dataToUpdate: dataToUpdateWith,
      dataToSet: dataToSet,
      collectionPath: collectionPath,
      documentPathToUpdate: documentPathToUpdate,
      documentPathToSetTo: documentPathToSetTo,
      documentPathToDelete: documentPathToDelete);

  final DocumentSnapshot documentSnapshotForUpdate =
      await documentReferenceToUpdate.get();
  final DocumentSnapshot documentSnapshotForSet =
      await documentReferenceToSetTo.get();
  final DocumentSnapshot documentSnapshotForDelete =
      await documentReferenceToDelete.get();

  final actualDataFromUpdate = documentSnapshotForUpdate.data();
  final actualDataFromSet = documentSnapshotForSet.data();
  final actualDataFromDelete = documentSnapshotForDelete.data();

  expect(actualDataFromUpdate, expectedUpdatedData);
  expect(actualDataFromSet, dataToSet);
  expect(actualDataFromDelete, null);
});
```

### Test For Running A Batched Write

#### *runBatchedWrite's Test*

The runBatchedWrite method can be tested by:

- Injecting the mock Firestore object, `fakeFirebaseFirestore`, to the service.
- Prepopulating the document at `documentPathToUpdate` via the fakeFirebaseFirestore object with the `data` object.
- Prepopulating the document at `documentPathToDelete` via the fakeFirebaseFirestore object with the `data` object.
- Running the `runBatchedWrite` method and passing `dataToUpdateWith` and `dataToSet` which will be used by the method to write to `documentPathToUpdate` and `documentPathToSetTo`.
- Retrieving the actual data contained in the `documentPathToUpdate`, `documentPathToSetTo` and `documentPathToDelete` document paths.
- Asserting that the data in `documentPathToUpdate` is equal to the expected updated data, `expectedUpdatedData` i.e `dataToUpdateWith` merged with `data`.
- Asserting that the data in `documentPathToSetTo` is equal to the expected set data i.e `dataToSet`.
- Asserting that the data in `documentPathToDelete` is null.

This is shown below:

```dart
 test('runBatchedWrite runs the correct batched write operation',
    () async {
  final FirestoreService firestoreService =
      FirestoreService(firestore: fakeFirebaseFirestore!);
  const String collectionPath = 'collectionPath';
  const String documentPathToUpdate = 'documentPathToUpdate';
  const String documentPathToSetTo = 'documentPathToSetTo';
  const String documentPathToDelete = 'documentPathToDelete';

  final CollectionReference collectionReference =
      fakeFirebaseFirestore!.collection(collectionPath);

  final DocumentReference documentReferenceToUpdate =
      collectionReference.doc(documentPathToUpdate);
  final DocumentReference documentReferenceToSetTo =
      collectionReference.doc(documentPathToSetTo);
  final DocumentReference documentReferenceToDelete =
      collectionReference.doc(documentPathToDelete);

  documentReferenceToUpdate.set(data);
  documentReferenceToDelete.set(data);

  const Map<String, dynamic> dataToUpdateWith = {'updated_data': '43'};
  const Map<String, dynamic> dataToSet = {'data': 44};

  const Map<String, dynamic> expectedUpdatedData = {
    ...data,
    ...dataToUpdateWith
  };

  await firestoreService.runBatchedWrite(
      dataToUpdate: dataToUpdateWith,
      dataToSet: dataToSet,
      collectionPath: collectionPath,
      documentPathToUpdate: documentPathToUpdate,
      documentPathToSetTo: documentPathToSetTo,
      documentPathToDelete: documentPathToDelete);

  final DocumentSnapshot documentSnapshotForUpdate =
      await documentReferenceToUpdate.get();
  final DocumentSnapshot documentSnapshotForSet =
      await documentReferenceToSetTo.get();
  final DocumentSnapshot documentSnapshotForDelete =
      await documentReferenceToDelete.get();

  final actualDataFromUpdate = documentSnapshotForUpdate.data();
  final actualDataFromSet = documentSnapshotForSet.data();
  final actualDataFromDelete = documentSnapshotForDelete.data();

  expect(actualDataFromUpdate, expectedUpdatedData);
  expect(actualDataFromSet, dataToSet);
  expect(actualDataFromDelete, null);
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

    // Document operations and Collection operations tests removed for readability

    group('Transaction Operation', () {
      test('runTransaction runs the correct transaction operation', () async {
        final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);

        const String collectionPath = 'collectionPath';
        const String documentPathToUpdate = 'documentPathToUpdate';
        const String documentPathToSetTo = 'documentPathToSetTo';
        const String documentPathToDelete = 'documentPathToDelete';

        final CollectionReference collectionReference =
            fakeFirebaseFirestore!.collection(collectionPath);

        final DocumentReference documentReferenceToUpdate =
            collectionReference.doc(documentPathToUpdate);
        final DocumentReference documentReferenceToSetTo =
            collectionReference.doc(documentPathToSetTo);
        final DocumentReference documentReferenceToDelete =
            collectionReference.doc(documentPathToDelete);

        documentReferenceToUpdate.set(data);
        documentReferenceToDelete.set(data);

        const Map<String, dynamic> dataToUpdateWith = {'updated_data': '43'};
        const Map<String, dynamic> dataToSet = {'data': 44};

        const Map<String, dynamic> expectedUpdatedData = {
          ...data,
          ...dataToUpdateWith
        };

        await firestoreService.runTransaction(
            dataToUpdate: dataToUpdateWith,
            dataToSet: dataToSet,
            collectionPath: collectionPath,
            documentPathToUpdate: documentPathToUpdate,
            documentPathToSetTo: documentPathToSetTo,
            documentPathToDelete: documentPathToDelete);

        final DocumentSnapshot documentSnapshotForUpdate =
            await documentReferenceToUpdate.get();
        final DocumentSnapshot documentSnapshotForSet =
            await documentReferenceToSetTo.get();
        final DocumentSnapshot documentSnapshotForDelete =
            await documentReferenceToDelete.get();

        final actualDataFromUpdate = documentSnapshotForUpdate.data();
        final actualDataFromSet = documentSnapshotForSet.data();
        final actualDataFromDelete = documentSnapshotForDelete.data();

        expect(actualDataFromUpdate, expectedUpdatedData);
        expect(actualDataFromSet, dataToSet);
        expect(actualDataFromDelete, null);
      });
    });

    group('Batched Write Operation', () {
      test('runBatchedWrite runs the correct batched write operation',
          () async {
        final FirestoreService firestoreService =
            FirestoreService(firestore: fakeFirebaseFirestore!);
        const String collectionPath = 'collectionPath';
        const String documentPathToUpdate = 'documentPathToUpdate';
        const String documentPathToSetTo = 'documentPathToSetTo';
        const String documentPathToDelete = 'documentPathToDelete';

        final CollectionReference collectionReference =
            fakeFirebaseFirestore!.collection(collectionPath);

        final DocumentReference documentReferenceToUpdate =
            collectionReference.doc(documentPathToUpdate);
        final DocumentReference documentReferenceToSetTo =
            collectionReference.doc(documentPathToSetTo);
        final DocumentReference documentReferenceToDelete =
            collectionReference.doc(documentPathToDelete);

        documentReferenceToUpdate.set(data);
        documentReferenceToDelete.set(data);

        const Map<String, dynamic> dataToUpdateWith = {'updated_data': '43'};
        const Map<String, dynamic> dataToSet = {'data': 44};

        const Map<String, dynamic> expectedUpdatedData = {
          ...data,
          ...dataToUpdateWith
        };

        await firestoreService.runBatchedWrite(
            dataToUpdate: dataToUpdateWith,
            dataToSet: dataToSet,
            collectionPath: collectionPath,
            documentPathToUpdate: documentPathToUpdate,
            documentPathToSetTo: documentPathToSetTo,
            documentPathToDelete: documentPathToDelete);

        final DocumentSnapshot documentSnapshotForUpdate =
            await documentReferenceToUpdate.get();
        final DocumentSnapshot documentSnapshotForSet =
            await documentReferenceToSetTo.get();
        final DocumentSnapshot documentSnapshotForDelete =
            await documentReferenceToDelete.get();

        final actualDataFromUpdate = documentSnapshotForUpdate.data();
        final actualDataFromSet = documentSnapshotForSet.data();
        final actualDataFromDelete = documentSnapshotForDelete.data();

        expect(actualDataFromUpdate, expectedUpdatedData);
        expect(actualDataFromSet, dataToSet);
        expect(actualDataFromDelete, null);
      });
    });
  });
}

// MapListContains Matcher code removed for readability
```


# Conclusion

This article demonstrates how to write unit tests for Firebase Firestore's atomic operations, that is, Transactions and Batched Writes. 

The complete code can be found on [GitHub](https://github.com/victoreronmosele/firestore-unit-test-flutter).