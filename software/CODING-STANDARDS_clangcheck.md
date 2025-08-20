# SPDX-FileCopyrightText: 2025 Pierre-Noel Bouteville <pierre-noel.bouteville@allcircuits.com>
#
# SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1

# Table of Contents

- [Table of Contents](#table-of-contents)
- [Clang Static Analyzer – Default Checkers for C Language](#clang-static-analyzer--default-checkers-for-c-language)
	- [Summary](#summary)
	- [core](#core)
		- [core.BitwiseShift](#corebitwiseshift)
		- [core.CallAndMessage](#corecallandmessage)
		- [core.DivideZero](#coredividezero)
		- [core.FixedAddressDereference](#corefixedaddressdereference)
		- [core.NonNullParamChecker](#corenonnullparamchecker)
		- [core.NullDereference](#corenulldereference)
		- [core.StackAddressEscape](#corestackaddressescape)
		- [core.UndefinedBinaryOperatorResult](#coreundefinedbinaryoperatorresult)
		- [core.VLASize](#corevlasize)
		- [core.uninitialized.ArraySubscript](#coreuninitializedarraysubscript)
		- [core.uninitialized.Assign](#coreuninitializedassign)
		- [core.uninitialized.Branch](#coreuninitializedbranch)
		- [core.uninitialized.CapturedBlockVariable](#coreuninitializedcapturedblockvariable)
		- [core.uninitialized.UndefReturn](#coreuninitializedundefreturn)
	- [optin](#optin)
		- [optin.mpi.MPI-Checker](#optinmpimpi-checker)
		- [optin.performance.Padding](#optinperformancepadding)
		- [optin.portability.UnixAPI](#optinportabilityunixapi)
	- [optin.taint](#optintaint)
		- [optin.taint.GenericTaint](#optintaintgenerictaint)
		- [optin.taint.TaintedAlloc](#optintainttaintedalloc)
	- [security](#security)
		- [security.ArrayBound](#securityarraybound)
		- [security.FloatLoopCounter](#securityfloatloopcounter)
		- [security.insecureAPI.UncheckedReturn](#securityinsecureapiuncheckedreturn)
		- [security.insecureAPI.bcmp](#securityinsecureapibcmp)
		- [security.insecureAPI.bcopy](#securityinsecureapibcopy)
		- [security.insecureAPI.bzero](#securityinsecureapibzero)
		- [security.insecureAPI.getpw](#securityinsecureapigetpw)
		- [security.insecureAPI.gets](#securityinsecureapigets)
		- [security.insecureAPI.mkstemp](#securityinsecureapimkstemp)
		- [security.insecureAPI.mktemp](#securityinsecureapimktemp)
		- [security.insecureAPI.rand](#securityinsecureapirand)
		- [security.insecureAPI.strcpy](#securityinsecureapistrcpy)
		- [security.insecureAPI.vfork](#securityinsecureapivfork)
		- [security.insecureAPI.DeprecatedOrUnsafeBufferHandling](#securityinsecureapideprecatedorunsafebufferhandling)
		- [security.MmapWriteExec](#securitymmapwriteexec)
		- [security.PointerSub](#securitypointersub)
		- [security.PutenvStackArray](#securityputenvstackarray)
		- [security.SetgidSetuidOrder](#securitysetgidsetuidorder)
	- [unix](#unix)
		- [unix.API](#unixapi)
		- [unix.Malloc](#unixmalloc)
		- [unix.MallocSizeof](#unixmallocsizeof)
		- [unix.MismatchedDeallocator](#unixmismatcheddeallocator)
		- [unix.cstring.BadSizeArg](#unixcstringbadsizearg)
		- [unix.cstring.NotNullTerminated](#unixcstringnotnullterminated)
		- [unix.cstring.NullArg](#unixcstringnullarg)
		- [unix.StdCLibraryFunctions](#unixstdclibraryfunctions)
		- [unix.Stream](#unixstream)


# Clang Static Analyzer – Default Checkers for C Language

This document summarizes the default checkers of the Clang Static Analyzer that are specifically relevant to the C language, with explanations and code examples. For more details, see the [official documentation](https://clang.llvm.org/docs/analyzer/checkers.html#default-checkers).

---

## Summary

The Clang Static Analyzer provides a set of default checkers for C code to detect common bugs, security issues, and API misuse. These checkers help catch undefined behavior, memory errors, use of dangerous functions, and more, improving code quality and safety.

---

## core

### core.BitwiseShift
**Detects undefined behavior with bitwise shift operators.**

**Explanation:** Warns when shifting by a negative value or by more than the bit width of the type.

**Example:**
```c
void example(int a, int b) {
	if (b < 0) {
		a = a << b; // warn: right operand is negative
	} else if (b >= 32) {
		a = a >> b; // warn: right shift overflows 'int'
	}
}
```

### core.CallAndMessage
**Checks for logical errors in function calls.**

**Explanation:** Detects calls to null or uninitialized function pointers.

**Example:**
```c
void test() {
	void (*foo)(void) = 0;
	foo(); // warn: function pointer is null
}
```

### core.DivideZero
**Detects division by zero.**

**Example:**
```c
void test(int z) {
	int x = 1 / z; // warn if z == 0
}
```

### core.FixedAddressDereference
**Checks for dereferencing pointers with fixed addresses.**

**Example:**
```c
void test() {
	int *p = (int *)0x020;
	int x = p[0]; // warn
}
```

### core.NonNullParamChecker
**Checks for null pointers passed to non-null arguments.**

**Example:**
```c
int f(int *p) __attribute__((nonnull));
void test(int *p) {
	if (!p)
		f(p); // warn
}
```

### core.NullDereference
**Detects dereferencing of null pointers.**

**Example:**
```c
void test(int *p) {
	if (!p)
		*p = 0; // warn
}
```

### core.StackAddressEscape
**Checks that stack addresses do not escape the function.**

**Example:**
```c
char const *p;
void test() {
	char const str[] = "string";
	p = str; // warn
}
```

### core.UndefinedBinaryOperatorResult
**Detects undefined results of binary operators.**

**Example:**
```c
void test() {
	int x;
	int y = x + 1; // warn: x is uninitialized
}
```

### core.VLASize
**Checks for invalid Variable Length Array (VLA) sizes.**

**Example:**
```c
void test() {
	int x;
	int vla[x]; // warn: x is uninitialized
}
```

### core.uninitialized.ArraySubscript
**Detects use of uninitialized values as array subscripts.**

**Example:**
```c
void test() {
	int i, a[10];
	int x = a[i]; // warn: i is uninitialized
}
```

### core.uninitialized.Assign
**Detects assignments from uninitialized values.**

**Example:**
```c
void test() {
	int x;
	x |= 1; // warn: x is uninitialized
}
```

### core.uninitialized.Branch
**Detects use of uninitialized values in branch conditions.**

**Example:**
```c
void test() {
	int x;
	if (x) // warn: x is uninitialized
		return;
}
```

### core.uninitialized.CapturedBlockVariable
**Detects blocks that capture uninitialized values.**

**Example:**
```c
void test() {
	int x;
	^{ int y = x; }(); // warn: x is uninitialized
}
```

### core.uninitialized.UndefReturn
**Detects returning uninitialized values from functions.**

**Example:**
```c
int test() {
	int x;
	return x; // warn: x is uninitialized
}
```

---

## optin

### optin.mpi.MPI-Checker
**Checks for correct usage of non-blocking MPI calls.**

**Example:**
```c
void test() {
	double buf = 0;
	MPI_Request sendReq1;
	MPI_Ireduce(MPI_IN_PLACE, &buf, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD, &sendReq1);
} // warn: request 'sendReq1' has no matching wait.
```

### optin.performance.Padding
**Detects structs with excessive padding.**

**Explanation:** Warns if struct field order causes wasted memory.

**Example:**
```c
struct NonOptimal {
	char c1;
	long big1;
	char c2;
	long big2;
}; // warn: excessive padding
```

### optin.portability.UnixAPI
**Warns when 0 is passed as the size argument to allocation functions.**

**Example:**
```c
void test() {
	void *p = malloc(0); // warn: allocation size of 0 bytes
}
```

---

## optin.taint

### optin.taint.GenericTaint
**Performs taint analysis for untrusted data.**

**Explanation:** Tracks data from untrusted sources to sensitive operations (e.g., command injection).

**Example:**
```c
int main() {
	char cmd[2048] = "/bin/cat ";
	char filename[1024];
	scanf("%1023s", filename); // attacker can inject here
	strcat(cmd, filename);
	system(cmd); // warn: untrusted data to system()
}
```

### optin.taint.TaintedAlloc
**Warns if allocation size comes from an untrusted source.**

**Example:**
```c
void vulnerable(void) {
	size_t size = 0;
	scanf("%zu", &size);
	int *p = malloc(size); // warn
	free(p);
}
```

---

## security

### security.ArrayBound
**Detects out-of-bounds accesses to arrays and memory.**

**Example:**
```c
void test(int x) {
	int buf[100];
	buf[x] = 1; // warn if x < 0 or x >= 100
}
```

### security.FloatLoopCounter
**Warns on using floating-point values as loop counters.**

**Example:**
```c
void test() {
	for (float x = 0.1f; x <= 1.0f; x += 0.1f) {} // warn
}
```

### security.insecureAPI.UncheckedReturn
**Warns when return value of critical functions is ignored.**

**Example:**
```c
void test() {
	setuid(1); // warn: return value not checked
}
```

### security.insecureAPI.bcmp
**Warns on uses of the `bcmp` function.**

**Example:**
```c
void test() {
	bcmp(ptr0, ptr1, n); // warn
}
```

### security.insecureAPI.bcopy
**Warns on uses of the `bcopy` function.**

**Example:**
```c
void test() {
	bcopy(src, dst, n); // warn
}
```

### security.insecureAPI.bzero
**Warns on uses of the `bzero` function.**

**Example:**
```c
void test() {
	bzero(ptr, n); // warn
}
```

### security.insecureAPI.getpw
**Warns on uses of the `getpw` function.**

**Example:**
```c
void test() {
	char buff[1024];
	getpw(2, buff); // warn
}
```

### security.insecureAPI.gets
**Warns on uses of the `gets` function.**

**Example:**
```c
void test() {
	char buff[1024];
	gets(buff); // warn
}
```

### security.insecureAPI.mkstemp
**Warns if `mkstemp` is called with fewer than 6 X's.**

**Example:**
```c
void test() {
	mkstemp("XX"); // warn
}
```

### security.insecureAPI.mktemp
**Warns on uses of the `mktemp` function.**

**Example:**
```c
void test() {
	char *x = mktemp("/tmp/zxcv"); // warn
}
```

### security.insecureAPI.rand
**Warns on uses of inferior random number functions.**

**Example:**
```c
void test() {
	random(); // warn
}
```

### security.insecureAPI.strcpy
**Warns on uses of `strcpy` and `strcat`.**

**Example:**
```c
void test() {
	char x[4];
	char *y = "abcd";
	strcpy(x, y); // warn
}
```

### security.insecureAPI.vfork
**Warns on uses of the `vfork` function.**

**Example:**
```c
void test() {
	vfork(); // warn
}
```

### security.insecureAPI.DeprecatedOrUnsafeBufferHandling
**Warns on unsafe or deprecated buffer handling functions.**

**Example:**
```c
void test() {
	char buf[5];
	strncpy(buf, "a", 1); // warn
}
```

### security.MmapWriteExec
**Warns on calls to `mmap()` with write and execute permissions.**

**Example:**
```c
void test() {
	void *c = mmap(NULL, 32, PROT_READ | PROT_WRITE | PROT_EXEC, MAP_PRIVATE | MAP_ANON, -1, 0); // warn
}
```

### security.PointerSub
**Detects pointer subtractions between different memory objects.**

**Example:**
```c
void test() {
	int a, b, c[10], d[10];
	int x = &c[3] - &c[1];
	x = &d[4] - &c[1]; // warn
}
```

### security.PutenvStackArray
**Detects calls to `putenv()` with stack-allocated arrays.**

**Example:**
```c
int f() {
	char env[] = "NAME=value";
	return putenv(env); // warn
}
```

### security.SetgidSetuidOrder
**Checks for correct order of `setgid` and `setuid` calls.**

**Example:**
```c
void test1() {
	if (setuid(getuid()) != 0) {
		handle_error();
		return;
	}
	if (setgid(getgid()) != 0) { // warn
		handle_error();
		return;
	}
}
```

---

## unix

### unix.API
**Checks calls to UNIX/POSIX functions for correct usage.**

**Example:**
```c
void test(const char *path) {
	int fd = open(path, O_CREAT); // warn: missing third argument
}
```

### unix.Malloc
**Detects memory leaks, double free, and use-after-free.**

**Example:**
```c
void test() {
	int *p = malloc(1);
	free(p);
	free(p); // warn: double free
}
```

### unix.MallocSizeof
**Detects dubious `malloc` arguments involving `sizeof`.**

**Example:**
```c
void test() {
	long *p = malloc(sizeof(short)); // warn
	free(p);
}
```

### unix.MismatchedDeallocator
**Detects mismatched deallocation (e.g., using `delete` on `malloc`).**

**Example:**
```c
void test() {
	int *p = (int *)malloc(sizeof(int));
	delete p; // warn
}
```

### unix.cstring.BadSizeArg
**Checks the size argument passed to C string functions.**

**Example:**
```c
void test() {
	char dest[3];
	strncat(dest, "*", sizeof(dest)); // warn
}
```

### unix.cstring.NotNullTerminated
**Checks that arguments to string functions are null-terminated.**

**Example:**
```c
void test1() {
	int l = strlen((char *)&test1); // warn
}
```

### unix.cstring.NullArg
**Checks for null pointers passed to C string functions.**

**Example:**
```c
int test() {
	return strlen(0); // warn
}
```

### unix.StdCLibraryFunctions
**Checks calls to standard C library functions for argument violations.**

**Example:**
```c
void test_alnum_concrete(int v) {
	int ret = isalnum(256); // warn: argument outside allowed range
}
```

### unix.Stream
**Checks for correct handling of C streams.**

**Example:**
```c
void test1() {
	FILE *p = fopen("foo", "r");
} // warn: opened file is never closed
```
