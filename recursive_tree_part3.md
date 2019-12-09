## Recursion Tree Fairy Tail (Part3)


<img src="./assets/bedtimestory.jpg" />

### Once Upon a Time
... a boy was born within a lovely family. Within the boy an evil entity began to grow slowly but steadily.
Not very much later this evil entity overpowered him and his craving for more got out of control. Without any wealth, his 
greed told him to make cheap ass paid courses about getting rich quick. The boy couldn't be happier, it worked and students were
buying his courses like ice scream. Every student whom followed the boys' courses had been contaminated with the ever growing evil
and were already planning to make a course about getting rich quick to satisfy their hunger. They won't live happily ever after.

Although this fairy tail isn't officially a [greedy algorithm](https://en.wikipedia.org/wiki/Greedy_algorithm) it sure sounds like one. 

### Recursion Gone Wrong
Having no correct base case in recursion results in a catastrophe. Consider this old-fashioned fairy tail.

Unfortunately the universe does not have a built in maximum call stack exceed error. 
This quackery will only stop if the human race has been destroyed or their destructive swathes have consumed the
earth. 

#Searching

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




