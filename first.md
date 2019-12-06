## Recursion tree has no incentive to leaf

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

The leaf node is the important part where all the magic happens. Whereas the tree element is just a handy container for the root leaf
where it all begins. For example you could put methods in the tree element for manipulating the whole tree, this comes in handy later on
when doing the searching. So the tree element has one leaf (the root) contains child leafs. The root leaf from won't
be visible at all, has only children and is a virtual leaf as a mounting point for the first level of leafs.

As you can observe from the above diagram it will be a play where recursion and composition will be in the spot light together.
The composition part comes to light where the leaf has a reference to its parent and children therefor a parent is able to do something 
with children. For the record this is not an advocation for pedopilia.  

These tree classes originated from shown diagram are the bare skeleton the data binding comes up next.
```
//app.js
// this is the main App class where the app starts from
class MyApp extends LitElement {
    render() {
        return html`
            <custom-tree></custom-tree>
        `;
    }
}

```
```
//tree.js
// this class gets registered to <custom-tree></custom-tree>
export class Tree extends LitElement{

    render() {
        return html`
            <custom-leaf></custom-leaf>
        `
    }
}
```
```
//leaf.js
// this class gets registered to <custom-leaf></custom-leaf>
export class Leaf extends LitElement{

    render() {
        return html`
            ${this ? html`
                ${
                    this.leaf.name ? html`
                        <div class="node">                            
                            <div class="item">
                                <p>${this.leaf.name}</p>
                            </div>
                        </div>
                    `: ''
                }
                <div>
                    ${this.hasChildren() ? this.renderChildren() : ''}
                </div>
            `: ''}
        `
    }

    // loops through child leaf references and outputs the html for each element
    // if it has no children no html will be produced
    renderChildren(){
        return html`
            <ul>
                ${this.childrenRef.map(leaf => html`
                    ${leaf}
                `)}
            </ul>
        `
    }

    hasChildren(){
        return !!this.childrenRef.length;
    }
}


```

### Lit & Data Binding
To go any further some explaining on the Lit and data binding front is essential to get the gist of my story. 

```
//within a Class that extends LitElement

// define the properties of this class which Lit Element has to monitor for changes
static get properties() {
    return {
        leaf: { type: Object },
    };
}

// The Render function renders the html with a Lit HTML helper.
// Everytime the this.leaf prop changes then the html of that part will be rerendered
// with the new values (the lit html helper takes care of all that stuff)
render() {
    return html`
         <div>
             ${this.leaf.name}
         </div>
    `
}

// this updated function is being called everytime 
// one of the props of the properties() function has changed
updated(_changedProperties) {
    if(_changedProperties.has('leaf')){
        //do something when the class' leaf property has changed 
    }
}
```

For example the MyApp needs a data property so we define a data property in the properties() function.
Everytime the this.data gets changed all the html element that are bound to this prop will get notified with
the new data. In our case the custom-tree element has the data prop van MyApp bound to its own data prop.

**Recap**: MyApp changes this.data -> the &lt;custom-tree&gt; is bound and gets notified -> the updated() function of
&lt;custom-tree&gt; is being called -> you could react on this change


Everytime -> britney spears youtube.

```
const apiData = require('../data.json');

class MyApp extends LitElement {

    static get properties() {
        return {
            data: { type: Object },
        };
    }

    constructor(){
        super();
        this.data = prepData(apiData);
    }
    render() {
        return html`
            <custom-tree .data="${this.data}"></custom-tree>
        `;
    }
}
```

Each leaf represents a category ... yeah I know it's unoriginal and boring (probably the oxygen deprivation talking). But it is able to illustrate
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