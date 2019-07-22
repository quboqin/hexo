I think the most important concept in Vue.js is the component system. Each Vue instance can be seen as an view model. 
There are several essential elements in an Vue component that we must know how to distinguish them, and use them at 
the right time.
1. A component defines its own state by `Data`, and the parent can pass down some `Data` to its components through a one-way-down
binding. These `Data` are binding with `Props` in its components. Because of the one-way binding, we can not modify `Props`. `Props` are immutabel. In the component, if we want to change the value of a `Prop`, we need to initialize a `Data` using the value of the `Prop`. 
``` Javascript
```
2. The `DOM` defined in the `Template` can generate some events, and these events will trigger some method calls. Then these methods will change the values of `Data`. And some of these calls come from the `Hooks` of the life cycle of the component. In these methods, we can also use asynchonize API to update `Data` from the backend. To compare with these methods, the `Computed Properties` will cache their dependencies, if none of their reactive dependencies is changed, it will not be re-evaluated. 
3. If `Methods` return some `Data`, they will also can bind onto the `DOM`s, just like `Computed Properties` and `Data` itself.
4. Sometimes when the `Data` or `Computed Properties` was changed, we want to apply some strategies before we actually take actions. If these strategies are synchonized, we can define another `Computed Properties`, but if these strategies are asynchonized, we can use `Watchers`. For examples, we want to delay some operations when some values were changed, and don't want to handle immediately.
5. The `Component` can emit a value with an event to its parent.

Vue doesn't support the two-way binding, because the two-way binding can create maintainance issues. But in some cases, we still can use `Props` and `Events` to figure out a two-way binding. The `v-model` is just a syntax sugar for the two-way data binding.
1. On the child component side, we cann't modify the `Props`, but it can emit a new value using `update:var_name` to its parent.
2. On the parent side, we use `v-bind:var_name` and `v-on:update:var_name` to send the value to its child components, and listen on that event to monitor the value will be changed. In 2.3.0+ version, we can merge these two directives into one directive `v-bind:var)name.sync`.
3. `v-model` is just a a syntax sugar for the two-way data binding.
``` Javascript
```

Another way to communicate between `Components` and its parent is using `Slots`, and we can also override the default appearance or implements of a `Component`
1. Because the `Slots` are belong to the parent, if the parent want to access the data in its components, the components need to bind its `Data` and send back to its parent. Then the parent can get the values by `v-slot`




