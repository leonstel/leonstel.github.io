

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

#Custom map component
```
export class XMenMap extends MapBase{

    constructor(){
        this.listToPropAfterMapInit('prop1').subscribe((val: any) => {
            // do something after prop1 changes but only after the map has been initialized
        });
    }

    doMapInitLogic(): void {

        //you could here do some initializing on the google maps

        // this lets the store know that map has been initialized
        this.mapIsInitialized();    
    }

    firstAfterMapInitialize(): void {
        // TODO do some panning or so for demonstration purpose
    }

    // to show multiple ways to do the same thing with google maps
    markerClicked(marker: MutantMarker): void {
        // do something when markers has been clicked on map
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
    
    

    

