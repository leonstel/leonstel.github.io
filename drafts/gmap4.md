

# Finding  X MEN The Right Direction

### MapBase

// TODO diagram here

- Usable methods
- Make listen After map prop
    - nullpointer check
    - every where map init listeners
- IMAP

```
export interface IMap {
    doMapInitLogic(): void;
    firstAfterMapInitialize(): void;
    markerClicked(marker: MutantMarker): void;
}

    protected googleMapsInstance;

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

    private afterMapLoaded(){
        this.setupMap();
        this.doMapInitLogic();
    }

    protected mapIsInitialized(): void {
        Store.set('mapInit', true);
    }

    // Method to be overwritten
    afterMapInit(){}

    //SCENARIOS
    // TODO write scenarios for blog
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
    firstAfterMapInitialize(): void {}
```


### Custom Map Component
every custom component is IMap

```
export class XMenMap extends MapBase{

    constructor(){
        this.listToPropAfterMapInit('prop1', 'professorX').subscribe((val: any) => {
            
        });

        // the second argument indicates that it waits on professorX to be defined
        this.listToPropAfterMapInit('professorX', 'professorX').subscribe((profX: ProfX) => {
            googleMapsInstance.changeProfXRange(profX.radius);
        });
    }

    doMapInitLogic(): void {
        this.mapIsInitialized();
    }

    firstAfterMapInitialize(): void {
        // TODO do some panning or so for demonstration purpose
    }

    // to show multiple ways to do the same thing with google maps
    markerClicked(marker: MutantMarker): void {
        recruit(marker.data.mutant.id);
    }

    private afterMapLoaded(){
        this.setupMap();
        this.doMapInitLogic();
    }

    protected mapIsInitialized(): void {
        Store.set('mapInit', true);
    }

    private setupMap() {
        initGoogleMaps();
        this.googleMapsInstance = googleMapsInstance;
        this.googleMapsInstance.setContext(this);
        this.element.appendChild(googleMapsInstance.el);
    }

    // IMap interface methods, that must be overwritten by its children (custom map components)
    doMapInitLogic(): void {}
    markerClicked(marker: MutantMarker): void {}
    firstAfterMapInitialize(): void {}

}
```


### Context and interaction
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
export class XMenMap extends MapBase{
    
    markerClicked(marker: MutantMarker): void {
        recruit(marker.data.mutant.id);
    }
}
```


### Map Interaction
- clickhandler to custom map component through context


## Map instance  indepth
- Mapbase related setContext
    - click handler -> context (MapBase structure)
    markerClickHandler(marker: MutantMarker){}

 - helper methods like:
    private getMarkerOfId(mutantId: string): MutantMarker | undefined {}
    private getMarkerOfType(mutantType: MutantType): MutantMarker | undefined {}


    
### Markers
- adding markers
    - marker factory methods
    
    

    

