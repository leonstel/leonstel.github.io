

# Finding  X MEN Gathering

// TODO yt video embed of working app
// same as post 1

### Api data


### initializing
```
// src/main.ts

const apiMutants = require('./mutants.json');

loadGoogleMapScripts();
initializeRealtimePosition();

Store.set('professorX', {
    ...apiMutants.xmen.professorX,
    radius: 1500
});
Store.set('apiMutants', apiMutants);

new XMenMap();
```

- store


Adding other mutants
```
firstTimeTrue('mapInit').subscribe(() => {
    googleMapsInstance.addMutants(apiMutants.alpha, MutantType.Alpha);
    googleMapsInstance.addMutants(apiMutants.beta, MutantType.Beta);
    googleMapsInstance.addMutant(apiMutants.xmen.wolverine, MutantType.Wolverine);
    googleMapsInstance.addMutant(apiMutants.xmen.professorX, MutantType.ProfessorX);

    Store.set('recruited', [
        apiMutants.xmen.professorX.id,
        apiMutants.xmen.wolverine.id
    ]);

    googleMapsInstance.show();
});
```



### UI
```
new UI();
```

    

