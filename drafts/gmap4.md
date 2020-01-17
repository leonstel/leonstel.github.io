

# Finding  X MEN The Right Direction

### MapBase

// TODO diagram here

- Usable methods
- Make listen After map prop
    - nullpointer check
    - every where map init listeners
- IMAP

### Context and interaction, how it all starts
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
export class MapBase {
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
    
    

    

