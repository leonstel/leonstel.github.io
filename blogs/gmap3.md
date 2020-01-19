

# Finding  X MEN Without Italian Delicacy
<img width="100%" src="../assets/finding_xmen/recipe.jpg" />

*Recipe: Spaghetti*
  
*Ingredients*
- *No state management*

In previous article we have talked about the `GoogleMapInstance` which is a singleton like method wrapper for maintaining 
and manipulating the google maps. Today we will discuss how to integrate some state management so data could be effectively saved and notify other
components. 

### Learning Points

- What does it mean to have a store?
- Rxjs observable streams
- How does it relate to our google maps application?

### Store
In this project I have chosen to make a store ourselves I believe this enriches your core knowledge about the general
flux pattern. Our most basic store is the single source of truth of your app for your data and it will notify listeners when some
state changes. These listener are then able to act upon that change. From my own experience javascript is notorious
 for making a mess when you don't think about what you code. With the store on the other hand you know for sure what and 
 when to expect. 

To clarify we won't use any out of the box store libraries like Redux or MobX. It is a little bit 
 rudimentary and does not cover all edge cases of the universe so don't use this store code in
production!

*Flux*  
The Flux pattern, which has been implemented by Redux and MobX, builds upon above description and is meant for creating
a lifecycle related to data. The data flow of flux is unidirectional.


    - You set some prop in the store
    - Something listens to that state
    - The listeners for that state could rerender their UI respectively
    - A button in UI gets clicked
    - In its handler you set some prop in the store
    ... and it will go to step 2 again
    
    Do you see the lifecycle here?

The store will first be updated then the listeners will do their thing with the new state.
It will always comes first, it will NEVER occur that the UI of some variable is first updated locally and then
set into the store.

*Pros*
- Debuggable
- Single source of truth, know what to expect
- Cleaner and more readable code
- No more endless prop drilling

*Cons*
- Our example is simple and covers our cases but not yet ready for full use
- The Redux library uses some templating to get it working
- MobX is easier with less templating but is less explicit


That being said, put on your driving suit because you will be lowered in the shark cage to see some real learning.
```
// src/app/store.ts

// The type for the state
interface State {}

class Store {

    //Holds the state object of the entire app
    private state: State = {};
  
    // Sets state of name
    set(name: string, val: any){
        // Compose new state object immutably
        // Set new state
    }

    // Gets state of name
    get(name: string){
        // Return value of name
    }

    // Returns Rxjs observable stream which notifies its listener when input 
    // param prop changes 
    changed(prop){
        // observable
    }
}

// Exports created Store Class, only one is allowed to exists within app
export default new Store();

```

#### Example usage
```
// Set new state of ApiMutant in store
Store.set('apiMutants', apiMutants);

// Callback called everytime the prop changes in store
Store.changed('discovered').subscribe((mutantIds: string[]) => {
    //... do something with new value
});
```


## Rxjs

To get the state observing functionality working we will use the Rxjs library which is a nice observable based library 
with lots of features for handling and manipulating observable streams. We will not going in depth of each and every feature.
I will do my best to explain the things I use.

I hope that the set and get function of the store are self explanatory, they will get/set state on the global state object.

Rxjs is solely build on the observable principle that means that you can subscribe to an observable
and define listeners that will receive new values whenever a change happens.

In Rxjs you have several Observable types
- Observable
- Subject
- BehaviourSubject

I have chosen a behaviour subject that one will trigger the subscription callback with the initial value
when it is first subscribed to.
To elaborate on that if you call `Store.changed('prop1')` it returns an Rxjs observable and the listener will immediately 
be triggered with the current value. So this first time calling even happens if the `prop1` property of the Store has not been 
changed yet (the default state).

```
// The initial value here is the object with current and prev (empty) state
new BehaviorSubject<any>({current:this.state, prev: {}});
```

The specified value object of the observable contains a current and previous state so that when a change happens
some logic could check if that prop has changed or not in comparison with the previous state.

