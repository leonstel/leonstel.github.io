

# Finding  X MEN Gathering

// TODO yt video embed of working app
// same as post 1

#### Mutant

So what actually is a mutant

```
// single mutant
interface Mutant {
    id: string;
    name: string;
    img: string;
    location?: Location;
}

```

### Api data
```

// api response
interface ApiMutants {
    alpha: Mutant[],
    beta: Mutant[],
    xmen: {
        [key: string]: Mutant;
    }
}
```

##### Mock data
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

### Store

Default state of store
```
interface State {
    mapLoaded: boolean;
    mapInit: boolean;
    realTimeLocation: Location;
    professorX: ProfX;
    apiMutants: any;
    recruited: string[];
    discovered: string[];
    isRecruiting: boolean;
}
```

### XMenMap Component

Brace yourself, a bunch of code comming your way. Please dont be intimidated. 

```
export class XMenMap extends MapBase{

    constructor(){
        const xmenMapContainer = document.querySelector('#xmen-map');
        super(xmenMapContainer);

        const apiMutants = Store.get('apiMutants');
        const wolverine = apiMutants.xmen.wolverine;

        // listen for realTimeLocation and mapInit but wait for professorX, observable stream
        this.listToPropAfterMapInit('realTimeLocation', 'professorX').subscribe((loc: Location) => {

            if(loc){
                // because the listener is only getting fired if the professorX exists in store, we can be
                // certain that it is not undefined
                const professorX: ProfX = Store.get('professorX');
                googleMapsInstance.move(loc, professorX.id);

                const isRecruiting: boolean = Store.get('isRecruiting');
                if(!isRecruiting){
                    googleMapsInstance.moveTo(wolverine.id, professorX.id);
                }

                // let the recruited ones follow professorX
                const recruited: string[] = Store.get('recruited').filter( id => id !== wolverine.id && id !== professorX.id );
                recruited.forEach((mId: string) => {
                    googleMapsInstance.moveTo(mId, professorX.id);
                });

                googleMapsInstance.discoverMutants();
            }

        });

        // the second argument indicates that it waits on professorX to be defined
        this.listToPropAfterMapInit('professorX', 'professorX').subscribe((profX: ProfX) => {
            googleMapsInstance.changeProfXRange(profX.radius);
        });
    }

    doMapInitLogic(): void {
        this.mapIsInitialized();
    }

    // to show multiple ways to do the same thing with google maps
    markerClicked(marker: MutantMarker): void {
        recruit(marker.data.mutant.id);
    }

}

```

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

    

