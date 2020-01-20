

# Finding  X MEN - The Gathering

<img src="../assets/finding_xmen/willy.jpg" />

We have covered much ground over the last three posts now it's time to finish up the app and be done with it once and for all.
After all the sweat and tears now will be the time to harvest the fruits from the three subjects where we discussed the Store, 
Google Maps mini Api and the infrastructure to quickly create new maps.

Professor X will be satisfied with the results! 

// TODO yt video embed of working app
// same as post 1

I will try to use less words this time and show you the code how this all comes together, too many words have already been said!

#### Mutant

The entire app is based on mutants so what actually is a mutant?

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

Mocking some api data which gives us a list of mutants. Below is defined the ApiResponse type and the actual json mock data.
```
// Api response type
interface ApiMutants {
    alpha: Mutant[],
    beta: Mutant[],
    xmen: {
        [key: string]: Mutant;
    }
}
```

```
// src/mutants.json

// Actual data
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

## Main roles

#### Professor X
Professor X is the main character in this play. Image that Professor X walks around town watching this app to discover new 
mutants and sending his partner Wolverine to recruit them. For demo purposes I have simulated the realtime location with 
some coordinates in an array which I am looping 
over. You can find that code in `src/app/position.ts`. For brevity I won't show it here.

ProfessorX has a radius on the map (blue circle), every mutant which comes within that range will be discovered

// PHOTO within app

#### Wolverine

Via an interface button you can tell the Wolverine to recruit one of the discovered mutants. Recruiting involves setting the
Wolverine marker to the target mutant, stay for a short term at that position and lastly let it follow the Professor X marker.

// PHOTO within app

### Store

Our store object will look likes this.
```
// src/app/store.ts

interface State {
    mapLoaded: boolean;         // Indicates if the map has been loaded
    mapInit: boolean;           // Indicates if the map has been initialized
    realTimeLocation: Location; // Gets update every time then the simulated real time position loops
    professorX: ProfX;          // The professor X mutant contains a radius as well
    apiMutants: any;            // The mocked api response object
    discovered: string[];       // An array of mutants that are discovered
    recruited: string[];        // An array of mutants that are recruited (the X Men team)
    isRecruiting: boolean;      // Let the app know if the Wolverine is busy recruiting
}
```

### initializing

How it all starts.

```
// src/main.ts

// Load mock data
const apiMutants = require('./mutants.json');

// Load external google maps script into DOM
loadGoogleMapScripts();

// Initialze the realtime position simulation loop
initializeRealtimePosition();

// Save professor X seperately in the store
Store.set('professorX', {
    ...apiMutants.xmen.professorX,
    radius: 1500
});

// Save the mocked api response obj in the store
Store.set('apiMutants', apiMutants);

// Create new custom map
new XMenMap();
```

Do some initial things with the map when the map when the map first initializes.

```
// src/main.ts

// Listen in the store when the map has been initialized for the first time
firstTimeTrue('mapInit').subscribe(() => {

    // Add the mutants to the map with their corresponding types 
    googleMapsInstance.addMutants(apiMutants.alpha, MutantType.Alpha);
    googleMapsInstance.addMutants(apiMutants.beta, MutantType.Beta);
    googleMapsInstance.addMutant(apiMutants.xmen.wolverine, MutantType.Wolverine);
    googleMapsInstance.addMutant(apiMutants.xmen.professorX, MutantType.ProfessorX);

    // Professor X and the Wolverine are marked as recruited on startup they are 
    // already on the X Men team
    Store.set('recruited', [
        apiMutants.xmen.professorX.id,
        apiMutants.xmen.wolverine.id
    ]);

    // Finally make all the added markers visible on the map
    googleMapsInstance.show();
});
```

### XMenMap Component

The custom map component which extends the `MapBase`.

```
/**
    extends from mapbase 
    - to get the map rendered in DOM, 
    - get map clickhandler
    - map lifecycle hooks
    - to use utility functions
**/
export class XMenMap extends MapBase{

