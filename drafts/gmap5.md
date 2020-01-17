

# Finding  X MEN Gathering

### Api data


### initializing
```
// src/main.ts

const apiMutants = require('./mutants.json');

loadGoogleMapScripts();
initializeRealtimePosition();

Store.set('professorX', {
    ...apiMutants.xmen.professorX,
    radius: 1500
});
Store.set('apiMutants', apiMutants);

new XMenMap();
```

- store

- xmap extends mapbase

Professor x


### UI
```
new UI();
```

    

