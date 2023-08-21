# Guess Word #
Guess Word is a multiplayer game in which a random word will be displayed 
on the screen, and one of the players will enact that word, the other player has to guess that word. For every correct guess the player gets 1 point and skipping the word decreases the score by 1. The player has to guess the word within 2 minutes, else the game ends and at the end the total score is shown.

When the device configuration changes, the the data in the activity/fragment is lost, as the activity/fragment is recreated. This issue can be addressed by saving data in bundle in `onSaveInstanceState` callback. But it has some limitaions, like extra logic to save and retrieve the data and limited size. Hence, in this project a better way is used to solve this issue using Lifecycle Library's architecture components called ViewModel and LiveData.

Note : To see how this problem can be solved using `onSaveInstanceState`, please refer this [Dessert Pusher project](https://github.com/pawanharariya/Dessert-Pusher).

## App Preview ##
<img alt="Guess Word preview 1" src="https://github.com/pawanharariya/Guess-Word/assets/43620548/e44bb7c3-2632-47c8-b378-10f025b37642" width="220" >
<img alt="Guess Word preview 2" src="https://github.com/pawanharariya/Guess-Word/assets/43620548/d71a20ce-ac60-44c5-abbc-249fa6aaa6d1)" width="220">
<img alt="Guess Word preview 3" src="https://github.com/pawanharariya/Guess-Word/assets/43620548/71a0133c-bc5b-4a99-b243-089e1e7b4402" height="250">

## Application Architecture ##
It is a way of designing an application's classes and relationships between them. This makes code organised, modular, debuggable, easily manageable and testable. However, there isn't a perfect architectural style for all needs, and it depends on circumstances and needs. Details of our app's architecture are given below:

* This app is based on MVVM (Model-View ViewModel) architecture. 

* It follows software principle called **Separation of Concerns**, in which we divide the code into classes, each with separate responsibilities.

* It has three types of classes, UI controller, ViewModel and LiveData.

* The UI controller is responsible for user interface related tasks, like displaying views and capturing input. We use UI controller only for UI related takes and not for any decision-making. For eg. we display the text on screen using UI controller, but what text is displayed is not controlled by it. Examples for UI controller are activity and fragment classes.

* The ViewModel is responsible for decision making. It holds the data to be displayed in activity or fragment. It can also perform calculations on the data. So, when a button is pressed, the UI controller is notified, but it simply passes the information to the ViewModel. ViewModel contains instances of LiveData class to hold the data. 

* LiveData holds the actual data and is responsible for communicating information for the ViewModel to the UI controller, that the screen should be updated or recreated.

* MVVM architecture has other components as well like Repository, Database and Network which are useful in case we have databases and backend. These classes are not used in this project.

## View Model ##
View Model is an abstract class of Lifecycle Library. It holds app's UI data and it can survive configuration changes. Unlike, onSaveInstanceState bundle, it have no size limitations. ViewModels are made for and associated with a single fragment or activity.
 
ViewModel get destroyed when the fragment or activity that ViewModel is associated with, is finally destroyed and `onCleared()` callback of ViewModel is called before it.

To use the ViewModel, we make a reference to it in our fragment. However, we never create ViewModel ourselve because in that case, ViewModel would be created every time fragment is created. So, we request the ViewModel from `ViewModelProvider` class and the Lifecycle library creates the ViewModel for us. When we first request the ViewModel from provider, it creates a new instance of our ViewModel. And later, when we reference to it again it provides the existing ViewModel.

**Note:** The ViewModel should never contain references to fragments, activities or views. As these get destroyed with configuration changes, and referencing something which is destroyed, can lead to errors. Since ViewModels don't contain references to activities or fragments, we can easily test and write unit tests that don't require Android framework.

### Limitations of the ViewModel ###
Although, the ViewModel solves the issues related to configuration changes, it has no way to communicate with the UI controller. This would be required in case some data is changed in ViewModel and we need to update the UI. Since it should not contain references to the fragments or activities, we use another lifecycle component called LiveData, to communicate back to UI controller.

## LiveData ##
LifeData is an observable data holder class that is lifecycle-aware. This means that UI controllers can observe the LiveData for changes in its state, the state changes when the data wrapped by LiveData changes. LiveData helps the ViewModel to communicate data changes back to the UI controller. 

LiveData is lifecycle aware which means it knows about lifecycle states of its UI controller observers. This helps in updating UI efficiently. Following are some feature of LiveData, that use lifecycle awareness.

1. LiveData only updates UI controllers that are actually on-screen. So, if the fragment is off-screen and value in LiveData changes, UI is not updated. When the UI controller is back on-screen, LiveData triggers the observer with most recent data. 

2. When a LiveData already exists with some data, and new UI controller starts observing it, it gets the current data immediately.

3. If the UI controller is destroyed, LiveData internally cleans up its own connection to observer. Hence we don't need to clean up the observation connection to LiveData in our `onDestroy()` callback. So basically, UI controller observers the LiveData for its data, and LiveData observes the UI controller for its lifecycle state.

## Event vs State ##
LiveData shouldn't be used for navigation. Consider a case in which, some data changes in ViewModel and based on it we want UI controller to navigate to another screen. Since ViewModel cannot reference the UI controller, it cannot do it. We can however maintain a state for navigating and use LiveData to communicate to UI controller when the state changes. But navigation is not a state, but an event and LiveData should be used only for data states.

**State** - State is current value of something. The state is retained until is changes. eg. score = 20, button_color = red.

**Event** - Events are one-time thing. An event happens once and it's done until triggered again. eg. pressing a button to navigate. 

There shouldn't be a state saying `navigating_to_final_screen`, because we navigate once and then we are done navigating. But for state, the data is retained until state changes, like once the score is 20, it should remain 20.

## Handling events using LiveData ## 
We still use LiveData to handle events, but in careful way. LiveData handles state, and we use to it represent the state of an event. We use LiveData with a `Boolean` data type which represents whether the event has occurred or not and then observe it in our UI controller. However once the event has been handled by the UI controller, we must reset the state back in our ViewModel, to avoid bugs.

When an event happened and the UI controller is notified, it handles it. Now suppose, the UI controller gets recreated due to configuration change, in that case it gets the current data of LiveData, which says the event happened. However, the event had already happened before. So, to avoid such bugs, the state should be reset after the event is handled.

## ViewModel Factory ##
Suppose we need to pass in some data to a ViewModel during initialization. To do this we can use a setter property or create a constructor in our ViewModel. However, we don't create ViewModels, it is provided to us by the ViewModelProvider. So to have a ViewModel with constructor, we use a ViewModel Factory, that creates ViewModels for us. We can pass in whatever data we need in our ViewModel, to this factory. Then we pass this factory to our ViewModelProvider, so that we get a ViewModel that was created by this particular factory.

### Quick Tips: ###
1. To remove the UI controller as an intermediary between ViewModel and the direct xml views, we can directly add our ViewModel to Data Binding.

2. By adding add `binding.lifecycleOwner = this` to the fragment code, we can use LiveData with Data Binding to directly update our xml views whenever the data changes. 

3. Use Transformations to manipulate and combine data from one or more LiveData.
