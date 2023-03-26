# BLOC_Flutter
3 Example to show how we can implement the BLOC or Business Logic Component in our Flutter application.

The goal of this package is to make it easy to implement the BLoC Design Pattern (Business Logic Component).

This design pattern helps to separate presentation from business logic. Following the BLoC pattern facilitates testability and reusability. This package abstracts reactive aspects of the pattern allowing developers to focus on writing the business logic.

## CUBIT

![image](https://user-images.githubusercontent.com/63160825/227774033-4e2f8024-411d-4597-aa98-599c9437fcf6.png)

A Cubit is class which extends BlocBase and can be extended to manage any type of state. Cubit requires an initial state which will be the state before emit has been called. The current state of a cubit can be accessed via the state getter and the state of the cubit can be updated by calling emit with a new state.

![image](https://user-images.githubusercontent.com/63160825/227774044-84f5366c-1177-4c27-ac06-c710fb22e88e.png)

State changes in cubit begin with predefined function calls which can use the emit method to output new states. onChange is called right before a state change occurs and contains the current and next state.
