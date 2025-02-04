# Flutter BLoC Architecture Template

Template repository showing the application of the Business Logic Component (BLoC) architectural design pattern to **Flutter** based applications. 

However, the principals and patterns discussed in this repository can be applied to any application independant of frameworks used.

For those of you who dont know, in this context.
```
Architecture is the structure and logical organization of software system
```

## Objective

The purpose of this repository is to provide a solution to the following question:
```
"How should i structure my software projects?"
```
But more specifically the object is the same as any application level architecture
```
"The separation of concerns by structuring code into distinct layers"
```
This seperation of concerns results in code that is:

- maintainable (organised and human readable)
- extendable (i.e. modular (reuseable))
- testable, and
- enables collaboration

And ultimately is one **major** piece in the puzzle towards the ultimate goal of having code which is CLEAN:
```
Clean code is code that is easy to understand and easy to change.
```

I also suggest implenting some good design patterns and principals to help work towards this goal. Such as
- SOLID
- DRY 
- KISS

## Background

Building and maintaining software applications is complicated and hard. Without a good overall structure (architecture) applications can become very hard to change and hard to understand, usually reffered to as "Spaghetti Code". 

This results in **a much higher risk of project failure**. And we definately want to avoid this.

### Option Select Study 

As such a **brief** study was conducted to determine the architectural pattern for Flutter that best fits the Objective of this template. The study was conducted as follows:

- online research to determine the available architectures
- review of each of these architectures
- online research to determine what the community largely thinks is the "best Flutter architecture"
- subjective conclusion from these results

The study can be summarised as follows:

- The following architectures are available for Flutter
    - CLEAN
    - Business Logic Component (BLoC)
    - Onion
    - Hexagonal 
    - Model-View-Controller (MVC)
    - Model-View-View Model (MVVM)
- Due to time constraints only the CLEAN and BLoC architectures were looked at in detail
    - CLEAN - good separation of concerns, however with high levels of complexity
    - BLoC - good separation of concerns, lower levels of complexity, well documented and supported by the community
- The community in general considers BLoC as the better patten for Flutter projects

For these reasons the BLoC pattern was selected as the architectural pattern for this template.

Note: Details of the other architectures considered can be found in the following folder in the repository:

```
./theory/architectures/
```

## Theory

## Business Logic Component (BLoC) Pattern

