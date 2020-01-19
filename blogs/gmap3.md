

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
- How does it relate to our google maps application?
- Rxjs observables

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
    - A button in UI get clicked
    - In its handler you set some prop in the store
    ... and it will go to step 2 again
    
    Do you see the lifecycle here?

The store will first be updated then the listeners will do their thing with the new state.
It will always comes first, it will NEVER occur that the UI of some state is first updated locally and then
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


That being said, put on your diving suit because you will be lowered in the shark cage to see some real learning.
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


## rxjs

To get the prop observing functionality working, I will use RXJS.
Rxjs is a nice observable based library with lots of features. I am not going in depth of each and every feature.
I will do my best to explain the things I use.

The set function will set a property

The typing could be stricter for sure for the 

Rxjs is solely build on the observable principle, which means that you can subscribe to an observable
and that code get notified with a new changed value if a value change or something other happens.

In rxjs you have several Observable types
- Observable
- Subject
- BehaviourSubject

I have chosen a behaviour subject. A behaviour subject will trigger the subscription callback with the initial value
when it is subscribed to.
This means that if you call `Store.changed('prop1')` returns a RXJS observable and the subscribe will immediately trigger and gives
the current value back. So this first time calling even happens if the `prop1` property of the Store has not been 
changed yet (the default state).

```
// the initial value here is the object with current and prev (empty) state
new BehaviorSubject<any>({current:this.state, prev: {}});
```

The specified value object of the observable contains a current and previous state so that when a change happens
some logic could check if that props has changed or not in comparison with the previous state.
changed.

[rxjs documentation](https://www.learnrxjs.io/)

The set function in the store will set the incoming value as its new state and then calls `next` on the observable.
This will tell rxjs to call the subscriptions who were listening with the new value. In this case the new value is an 
object with the new state and the previous state. 
```
// src/store.ts

class Store {

    // the initial value here is the object with current and prev (empty) state
    private changedObs = new BehaviorSubject<any>({current:this.state, prev: {}});
    
    ...

    set(name: string, val: any){
        // check if prop exists
        if(!this.state.hasOwnProperty(name)) throw Error(`prop ${name} not defined in store!`);

        const prev = this.state;
        this.state = {
            ...this.state,
            [name]: val,
        };

        this.changedObs.next({
            current: this.state,
            prev
        })
    }

}

```

### Update 
For one of my cases I needed a way to not just set a property but also to update an object property
of store. The set would just override it, the update would merge the new values into the current.
The update function in store is practically the most but merges the object instead of setting. You 
can find the entire code in the repo.
```
// src/store.ts

update(name: string, val: object){

    // update state object
    val = this.state[name] =  {
        ...this.state[name],
        ...val
    };

    // set new state
    ....
}
```

### Changed

I want the subscriptions
of changed listeners be called if it subscribes the first time (with current value) and after that only if the value has 
Describe how change must behave

```
// src/store.ts

private changedObs = new BehaviorSubject<any>({current:this.state, prev: {}});

changed(prop){
    if(!this.state.hasOwnProperty(prop)) throw Error('prop not defined in store! '+ prop, );

    return this.changedObs.pipe(
        switchMap((state, index) => {
            if(index === 0) return of(state.current[prop]);

            // could be better check but it is about the idea
            if(Array.isArray(state.current[prop])){

                if(state.prev[prop].toString() !== state.current[prop].toString()) 
                    return of(state.current[prop])

            } else if(state.prev[prop] !== state.current[prop]) {
                return of(state.current[prop]);
            }
            return NEVER;
        })
    );
}
```

So what the hell did I just see? Step by step

1\. Some code want to listen to a state's property of choice
```
Store.changed('prop1').subscribe(val => {});
```
2\. Store changed returns an piped observable stream  
`pipe` is able to chain multiple operators together to manipulate the observable's stream. 
For example you could switch to another observable when a specific value gets passed through the stream.
Or you could filter a stream, only let this stream reach the listeners if value === 5.

```
// src/app/store.ts

private changedObs = new BehaviorSubject<any>({current:this.state, prev: {}});

changed(prop){
        return this.changedObs.pipe(...)
}
```

3\. Because it is a behaviourSubject will immediately fire the listeners subscription method with the value through to 
observable stream

4\. The stream for the first time  
Like described above the first time the current value must be returned directly

`switchMap` switches to a new observable. For example the switchMap gives the value from the observable stream
and now you can switch to different observable depending how you want to react on the given value. The first param
is the value and the second the index. The index is the xth time the this observable
has been called next on. If `index === 0` it is the first time.
 
`of` makes an observable of the given value. When switching you have to switch to a new observable to not break the
observable stream.

In the `of ` which creates a new observable you pass in the the asked prop's current value



```
// src/app/store.ts

changed(prop){
    return this.changedObs.pipe(

        // recall that the state param will be {current: ..., prev: ..}
        // the index param indicates how many this stream has been call like array[index];
        switchMap((state, index) => {

            // if the index is 0 it is the first stream, so return the current prop state from store
            if(index === 0) return of(state.current[prop]);
            ...
        }
    )
}
```

5\. After some time other code sets the property
next
prev-current will be used for change detection
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

    // put new value so a that a new observable stream starts
    // this will go into the pipe function I have described above
    // because it get retriggerd with these new composed object
    // the currentState will be compared with the previous state to
    // detect if the prop has been changed
    this.changedObs.next({
        current: this.state,
        prev
    })
}
```

6\. The observable stream for all the listeners gets retriggered again, because the `next` has been called on de `BevahiouralSubject`

again in the change observable stream. But this time because it is the second time it is being called (by the Store.set())
the `index === 1`.  So then other logic kicks in.
Check if prop has changed `state.prev[prop] !== state.current[prop]`. If true then changed
then resume the observable stream with the latest prop value.
If it has not being changed return `NEVER`

`NEVER` is from the rxjs api. It is a constant with is an observable that tells the stream
to not go further. This means if you switchmap to a `NEVER` that listeners listening to this piped
stream will not get notified (the streams has prematurely ended)

```
// src/app/store.ts

