## Recursion Tree Hide and Seek (Part2)

| Before  | After |
| ------------- | ------------- | ------------- | ------------- |
| <img src="./assets/tree_simple_part1.gif" width="150" /> | <img src="./assets/treeviewcase3.gif" width="150" />|

### Hide and Seek
To recall, our data has been prepped so that every leaf has got some extra flags. Every leaf 
can be sure to receive those flags with its detaults on start up because of the prepping. Yeah it would be possible to leaf the hide
prop null without prepping that part, but that does not line up with my data consistency rule.

Hiding a branch means collapsing its child leafs when clicking on the arrow like shown in the intro image.

```
// This will be the leaf object after prepping the raw data.json
// Type Leaf
{
  name: String                  // category name
  children: Leaf[],             // the child leaf of this leaf
  hide: Boolean,                // flag used for collapsing the branches visually (like the first simple tree gif shows)
  included: Boolean,            // flag used for showing only the branches that contain the searh term 
}


//leaf.js

// add hide prop to prop array because we want to bind it with html element
static get properties() {
    return {
        ....
        hide: {type:Boolean}
    };
}
```

Html conditional

The root node could be detected it does not have any parentRef. Else ouput empty.
But keep in mind that the root node still has to render it children.

```
render() {
    return html`
        ${
        // When the leaf is the root node don't render anything
        this.parentRef  ? html`
            //leaf render here
        ` : '';
}
```

Arrow. Are svgs and modified with class to show right and up arrows

```
${  
    this.hasChildren() ? html`
        <div class="arrow">
            <img class="${!this.hide ? 'down' : ''}" @click="${this.fold}" src="./src/assets/arrow.svg" />
        </div>       
    ` : html``
}

fold(){
    this.hide = !this.hide;
}
```

With the hide flag included in the incoming data it is now possible the hide the leafs from external code.

Status
```
const STATUS = {
    NONE: 1,
    CHECKED: 2,
    INDETERMINATE: 3,
};
```

I will handle three cases of selection.

As mentioned before every subcategory must be a subset of its parent.

| Case1  | Case2 | Case3 |
| ------------- | ------------- | ------------- | ------------- |
| <img src="./assets/treviewcase1.gif" width="150" />  | <img src="./assets/treeviewcase2.gif" width="150" /> | <img src="./assets/treeviewcase3.gif" width="150" />


