## Recursion Tree Fairy Tail (Part3)


<img src="./assets/bedtimestory.jpg" />

### Once Upon a Time
... a boy was born within a lovely family. Within the boy an evil entity began to grow slowly but steadily.
Not very much later this evil entity overpowered him and his craving for more got out of control. Without any wealth, his 
greed told him to make cheap ass paid courses about getting rich quick. The boy couldn't be happier, it worked and students were
buying his courses like ice scream. Every student whom followed the boys' courses had been contaminated with the ever growing evil
and were already planning to make a course about getting rich quick to satisfy their hunger. They won't live happily ever after.

Although this fairy tail isn't officially a [greedy algorithm](https://en.wikipedia.org/wiki/Greedy_algorithm) it sure sounds like one. 

### Base Case
As above tail makes clear, having no base case within a recursion loop will result in an infinity amount of callbacks 
because it keeps going forever. Eventually it will throw a maximum call stack exceeded error. A base case is a condition
within the recursive function that returns some kind of value (could be an empty return as well) to stop the feedback loop. 
Although morbidly, the universe has no built-in base case so the fairy tail goes on and on till the human race goes 
extinct ...happy hollidays!

##### Repo
The git repo contains the runnable code
[Git Repo recursive part3 branch](https://github.com/leonstel/techblog_recursive_tree/tree/part3)

### Do I Have Children?

This question mostly asked to yourself when the doorbell rings and a vaguely familiar woman stands in front of your 
door. It then kicks in... that night from years ago that has some memory gaps. 

Spoiler, no it's not like that just talking about the children of the recursive tree.

In previous article (part 2) we have covered some selection cases. The only one left involves the indeterminate state
of the leaf if it is in hiding and some of its children are selected. How does the leaf know if any childs are checked?

<img src="./assets/treeviewcase3.gif" width="150" />

For that I've come up with the recursive ```hasCHeckedInBranch()``` method that returns ```true``` if it has any checked childs 
and ```false``` if none of them are selected. Although it seems like a simple and brief function getting it right was
driving me up the wall at some moments.

```
//leaf.js
hasCheckedInBranch() {
    for (let childRef of this.childrenRef){
        const flag = childRef.isChecked() || childRef.hasCheckedInBranch();
        if(flag) return true;
    }
    return false;
}   
```

The image shows the dissected process of the taken steps to get the answer from the recursive chain.

<img src="./assets/tree_has_checked_children.jpeg" width="800" />

### Searching

Let's do some searching on the tree!

<img src="./assets/treesearch.gif" width="150" />

##### What Does Searching Mean?
```
//Whole example tree
- leaf1
    - leaf2
        - tobefound
            - leaf5
        - leaf4
    - leaf3

// if you search this tree with term "found" the only included leafs will be
- leaf1
    - leaf2
        - tobefound
```

I hope that the earlier function is not lost on you and that the image clarifies a lot.
Thank god that the recursive ```searchRecursive()``` is practically the same mechanic as the ```hasCheckedInBranch()```
so you have killed two birds with one stone. Though that being said, there is a big difference between the implementations.

The ```hasCheckedInBranch()``` operates on the leaf component itself whereas to ```searchRecursive()``` changes the
entire input data from within the ```Tree```. Remember that the tree component has the entire tree data bound and the child leafs are being drawn
from that data. The search function changes te input data which then triggers the tree to rerender its leafs with that 
newly set data. 

Again I am going to show you the prepped data from the [part 1 article](http://leonstel.github.io/recursive_tree_part1). 
Back then I have put the ```included``` prop in the data set which is just what we need to get the search working. It
is a flag to let the leaf know if it is allowed to render itself. 
```
// This will be the leaf objects within the data tree after prepping the raw data.json
// Type Leaf
{
  name: String                  // category name
  children: Leaf[],             // the child leaf of this leaf
  hide: Boolean,                // flag used for collapsing the branches visually (like the first simple tree gif shows)
  included: Boolean,            // flag used for showing only the branches that contain the searh term 
}

//leaf.js
render() {
    return html`

        // the node is only allowed to render if the included prop === TRUE
        ${this.node && this.node.included ? html`
            //... render node 
        `: ''
    `
}
```

Finally the search function within the ```Tree``` component, recall that it is the same boolean propagation as 
```hasCheckedInBranch()```. The only addition is that the included prop is being set as well. It checks for each leaf 
if its name contains the search term with ```child.name.toLowerCase().includes(s.toLowerCase()```, case insensitive.
When it is found it traverses up a `TRUE` bool.  
```
//tree.js
// s param is the search term as a String
searchRecursive(children, s) {
    let found = false;
    if (children) {
        for (let child of children) {
            const searchFound = this.searchRecursive(child.children, s) || child.name.toLowerCase().includes(s.toLowerCase());
            child.included = searchFound;   //<------ the included prop is set, 
                                            //         if false then the child is not drawn
            if (searchFound) {
                found = true;
            }
        }
    }
    return found;
}
```

In the end it all comes to together by filling up the ```search()``` within the Tree component. This search function 
could be called from any code that has access to this tree component.  

```
//tree.js
// the search param is the string to be search
search(search){
    // It creates a copy of the existing data
    const copyChildren = [...this.data.children];

    // Mutate the copied data by calling searchRecursive.
    // This will reavaluate the included prop of each child of the copied set 
    // according to what the search term is.
    this.searchRecursive(copyChildren, search);

    // Reset the data by assigning the copied set to the data prop.
    // This triggers the leafs, that are bound to this.data, to be redrawn.
    // The root node must be included at all times therefor this hardcoded true value.
    this.data = {children:copyChildren, included: true}; 
}
```

### Afterthought
Why use two properties in the data for hiding the leafs, ```hide``` & ```included```, don't they do kind of the same 
thing? Yeah I understand the confusion let me explain. Visually they indeed do the same thing, but data wise they can't
be mixed up as they serve two totally different purposes. Imagine if they would do the same thing with only one 
property and you have searched for an item and then hid one of the search result leafs by clicking the arrow. If you then
took the data you could never be certain that the adjusted data contains all search results. But with two properties 
you could always identify from the data which leafs are search result (the ones with ```included```) no matter which 
leafs you have hidden manually (because that correlates with ```hide```).  

### Fitting the Pieces

To wrap up, the tree is now a full grown one in one of the greatest jungles. Whereas traversing and searching trees, if done 
right, is accompanied with a healthy dose of recursion. The most important thing to grasp of this article is the boolean 
propagation principle so that a leaf is able to communicate efficiently with its children.

The purpose of this series is to level up your recursion skills since it is a valuable topic and tool for writing and 
understanding code better I believe. You must have traveled long distances to get at this point, I appreciate it that 
you took the time to give my read a shot! Furthermore the most important thing, I hope you have learned something. 

The git repo contains the runnable code
[Git Repo recursive part3 branch](https://github.com/leonstel/techblog_recursive_tree/tree/part3)

[Recursion Tree Has No Incentive to Leaf (Part1)](http://leonstel.github.io/recursive_tree_part1)  
[Recursion Tree Hide and Seek (Part2)](http://leonstel.github.io/recursive_tree_part2)  
[Recursion Tree Fairy Tail (Part3)](http://leonstel.github.io/recursive_tree_part3)

[Home](http://leonstel.github.io/)

<img src="./assets/jungle.jpg" />


