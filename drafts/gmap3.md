

# Finding  X MEN Without Italian Delicacy
<img width="100%" src="../assets/finding_xmen/recipe.jpg" />

*Recipe: Spaghett*  
*Ingredients*
- *No state management*

The google map instance will take care of a part the unmaintainability. But I believe that a proper immutable
state management strategy is very important. For this demo I choose to
make a store prototype myself for the sake of learning.

### Store
For learning purpose understanding principle.
Dont use this in production code!

I began to start wrapping my head around redux principle only after
I had created when from vanilla myself.  
Correct immutable state management is a must. Single source of truth.
So you know what and when to expect, without prop drilling or any other
bug introducing spaghetti recipe.   



```
//store.ts

// the type for the state
interface State {}

class Store {

    //default state object state type
    private state: State = {};
  
    set(name: any, val: any){
        // compose new state object
        // set new state
    }

    get(name: any){
        // return prop
    }

    // observable stream, notify listener when their subscribed prop changes
    changed(prop){
        // observable
    }
}

export default new Store();

```

Store example usage
```
// set prop in store
Store.set('apiMutants', apiMutants);

// callback called everytime the prop changes in store
Store.changed('discovered').subscribe((mutantIds: string[]) => {
    this.showInPanel(this.XMenDiscoveredEl, mutantIds, mutantsList)
});
```


## rxjs

Rxjs is a nice observable based library. I am not going in depth of each and every feature.
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
This means that if you call `Store.changed('prop1')` the subscribe will immediately trigger and gives
the current value back. So this first time calling even happens if the `prop1` property of the Store has not been 
changed yet. 

```
// the initial value here is the object with current and prev (empty) state
new BehaviorSubject<any>({current:this.state, prev: {}});
```

The specified value object of the observable contains a current and previous state so that when a change happens
some logic could check if that props has changed or not in comparison with the previous state.
changed.

[rxjs documentation](https://www.learnrxjs.io/)

The set function in the store will set the incoming value as its new state and then calls `next` on the observable.
This will tell rxjs to call the subscriptions with the new value. In this case the new value is an object with the new state and the
previous state. 
```
// src/store.ts

class Store {

    // the initial value here is the object with current and prev (empty) state
    private changedObs = new BehaviorSubject<any>({current:this.state, prev: {}});
    
    ...

    set(name: string, val: any){
        if(!this.state.hasOwnProperty(name)) throw Error(`prop ${name} not defined in store!`);

        const prevState = this.state;
        this.state = {
            ...this.state,
            [name]: val,
        };

        this.changedObs.next({
            current: this.state,
            prev: prevState
        })
    }

}

```

### Update 
For one of my cases I needed a way to not just set a property but also to update an object property
of store. The set would just override it, the update would merge the new values into the current.
The update function in store is practically the most but merges the object instead of setting. You 
can find the code in the repo.
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
2\. Store changed returns an piped subscriber  
`pipe` is able to chain multiple operators together to manipulate the observable's stream. 
For example you could switch to another observable when a specific value gets passed through the stream.
Or you could filter a stream, only let this stream reach the listeners if value === 5.

```
private changedObs = new BehaviorSubject<any>({current:this.state, prev: {}});

changed(prop){
        return this.changedObs.pipe(...)
}
```

3\. Because it is a behaviourSubject will immediately fire the value through to observable stream

4\. The stream for the first time  
Like described above the first time the current value must be returned directly

`switchMap` switches to a new observable. For example the switchMap gives the value from the observable stream
and now you can switch to different observable depending how you want to react on the given value. The first param
is the value and the second the index. The index is the xth time the this observable
has been called next on. If `index === 0` it is the first time.
 
`of` makes an observable of the given value. When switching you have to switch to a new observable to not break the
observable stream.

```
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
//somewhere else
store.set('prop1', 'new value');

class Store {
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
}
```

6\. The 

again in the change observable stream. But this time because it is the second time it is being called (by the Store.set())
the `index === 1`.  So then other logic kicks in.
Check if prop has changed `state.prev[prop] !== state.current[prop]`. If true then changed
then resume the observable stream with the latest prop value.
If it has not being changed return `NEVER`

`NEVER` is from the rxjs api. It is a constant with is an observable that tells the stream
to not go further. This means if you switchmap to a `NEVER` that listeners listening to this piped
stream will not get notified (the streams has prematurely ended)

```
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
Array toString both and then check if that string is equal.
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
- First Time True
- Changed But Wait For

Describe what happens in what cases

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

Change only if a state change to something the first time  
For example a boolean flag with default value false and that you only want to get
a callback if that prop change to true the first time true

```
// src/app/utils.ts

export const firstTimeTrue = (prop: any) => Store.changed(prop).pipe(
    mergeMap( (flag: boolean) => {
        return iif( () => flag, of(flag))
    }),
    first( (flag: boolean) => flag),
);
```

Listen in store if they exists && not undefined

```
// src/app/utils.ts

export const changedButWaitFor = (mainProp, ...ifDefinedProp) => {
    const waitIfDefinedProms = ifDefinedProp.map((prop) => {
        return Store.changed(prop)
            .pipe(
                filter( (val: any) => !!val),
                first()
            )
            .toPromise();
    });

    return Store.changed(mainProp).pipe(
        switchMap(async (res: any) => {
            await Promise.all(waitIfDefinedProms);
            return res
        }),
    );
};
```
Important thing to notice:  
In the `changedButWaitFor()` a property will be checked on existence with `filter( (val: any) => !!val)`
The following statements are correct on the sample state   

<sub>In sample state prop2 is undefined and mapInit is boolean</sub>  
`!!this.state.prop1 === true`   
`!!this.state.prop2 === false`  
`!!this.state.mapInit === true`

#### related to gmaps

To extend in this store concept with the extra observable streams we could for example use the
changedButWaitFor method to listen to props only after the map has been initialized. This comes in very handy
if you want the catch a props changed and then do something with that on the googleMapInstance.
If you don't wait for the map to be initialized the googelMapsInstance could be `undefined`. Like
the scenario I did tell you about, where the asynchrounously gmap script loading gets delay.
In the next article I will take this idea and combine it with another concept to make it even more practical.  

## Further reading
Redux
    - reducer
    - actions
MobX, more implicit state management
Flux pattern, multiple store

Store
Dont use in production
Not really a stable way of having deep layers of state (detecting change)
