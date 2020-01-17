

# Finding  X MEN - The Gathering

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

### ProfessorX
ProfessorX is the main character in this play. Image that Profx walks around town watching this app to discover new 
mutants and sending his partner Wolverine to recruit them. For demo purposes I have simulated the realtime location with some coordinates in an array which I am looping 
over. You can find that code in `src/app/position.ts`. For brevity I won't show it here.

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

// other post
loadGoogleMapScripts();

// initializing the realtime simulation loop
initializeRealtimePosition();

// save api data to the store
// professor x will be saved separatly because its 
// radius will be controlled through the store
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

`MapBase` from other post

```
/**
    extends from mapbase 
    - to get the map rendered in DOM, 
    - get map clickhandler
    - map lifecycle hooks
**/
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

        // listen to the professorX property, but wait for it to exist and for map initialization
        this.listToPropAfterMapInit('professorX', 'professorX').subscribe((profX: ProfX) => {
            googleMapsInstance.changeProfXRange(profX.radius);
        });
    }

    // call by the MapBase after the map has been loaded
    // let the MapBase know that the map initialization is done
    doMapInitLogic(): void {
        // you could do some initialization for the google map here
        this.mapIsInitialized();
    }

    // when the marker has been clicked the googleMapInstance will call this 
    //handler through the context
    markerClicked(marker: MutantMarker): void {

        // recruit the marker's mutant 
        recruit(marker.data.mutant.id);
    }

}

```



### UI
```
// src/main.ts

new UI();
```

```
// src/app/UI.ts

export class UI {

    constructor(){
        // init listeners only once after the map has been initialized
        const mapLoaded = firstTimeTrue('mapInit');
        mapLoaded.subscribe( this.initListeners.bind(this));

        //... renders html content
    }

    // map will be available when initialzing listeners
    initListeners() {
        //... button listeners

        Store.changed('discovered').subscribe((mutantIds: string[]) => {
            // update ui when discovered mutants state prop changed
        });

        Store.changed('recruited').subscribe((mutantIds: string[]) => {
            // update ui when recruited mutants state prop changed
        });
    }

    private mutantClicked(e){
        const mutantId: string = e.target.getAttribute('mutant-id');
        googleMapsInstance.panTo(mutantId)
    }

    private toggleDisplay(mutantType: MutantType, hide=false) {
        if(!hide) googleMapsInstance.show(mutantType);
        else googleMapsInstance.hide(mutantType);
    }

    private profXRangeClicked(){
        const max = 2500, 
            min = 500, 
            randomRadius = Math.floor(randomInBetween(max,min));

        Store.update('professorX', {
            radius: randomRadius
        });
    }

    private async recruitClicked(){
        const [firstId] = Store.get('discovered');
        await recruit(firstId)
    }
}

```

    