[Rxjs documentation ](https://www.learnrxjs.io/)

The set function in the store will set the incoming value as its new state and then calls `next` on the observable.
This will tell Rxjs to call the subscriptions who were listening with the new value. In this case the new value is an 
object with the new state and the previous state. 
```
// src/app/store.ts

class Store {

    private state: State = {};    

    // Init observable with default object
    private changedObs = new BehaviorSubject<any>({current:this.state, prev: {}});
    
    ...

    set(name: string, val: any){
        // Check if prop exists in store object
        if(!this.state.hasOwnProperty(name)) throw Error(`prop ${name} not defined in store!`);

        // Compose new state object of input name with new value
        const prev = this.state;
        this.state = {
            ...this.state,
            [name]: val,
        };

        // Notify all the listeners that some state has changed
        this.changedObs.next({
            current: this.state,
            prev
        })
    }

}

```

### Update 
For one of my cases I need a way to not just set a property but  to update an object property
of the store as well. The `set` would just override it whereas the update would merge the new values into the current.
You can find the entire code in the repo.
```
// src/app/store.ts

update(name: string, val: object){

    // update state object
    val = this.state[name] =  {
        ...this.state[name],
        ...val
    };

    // set new state, call .next() on observable
    ....
}
```

### Changed

We want the subscriptions
of changed listeners to be called if it subscribes the first time (with current value) and after that only if the value has 
changed.

```
// src/app/store.ts

// Lots of confusing code... we will go over it step by step

private changedObs = new BehaviorSubject<any>({current:this.state, prev: {}});

changed(prop){
    if(!this.state.hasOwnProperty(prop)) throw Error('prop not defined in store! '+ prop, );

    return this.changedObs.pipe(
        switchMap((state, index) => {
            if(index === 0) return of(state.current[prop]);

            // Check if the prop's state has been changed
            if(state.prev[prop] !== state.current[prop]) {
                return of(state.current[prop]);
            }
            return NEVER;
        })
    );
}
```

So what the hell did I just read? 

#### Step by step

1\. Some code wants to listen to a state's property of choice
```
Store.changed('prop1').subscribe(val => {});
```
2\. The Store's `changed()` returns an piped observable stream.  

`pipe` is able to chain multiple operators together to manipulate the observable's stream.

 
For example you could switch to another observable when a specific value gets passed through the stream.
Or you could filter a stream to only let this stream reach the listeners if its passing `value === 5.`

```
// src/app/store.ts

private changedObs = new BehaviorSubject<any>({current:this.state, prev: {}});

changed(prop){
        return this.changedObs.pipe(...)
}
```

3\. Because it is a Rxjs BehaviourSubject it will immediately fire the listeners subscription method with the value through to 
observable stream.

4\. The stream for the first time.
  
Like described above the first time some code subscribes to it the current value must be returned immediately.

```
// src/app/store.ts

changed(prop){
    return this.changedObs.pipe(

        // Recall that the state param will be {current: ..., prev: ..}
        // The index param indicates how many this stream has been call like array[index];
        switchMap((state, index) => {

            // If the index is 0 it is the first stream, so return the current prop state from store
            if(index === 0) return of(state.current[prop]);
            ...
        }
    )
}
```

`switchMap` switches to a new observable. For example the switchMap gives the value from the observable stream
and now you can switch to a new observable depending how you want to react on the that value. 

The first param
is the value and the second the index. The index is the xth time this observable
has been called next on. If `index === 0` it is the first time.
 
`of` makes an observable of the given value. When switching you have to switch to a new observable to not break the
observable stream. It will be given the asked prop's current value.

5\. After some time other code sets the property in the store by calling `.next()`.
```
store.set('prop1', 'new value');
```
```
// src/app/store.ts

set(name: string, val: any){    

    // compose the current state with the incoming value for the name param
    const prev = this.state;
    this.state = {
        ...this.state,
        [name]: val,
    };

    // All the listeners will be notified with the current and previous state
    this.changedObs.next({
        current: this.state,
        prev
    })
}
```

6\. The observable stream for all the listeners gets retriggered again because the `next` has been called on de `BevahiouralSubject`

This second time it is being called with the `index === 1` so other logic kicks in.
```
// src/app/store.ts

changed(prop){
    if(!this.state.hasOwnProperty(prop)) throw Error('prop not defined in store! '+ prop, );
    return this.changedObs.pipe(
        switchMap((state, index) => {
            if(index === 0) return of(state.current[prop]);

            // Check if the prop's state has changed
            if(state.prev[prop] !== state.current[prop]) {
                return of(state.current[prop]);
            }
            return NEVER;
        })
    );
}
```

Change detection follows with `state.prev[prop] !== state.current[prop]`. When the prop's previous state is not equal to
its current state then some change has happened. 
It will then resume the observable stream with the latest state value of the prop.


In the end whenever no changes has been detected for this prop it stops the observable stream with `NEVER`.
`NEVER` is a constant from the Rxjs library and contains is an observable that tells the stream
to not go any further. A listener won't get notified when its piped stream will prematurely `switchMap` to a `NEVER`.

####  Extra Observable Stream Utils

To make the store even more helpful the app contains to extra util methods.

- `firstTimeTrue`  
Listens for a state prop and fires only once when its value changes to true
- `changedButWaitFor`  
Listens for a state prop but gets only notified if other specified props exists in the store

#### In Practice

Imagine a state with the following shape
```
// Sample state
state = {
    mapLoaded: true,       // Indicates if the external gmaps script has been loaded
    mapInit: true          // Indicates if the google map obj has been initialized
    prop1: 'val1',
}
```

Usage

```
// src/app/utils.ts

// Called only once, the first time the mapInit state prop becomes true
firstTimeTrue('mapInit').subscribe((val) => {
    // do something once after map init
});

// Listens for prop1 and waits for mapInit
changedButWaitFor('prop1', 'mapInit').subscribe((val) => {
    // if mapInit becomes true and prop1 has not changed and callback has not been fired once
        // gets called with current value
    
    // if mapInit is true and prop1 has changed after that
        // gets called with the changed value
});

// The only difference here is that it listens for prop1 and waits for prop2 and mapInit
changedButWaitFor('prop1', 'prop2', 'mapInit').subscribe((val) => {
    //.. 
});

// You could listen and wait for unlimited props 
// Listens for prop1 and waits for ...
changedButWaitFor('prop1', 'prop2', 'prop3', 'etc, 'mapInit')

```

#### First Time True

Fires only the first time the passed through value becomes true

```
// src/app/utils.ts

export const firstTimeTrue = (prop: any) => Store.changed(prop).pipe(
    mergeMap( (flag: boolean) => {
        return iif( () => flag, of(flag))
    }),
    first( (flag: boolean) => flag),
);
```

`firstTimeTrue` is like encountered earlier an Rxjs observable stream. 

`mergeMap` sort of like `switchMap` but this one merges/adds a new observable instead of replacing it.

`iif` filters a stream through a condition

`first` gets the first passed observable that reaches it followed by ending / completing the stream.

#### Changed But Wait For

Listens to a store's prop and waits for other specified props to be true or existence.

```
// src/app/utils.ts

export const changedButWaitFor = (propToListenFor, ...ifDefinedProp) => {
    const waitIfDefinedProms = ifDefinedProp.map((prop) => {
        return Store.changed(prop)
            .pipe(
                filter( (val: any) => !!val),
                first()
            )
            .toPromise();
    });

    return Store.changed(propToListenFor).pipe(
        switchMap(async (res: any) => {
            await Promise.all(waitIfDefinedProms);
            return res
        }),
    );
};
```

**Part1** 

```
(prop) => {
    return Store.changed(prop)
        .pipe(
            filter( (val: any) => !!val),
            first()
        )
        .toPromise();
}
```
Important thing to notice:  
In the `changedButWaitFor()` a property will be checked on existence and waits for a property to change on store.
It then filters that observable stream with `filter( (val: any) => !!val)`. 

The following statements are correct on the sample state. 

<sub>In sample state prop2 is undefined and mapInit is boolean</sub>  
`!!this.state.prop1 === true`   
`!!this.state.prop2 === false`  
`!!this.state.mapInit === true`

If the stream passes through the `filter` then take the `first`.
Our main goal of this piping is to convert it to a `Promise` so you could
wait till the prop exists in the store. Without the `first` the promise (with `toPromise`) never gets called
because the stream won't ever end. The `first` takes the first stream it encounters and then ends/completes that stream.
At that moment the transformed promise will be called 

**Part2**

```
export const changedButWaitFor = (propToListenFor, ...ifDefinedProp) => {
    const waitIfDefinedProms = ifDefinedProp.map((prop) => {
        ...
    });

    ...
};
```

`propToListenFor` is the param that is going to be listened to while the spread operator gathers all the exceeding params
and assigns that array to `ifDefinedProp`. 

`map` over the `ifDefinedProp` array which will transform it to an array with Promises (Part1). 

You end up with an array of promises each of them waiting for their property to become true in the store.

**Part3** 

```
export const changedButWaitFor = (propToListenFor, ...ifDefinedProp) => {
   ...

    return Store.changed(propToListenFor).pipe(
        switchMap(async (res: any) => {
            await Promise.all(waitIfDefinedProms);
            return res
        }),
    );
};
```

Almost there, the `Store.changed` stream gets piped through a `switchMap` (mergeMap or a `tap` would probably do the job as well). That means every time a observable stream gets fired
with `next` it will first go to this pipe before reaching the listener.

Finally the last thing that happens is that within the `switchMap` it will wait for all the promises from Part 3 to succeed with `Promise.all` 

To simplify, until all other properties exists in the Store don't go any further.

#### Related To Google Maps

We have seen a lot of concepts come by but how does it relate to our app?

- The store will be the central point for keeping our data
- It will keep track of when the google maps has been loaded and initialized
- You will be able to globally listen to this loading and initializing state
- `firstTimeTrue()` is a handy method for doing something once after the map has been initialized
- `changedButWaitFor()` will be convenient for reacting to map related state properties after the map has been initialized
to elude those pesky undefined checks.


### Fitting the Pieces

To recap we have our store in place with some extra utils. Alongside Rxjs has accompanied our trip to the finish line.
Hopefully you have had some aha moments working with state in combination with google maps. In the next article I will 
take the `GoogleMapInstance` and `Store` principles and combine it with another concept to make it even more practical
to create custom google map components.  

### Happy diving

<p align="center">
    <img src="../assets/finding_xmen/shark_cage.jpg" />
</p>

//TODO Links to next articles
// TODO Link to repo
