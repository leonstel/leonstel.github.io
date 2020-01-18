

# Finding X MEN
<img src="../assets/finding_xmen/xmen.jpg" />

### The app
On the other day you get a phone call of someone called Charles, Charles Francis Xavier, he prefers to be called
Professor X. After elaborately sifting through the yellow pages he has come across your phone number as the most 
excellent freelance developer. You won't turn down this great opportunity and immediately ask him more about his wishes. 
Charles tells you that his Cerebro is broken and wants an upgrade so that discovering mutants will be even easier.
He has used Cerebro to easily discover hidden mutants in the area. 

<p align="center">
    <img src="../assets/finding_xmen/prof-x-cerebro.jpg" />
</p>

When you are done talking about the X Men movies you
represent him your ideas. Luckily the Cerebro already has an API implementation so that you are able to control it 
remotely. The main objective is to help professor X gather his yet to become X Men team.

// TODO correct youtube video here of working app
<p align="center">
    <iframe src="https://www.youtube.com/embed/hGlEOLbH00A" width="533" height="300" frameborder="0" allowfullscreen> </iframe>
</p>

## Features
- The realtime position will be linked to a professor X on the map (//TODO time in video)
- Control the radius of the range of the Cerebro and reflect it on the map (the blue circle around professor X) (//TODO time in video)
- Several types of mutants do exists (Alpha, Beta etc) so the option to control mutant markers of a specific type is (//TODO time in video)
desired
- When a mutant resides within the reach of the constantly moving professor X it will be marked as discovered (//TODO time in video)
- Wolverine will be sent after those discovered mutants to recruit them on the X Men team. (//TODO time in video)

## Learning points

*Store principle*  
What does state management mean? We will build a simple state manager from scratch with Rxjs observables. When
using state management in your own app I encourage you to look at redux or mobX these implement the flux pattern
in their own way. Back then I found it helpful to see the store principle in its most basic form. Maybe you have 
encountered it, the angular version (ngRx) uses a lot of boilerplate and a lot is done for you. When you are new 
to the flux storage principle that could be daunting.

*Google maps infrastructure*  
Google maps is the most used frontend mapping library I believe. The world would end up in ruins without it, climate 
change is child's play compared to not having google maps. After working a lot with it I believe that an thought out
structure is a necessity for writing good navigational code.

*js*  
While doing all this fancy stuff we will use Typescript and the `@types/googlemaps` package (type definition for google 
maps lib). Typescript will enforce you to write code strictly and eliminates many unnecessary bugs. Furthermore ES6 
features, factory methods and method abstraction will accompany our road.

## Google Maps Instance

First things first, what is the first step you could take for structuring google maps code? The answer is the google 
map instance, this is an instance with a singleton like feel. That class is responsible for manipulating the raw google 
maps object and other code is not meant to touch that. The idea is that other code could call methods on the 
GoogleMapInstance to do something with the map such as adding or moving markers. 

Only one raw google maps object reference will live in the application and it will be reused when switching maps.
This makes it easy for external code to do things with the map as well they only have to call methods on the 
GoogleMapInstance. Because every code deals with the same google map object reference changes made from another place 
will reflect directly on the map. For example you have created a google map via the GoogleMapInstance and have put that 
reference in the DOM. If somewhere else at a later time asynchronously fetched some api data and added markers via the 
GoogleMapInstance in its response then those marker will directly be visible on the earlier created map. No need to 
keep track of multiple google map references and their markers.
    
#### Mini Api

- why
    - initializes raw gmap obj
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
            draggable: true,
            zoom: 13,
            //... other init props here
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

##Initialize instance

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
// When we go further we code an infrastructure so 
// we don't have to this all the time any more and the instance existence 
// can be garantued
const mutantId = "...someid"
if(googleMapsInstance){
    googleMapsInstance.panTo(mutantId)      // manipulate map
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

The reason, this way you have no control when it is being initialize (as you can see in the constructor of the GoogleMapInstance
`this.googleMaps = new google.maps.Map(...) // google.maps.Map comes from external gmap lib`
). The google maps instance
use the google maps for initialization and the gmaps script is loaded asyncly. So it the app load
and the first js file that imports the googleMapsInstance will initialize the object.
The problems arise if the external google maps script has been somewhat delayed then 
the initialization happens too early and you get a null pointer. Later on I will talk about
catching a successful load of the libary script and then initiliazing the gmapInstance with `initGoogleMaps` 
on the right time 

## loading Google maps

All sounds good but without the actual gmaps lib it won't work. Yeah 

```
//src/app/init.ts

multiple because other google map extension comes sometime with different library

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
```

```
// src/main.ts      (js file where it all starts)

loadGoogleMapScripts();
```

### Fitting the Pieces



Link to next article 
