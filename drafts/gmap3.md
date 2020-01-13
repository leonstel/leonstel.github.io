

# Finding  X MEN Without Italian Delicacy
<img width="100%" src="../assets/finding_xmen/recipe.jpg" />

*Recipe: Spaghett*  
*Ingredients*
- *No state management*
- *No*



## GoogleMaps Wrapper
- Googlemaps instance
    - initialize in constructor
    - simple gmap manipulation (these two dont exist in current app)
        - zoom  
        - pan


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

```
//src/main.ts

loadGoogleMapScripts();
initializeRealtimePosition();
```


