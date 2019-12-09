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



### Do I Have Children?

This question mostly asked to yourself when the doorbell rings and a vaguely familiar woman stands in front of your 
door. It then kicks in... and you remember that night from years ago with some memory gaps. 



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

### Searching

<img src="./assets/treesearch.gif" width="150" />

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




