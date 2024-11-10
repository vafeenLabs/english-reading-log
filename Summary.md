
1. In this article we will talk about how to create a user interface using composable functions
2. A single answer in the survey can be written in Compose as a function containing a Row with an Image, Text, and a RadioButton.
3. UI components in a compose are functions annotated with a composable annotation.
4. For example, to create a list of possible answers, you can call a function for each answer, in which you can put the blocks defined above. 
5. Composable functions do not return anything, they only create the UI.
6. Composable functions are immutable; you can only pass parameters to them when calling. 
7. Composable functions are written in Kotlin and therefore we no longer need to use the XML declaration. 
8. Composable functions must be fast and free of side-effects and it must behave the same when we call it in different times with the same arguments. 
9. Composable functions always control the UI, and they will redraw the UI if we call them again with different arguments.
10. This process is called recomposition - updating the UI with a new state
11. For example, if the answer is not initially selected, then when selected, the UI should update, but it will not update because you need to use a mutable state but not a simple state.
12. If this is a mutable state, then you must change its internal value, not itself
13. And you don’t need to explicitly observe State, or manually update the UI
14. composable functions can be drawn non-sequentially and executed in parallel.
15. the compose updates only those parts that require it 
16. If you need to perform long-running operation, do it in other thread and only use the result in composable. 
17. In the end i said that we have learned to create UI with composable funtions, which must be fast, should be side-effect free, and also can run in parallel and be skipped.