In the previous MAD Skills Compose Basics article, you learned how to think in Compose — you describe your UI in Kotlin as functions. No more XML needed! In this article, we will dive deeper into these functions and how you can build UI with them.

> As a reminder, we will be answering your questions on Compose Basics in our live Q&A session on October 13th. Make sure to leave a comment here, on YouTube, or by using the #MADCompose hashtag on Twitter.

To understand how these functions work, let’s see how we can build the single choice question screen of Jetsurvey one of our Compose samples.

# A simple composable function

A single answer in the survey can be written in Compose as a function containing a Row with an Image, Text, and a RadioButton.

To create a UI component in Compose, we must annotate a function with the `@Composable`. annotation. This annotation tells the Compose compiler that this function is intended to convert data into UI, so an answer into UI.¹

Functions with this annotation are also called **composable** **functions**, or composables for short. These functions are the building blocks of UI in Compose. It’s quick and easy to add this annotation encouraging you to organize your UI into a library of reusable elements.

For example, to implement a list of possible answers to select from, we can define a new function called `SingleChoiceQuestion` that takes in a list of answers, and then call the `SurveyAnswer` function that we have just defined.

`SingleChoiceQuestion` accepts parameters which allows it to be configured by the app’s logic. In this case, it accepts a list of possible answers so that it can display these options to the UI. Notice that the composable doesn’t return anything (it returns `Unit`), but instead, it **emits UI**. Specifically, it emits the `Column` layout composable that’s part of the Compose toolkit which arranges items vertically. Within the column, it emits a `SurveyAnswer` composable for each answer.

Composables are **immutable**. You can’t hold a reference to them — like holding a reference to a single answer — and later update its contents. You need to pass any and all information as parameters when you call it.

Notice that since the function is written in Kotlin, we can use the full Kotlin syntax and control flow to produce our UI. Here we’re using `forEach` to iterate through each answer and call `SurveyAnswer` to display them. If we want to conditionally display something else, it’s as simple as using an if statement. No `View.visibility = View.GONE` or `View.INVISIBLE` needed. With a declarative UI framework, like Compose, if you want your UI to look differently depending on the inputs provided, the composable must describe how your UI should look for every possible value of inputs. Using conditional statements like this snippet is how you can achieve this.

Composables must be **fast and free of side-effects**. It must behave the same when called multiple times with the same argument, and it should not modify properties or global variables. We say that functions with this property are _idempotent_. This property is necessary for all composables so that UI can be emitted correctly when functions are reinvoked with new values.

Notice that the parameters provided to the functions **entirely** **control the UI**. This is what we mean by transforming state into UI. The logic in the function will guarantee that the UI can never get out of sync. If the list of answers changes, then a new UI is generated from the new list of answers by executing this function again and redrawning the UI if necessary.

This process of regenerating UI when state changes is called **recomposition.** Since composables are immutable, recomposition is the mechanism for updating UI with new state.

# Recomposition and State of composable functions

Recomposition happens when a composable is reinvoked with different function parameters. This happens because State that the function depends on changes.

For example, say the `SurveyAnswer` composable accepts a parameter `isSelected` which dictates if the answer is selected or not. Initially, no answer is selected.

In the View world, interacting by tapping on one of the answer UI elements would visually toggle it as Views hold their own state. In the Compose world, however, since **false** is provided to all `SurveyAnswer` composables, all answers would remain unselected despite user interaction. To make them visually respond to user interaction, the composable needs to be recomposed so that the UI can be regenerated with new state.

To do that, a new variable needs to be introduced which holds the selected answer. Additionally, the variable needs to be a `[MutableState]
— an observable type that is integrated within the Compose runtime. Any changes to the state will automatically schedule a recomposition for any composables that read it. A new `MutableState` can be created with the `[mutableStateOf]`method like so.

Note in the snippet above that the value of `isSelected` has been updated to compare the current answer to `selectedAnswer`. Since `selectedAnswer` is of type `MutableState`, we have to use the value property to get the selected answer. When this value changes, Compose will automatically re-execute the `SurveyAnswer` so that the selected answer is highlighted.

The snippet above won’t quite work, however. The value of `selectedAnswer` needs to be remembered across recomposition so that it is not overwritten when `SingleChoiceQuestion` is reinvoked. To fix that, the `mutableStateOf` call should be performed within a call to remember. This guarantees that the value is remembered, and not reset, when the composable recomposes. To remember values across configuration changes, we can also use `rememberSaveable`.

The above code snippet can further be refined by using Kotlin’s delegated property syntax on the `selectedAnswer` variable. Doing so changes the type from `MutableState<Answer?>` to simply, `Answer?`. This syntax is quite nice as we can work directly with the value of the underlying state — no more calls to the `value` property on a `MutableState` object.

With our newly introduced state, we can pass a lambda function for the `onAnswerSelected` parameter so we can perform an action when the user makes a selection. In the definition of this lambda, we can set the value of `selectedAnswer` to the new one.

If you recall from the previous article, events are the mechanism for how State is updated. Here, the event `onAnswerSelected` would be invoked when the user interacts by tapping an answer.

The Compose runtime automatically keeps track of where state reads occur so that it can intelligently recompose composables that depend on that state. As a result, **you don’t need to explicitly observe State, or manually update the UI**.

# Behaviors and properties of composables

There are some other behavioral properties of composable functions you should also be aware of. Because of these behaviors, **it’s important that your composable functions are side-effect free, and behave the same when called multiple times with the same argument**.

Looking at the snippet below, you might assume that the code runs sequentially. But this isn’t necessarily true. Compose can recognize that some UI elements are higher priority than others, and so those elements might be drawn first. Say for example you have code that draws three screens in a tab layout. You might assume that `StartScreen` is executed first, however, these executions can happen in any order.

Composables can run in parallel thereby taking advantage of multiple cores which improves the rendering performance of a screen. In the code snippet below, the code runs side-effect free and transforms the input list to UI.

However, if the function writes to a local variable, such as in the snippet below, the code is no longer considered side-effect free. Doing something similar to the code below may result in odd behaviors in your UI.

Compose does its best to recompose only portions of that UI that need to be updated. If a composable doesn’t use the state that triggered recomposition, then it will be skipped. In the snippet, if the name string changes, the `Header` and `Footer` composables will not be re-executed since it does not depend on that state.

Recomposition is optimistic — this means that Compose expects to finish recomposition before the parameters change again. If a parameter does change before recomposition finishes, Compose might cancel the recomposition and restart it with the new parameter.

Lastly, composable functions might run frequently. This might be the case if your composable function contains an animation that needs to be executed for every frame. This is why it’s important to make sure that your composable functions are fast to avoid dropped frames.

If you need to perform long-running operation, don’t do it in the composable function, Instead perform it off the UI thread and only use the result in the composable.
# Summary

We covered a lot! To summarize:

- You can create composable functions by using the `@Composable` annotation
- It’s quick and easy to create composables, encouraging you to organize your UI into a library of reusable components.
- Composables can and should accept parameters to configure their behavior
- **MutableState, remember** and **rememberSaveable** can be used to store a component’s state and have Compose automatically track and recompose changes
- Composables should be side-effect free

We also learned about some interesting properties of composables. Composables can:

- Execute in any order
- Run in parallel
- Be skipped
- Run frequently

The Compose toolkit provides a lot of foundational and powerful composables that help you build beautiful apps. 
