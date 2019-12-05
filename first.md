## Rescursive search tree

<img src="./assets/treeview.gif" width="150" />



### Data Preparation

The data.json in the git repo contains the full data, but for the sake of clarity only 
a minimal sample has been displayed for demonstration purpose.

```
//data.json
//First level of branch to demonstrate the structure
{
  "name": "root",
  "children": [
    {
      "name": "Education",
      "children": [
        {
          "name":"School",
        }
      ]
    },
  ]
}
```

After data prep in utils
Consistency, one source of truth. Prevent nullpointers. Your nodes have to known what
data it can expect. With javascript you lose control easily, when you keep adding
properties and properties on run time

```
// Type NODE
{
  name: String
  children: Node[],
  hide: Boolean,
  included: Boolean,
}
```

Utils data prep function
```
//utils.js
export const prepData = (data) => {
    if(!data) {
        console.warn('the input data is undefined so nothing to prep');
        return createNode(data,true,false);
    }

    if(Array.isArray(data)) throw Error('Could not prep, input must be object');
    if(data.children && !Array.isArray(data.children)) throw Error('if children prop exist it must be an array');

    let preppedData = createNode(data,false,true);

    if(data.children){
        let children = [];
        for(let child of data.children){
            children.push(prepData(child))
        }
        preppedData.children = children;
    }
    return preppedData;
};

```




### Node Selection

States
```
const STATUS = {
    NONE: 1,
    CHECKED: 2,
    INDETERMINATE: 3,
};
```

| Case1  | Case2 | Case3 |
| ------------- | ------------- | ------------- | ------------- |
| <img src="./assets/treviewcase1.gif" width="150" />  | <img src="./assets/treeviewcase2.gif" width="150" /> | <img src="./assets/treeviewcase3.gif" width="150" />

### Search
<img src="./assets/treesearch.gif" width="150" /> 

Recursive search function

```
searchRecursive(children, s) {
    let found = false;
    if (children) {
        for (let child of children) {
            const searchFound = this.searchRecursive(child.children, s) || child.name.toLowerCase().includes(s.toLowerCase());
            child.included = searchFound;
            if (searchFound) {
                found = true;
            }
        }
    }
    return found;
}
```