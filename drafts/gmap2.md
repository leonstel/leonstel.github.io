

# Finding X MEN
<img src="../assets/finding_xmen/xmen.jpg" />

### The app
// TODO correct youtube video here of working app
<p align="center">
    <iframe src="https://www.youtube.com/embed/hGlEOLbH00A" width="533" height="300" frameborder="0" allowfullscreen> </iframe>
</p>

Charles xavier you get phone call  
You won't turn down this great opportunity
Charles tells you that his Cerebro has gone broke and that he cant control it anymore manually,
only through the Cerebro Api.   
There will be mutants among you to be found  
Can you make an app to help me find mutants and gather the yet to come X MEN team?

##App features
- Separation of multiple mutants types (alpha beta) -> set video time where this happens
    - you can hide | show all mutants of type x at once -> set video time where this happens
- The Cerebro helmet has a fluctuating range radius ( the blue circle around professor X), range must be dynamically updated every time -> set video time where this happens
- When mutants are detected within professor x range mark them as discovered -> set video time where this happens
- When mutants are discovered you can send the Wolverine to recruit them on the X MEN team -> set video time where this happens


## What you will be learning in this series?
- Store principle vanilla js
- Getting you in the right direction with reusable googlemaps infrastructure
- Typescript based, know what you are programming with
- How to make maintainable code
    - factory methods
    - Method abstraction with wrapper class
- js ES6


<img src="../assets/finding_xmen/gmap_structure_test.jpeg" />

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

    update(name: any, val: object){
        // update state object
        // set new state
    }

    get(name: any){
        // return prop
    }

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
```
//store.ts

class Store {

    private mapLoadedObs = new BehaviorSubject<any>({current:this.state, prev: {}});
    
    

}

```

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

### Atherthough
Dont use in production
Not really a stable way of having deep layers of state (detecting change)
