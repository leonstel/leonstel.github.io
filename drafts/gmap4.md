

# Finding  X MEN The Right Direction

### MapBase

// TODO diagram here

- Usable methods
- Make listen After map prop
    - nullpointer check
    - every where map init listeners
- IMAP

### Basic Infrastructure
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

    doMapInitLogic(): void {
        //you could here do some initializing on the google maps

        // this lets the store know that map has been initialized
        // once this has been called then every listToPropAfterMapInit observers
        // finally be called
        this.mapIsInitialized();    
    }

    // to show multiple ways to do the same thing with google maps
    markerClicked(marker: MutantMarker): void {
        // do something when markers has been clicked on map
    }

}
```

### Map Base in depth
```
// src/app/store.ts

private state: State = {
    mapLoaded: false,       // indicates if the external gmaps script has been loaded
    mapInit: false,         // indicates if the google maps has been initialized
};
```

```
export interface IMap {
    doMapInitLogic(): void;
    markerClicked(marker: MutantMarker): void;
    afterMapInit(): void;
}
```

```
// src/app/map/MapBase.ts

constructor(private element){
    if(!this.element) throw Error('html element required for MapBase');

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

#### Timeout

```
// src/app/map/MapBase.ts

constructor(private element){
    if(!this.element) throw Error('html element required for MapBase');

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


```
export class MapBase implements IMap{
    protected googleMapsInstance;

    

    private afterMapLoaded(){
        this.setupMap();
        this.doMapInitLogic();
    }

    protected mapIsInitialized(): void {
        Store.set('mapInit', true);
    }

    // Method to be overwritten
    afterMapInit(){}
    protected listToPropAfterMapInit = (prop: any, ...rest) => {
        const params = [
            prop,
            ...rest,
            'mapInit'
        ];
        return changedButWaitFor.apply(null, params);
    };

    private setupMap() {
        initGoogleMaps();
        this.googleMapsInstance = googleMapsInstance;
        this.googleMapsInstance.setContext(this);
        this.element.appendChild(googleMapsInstance.el);
    }

    // IMap interface methods, that must be overwritten by its children
    doMapInitLogic(): void {}
    markerClicked(marker: MutantMarker): void {}
}
```


    
### Markers
- adding markers
    - marker factory methods
    
    

    