The BLoC pattern is an architectural design pattern proposed by *Paolo Soares* and *Cong Hui* from Google at DartConf 2018. See it on [YouTube](https://www.youtube.com/watch?v=PLHln7wHgPE&t=126s).

At a very high level it **decouples** (separates) the user interface, business logic and external data sources/systems from each other into 3 separate layers as follows:

- Presention
- Domain (BLoC)
- Data

The overall structure can be best depicted by the following diagram taken from the official [flutter bloc library documentation](https://bloclibrary.dev/#/architecture):

![bloc pattern architecture](./img/bloc_architecture_full.png)

Note:
- Dots on the diagram represent an asynchronous stream of events and states going into and out of the bloc.

A detailed description of each layer is provided below.

## Presentation Layer
```
"The presentation layer's responsibility is everything associated to the user interface."
```
These responsibilites are as follows:
- user interface (pages, widgets, animations)
- passing events to the domain layer (bloc)
- rendering itself based on states returned by the domain (bloc) layer

This layer can only depend on the Bloc Layer.

A sample widget demonstrating the theory is provided as follows:

```dart
class PresentationComponent extends StatelessWidget {
    const PresentationComponent({
        Key? key,
    }) : super(key: key);

    final Bloc bloc = Bloc();

    @override
    Widget build(BuildContext context) {
        return StreamBuilder<BlocState>(
            stream: bloc.stream,
            builder: (BuildContext context, AsyncSnapshot<BlocState> snapshot) {
                // handle connection states
                // ...
                // get state
                BlocState state = snapshot.data;
                // build ui off state
                if (state == LoadingState) {
                    // return loading state
                } else if (state == CompletedState) {
                    // return completed state
                }
            }
        );
    }
}

```

### Widgets / Pages / Animations

All widgets in the presentation layer should follow the composite design pattern outlined in the theory folder of this repository.

A widget is defined as:

```
Individual components that make up the page.
```

A animation is defined as:

```
Individual animations that are used in the page.
```

A page is defined as:

```
a single page/screen a user views, typically the overall layout of the page.
```

A page is comprised of two main parts:

- Page
```
"Responsible for Creating a BLoC and providing it to the View."
```
- View
```
"Responsible for Consuming the state and interacting with the BLoC."
```

It's important to separate or decouple the creation of a BLoC (page) from the consumption of a BLoC (view) in order to have code that is much more testable and reusable. This will allow you to easily inject mock instances and test your view in isolation.

## Domain Layer

```
"Responsibility: All logic independant of the presentation and data layers."
```
The Domain layer is subject to the dependancy rule and cannot depend on the presentation or data layers. Therefore it is the most stable layer of our application and least likely area to change.

This layer is divided into 3 key areas
- entities
- repository interfaces
- blocs

And one optional area:
- value objects

All of which are explained in detail below.

### Entities

The responsibility of the entities are the same as that defined in CLEAN architecture.

```
"An entity can be an object with methods, or it can be a set of data structures and functions. It doesn’t matter **so long as the entities could be used by many different applications**."
```

In the case of BLoC architecture we will mostly use them to represent a data structure.

Additionally entities can be thought of as the inner most layer of our application and therefore and must be **INDEPENDANT** from all other areas in the domain layer.

An example entitity is provided below.
```dart
class RectangleEntity {
  final int base;
  final int height;

  RectangleEntity({
    required this.base,
    required this.height,
  });
}
```

### Repository Interfaces

An interface defines WHAT something does. And not HOW it does it. A repository is somewhere we get data from. Therefore...

```
"The responsibility of the repository interfaces are to define WHAT data the BLoCs require and NOT the implementation."
```

This is what allows the domain Layer to be independant of the data layer.

An example of a repository interface is defined below.

```dart
// I = Interface = Standard nomenclature for noting a software entity as a interface.
abstract class IRepository {
  Future<List<RectangleEntity?>> getAllRectangles();
  Future<RectangleEntity?> getSpecificRectangle();
}
```

### Bloc

```
"The business logic component's responsibility is to respond to events from the presentation layer with new states."
```
i.e. the BLoC contains all the logic to co-ordinate/orchestrate this entire process and as such will be the location where the majority of our domain layer code lives.

A bloc is separated into 3 separate files:
- bloc
- event
```
Events are the input to a Bloc. They are commonly added in response to user interactions. Such as button presses or lifecycle events like page loads.
```
- state
```
The present condition of something / "information" that you need to remember.
```

```dart
// event.dart
abstract class Event {}
class GetRectangle extends Event {}

// state.dart
abstract class State {}
class Error extends State {}
class Loaded extends State {
    final RectangleEntity rectangle;
    const Loaded({required this.rectangle});
}

// bloc.dart
class RectangleBloc {
    final Repository repository;

    Stream mapEventToState(event) async* {
        if (event is GetRectangle) {
            try {
                final RectangleEntity? data = await repository.getSpecificRectangle();
                yield Loaded(data);
            } catch (error) {
                yield Error(error);
            }
        }
        // other event handling...
    }
}
```

### Value Objects

```
A Value Object represents a typed value in your domain. It has three fundamental characteristics: immutability, value equality and self validation. An Age, for example. 
```

Value objects are the fields that make up your (data structure) entities. They are a great way to abstract your validation logic away from your presentation layer.

An example is provided below.

```dart
@immutable
class EmailAddress {
  final String value;

  factory EmailAddress(String input) {
    assert(input != null);
    return EmailAddress._(
      validateEmailAddress(input),
    );
  }

  const EmailAddress._(this.value);

  // toString, equals, hashCode...
}

String validateEmailAddress(String input) {
  // Maybe not the most robust way of email validation but it's good enough
  const emailRegex =
      r"""^[a-zA-Z0-9.a-zA-Z0-9.!#$%&'*+-/=?^_`{|}~]+@[a-zA-Z0-9]+\.[a-zA-Z]+""";
  if (RegExp(emailRegex).hasMatch(input)) {
    return input;
  } else {
    throw InvalidEmailException(failedValue: input);
  }
}

class InvalidEmailException implements Exception {
  final String failedValue;

  InvalidEmailException({@required this.failedValue});
}
```

## Data Layer

```
"The data layer's responsibility is to retrieve/transform data from one or more sources."
```

The data layer can only depend on the domain layer.

The data layer is divided into 3 sections:
- data proividers
- models
- repository implementations

All of which are clearly defined in the sections below.

### Data Providers

```
"The data provider's responsibility is to provide raw data from a single source."
```

The data provider will usually expose simple APIs to perform CRUD operations on a database, REST API, GraphQL API, or other external data source.

Data providers separated into two areas: 

- Local: Data from local device. e.g. cache, local db, etc.
- Remote: Data from an external location. e.g. REST/GraphQL API, SQL/NoSQL DB, etc.

Example code for a data provider is as follows:

```dart
class DataProvider {
    Future<RawData> readData() async {
        // Read directly from DB, make network request etc...
        // return RAW data from the data source
    }
}
```

### Models

```
"The models responsibility is to extend entities with functions that convert data formats and datasource specific functionality."
```

Usually this is done with a mapping function which converts the raw source data data format (e.g. JSON, XML, etc.) to the format expected by the application (entity format) and visa versa. E.g. fromJson, toJson, fromXml and toXml.

```dart
class RectangleModel extends RectangleEntity {

    RectangleModel({
        required this.base,
        required this.height,
    }): super(base: base, height: height);

    factory RectangleModel.fromJson(Map<String, dynamic> json) {
        return RectangleModel(
            base: json['base'] as num,
            height: json['height'] as num,
        );
    }

    Map<String, dynamic> toJson() {
        return {
            'base': base,
            'height': height,
        };
    }
}
```

### Repository Implementations

```
"The repository implementions reposnibility is to implement the repository interface using the data providers and models. The Bloc directly communicates with the repository implementation."
```

```dart
class RectangleRepository implements IRectangleRepository {
    final DataProvider dataProvider;
 
    Future<List<RectangleModel?>> getAllRectangles() async {
        final List<RawData?> rectangles = await dataProvider.readData();
       
        // perform transformations on data. 
        // e.g. filter
        final List<RawData?> filteredRectangles = _filterData(rectangles);
        // and parse
        final List<RectangleModel?> parsedRectangles = RectangleModel.fromMap(filteredRectangles);

        return parsedRectangles;
    }

    // other interface implmentations
    // Future<RectangleEntity?> () async {
    //     
    // }
}
```
## General Design Principals

All functions within the application should be pure functions where possible.

```
pure function is a function where the result of calling it depends only on its arguments.
```

## Decoupling Data Layer and Domain Layer

Given we have decoupled the data layer and the domain layer using repository interfaces and implementations in the architecture above. You may have noticed that this has introduced a new problem for us...

```
How do we get the implementation into the interface?
```

We do this by applying the principals of Inversion of Control (IoC) throught the application of an IoC container. An IoC container...

```
creates an object of a specified class and also injects all the dependency objects through a constructor, a property or a method at run time and disposes it at the appropriate time.
```

So therefore instead of creating instances of BloCs and directly passing the repository implementations. We instead ask the IoC container to create us a BLoC and handle all the dependancy injection.

An example IoC container is provided below:

```dart
abstract class IoCContainer {
    // all requests for a [RectangleRepository] in our app should be replaced by this function
    IRectangleRepository getRectangleRepository() {
        return RectangleRepository();
    }
}
```

## Packages

A package should be created anytime code is intended to be used across projects / applications.

The general theory on how a package should be created is as follows:
- convert all layers of a feature
- convert one or more layers of a feature

For example. If you were converting an Authentication Repository to a package. You would take the entire data layer then the interface, entities and value objects from the domain layer. This would enable the use of the repository across many different projects and also adding different implementations of the repository to the package, i.e. one for AWS, GCP and Azure.

## Implementation

### File Structure

The best way to practically follow and enforce this architecture into your applications is via a well thought out file structure.

The file structure i propose reflects the layers described above and separates each layer by core functionality and feature specific functionality.

A feature can be defined as:
```
A distinguishing characteristic of a software item.
```
But i like to think of it as functionality that your app offers from the perspective of the user.

Examples might include:
- Authentication
- User profile
- Search
- Live feed of posts
- etc.

Features usually align with a page in the application.

Core functionality can be defined as
```
any software which is used accross more than 1 feature in a layer.
```

By isolating own app features in corresponding directories. This enables us to scale as the number of features increases and allows developers to work on different features in parallel.

The file structure is shown below

```
src/
    data/                                       <--- data layer
        core/
            data providers/
                local/
                remote/
            models/
            repository_implementations/
        feature 1/
            data providers/
                local/
                remote/
            models/
            repository_implementations/
        .../
    domain/                                     <--- domain layer
        core/
            bloc/
                bloc.dart
                event.dart
                state.dart
            entities/
            repository_interfaces/
        feature 1/
            bloc/
                bloc.dart
                event.dart
                state.dart
            entities/
            repository_interfaces/
        .../
    presentation/                               <--- presentation layer
        core/
            pages/                              
            widgets/
            animations/
        feature 1/
            pages/
            widgets/
            animations/
        .../
    ioc_container                               <--- inversion of control container
    main.dart                                   <--- entry point for application
packages/
    package_1/
        src/                                    <--- file structure as above
        pubspec.yaml                            <--- package details
    .../
test/                                           <--- all unit and widget tests, file structure as per src folder
test_driver/                                    <--- integration test folder
    app.dart                                    <--- configure application and drivers for testing
    app_test.dart                               <--- integration tests
```

If you find that this file structure does not fully support your requirements some additional folders that may be added are suggested below:
```
constants/                  <-- layer specific constants such as API keys, routes, urls, etc.
utils/                      <-- layer specific useful functionality that does not fit into outher folders. i.e. misc functionality.
```

### BLoC Library

The Bloc library is an implementation of the BLoC pattern created by Felix Angelov. 

You can review it in detail in the following [link](https://github.com/felangel/bloc/).

This librabry is a great way to help enforce implementation of the BLoC pattern and help ensure best practices are used for the software files created in each layer.

Some of the major benefits of this library include:
- immutable (unchangeable) states which means a history of states are kept so you can roll back states at any point in time.

### Get_It Package

The Get_It package is a implementation of IoC using the service locator architectural pattern.

You can review it in detail in the following [link](https://pub.dev/packages/get_it).

This library is a great way for us to enforce the use of IoC and ensure best practice is followed for the IoC container itself.

# Testing

Testing tests a function, widget or application operates as expected. It is used main benefits are:
- higher confidence in code
- documenting of code operation

## Architecture

The overall structure of for writing tests in flutter is represented below:

```dart
void main() {
    group('some group name',() {

        /// SETUP

        // Registers a function to be run once before all tests
        setUpAll(() async {
            
        });

        // Registers a function to be run once after all tests
        tearDownAll(() async {
            
        });

        // any other global setup configuration required

        // test naming should following the GIVEN, WHEN, THEN convention
        // GIVEN - some context
        // WHEN - some event happens
        // THEN - some result is expected
        test('name', () async {
            // function structure should use the ARRANGE, ACT, ASSERT convention
            
            // ARRANGE - Setup everything to be used by the test
            // E.g. create Mock
            //      configure how Mock responds when certain functions are called.

            // ACT - Call the class or function to be tested and store the result

            // ASSERT - Check the function output matches expected values
            expect(actualResult, expectedResult);
        });
    });
}
```

## Unit Tests

```
Tests a single function OR class operates as expected.
```

```dart
void main() {
    /// UNIT Test
    test('some name', () async {
        // Structure as above
    }); 
}
```

## Widget Tests

```
Tests a single widget looks and behaves as expected.
```

```dart
void main() {
    // WIDGET Test
    testWidgets('name', (WidgetTester tester) async {
        // WidgetTester can be thought of as the user

        // ARRANGE / ASSEMBLE
        
        // Render the page
        await tester.pumpWidget(
            MaterialApp(
                /// Widget to be tested
                home: MyHomePage(),
            ),
        ); 

        // Find widgets to be tested
        final button = find.byType(FloatingActionButton)

        /// ACT
        await tester.tap(button); // tap the button
        await tester.pump(); // re-render the widget
        final actualText = find.text('some text in a widget');

        /// ASSERT
        expect(actualText, findsOneWidget);
    });
}
```

## Integration Tests

```
Tests a large part or entire application to verify that multiple functions, classes, widgets and services operate as expected.
```

## app.dart

app.dart file configures the test driver and application to be tested.

```dart
import 'package:flutter_driver/driver_extension.dart'; // integration testing library
import '../lib/main.dart' as app; // import entire flutter application

void main() {
    // enable integration testing library
    enableFlutterDriverExtension();
    // run flutter application
    app.main();
}
```

## app_test.dart

app_test.dart performs all the integration tests on the configured app and driver.

```dart
import 'package:flutter_driver/flutter_driver.dart'; // import drivers from testing library

void main() {
    group('some name', () {
        
        /// SETUP

        /// Drives a Flutter Application running in another process
        FlutterDriver driver;

        setUpAll(() async {
            driver = await FlutterDriver.connect();
        });

        tearDownAll(() async {
            if (driver != null) {
                driver.close();
            }
        });

        final button = find.byType('FloatingActionButton');

        test('some name', () {
            /// ARRANGE - N/A

            /// ACT
            await driver.tap(button);
            final textWidget = find.text('some text');
            final actualText = await driver.getText(textWidget);

            /// ASSERT
            expect(actualText, 'someText');
        });
    });
}
```