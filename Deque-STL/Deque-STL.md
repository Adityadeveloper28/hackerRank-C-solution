# Deque (STL) — Sliding window maximum

Double-ended queue or Deque (part of the C++ STL) is a sequence container with dynamic size that can be expanded or contracted on both ends (either its front or its back). This example uses a deque to solve the classic "maximum of all subarrays of size K" (sliding window maximum) problem in O(n) time per array.

Problem

Given a set of arrays of size N and an integer K, you have to find the maximum integer for each and every contiguous subarray of size K for each of the given arrays.

Input Format

First line contains the number of test cases T. For each test case, you'll be given two integers N and K followed by N integers — the elements of the array A.

Output Format

For each test case, print the maximum for every contiguous subarray of size K separated by spaces. Each test case's output should appear on its own line.

Constraints

Each array element fits in a signed 32-bit integer.

Approach (using deque)

Use a deque to store indices of array elements. The deque maintains indices in decreasing order of their corresponding values (front of deque has the index of the maximum in the current window). For each index i:

- Remove indices from the front that are out of the current window (i - k).
- Remove indices from the back whose corresponding values are <= arr[i], since they can't be the maximum while arr[i] is in the window.
- Push the current index to the back.
- Once we've processed at least k elements (i >= k-1), the front of the deque is the index of the maximum for the current window.

This guarantees each element is added/removed at most once, yielding O(n) time per test case.

Example

Input:

```
1
7 4
3 4 5 8 1 4 10
```

Processing windows of size 4:
- window [3,4,5,8] -> max 8
- window [4,5,8,1] -> max 8
- window [5,8,1,4] -> max 8
- window [8,1,4,10] -> max 10

Output:

```
8 8 8 10
```

C++ solution

```cpp
#include <cmath>
#include <cstdio>
#include <vector>
#include <iostream>
#include <algorithm>
#include <deque>
using namespace std;

int main() {
	/* Enter your code here. Read input from STDIN. Print output to STDOUT */  
	int t;
	cin>>t;
	while(t--){
		int n,k; // n = array size and k = window-size
		cin>>n>>k;
		vector<int> arr(n);
		for(int i=0;i<n;i++){
			cin>>arr[i];
		}
		deque<int> dq; // stores indices, front = index of current max
		vector<int> result;
        
		for(int i=0;i<n;i++){
			// 1) Remove indices which are out of this window
			while(!dq.empty() && dq.front()<=i-k){
				dq.pop_front();
			}
			// 2) Remove indices whose corresponding values are <= arr[i]
			while(!dq.empty() && arr[dq.back()]<=arr[i]){
				dq.pop_back();
			}
			// 3) Add current index
			dq.push_back(i);
			// 4) If window has k elements, record the current max
			if(i>=k-1){
				result.push_back(arr[dq.front()]);
			}
		}
        
		for(int i=0;i<result.size();i++){
			cout<<result[i];
			if(i!=result.size()-1){
				cout<<" ";
			}
		}
		cout<<endl;
	} 
    
	return 0;
}
```

Notes

- Time complexity: O(n) per test case.
- Space complexity: O(k) for the deque in the worst case.

References

The deque-based technique is a standard pattern for sliding-window min/max problems and is useful in many competitive programming tasks.

