---
layout: post
title: "Simple script to hack during contests"
author: "Sukeesh"
date: 2015-12-18
---

This is a simple shell script to hack during codeforces/topcoder rounds. `your_code.cpp` should be your code, `victim_code.cpp` should be victim's code. Write test case generator in `test.py` and run this bash file by `bash hack.sh`. This script keeps on iterating until it finds a test case where your_code.cpp and victim_code.cpp gives a different output. Once it finds a hack test case, this stops iterating. Now, you can copy the test case and hack.

### Happy Hacking!


```bash
while true
do
python test.py > in
g++ your_code.cpp
./a.out <in> sukeesh
g++ victim_code.cpp
./a.out <in> victim
diff --brief <(sort victim) <(sort sukeesh) >/dev/null
comp_value=$?
if [ $comp_value -eq 1 ]
then
	echo " "
	echo "--------- HACK! --------"
	echo " Input : "
	cat ./in
	echo " Expected : "
	cat ./sukeesh
	printf "\n"
	echo " Recieved : "
	cat ./victim
	printf "\n"
	break
else
	echo " "
	echo "----------Same----------"
	echo " Input : "
	cat ./in
	echo " Expected : "
	cat ./sukeesh
	printf "\n"
	echo " Recieved : "
	cat ./victim
	printf "\n"
fi
done

```

This is the script