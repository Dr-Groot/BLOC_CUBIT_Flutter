# BLOC_Flutter

Example to show how we can implement the BLOC or Business Logic Component in our Flutter application.

The goal of this package is to make it easy to implement the BLoC Design Pattern (Business Logic Component).

This design pattern helps to separate presentation from business logic. Following the BLoC pattern facilitates testability and reusability. This package abstracts reactive aspects of the pattern allowing developers to focus on writing the business logic.

## BLOC

![image](https://user-images.githubusercontent.com/63160825/227774126-50756e18-6af0-4239-90f7-e7b4430410ea.png)

A Bloc is a more advanced class which relies on events to trigger state changes rather than functions. Bloc also extends BlocBase which means it has a similar public API as Cubit. However, rather than calling a function on a Bloc and directly emitting a new state, Blocs receive events and convert the incoming events into outgoing states.

![image](https://user-images.githubusercontent.com/63160825/227774136-5e3f7bd0-df14-4ecf-8866-fbff47f662d3.png)

State changes in bloc begin when events are added which triggers onEvent. The events are then funnelled through an EventTransformer. By default, each event is processed concurrently but a custom EventTransformer can be provided to manipulate the incoming event stream. All registered EventHandlers for that event type are then invoked with the incoming event. Each EventHandler is responsible for emitting zero or more states in response to the event. Lastly, onTransition is called just before the state is updated and contains the current state, event, and next state.

## CUBIT

![image](https://user-images.githubusercontent.com/63160825/227774033-4e2f8024-411d-4597-aa98-599c9437fcf6.png)

A Cubit is class which extends BlocBase and can be extended to manage any type of state. Cubit requires an initial state which will be the state before emit has been called. The current state of a cubit can be accessed via the state getter and the state of the cubit can be updated by calling emit with a new state.

![image](https://user-images.githubusercontent.com/63160825/227774044-84f5366c-1177-4c27-ac06-c710fb22e88e.png)

State changes in cubit begin with predefined function calls which can use the emit method to output new states. onChange is called right before a state change occurs and contains the current and next state.

## Example 1

[Download Example 1 Files](https://github.com/Dr-Groot/BLOC_Flutter/blob/main/Example_1.zip)

First comes **Cubit** :
It contains the initial state as 0 and two methods for increment and decrement, here int is acting as a state.

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  /// Add 1 to the current state.
  void increment() => emit(state + 1);

  /// Subtract 1 from the current state.
  void decrement() => emit(state - 1);
}
```

**Bloc Provider** :
Here we are mentioning the initial state, which is 0 in Cubit.

```dart
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => CounterCubit(),
      child: const CounterView(),
    );
  }
```

**Bloc Builder** :

```dart
  child: BlocBuilder<CounterCubit, int>(
    builder: (context, state) {
      return Text('$state', style: textTheme.displayMedium);
    },
  ),
```

We can call cubit function by:
```dart
context.read<CounterCubit>().increment()
///OR
context.read<CounterCubit>().decrement()
```


## Example 2

[Download Example 2 Files](https://github.com/Dr-Groot/BLOC_Flutter/blob/main/Example_2.zip)

**Cubit**:
We are emiting List of Post as state, initial state is empty list and getPosts function helps us to get data from the network request and emit as a
list of Post.

```dart
class PostCubit extends Cubit<List<Post>> {
  final _dataService = DataService();

  PostCubit(): super([]);

  void getPosts() async => emit(await _dataService.getPost());
}
```

**Bloc Provide** :
If we want to call any function in cubit when initial state is called by adding three dots after class name and calling the function.

```dart
home: BlocProvider<PostCubit>(
          create: (context) => PostCubit()..getPosts(),
          child: PostView()
      ),
```

**Bloc Builder** :

```dart
body: BlocBuilder<PostCubit, List<Post>>(
        builder: (context, posts) {
          if (posts.isEmpty) {
            return const Center(
              child: CircularProgressIndicator(),
            );
          }

          return ListView.builder(itemBuilder: (context, index) {
            return Card(
              child: ListTile(
                title: Text(posts[index].title ?? ""),
              ),
            );
          });
        },
      ),
```


## Example 3

[Download Example 2 Files](https://github.com/Dr-Groot/BLOC_Flutter/blob/main/Example_3.zip)

In this example we have a state in a separate file.

**Cubit**: 

In this we have a HeroState as a state. We can call a function at a initial state by adding the function like this:

```dart
class HeroCubit extends Cubit<HeroState> {
  final HeroNetwork _heroNetwork;

  HeroCubit(this._heroNetwork) : super(HeroLoadingState()) {
    fetchHeroList();
  }

  void fetchHeroList() async {
    emit(HeroLoadingState());
    try {
      final heros = await _heroNetwork.getHeroData();
      emit(HeroLoadedState(heros));
    } catch (e) {
      emit(HeroErrorState());
    }
  }

  void fetchOneName() {
    emit(HeroDemoState([HeroModel(name: 'Big', localisedName: 'Bee')]));
  }

}
```

**State:**

```dart
@immutable
abstract class HeroState extends Equatable  {}

class HeroLoadingState extends HeroState {

  @override
  List<Object?> get props => [];
}

class HeroLoadedState extends HeroState {
  final List<HeroModel> heros;

  HeroLoadedState(this.heros);

  @override
  List<Object?> get props => [heros];
}

class HeroErrorState extends HeroState {

  @override
  List<Object?> get props => [];
}

class HeroDemoState extends HeroState {
  final List<HeroModel> heros;

  HeroDemoState(this.heros);

  @override
  List<Object?> get props => [heros];

}
```

**Bloc Provider:**
We can mention the cubit name at starting of BlocProvider, which helps us to indetify for which cubit it works.

```dart
home: BlocProvider<HeroCubit>(
        create: (BuildContext context) => HeroCubit(HeroNetwork()),
          child: HeroView()
      ),
```

**Bloc Builder:**
Now we can show the UI for different state.

```dart
body: BlocBuilder<HeroCubit, HeroState>(
        builder: (context, state) {

          if (state is HeroLoadingState) {
            return const Center(
              child: CircularProgressIndicator(),
            );
          }

          if (state is HeroLoadedState) {
           List<HeroModel> heroList = state.heros;
           heroList.forEach((element) {
             if (element.localisedName == 'Axe') {
               debugPrint('AXE FOUND');
             }
           });
           // context.read<HeroCubit>().fetchOneName();
           return ListView.builder(
               itemCount: heroList.length,
               itemBuilder: (_, index) {
                 return Card(
                   color: Colors.black,
                   child: ListTile(
                     title: Text(
                       '${heroList[index].localisedName}',
                       style:
                       const TextStyle(color: Colors.white, fontSize: 17.0),
                     ),
                     subtitle: Text(
                       '${heroList[index].name}',
                       style: const TextStyle(color: Colors.red),
                     ),
                   ),
                 );
               });
          }

          if (state is HeroErrorState) {
            return Center(
              child: Container(
                child: const Text('Error')
              ),
            );
          }

          if (state is HeroDemoState) {
            List<HeroModel> demoList = state.heros;
            return Center(
              child: Container(
                child: Text('${demoList[0].name}', style: TextStyle(color: Colors.white),),
              ),
            );
          }

          return Container();
        },
      ),
```




