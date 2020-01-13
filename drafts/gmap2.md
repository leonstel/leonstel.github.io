

# Finding X MEN
<img src="../assets/finding_xmen/xmen.jpg" />

### The app
// TODO correct youtube video here of working app
<p align="center">
    <iframe src="https://www.youtube.com/embed/hGlEOLbH00A" width="533" height="300" frameborder="0" allowfullscreen> </iframe>
</p>

Charles xavier you get phone call  
You won't turn down this great opportunity
Charles tells you that his Cerebro has gone broke and that he cant control it anymore manually,
only through the Cerebro Api.   
There will be mutants among you to be found  
Can you make an app to help me find mutants and gather the yet to come X MEN team?

##App features
- Separation of multiple mutants types (alpha beta) -> set video time where this happens
    - you can hide | show all mutants of type x at once -> set video time where this happens
- The Cerebro helmet has a fluctuating range radius ( the blue circle around professor X), range must be dynamically updated every time -> set video time where this happens
- When mutants are detected within professor x range mark them as discovered -> set video time where this happens
- When mutants are discovered you can send the Wolverine to recruit them on the X MEN team -> set video time where this happens


## Learning points
- Store principle vanilla js
- Getting you in the right direction with reusable googlemaps infrastructure
- Typescript based, know what you are programming with,
uses the ```@types/googlemaps``` packages to get google maps types
- How to make maintainable code
    - factory methods
    - Method abstraction with wrapper class
- js ES6


### Google Maps Instance
- why
    - initializes raw gmap obj
    - one raw gmap obj reference in whole app, so if something from an other place calls a manipulative
    method on the instance then those change reflect to every other place where the gmap is being used.
    For example if you add in place B the gmap obj of the instance to the DOm and in place A add markers via the
    google map instance. The addition of marker will immediatly be reflected in the DOM because it is the same refenced gmap obj
    - little custom api around gmaps
    - debuggable
    - not everywhere in code lingers some raw google maps object code
    - easy way to maintain gmap enitities like markers in once place (with one reference)
    - handy abstraction layer for manipulating those enitites and google maps itself
- singleton feel
- method wrapper
- initialize map in constructor
- simple gmap manipulation (these two dont exist in current app)
- demonstrate methods
    - zoom  
    - pan
- how can it be called
- this is just the begin, later on the MapBase class

The methods in this instance manipulating google map in many ways.
Read the comments what the function do. The body code of the functions
will be discussed in following articles. I hope this gives a concise impression
what this instance does.

All the methods covers the app features listed above. 

The ```private googleMaps: google.maps.Map;``` property contains the raw google map object.
The function uses this object to make something happend on google maps like adding markers
or change center etc.

```
// src/app/map/GoogleMapInstance.ts

export class GoogleMapsInstance {
    private googleMaps: google.maps.Map;
    public el: HTMLElement;
    private context: IMap;

    private markers: MutantMarker[] = [];

    constructor() {
        console.log('Gmap pinstnace construct');

        this.el = document.createElement('div');
        this.el.setAttribute('id', 'map');

        this.googleMaps = new google.maps.Map(this.el, {
            center: new google.maps.LatLng(defaultLocation.lat, defaultLocation.lon),
            // @ts-ignore
            styles: mapStyles,
            streetViewControl: false,
            rotateControl: false,
            fullscreenControl: false,
            disableDefaultUI: true,
            zoomControl: false,
            gestureHandling: 'greedy',
            clickableIcons: true,
            draggable: true,
            zoom: 13,
        });
    }

    //bodies are left empty for brevity
    private addMarker(marker: MutantMarker): void
    public show(mutantType?: MutantType): void
    public hide(mutantType?: MutantType): void
    public addMutants(mutants: Mutant[], type: MutantType): void
    public addMutant(mutant: Mutant, type: MutantType): void
    public move(loc: Location, mutantId: string): void
    public moveTo(toBeMovedId: string, targetId: string): void
    public panTo(mutantId: string): void
    public changeProfXRange(radius): void
    public discoverMutants(): void
    public recruit(mutantId: string): PromiseLike<void>
}
```

#Initialize instance

```
// src/app/map/GoogleMapInstance.ts

export const initGoogleMaps: () => void = (): void => {
    if(!googleMapsInstance) {
        googleMapsInstance = new GoogleMapsInstance();
    }
};
```

```
// somewhere else

import {initGoogleMaps, googleMapsInstance} from "./map/GoogleMapsInstance";
initGoogleMaps();                       // only done once in app

// In this example without context where this snippet is being run it 
// is important to check if the instance exists. 
// When we go further we code an infrastructure when 
// we don't have to this all the time any more
const mutantId = "...someid"
if(googleMapsInstance){
    googleMapsInstance.panTo(mutantId)      // manipulate map}
}
```

ES6 export
I hear you think, but why don't you defined it not this way. Isn't above a little
cumbersome? I get your point when you export an created object js will everywhere it is imported give the reference to that created object (like the singleton principle)
, only the first time it has been called and it does not exist yet it will execute the expression
and creates a new object.

```
export let googleMapsInstance: GoogleMapsInstance = new GoogleMapsInstance();
```

The reason, this way you have no control when it is being initialize. The google maps instance
use the google maps for initialization and the gmaps script is loaded asyncly. So it the app load
and the first js file that imports the googleMapsInstance will initialize the object.
The problems arise if the external google maps script has been somewhat delayed then 
the initialization happens to early and you get a null pointer. Later on I will talk about
catching a successful load of the libary script and then initiliazing the gmapInstance with `initGoogleMaps` 
on the right time 

## loading Google maps

```
//src/app/init.ts

const addGoogleMapsScript = (): PromiseLike<void> => {
    return new Promise((resolve, reject) => {
        const googleMapsUrl: string = 'https://maps.googleapis.com/maps/api/js?key=AIzaSyAvOv3Il7OP5L8z3T7u6wSiz72XZY1XIDo';

        if (!document.querySelectorAll(`[src="${googleMapsUrl}"]`).length) {
            document.body.appendChild(Object.assign(
                document.createElement('script'), {
                    type: 'text/javascript',
                    src: googleMapsUrl,
                    onload: (): void => {
                        resolve();
                    },
                    onerror: (e:any): void => {
                        reject();
                    },
                }));
        }
    });
};

export const loadGoogleMapScripts = (): void => {
    const loadGoogleMapsMainScript = addGoogleMapsScript();

    //Promise all because google maps have some things separated in different libs and different files
    Promise.all([loadGoogleMapsMainScript]).then(() => {
        Store.set('mapLoaded', true);
    }).catch((e:any) => {
        throw Error('Could not load Google Maps properly!');
    });
};
```

### Atherthough
Dont use in production
Not really a stable way of having deep layers of state (detecting change)


Link to next article 
