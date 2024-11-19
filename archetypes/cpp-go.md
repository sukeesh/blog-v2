---
title: "Go vs C++ | Which is Faster? | Sieve of Eratosthenes"
date: 2020-05-24
author: "Sukeesh"
description: "A comparison of the Sieve of Eratosthenes implementation in Golang and C++ to see which is faster."
---

[![Golang vs C++](https://raw.githubusercontent.com/sukeesh/blog/refs/heads/gh-pages/assets/images/may2020/gocbanner.png)](https://amzn.to/2ZJNefN)

Comparing the Sieve of Eratosthenes in Golang vs C++—two of my favorite programming languages!

---

## Sieve of Eratosthenes

![Sieve of Eratosthenes](https://upload.wikimedia.org/wikipedia/commons/b/b9/Sieve_of_Eratosthenes_animation.gif)

This post doesn't aim to explain the algorithm itself. If you'd like to learn more about it, check out the [Wikipedia page](http://en.wikipedia.org/wiki/Sieve_of_Eratosthenes).

---

## Code

Here’s the implementation of the Sieve of Eratosthenes in both Golang and C++:

### [Golang Code](https://amzn.to/3cZ9sOD)

```golang
package main

import(
	"strconv"
	"os"
)

func generate_sieve(n int) {
	isPrime := make([]bool, n + 1)
	for i, _ := range (isPrime) {isPrime[i] = true}

	isPrime[0] = false
	isPrime[1] = false
	isPrime[2] = true

	i := 2

	for i <= n {
		if isPrime[i] {
			j := 2
			for i * j <= n {
				isPrime[i * j] = false
				j = j + 1
			}
		}
		i = i + 1
	}
}

func main() {
	n, _ := strconv.Atoi(os.Args[1])
	generate_sieve(n)
	return
}
```

---

### [C++ Code](https://amzn.to/2ARSPGc)

```cpp
#include <bits/stdc++.h>
#include <sstream>

using namespace std;

void generate_sieve(int n){
	bool isPrime[n + 1];
	memset(isPrime, 1, sizeof(isPrime));

	isPrime[0] = 0;
	isPrime[1] = 0;
	isPrime[2] = 1;

	for (int i = 2 ; i <= n ; i ++ ){
		if (isPrime[i]) {
			for (int j = 2 ; i * j <= n ; j ++ ) {
				isPrime[i * j] = 0;
			}
		}
	}
}


int main(int argc, char** argv){
	stringstream sn(argv[1]);
	int n;
	sn >> n;
	generate_sieve(n);
	return 0;
}
```

---

## Hardware Used

To ensure a fair comparison, I used the following hardware:

- **Device:** MacBook Pro (15-inch, 2018)
- **Processor:** 2.9 GHz 6-Core Intel Core i9
- **Memory:** 32 GB 2400 MHz DDR4

---

## Results

I measured the performance using the `time` command in a zsh shell. 

### Example usage of the `time` function:

```bash
$ time ./a.out 4194304
./a.out 4194304  0.05s user 0.00s system 95% cpu 0.057 total
```

I tested the code for generating the following ranges of prime numbers: `[2, 4, 8, 16, 32, ..., 4194304]`. Here’s how they performed:

- Until **131072**, C++ performed slightly better than Go.
- Beyond that, Go started dominating with an exponential curve.

![Performance Comparison](https://raw.githubusercontent.com/sukeesh/blog/refs/heads/gh-pages/assets/images/may2020/go_cpp.png)

---

## Final Thoughts

Both languages are great for their use cases. However, Go's performance for larger datasets was quite impressive in this particular test case.

Let me know your thoughts or share your benchmarks!
