

# Finding  X MEN Gathering

// TODO yt video embed of working app
// same as post 1

### Api data

#### Mutant
```
export interface ApiMutants {
    alpha: Mutant[],
    beta: Mutant[],
    xmen: {
        [key: string]: Mutant;
    }
}
```

#### Mock data
```
// src/mutants.json

{
  "alpha": [
    {
      "id": "6894f4f8-2720-11ea-978f-2e728ce88125",
      "name": "Magneto",
      "img": "magneto.png",
      "location": {
        "lat": 51.8148042,
        "lon": 4.6846541
      }
    },
    ...
  ],
  "beta": [
    ...
  ],
  "xmen": {
    "professorX": {
        ...
    },
    "wolverine": {
        ...
      }
    }
  }
}

```

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

    