changed(prop){
    if(!this.state.hasOwnProperty(prop)) throw Error('prop not defined in store! '+ prop, );
    return this.changedObs.pipe(
        switchMap((state, index) => {
            if(index === 0) return of(state.current[prop]);

            // could be better check but it is about the idea
            if(state.prev[prop] !== state.current[prop]) {
                return of(state.current[prop]);
            }
            return NEVER;
        })
    );
}
```

####Addition array is equal
One prop in my store is array, and I wanted to check if the array items has changed
for demo purposes I used this. Dont use this in production! It is only handy for an 
array if it only contains strings and nothing else!
Array toString both and then check if that string is equal. It then returns a new observable with 
the props value with rxjs's `of`. THe same thing as before described in this step.
```
changed(prop){
    ...
    return this.changedObs.pipe(
        switchMap((state, index) => {
           ...
            if(Array.isArray(state.current[prop])){
                if(state.prev[prop].toString() !== state.current[prop].toString()) return of(state.current[prop])
            } 
           ...
        })
    );
```

## Extra observable streams utils

2 utils function I wrote for state
- First Time True: listens for a prop on store and fires only once its value change to true
- Changed But Wait For: listens for a prop but gets only notified if other specified props exists in the store

Usage with examples

Image a state with the following shape
```
// state within Store object

// sample state
state = {
    mapLoaded: true,       // indicates if the external gmaps script has been loaded
    mapInit: true          // indicates if the google maps has been initialized
    prop1: 'val1',
}
```

Practical example how you would use these functions


```
// src/app/utils.ts

// this will be called only once, the first time the mapInit state prop becomes true
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

// you could listen and wait for unlimited props 
// list to prop1 and wait for ...
changedButWaitFor('prop1', 'prop2', 'prop3', 'prop4', 'etc, 'mapInit')

```

###firstTimeTrue

Change only if a state change to something the first time  

```
// src/app/utils.ts

export const firstTimeTrue = (prop: any) => Store.changed(prop).pipe(
    mergeMap( (flag: boolean) => {
        return iif( () => flag, of(flag))
    }),
    first( (flag: boolean) => flag),
);
```
For example a boolean flag with default value false and that you only want to get
a callback if that prop change to true the first time true
// TODO example code what happens

###changedButWaitFor

Listen in store if they exists && not undefined

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

*Part1* 

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
In the `changedButWaitFor()` a property will be checked on existence. Wait for a property to change on store (and first time)
and then filter that observable stream with `filter( (val: any) => !!val)`. The following statements are correct on the 
sample state. When the changed observable gets trigger (thus the prop has changed) then first `filter` the stream.
With the `filter` operator you only let the observable true if it return true.
`filter( (val: any) => !!val)` convert value to boolean. 

<sub>In sample state prop2 is undefined and mapInit is boolean</sub>  
`!!this.state.prop1 === true`   
`!!this.state.prop2 === false`  
`!!this.state.mapInit === true`

If the stream passes through the `filter` then take the `first`.
My main goal of this piping is to convert it to a `Promise`. Then you could
wait till the prop exists in the store. Without the first the promise (with `toPromise`) never gets called
because the stream won't ever end. When you convert an observable to a promise, the promise will only be 
called if the stream end. The `first` takes the first stream it encounters and then ends/completes that stream.
At that moment the transformed promise will be called 

*Part2* 

```
export const changedButWaitFor = (propToListenFor, ...ifDefinedProp) => {
    const waitIfDefinedProms = ifDefinedProp.map((prop) => {
        ...
    });

    ...
};
```

Prop to lister for 
Spread, takes every exceeding param and gives back and array
Array of promises: `waitIfDefinedProms`
It maps over the restArrayParams and creates for each param a Promise like Part1.
So after the map you have an array with promises that wait for their property to be true


*Part3* 

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

Store.changed functionality as described early
but I pipe this observable stream through a `switchMap`. That means every time a observable stream gets fired
with `next` it will first go to this pipe before reaching the listener

The last thing that happens. There will be listened to the `propToListenFor` on the start with `changed`.
When that observable is being triggered it will first being piped through a `switchMap` Within
that it will wait until all the promises that have been created in part2 are completed with 
`await Promise.all(waitIfDefinedProms)`

To recap that means that it will wait until all other specified props are true in the store, only then the listener will
be triggered.

#### related to gmaps

To extend in this store concept with the extra observable streams we could for example use the
changedButWaitFor method to listen to props only after the map has been initialized. This comes in very handy
if you want the catch a props changed and then do something with that on the googleMapInstance.
If you don't wait for the map to be initialized the googelMapsInstance could be `undefined`. Like
the scenario I did tell you about, where the asynchrounously gmap script loading gets delay.
In the next article I will take this idea and combine it with another concept to make it even more practical.  
