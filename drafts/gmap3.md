

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

In rxjs you have several Observable types
- Observable
- Subject
- BehaviourSubject

I have chosen a behaviour subject. A behaviour subject will trigger the subscription callback with the initial value
when it is subscribed to.
This means that if you call `Store.changed('prop1')` the subscribe will immediately trigger and gives
the current value back. So this first time calling even happens if the `prop1` property of the Store has not been 
changed yet.

[rxjs documentation](https://www.learnrxjs.io/)

```
//store.ts

class Store {

    // the initial value here is the object with current and prev (empty) state
    private mapLoadedObs = new BehaviorSubject<any>({current:this.state, prev: {}});
    
    ...

    set(name: string, val: any){
        if(!this.state.hasOwnProperty(name)) throw Error(`prop ${name} not defined in store!`);

        const prevState = this.state;
        this.state = {
            ...this.state,
            [name]: val,
        };

        this.mapLoadedObs.next({
            current: this.state,
            prev: prevState
        })
    }

}

```

### Update 
For one of my cases I needed a way to not just set a property but also to update an object property
of store. The set would just override it, the update would merge the new values into the current.
```
    update(name: any, val: object){
        // update state object
        // set new state
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
export const firstTimeTrue = (prop: any) => Store.changed(prop).pipe(
    mergeMap( (flag: boolean) => {
        return iif( () => flag, of(flag))
    }),
    first( (flag: boolean) => flag),
);
```

Listen in store if they exists && not undefined

```
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

For our map example usage

```

changedButWaitFor('prop1', 'mapInit').subscribe(() => {
    // get called if prop1 changes in store and mapInit is true
});
changedButWaitFor('prop1', 'prop2', 'prop3', 'mapInit').subscribe(() => {
    // get called if prop1 changes in store 
    // and prop2, prop3 and mapInit is true
});

// this will be called only once, the first time the mapInit becomes true
firstTimeTrue('mapInit').subscribe(() => {
    // do something after map init
});
```


## Further reading
Redux
    - reducer
    - actions
MobX, more implicit state management
Flux pattern, multiple store

Store
Dont use in production
Not really a stable way of having deep layers of state (detecting change)
