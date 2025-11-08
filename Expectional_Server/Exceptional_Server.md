## Exceptional Server (exception handling)

In this challenge, you are required to handle error messages while working with a small computational server that performs complex calculations.

The server provides a function that takes large numbers as input and returns a numeric result. Various exceptions may occur during execution; your task is to complete the code so that it prints appropriate messages when exceptions occur.

Expected behavior

- If the compute function runs fine with the given arguments, print the result of the function call.
- If it fails to allocate the memory that it needs, print: `Not enough memory`.
- If any other standard C++ exception occurs, print: `Exception: S` where `S` is the exception's error message (the result of `what()`).
- If any non-standard exception occurs, print: `Other Exception`.

Input Format

The first line contains an integer, `T`, the number of test cases. Each of the `T` subsequent lines describes a test case as two space-separated integers `A` and `B`.

Output Format

For each test case, print a single line containing the appropriate message described above. After all test-case messages have been printed, the locked stub code prints the server load (the number of times `compute` was invoked).

Sample Input

```
2
-8 5
1435434255433 5
```

Sample Output

```
Exception: A is negative
Not enough memory
2
```

Explanation of sample

1) For `A = -8, B = 5` the `compute` function checks `A` and throws a `invalid_argument("A is negative")`. That is a standard exception, so the program prints `Exception: A is negative`.
2) For the second test case `A = 1435434255433`, trying to allocate a vector of size `A` fails and `std::bad_alloc` is thrown; the program prints `Not enough memory`.
Finally, the server load is `2` because `compute` was invoked twice (the `load` counter is incremented at the start of `compute`).

C++ solution

```cpp
#include <cmath>
#include <cstdio>
#include <vector>
#include <iostream>
#include <algorithm>
#include <exception>
#include <string>
#include <stdexcept>
using namespace std;

class Server {
private:
	static int load;
public:
	static int compute(long long a, long long b) {
		load += 1;

		if (a < 0)
			throw invalid_argument("A is negative");

		vector<int> v(a, 0);  // may cause bad_alloc for large a
		int real = -1, cmplx = sqrt(-1);
		if (b == 0)
			throw 0; // non-standard exception (integer type)
		real = (a / b) * real;
		int ans = v.at(b);  // may throw out_of_range
		return real + a - b * ans;
	}

	static int getLoad() {
		return load;
	}
};

// initialize static variable
int Server::load = 0;

int main() {
	int t;
	cin >> t;

	while (t--) {
		long long a, b;
		cin >> a >> b;

		try {
			cout << Server::compute(a, b) << endl;
		}
		catch (bad_alloc &e) {
			cout << "Not enough memory" << endl;
		}
		catch (exception &e) {
			cout << "Exception: " << e.what() << endl;
		}
		catch (...) {
			cout << "Other Exception" << endl;
		}
	}

	cout << Server::getLoad() << endl;
	return 0;
}
```

How the exceptions are handled

- `bad_alloc` is caught first because it is a specific kind of failure (memory allocation failure) that the problem requests to be reported with the message `Not enough memory`.
- `catch (exception &e)` catches other standard exceptions derived from `std::exception` such as `std::invalid_argument` and `std::out_of_range`. The program prints `Exception: ` followed by `e.what()`.
- `catch (...)` is the fallback for non-standard exceptions (for example `throw 0;` in `compute`) and prints `Other Exception`.

Why the order of catch blocks matters

Specific exceptions (like `bad_alloc`) must be caught before the more general `std::exception&` handler; otherwise the general handler would intercept them and the `Not enough memory` message would never be printed.

Worked example (step-by-step for sample input)

- Input line 1: `-8 5` -> `compute` increments load to 1, detects `a < 0` and throws `invalid_argument("A is negative")`. This is caught by the `catch (exception &e)` block and prints `Exception: A is negative`.
- Input line 2: `1435434255433 5` -> `compute` increments load to 2, attempts `vector<int> v(a,0)` which fails and throws `std::bad_alloc`; the `catch (bad_alloc &e)` block prints `Not enough memory`.
- After processing all cases, `Server::getLoad()` prints `2`.

Compile & run notes

- Compile with a C++17 (or newer) compiler: `g++ -std=c++17 -O2 -o exceptional Exceptional_Server.cpp` (if you extract the code into a .cpp file).
- Provide input either interactively or via redirection from a file containing the sample input.

Important variables (what they are and why they matter)

- `t` — The number of test cases read in `main()`. The program loops `t` times and calls `Server::compute` for each `(a,b)` pair.
- `a`, `b` — The two input values for each test case (declared `long long` to allow very large inputs). They are passed to `Server::compute(a,b)` where most checks/operations happen.
- `Server::load` — A `static int` member of `Server` that is incremented at the start of every `compute` call. This tracks how many times the server attempted a computation and is printed after all test cases.
- `vector<int> v(a, 0)` — Attempts to allocate a vector of size `a`. If `a` is very large this allocation can fail and throw `std::bad_alloc`, which the program maps to the message `Not enough memory`.
- `real` and `cmplx` — Local variables used to represent parts of the (simulated) computation. `real` is initialized to `-1`. `cmplx = sqrt(-1)` is included to hint at a complex part of the computation (in the sample code this expression is only illustrative and may produce implementation-defined results when converted to `int`). These variables are not central to exception handling but are part of the returned calculation when no exception occurs.
- `if (b == 0) throw 0;` — This throws a non-standard (non-`std::exception`) exception (an `int`). It demonstrates how `catch(...)` is used as a last-resort handler: such throws are caught by `catch (...)` and reported as `Other Exception`.
- `int ans = v.at(b);` — Accessing the vector with `.at()` performs bounds checking and will throw `std::out_of_range` if `b` is outside `[0, a-1]`. This is a standard exception and is handled by `catch (exception &e)` which prints `Exception: <message>`.

Why these variables are important for exception behavior

- Memory-related issues arise from `v(a, 0)` — large `a` leads to `bad_alloc`.
- Logic or argument errors (for example negative inputs) are signaled by throwing `std::invalid_argument` (e.g. `A is negative`) and are reported via the general `std::exception` handler.
- Out-of-range accesses use `.at()` so we get a standard `std::out_of_range` exception instead of undefined behavior; those are also caught by `catch (exception &e)` and printed with `e.what()`.
- A deliberately thrown non-standard exception (`throw 0;`) verifies that `catch(...)` is necessary to handle arbitrary throwables that aren't derived from `std::exception`.


