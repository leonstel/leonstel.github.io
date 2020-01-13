

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

## Extra observable streams utils

- First Time True
- Changed But Wait For

Describe what happens in what cases

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

Image a state with the following shape
```
// sample state
state = {
    mapLoaded: true,       // indicates if the external gmaps script has been loaded
    mapInit: true          // indicates if the google maps has been initialized
    prop1: 'val1',
}
```

Important thing to notice:  
In the `changedButWaitFor()` a property will be checked on existence with `filter( (val: any) => !!val)`
The following statements are correct on the sample state   

<sub>In sample state prop2 is undefined and mapInit is boolean</sub>  
`!!this.state.prop1 === true`   
`!!this.state.prop2 === false`  
`!!this.state.mapInit === true`

Practical example how we will be using these functions

```
// this will be called only once, the first time the mapInit becomes true
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
