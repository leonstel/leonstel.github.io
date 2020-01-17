

# Finding  X MEN The Gathering

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

### Store

Default state of store
```
// src/app/store.ts

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

Initialization after the map has been initialized. Add mutants to google map with their corresponding type.
`firstTimeTrue` from other post



```
// src/main.ts

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




### XMenMap Component

Brace yourself, a bunch of code comming your way. Please dont be intimidated. 
If you read line by line hope it clearifies. After all the posts about small parts it is now
time to the complete class.

```
export class XMenMap extends MapBase{

    constructor(){
        // get the div from DOM which must contain the google map
        const xmenMapContainer = document.querySelector('#xmen-map');

        //send the element to the MapBase who takes care of putting it into the DOM
        super(xmenMapContainer);

        // get the mock data from store
        const apiMutants = Store.get('apiMutants');
        const wolverine = apiMutants.xmen.wolverine;

        // listen to realTimeLocation store prop, but wait for proffesorX and map initialization
        this.listToPropAfterMapInit('realTimeLocation', 'professorX').subscribe((loc: Location) => {

            // if realtime location has changed 

            if(loc){
                // because we are in the listener we can be certain that professorX prop is defined
                const professorX: ProfX = Store.get('professorX');

                // move the professorX marker via the googleMapsInstance to new location
                googleMapsInstance.move(loc, professorX.id);
                
                // let the wolverine follow the professorX marker, but only if the wolverine is not recruiting
                const isRecruiting: boolean = Store.get('isRecruiting');
                if(!isRecruiting){
                    googleMapsInstance.moveTo(wolverine.id, professorX.id);
                }

                // let the recruited mutant markers follow professorX
                const recruited: string[] = Store.get('recruited').filter( id => id !== wolverine.id && id !== professorX.id );
                recruited.forEach((mId: string) => {
                    googleMapsInstance.moveTo(mId, professorX.id);
                });

                // at every location change discover mutants that are in the radius of the professorX
                googleMapsInstance.discoverMutants();
            }

        });

        // listen to the professorX property, but wait for it to exist
        this.listToPropAfterMapInit('professorX', 'professorX').subscribe((profX: ProfX) => {
            googleMapsInstance.changeProfXRange(profX.radius);
        });
    }

    
    doMapInitLogic(): void {
        this.mapIsInitialized();
    }

    // when the marker has been clicked the googleMapInstance will call this 
    //handler through the context
    markerClicked(marker: MutantMarker): void {
        recruit(marker.data.mutant.id);
    }

}

```



### UI
```
new UI();
```

    

