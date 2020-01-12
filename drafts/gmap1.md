

# Finding  X MEN Without Spaghetti
<img src="../assets/finding_xmen/recipe.jpg" />
Ingredients
- No state management
- No factory methods
- No typing


## What you will be learning in this series?
- Store principle vanilla js
- Getting you in the right direction with reusable googlemaps infrastructure
- Typescript based, know what you are programming with
- How to make maintainable code
    - factory methods
    - Method abstraction with wrapper class
- js ES6

// youtube video here of working app

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

Store usage
```

Store.set('apiMutants', apiMutants);
Store.changed('discovered').subscribe((mutantIds: string[]) => {
    this.showInPanel(this.XMenDiscoveredEl, mutantIds, mutantsList)
});
```


## rxjs
```
private mapLoadedObs = new BehaviorSubject<any>({current:this.state, prev: {}});
```
