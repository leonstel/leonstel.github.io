

# Finding  X MEN Without Italian Delicacy
<img width="100%" src="../assets/finding_xmen/recipe.jpg" />

*Recipe: Spaghett*  
*Ingredients*
- *No state management*

```
//src/main.ts

loadGoogleMapScripts();
initializeRealtimePosition();
```

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
    
    ...

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


