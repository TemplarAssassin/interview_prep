```java
public List<Integer> printNums(int n) {
  // output 2 ^ n nums;
  // e.g. 00 -> 01 -> 11 -> 10 
  
  
  // 000 -> 001 -> 011 -> 010
  // 110 -> 111 -> 101 -> 100
  List<Integer> res = new ArrayList<>();
  int cur = 0;
  res.add(cur);
  
  
  for (int i = 0; i < n ; i++) {
    res.add(cur ^ (i << 1));
  }
  
}

public void dfs() {
  for (int i = 0; i < n; i++)
}

// dfs 
```