    constructor(){
        // Get the div from DOM which must contain the google map
        const xmenMapContainer = document.querySelector('#xmen-map');

        // Send the element to the MapBase who takes care of putting it into the DOM
        super(xmenMapContainer);

        // Get the mock data from store
        const apiMutants = Store.get('apiMutants');
        const wolverine = apiMutants.xmen.wolverine;

        // Listen to the realTimeLocation store prop, but wait for proffesorX and map to be initialized
        this.listToPropAfterMapInit('realTimeLocation', 'professorX').subscribe((loc: Location) => {

            // If realtime location has changed 

            if(loc){
                // Because we are in the listener we can be certain that professorX prop is defined
                const professorX: ProfX = Store.get('professorX');

                // Move the professorX marker via the googleMapsInstance to new location
                googleMapsInstance.move(loc, professorX.id);
                
                // Let the wolverine follow the professorX marker, but only if the wolverine is not recruiting
                const isRecruiting: boolean = Store.get('isRecruiting');
                if(!isRecruiting){
                    googleMapsInstance.moveTo(wolverine.id, professorX.id);
                }

                // Let the recruited mutant markers follow professorX
                const recruited: string[] = Store.get('recruited').filter( id => id !== wolverine.id && id !== professorX.id );
                recruited.forEach((mId: string) => {
                    googleMapsInstance.moveTo(mId, professorX.id);
                });

                // At every location change discover mutants that are in the radius of the professorX
                googleMapsInstance.discoverMutants();
            }

        });

        // Listen to the professor X property, but wait for it to exist and for the map to be initialized
        this.listToPropAfterMapInit('professorX', 'professorX').subscribe((profX: ProfX) => {
            googleMapsInstance.changeProfXRange(profX.radius);
        });
    }

    // Let the MapBase know that the map initialization is done
    doMapInitLogic(): void {
        // You could do some other map initialization for the google map here
        this.mapIsInitialized();
    }

    // When the marker has been clicked the GoogleMapInstance will call this 
    // handler through its context
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

// PHOTO within app

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

    // randomly change the radius of professorX
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

### Google Map Instance Methods
 I will show some of the methods in gmap instance with manipulative method on google maps obj. But I cant cover them all in this articles 
 otherwise it would be too long. You can find all of them in `src/app/map/GoogleMapInstance.ts`.

```
public changeProfXRange(radius){
    const marker = this.getMarkerOfType(MutantType.ProfessorX);
    if(!marker) throw Error('could not change rang of profx, marker not found');
    marker.data.drawing.setRadius(radius);
}
```

```
public discoverMutants(){
    const profXMarker = this.getMarkerOfType(MutantType.ProfessorX);
    if(!profXMarker) throw Error('could not change rang of profx, marker not found');

    const recruited: string[] = Store.get('recruited');
    const mutantsInRange: string[] = this.markers
        .filter((marker: MutantMarker) => {
            // only check if marker is not yet recruited and is a discoverable mutant
            if(isDiscoverableMutant(marker) && !recruited.find( id => id === marker.data.mutant.id)){
                return markerIsInProfXRange(marker, profXMarker)
            }
            return false;
        })
        .map((marker:MutantMarker)=> marker.data.mutant.id);

    const currentDiscoveredMutants: string[] = Store.get('discovered');

    // set because every item must be unique
    const uniqueSet: Set<string> = new Set([
        ...currentDiscoveredMutants,
        ...mutantsInRange
    ]);
    const discoveredMutants = [...uniqueSet];

    Store.set('discovered', discoveredMutants);
}
```

### Last things about prof x marker

The create thing with factory methods for creating markers is that you can easily create a new one
for a whole different type of marker with extra or other functionality in comparison with other markers.

```
// src/app/map/markers.ts

export const createProfessorX = (loc: Location, mutant: Mutant, googleMapInstance: GoogleMapsInstance): MutantMarker => {
    //... create marker object like before

    // add gmaps circle polygon to the marker's data (the blue circle)
    const profX: ProfX | undefined = Store.get('professorX');
    if(!profX) throw Error('no profx found in store while creating marker');

    // create drawing for prof X
    marker.data.drawing = new google.maps.Circle({
        center: new google.maps.LatLng(loc.lat, loc.lon),
        radius: profX.radius,
        map: googleMapInstance.getGmapObj(),
        fillColor: "#80c3e8",
        fillOpacity: 0.50,
        strokeColor: 'transparent',
        clickable: false,
    });

    return marker;
};
```
