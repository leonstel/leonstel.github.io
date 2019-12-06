## Rescursive search tree

How to code a searchable code tree? After hours, maybe weeks, of meditation I came to the conclusion that my next goal has been 
in front of my very eyes... RECURSION. Recursion is the key 
for creating managable tree structures. 
For sure you could harcode every branch and leaf but that gets unmaintainable very quickly.
From my experience I believe that most developers evade the recursion topic, but it could
be a helpful tool for many cases not just for trees. 

<img src="./assets/treeview.gif" width="150" />

Lets get started on building a simple and intuitive tree . I will use Lit Element but the concept is
applicable to infinity... and beyond. Lit is an light weight wrapper that relies heavily on web components and has
handy methods for data binding and property change callbacks. And the best part is that it does not get in the 
way of writing vanilla javascript with fancy framework stuff.

Here is the diagram that came to me on an adventurous bicycle ride at the mount everest without oxygen equipment
(they say that oxygen deprivation causes great inspirational sparks). So I stepped of my bike and got my laptop out to draw
it.

<img src="./assets/simple_tree_diagram.jpeg" width="650"/>

Recursion and compositions combinations
Comes into play

Each node represents a category ... yeah I know it's unoriginal and boring (probably the oxygen deprivation talking). But it is able to illustrate
my point very effictively so suck it up ;) 


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
//tree.js
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

So the conclusion is. If you want to implement recursion just step on your bike and any 
from of oxygen deprivation causes you to make great recursive solutions.

For a recursion tree you have to go to the intratuin now, they are in the discount