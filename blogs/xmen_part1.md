

# Finding X MEN
<img src="../assets/finding_xmen/xmen.jpg" />

## The App
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

*<sub>Unfortunately google maps has changed its policy while I was working on this app. Now you have to supply a creditcard
otherwise a grey development layer will occur. That's why you see a gray overlay, the functionality will stay the same
and should not interfere with learning new stuff :)</sub>*  

<p align="center">
    <iframe src="https://www.youtube.com/embed/T_Ob9p4vbrw" width="533" height="300" frameborder="0" allowfullscreen> </iframe>
</p>

#### Git Repo
[Finding X MEN git repo](https://github.com/leonstel/techblog_finding_xmen)

## Features
- The realtime position will be linked to a professor X on the map **(from 0:02 and on)**
- Control the radius of the range of the Cerebro and reflect it on the map (the blue circle around professor X) **(0:22)**
- Several types of mutants do exists (Alpha, Beta etc) so the option to control mutant markers of a specific type is desired **(0:40)**
- When a mutant resides within the reach of the constantly moving professor X it will be marked as discovered **(from 0:02 and on)**
- Wolverine will be sent after these discovered mutants to recruit them on the X Men team. **(1:20)**

## Learning Points

*Store principle*  
What does state management mean? We will build a simple state manager from scratch with Rxjs observables. When
using state management in your own app I encourage you to look at redux or mobX these implement the flux pattern
in their own way. Back then I found it helpful to see the store principle in its most basic form. Maybe you have 
encountered it, the angular version (ngRx) uses a lot of boilerplate and a lot is done for you. When you are new 
to the flux storage principle that could be daunting.

*Google maps infrastructure*  
Google maps is the most used frontend mapping library I believe. The world would end up in ruins without it, climate 
change is child's play compared to not having google maps. After working a lot with it I believe that a thought out
structure is a necessity for writing good navigational code.

*js*  
While doing all this fancy stuff we will use Typescript and the `@types/googlemaps` package (type definition for google 
maps lib). Typescript will enforce you to write code strictly and eliminates many unnecessary bugs. Furthermore ES6 
features, factory methods and method abstraction will accompany our road.

*Rxjs*  
Rxjs js observable streams will be used for the store state listeners.  

## Google Maps Instance

First things first, what is the first step you could take for structuring google maps code? The answer is the 
`GoogleMapsInstance`, this is an instance with a singleton like feel. That class is responsible for manipulating the raw google 
maps object whereas other code is not meant to touch that. The idea is that other code could call methods on the 
GoogleMapInstance to do something with the map such as adding or moving markers. 

#### Mini Api
Only one raw google maps object reference will live in the application and it will be reused when switching maps.
This makes it easy for external code to do things with the map as well they only have to call methods on the 
GoogleMapsInstance. Because every code deals with the same google map object reference changes made from another place 
will reflect directly on the map. For example you have created a google map via the GoogleMapsInstance and have put that 
reference in the DOM. Some code elsewhere at a later time will asynchronously fetch some api data and adds markers via the 
GoogleMapsInstance in its response. These markers will directly be visible on the earlier created map there is no need to 
keep track of multiple google map references and their markers. It is kind of an internal mini API.
    
*Pros*
- Debuggable
- Handy for configurations, easy map switching
- Easy manner to store and maintain gmap entities like markers and polygons
- Clean code. The codebase is not littered with raw google maps references everywhere.

*Cons*
- No two different maps could be shown at the same time

In one of the next posts the `GoogleMapsInstance` will be complemented by the `MapBase` to harvest even more advantages
to quickly create new Custom Map Components.

#### In Practice

Below some code to give an impression of what the `GoogleMapInstance` looks like in this project. All the public methods 
will be used to cover the above features by manipulate the raw google maps object in some 
way. The function bodies are left empty to give an eloquent overview. I hope that the function names tell you everything 
about what it is going on which is always a good practice.

*<sub>Note that the terms marker and mutant are interchangeable</sub>*

```
// src/app/map/GoogleMapInstance.ts

export class GoogleMapsInstance {

    // Contains the raw google maps object
    private googleMaps: google.maps.Map;

    // Contains the el which is linked to the google maps object
    public el: HTMLElement;

    // An array to keep track of all the markers of the map
    private markers: MutantMarker[] = [];

    constructor() {
        // Create virtual DOM element which will hold the map
        this.el = document.createElement('div');

        // Set some id could be handy for styling
        this.el.setAttribute('id', 'map');

        // Initialize raw google maps object with default config
        this.googleMaps = new google.maps.Map(this.el, {
            center: new google.maps.LatLng(defaultLocation.lat, defaultLocation.lon),
            draggable: true,
            zoom: 13,
            //... other init props here
        });
    }

    private addMarker(marker: MutantMarker): void

    // Methods which manipulate the raw google maps object      
    
    /**
        Show | hide markers on map of a mutantType
        Show | hide all if no type given
    **/
    public show(mutantType?: MutantType): void     
    public hide(mutantType?: MutantType): void


    /**
        Create markers | marker with a mutantType
    **/
    public addMutants(mutants: Mutant[], type: MutantType): void
    public addMutant(mutant: Mutant, type: MutantType): void

    /**
        move:   moves a marker of given ID to a Location (lat,lng)
        moveTo: moves a marker of given ID to another marker of ID
        panTo:  sets the center of the method to the location of 
                the marker with ID
    **/
    public move(loc: Location, mutantId: string): void
    public moveTo(toBeMovedId: string, targetId: string): void
    public panTo(mutantId: string): void

    // Change the range of the radius of the blue cirlce of the
    // prof X marker
    public changeProfXRange(radius): void

    // Every time on realtime location this methods will be called to 
    // check if some mutants are within the range of prof X
    public discoverMutants(): void

    // Recruits a mutant of ID (only done when mutant is discovered)
    public recruit(mutantId: string): PromiseLike<void>
}
```

#### Initialize Instance

Singleton principle is used which means that only one instantiation is allowed.

```
// src/app/map/GoogleMapsInstance.ts

export const initGoogleMaps: () => void = (): void => {
    if(!googleMapsInstance) {
        googleMapsInstance = new GoogleMapsInstance();
    }
};
```

Most basic example of using the `GoogleMapsInstance`

```
import {initGoogleMaps, googleMapsInstance} from "./map/GoogleMapsInstance";

// Initiated the GoogleMapInstance for later use
initGoogleMaps();                       

// Place the virtual DOM element in which the google map has ben place 
// in the actual DOM 
document.querySelector('body').appendChild(googleMapsInstance.el);

// It is not clear where this is being run so a proper undefined check is applicable
// The updated infrastructure in next posts will eliminate the need for most null checks 
if(googleMapsInstance){

    // Pan the google map to the marker with ID
    const mutantId = "...someid"
    googleMapsInstance.panTo(mutantId)
}
```

I hear you thinking on the ES6 export, why don't you define it this way? This achieves the same thing 
that you have said previously. Isn't your version a little cumbersome?  

```
export let googleMapsInstance: GoogleMapsInstance = new GoogleMapsInstance();
```
Yeah, I get your point when you export a created object js will everywhere it is imported give the reference to that 
created object (like the singleton principle). Only the first time it has been called and it does not exist yet it will 
execute the expression and creates a new object.

To give you an explanation, you will have more control when it is actually being initialized. Since the `GoogleMapsInstance` 
creates the raw google maps object in its constructor it needs the external gmap library to do that. 
That script will be loaded asynchronously this means that if it encounters a delay at loading the app could 
potentially come across a `googleMapsInstance.panTo()` or so. Well you guessed it, it would initialize the `GoogleMapsInstance` too early.

## Loading Google Maps

All sounds good but without the actual gmaps lib it won't work. So lets start some loading. 

```
//src/app/init.ts

export const loadGoogleMapScripts = (): void => {
    
    //Each returns a promise
    const loadGoogleMapsMainScript = addGoogleMapsScript();
    const loadingOther1 = addOtherScript1();
    const loadingOther2 = addOtherScript2();

    Promise.all([
            loadGoogleMapsMainScript,
            loadingOther1,
            loadingOther2
        ]).then(() => {
            Store.set('mapLoaded', true);
        }).catch((e:any) => {
            throw Error('Could not load Google Maps properly!');
        });
};
```

We will load all the google maps scripts asynchronously. I say all because some extensions on the general gmap lib have 
their own libs. Every individual script loading unit returns a promise that succeeds if the script has been loaded.
After that `Promise.all` digests all the loading promises and succeeds if every individual promise has succeeded. If one of 
them fails the `Promise.all` fails. 

At last the `Store` will be notified that google maps has loaded correctly. Other code
could act upon this `mapLoaded` prop as we will see in later posts.

#### Individual Loading Unit

A loading unit is my own fancy made up name for returning a `Promise` which succeeds when a script has been loaded and placed in the 
DOM.

```
//src/app/init.ts

const addGoogleMapsScript = (): PromiseLike<void> => {
    return new Promise((resolve, reject) => {
        const googleMapsUrl: string = 'https://maps.googleapis.com/maps/api/js?key={YOUR_KEY_HERE}';

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

And finally call the function at the highest level of your application.

```
// src/main.ts      (js file where it all starts)

loadGoogleMapScripts();
```

## Fitting the Pieces

Wow you have been through a lot and a break would be appropriate for sure. To recap we have contacted professor X with 
our game breaking idea. The whole concept pivots on google maps manipulation so we had to figure out a way to effectively 
go about it. Therefor we have come up with the GoogleMapsInstance which solely job is to manipulate the raw google maps object
by accessible methods like a mini API. We have done some loading and are ready for the next step.

In the next post we will go indepth into the basic Store architecture. Although terms like Redux, MobX and Flux will come
by we will focus on implementing the most basic form of what it means to have a Store and why state management is great.
This helped myself greatly to understanding the underlying principles.

After that the infrastructure is our next victim and will be made even better to easily create new Maps. It ends in taking
all discussed topics and combining them to finish up the app. 

#### Git Repo
[Finding X MEN git repo](https://github.com/leonstel/techblog_finding_xmen)

#### Blogs
[Finding X MEN (Part1)](http://leonstel.github.io/blogs/xmen_part1)  
[Finding  X MEN - Without Italian Delicacy (Part2)](http://leonstel.github.io/blogs/xmen_part2)  
[Finding  X MEN - The Right Direction (Part3)](http://leonstel.github.io/blogs/xmen_part3)  
[Finding  X MEN - The Gathering (Part4)](http://leonstel.github.io/blogs/xmen_part4)

[Home](http://leonstel.github.io/)

##### The Tip of the Iceberg
You have only seen the tip of the iceberg. Like the titanic we won't see it coming.
<p align="center">
    <img src="../assets/finding_xmen/titanic.jpg" />
</p>
