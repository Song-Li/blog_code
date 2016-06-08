title: "Quicksort"
date: 2015-10-17 14:16:04
tags: Algorithm
---
I finished merge sort, heap sort. If I leave quicksort along, he must be very upset.
So... Quicksort, here we go.
Quicksort, not only quick, but also ... quick.
The first quick means it runs quickly, the second one is it writes short and quick.
Here is my code
```
void quicksort(int *line, int from, int to){
    if(from == to) return ;
    int pivot = line[to - 1],now = from;
    for(int i = from;i < to - 1;++ i)
        if(line[i] < pivot) swap(line[i],line[now ++]);
    swap(line[now],line[to - 1]);
    quicksort(line, from, now);
    quicksort(line, now + 1, to);
}

```
