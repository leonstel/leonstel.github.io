

# Finding  X MEN The Right Direction

### Basic Infrastructure

// TODO diagram here

- Usable methods
- Make listen After map prop
    - nullpointer check
    - every where map init listeners
- IMAP

```
export class GoogleMapsInstance {
    private context: IMap;
    
    public setContext(context: MapBase): void {
        this.context = context;
    }

    private addMarker(marker: MutantMarker): void {
        marker.addListener('click', this.markerClickHandler.bind(this, marker));
    }

    markerClickHandler(marker: MutantMarker){
        this.context.markerClicked(marker);
    }
    ...
}
```

```
export class MapBase implement IMap {
    private setupMap() {
        initGoogleMaps();
        this.googleMapsInstance = googleMapsInstance;
        this.googleMapsInstance.setContext(this);
        this.element.appendChild(googleMapsInstance.el);
    }
}
```

```
export class XMenMap extends MapBase{
    
    markerClicked(marker: MutantMarker): void {
        recruit(marker.data.mutant.id);
    }
}
```

This way you can create really fast new map components. Then you can use to MapBases method to quickly
hook up with google maps.

The MapBase is responsible for putting the googlemaps in the DOM

### Map Base in depth

// TODO schema how the methods are being called after loading and initialize
// with store

```
export interface IMap {
    doMapInitLogic(): void;
    markerClicked(marker: MutantMarker): void;
    afterMapInit(): void;
}

export class MapBase implements IMap{
    ...
}
```

```
// src/app/store.ts

private state: State = {
    mapLoaded: false,       // indicates if the external gmaps script has been loaded
    mapInit: false,         // indicates if the google maps has been initialized
};
```

Describe flow from script loading to mapbases functions with diagram?

Uses articles 2 `firstTimeTrue` store utils function

```
// src/app/map/MapBase.ts

constructor(private element){
    if(!this.element) throw Error('html element required for MapBase');

    const mapLoaded = firstTimeTrue('mapLoaded')
    const mapInit = firstTimeTrue('mapInit');

    mapLoaded.subscribe( this.afterMapLoaded.bind(this));
    mapInit.subscribe( this.afterMapInit.bind(this));
}
```

### Timeout

Extend upon above constructor with external gmaps script loading timeout

```
// src/app/map/MapBase.ts

constructor(private element){
    if(!this.element) throw Error('html element required for MapBase');

    // timeout stream
    const TIMEOUT = 5000;
    const timer$ = timer(TIMEOUT).pipe(
        tap( x =>  {
            throw new Error(`Map took too long to load!, timeout is ${TIMEOUT}`)
        })
    );
    const mapLoaded = firstTimeTrue('mapLoaded').pipe(takeUntil(timer$));
    const mapInit = firstTimeTrue('mapInit');

    mapLoaded.subscribe( this.afterMapLoaded.bind(this));
    mapInit.subscribe( this.afterMapInit.bind(this));
}
```

Setupmap
```
// src/app/map/MapBase.ts

private afterMapLoaded(){
    this.setupMap();
    this.doMapInitLogic();
}

private afterMapInit(){

}

private setupMap() {
    initGoogleMaps();
    this.googleMapsInstance = googleMapsInstance;
    this.googleMapsInstance.setContext(this);
    this.element.appendChild(googleMapsInstance.el);
}

protected mapIsInitialized(): void {
    Store.set('mapInit', true);
}
```

Uses articles 2 `changedButWaitFor` store utils function


###Listening to map related props

```
// src/app/map/MapBase.ts

protected listToPropAfterMapInit = (prop: any, ...rest) => {
    const params = [
        prop,
        ...rest,
        'mapInit'
    ];
    return changedButWaitFor.apply(null, params);
};
```

#Custom map component
```
export class XMenMap extends MapBase{

    constructor(){
        // the container div where to map is going to be put in
        const xmenMapContainer = document.querySelector('#xmen-map');
        super(xmenMapContainer);

        this.listToPropAfterMapInit('prop1').subscribe((val: any) => {
            // do something after prop1 changes but only after the map has been initialized
        });
    }
    
    // overrides the MapBase' doMapInitLogic()
    doMapInitLogic(): void {
        //you could here do some initializing on the google maps

        // this lets the store know that map has been initialized
        // once this has been called then every listToPropAfterMapInit observers
        // finally be called
        this.mapIsInitialized();    
    }

    markerClicked(marker: MutantMarker): void {
        // do something when markers has been clicked on map
    }

}
```


    
### Markers

We will go into the mutant specifics in the next and last article. But these typescript interfaces are used when 
creating markers. 

```
export enum MutantType {
    Alpha = 'Alpha',
    Beta = 'Beta'

    // ... next post adding more types
}

```

```
export interface MutantMarker extends google.maps.Marker {
    data?: {
        mutant?: Mutant;
        mutantType: MutantType;
    };
}
```

#### Creating markers (factories)

```
const createMarker = (lat: number, lng: number, mutantType: MutantType, options: google.maps.MarkerOptions = {}): MutantMarker => {
    let markerOptions = defaultMarkerOptions(lat, lng);
    markerOptions = {
        ...markerOptions,
        ...options,
    };

    const marker: MutantMarker = new google.maps.Marker(markerOptions);
    marker.data = {
        mutantType: mutantType,
    };
    return marker;
};
```


Creating an AlphaMutant marker example

```
// src/app/map/markers.ts

export const createAlphaMutant = (mutant: Mutant): MutantMarker => {
    const icon: google.maps.Icon = {
        url : `../../assets/${mutant.img}`,
        scaledSize: new google.maps.Size(70, 70),
        anchor: new google.maps.Point(35, 40),
        labelOrigin: new google.maps.Point(0,0)
    };

    const markerOptions: google.maps.MarkerOptions = {
        icon,
        label: `${mutant.name}`,

    };

    // calls the above generalized createdMarker factory
    const marker: MutantMarker = createMarker(mutant.location.lat, mutant.location.lon, MutantType.Alpha, markerOptions);
    marker.data = {
        ...marker.data,
        mutant: <Mutant>{
            ...mutant,
        },
    };
    return marker;
};
```

Why did we add types with the factory to create a marker?

With those types you can easily make function to group those markers together.
For example if you have many mutant types and you want to define with one function call
if the input marker is a discoverable mutant. Those function keeps conditional marker checken
maintainable and flexible.


```
// src/app/map/markers.ts

export const isDiscoverableMutant = (marker: MutantMarker): boolean => {
    return(
        marker.data.mutantType === MutantType.Alpha ||
        marker.data.mutantType === MutantType.Beta
    );
};
```

Every you want to check if a marker is discoverable you call this function

```
// src/app/map/GoogleMapInstance.ts

if(isDiscoverableMutant(marker)){
    //... do something that is only intended for discoverable mutants
}
```



#### Adding markers

```
export class GoogleMapsInstance {
    private markers: MutantMarker[] = [];
        
}
```



    

    

