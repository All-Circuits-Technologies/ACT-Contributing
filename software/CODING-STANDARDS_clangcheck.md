# SPDX-FileCopyrightText: 2025 Pierre-Noel Bouteville <pierre-noel.bouteville@allcircuits.com>
#
# SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1

# 1. Available Checkers

The analyzer performs checks that are categorized into families or “checkers”.

The default set of checkers covers a variety of checks targeted at finding security and API usage bugs,
dead code, and other logic errors.

## 1.1. Default Checkers
-----------------------------------------------------------------------------

### 1.1.1. core

Models core language features and contains general-purpose checkers such as division by zero,
null pointer dereference, usage of uninitialized values, etc.*These checkers must be always switched on as other checker rely on them.*

#### 1.1.1.1. core.BitwiseShift

Finds undefined behavior caused by the bitwise left- and right-shift operator
operating on integer types.

By default, this checker only reports situations when the right operand is
either negative or larger than the bit width of the type of the left operand;
these are logically unsound.

Moreover, if the pedantic mode is activated by`-analyzer-config core.BitwiseShift:Pedantic=true`, then this checker also
reports situations where the _left_ operand of a shift operator is negative or
overflow occurs during the right shift of a signed value. (Most compilers
handle these predictably, but the C standard and the C++ standards before C++20
say that they’re undefined behavior. In the C++20 standard these constructs are
well-defined, so activating pedantic mode in C++20 has no effect.)

**Examples**

```c
static_assert(sizeof(int) == 4, "assuming 32-bit int")

void basic_examples(int a, int b) {
  if (b < 0) {
    b = a << b; // warn: right operand is negative in left shift
  } else if (b >= 32) {
    b = a >> b; // warn: right shift overflows the capacity of 'int'
  }
}

int pedantic_examples(int a, int b) {
  if (a < 0) {
    return a >> b; // warn: left operand is negative in right shift
  }
  a = 1000u << 31; // OK, overflow of unsigned value is well-defined, a == 0
  if (b > 10) {
    a = b << 31; // this is undefined before C++20, but the checker doesn't
                 // warn because it doesn't know the exact value of b
  }
  return 1000 << 31; // warn: this overflows the capacity of 'int'
}

```
**Solution**

Ensure the shift operands are in proper range before shifting.

#### 1.1.1.2. core.CallAndMessage

> Check for logical errors for function calls and Objective-C message expressions (e.g., uninitialized arguments, null function pointers).

```c
void test() {
   void (*foo)(void);
   foo = 0;
   foo(); // warn: function pointer is null
 }
```

#### 1.1.1.3. core.DivideZero

> Check for division by zero.

```c
void test(int z) {
  if (z == 0)
    int x = 1 / z; // warn
}

void test() {
  int x = 1;
  int y = x % 0; // warn
}

```

#### 1.1.1.4. core.FixedAddressDereference

Check for dereferences of fixed addresses.

A pointer contains a fixed address if it was set to a hard-coded value or it
becomes otherwise obvious that at that point it can have only a single fixed
numerical value.

```c
void test1() {
  int *p = (int *)0x020;
  int x = p[0]; // warn
}

void test2(int *p) {
  if (p == (int *)-1)
    *p = 0; // warn
}

void test3() {
  int (*p_function)(char, char);
  p_function = (int (*)(char, char))0x04080;
  int x = (*p_function)('x', 'y'); // NO warning yet at functon pointer calls
}

```
If the analyzer option `suppress-dereferences-from-any-address-space` is set
to true (the default value), then this checker never reports dereference of
pointers with a specified address space. If the option is set to false, then
reports from the specific x86 address spaces 256, 257 and 258 are still
suppressed, but fixed address dereferences from other address spaces are
reported.

#### 1.1.1.5. core.NonNullParamChecker

Check for null pointers passed as arguments to a function whose arguments are references or marked with the ‘nonnull’ attribute.

```c
int f(int *p) __attribute__((nonnull));

void test(int *p) {
  if (!p)
    f(p); // warn
}

```

#### 1.1.1.6. core.NullDereference

Check for dereferences of null pointers.

```c
void test(int *p) {
  if (p)
    return;

  int x = p[0]; // warn
}

void test(int *p) {
  if (!p)
    *p = 0; // warn
}

```
Null pointer dereferences of pointers with address spaces are not always defined
as error. Specifically on x86/x86-64 target if the pointer address space is
256 (x86 GS Segment), 257 (x86 FS Segment), or 258 (x86 SS Segment), a null
dereference is not defined as error. See [X86/X86-64 Language Extensions](https://clang.llvm.org/docs/LanguageExtensions.html#memory-references-to-specified-segments)for reference.

If the analyzer option `suppress-dereferences-from-any-address-space` is set
to true (the default value), then this checker never reports dereference of
pointers with a specified address space. If the option is set to false, then
reports from the specific x86 address spaces 256, 257 and 258 are still
suppressed, but null dereferences from other address spaces are reported.

#### 1.1.1.7. core.StackAddressEscape

Check that addresses to stack memory do not escape the function.

```c
char const *p;

void test() {
  char const str[] = "string";
  p = str; // warn
}

void* test() {
   return __builtin_alloca(12); // warn
}

void test() {
  static int *x;
  int y;
  x = &y; // warn
}

```

#### 1.1.1.8. core.UndefinedBinaryOperatorResult

Check for undefined results of binary operators.

```c
void test() {
  int x;
  int y = x + 1; // warn: left operand is garbage
}

```

#### 1.1.1.9. core.VLASize

Check for declarations of Variable Length Arrays (VLA) of undefined, zero or negative
size.

```c
void test() {
  int x;
  int vla1[x]; // warn: garbage as size
}

void test() {
  int x = 0;
  int vla2[x]; // warn: zero size
}

```
The checker also gives warning if the *TaintPropagation* checker is switched on
and an unbound, attacker controlled (tainted) value is used to define
the size of the VLA.

```c
void taintedVLA(void) {
  int x;
  scanf("%d", &x);
  int vla[x]; // Declared variable-length array (VLA) has tainted (attacker controlled) size, that can be 0 or negative
}

void taintedVerfieidVLA(void) {
  int x;
  scanf("%d", &x);
  if (x<1)
    return;
  int vla[x]; // no-warning. The analyzer can prove that x must be positive.
}

```

#### 1.1.1.10. core.uninitialized.ArraySubscript

Check for uninitialized values used as array subscripts.

```c
void test() {
  int i, a[10];
  int x = a[i]; // warn: array subscript is undefined
}

```

#### 1.1.1.11. core.uninitialized.Assign

Check for assigning uninitialized values.

```c
void test() {
  int x;
  x |= 1; // warn: left expression is uninitialized
}

```

#### 1.1.1.12. core.uninitialized.Branch

Check for uninitialized values used as branch conditions.

```c
void test() {
  int x;
  if (x) // warn
    return;
}

```

#### 1.1.1.13. core.uninitialized.CapturedBlockVariable

Check for blocks that capture uninitialized values.

```c
void test() {
  int x;
  ^{ int y = x; }(); // warn
}

```

#### 1.1.1.14. core.uninitialized.UndefReturn

Check for uninitialized values being returned to the caller.

```c
int test() {
  int x;
  return x; // warn
}

```

### 1.1.3. deadcode

Dead Code Checkers.

#### 1.1.3.1. deadcode.DeadStores

Check for values stored to variables that are never read afterwards.

```c
void test() {
  int x;
  x = 1; // warn
}

```
The `WarnForDeadNestedAssignments` option enables the checker to emit
warnings for nested dead assignments. You can disable with the`-analyzer-config deadcode.DeadStores:WarnForDeadNestedAssignments=false`.*Defaults to true*.

Would warn for this e.g.:
if ((y = make_int())) {
}

### 1.1.4. nullability

Checkers (mostly Objective C) that warn for null pointer passing and dereferencing errors.

#### 1.1.4.2. nullability.NullReturnedFromNonnull

Warns when a null pointer is returned from a function that has _Nonnull return type.

```c
- (nonnull id)firstChild {
  id result = nil;
  if ([_children count] > 0)
    result = _children[0];

  // Warning: nil returned from a method that is expected
  // to return a non-null value
  return result;
}

```
Warns when a null pointer is returned from a function annotated with `__attribute__((returns_nonnull))`

```c
int global;
__attribute__((returns_nonnull)) void* getPtr(void* p);

void* getPtr(void* p) {
  if (p) { // forgot to negate the condition
    return &global;
  }
  // Warning: nullptr returned from a function that is expected
  // to return a non-null value
  return p;
}

```

### 1.1.5. optin

Checkers for portability, performance, optional security and coding style specific rules.

#### 1.1.5.1. optin.core.EnumCastOutOfRange

Check for integer to enumeration casts that would produce a value with no
corresponding enumerator. This is not necessarily undefined behavior, but can
lead to nasty surprises, so projects may decide to use a coding standard that
disallows these “unusual” conversions.

Note that no warnings are produced when the enum type (e.g. *std::byte*) has no
enumerators at all.

```c
enum WidgetKind { A=1, B, C, X=99 };

void foo() {
  WidgetKind c = static_cast<WidgetKind>(3);  // OK
  WidgetKind x = static_cast<WidgetKind>(99); // OK
  WidgetKind d = static_cast<WidgetKind>(4);  // warn
}

```
**Limitations**

This checker does not accept the coding pattern where an enum type is used to
store combinations of flag values.
Such enums should be annotated with the *__attribute__((flag_enum))* or by the*[[clang::flag_enum]]* attribute to signal this intent. Refer to the[documentation](https://clang.llvm.org/docs/AttributeReference.html#flag-enum)of this Clang attribute.

```c
enum AnimalFlags
{
    HasClaws   = 1,
    CanFly     = 2,
    EatsFish   = 4,
    Endangered = 8
};

AnimalFlags operator|(AnimalFlags a, AnimalFlags b)
{
    return static_cast<AnimalFlags>(static_cast<int>(a) | static_cast<int>(b));
}

auto flags = HasClaws | CanFly;

```
Projects that use this pattern should not enable this optin checker.

#### 1.1.5.4. optin.mpi.MPI-Checker

Checks MPI code.

```c
void test() {
  double buf = 0;
  MPI_Request sendReq1;
  MPI_Ireduce(MPI_IN_PLACE, &buf, 1, MPI_DOUBLE, MPI_SUM,
      0, MPI_COMM_WORLD, &sendReq1);
} // warn: request 'sendReq1' has no matching wait.

void test() {
  double buf = 0;
  MPI_Request sendReq;
  MPI_Isend(&buf, 1, MPI_DOUBLE, 0, 0, MPI_COMM_WORLD, &sendReq);
  MPI_Irecv(&buf, 1, MPI_DOUBLE, 0, 0, MPI_COMM_WORLD, &sendReq); // warn
  MPI_Isend(&buf, 1, MPI_DOUBLE, 0, 0, MPI_COMM_WORLD, &sendReq); // warn
  MPI_Wait(&sendReq, MPI_STATUS_IGNORE);
}

void missingNonBlocking() {
  int rank = 0;
  MPI_Comm_rank(MPI_COMM_WORLD, &rank);
  MPI_Request sendReq1[10][10][10];
  MPI_Wait(&sendReq1[1][7][9], MPI_STATUS_IGNORE); // warn
}

```

#### 1.1.5.7. optin.performance.GCDAntipattern

Check for performance anti-patterns when using Grand Central Dispatch.

#### 1.1.5.8. optin.performance.Padding

Check for excessively padded structs.

This checker detects structs with excessive padding, which can lead to wasted
memory thus decreased performance by reducing the effectiveness of the
processor cache. Padding bytes are added by compilers to align data accesses
as some processors require data to be aligned to certain boundaries. On others,
unaligned data access are possible, but impose significantly larger latencies.

To avoid padding bytes, the fields of a struct should be ordered by decreasing
by alignment. Usually, its easier to think of the `sizeof` of the fields, and
ordering the fields by `sizeof` would usually also lead to the same optimal
layout.

In rare cases, one can use the `#pragma pack(1)` directive to enforce a packed
layout too, but it can significantly increase the access times, so reordering the
fields is usually a better solution.

```c
// warn: Excessive padding in 'struct NonOptimal' (35 padding bytes, where 3 is optimal)
struct NonOptimal {
  char c1;
  // 7 bytes of padding
  std::int64_t big1; // 8 bytes
  char c2;
  // 7 bytes of padding
  std::int64_t big2; // 8 bytes
  char c3;
  // 7 bytes of padding
  std::int64_t big3; // 8 bytes
  char c4;
  // 7 bytes of padding
  std::int64_t big4; // 8 bytes
  char c5;
  // 7 bytes of padding
};
static_assert(sizeof(NonOptimal) == 4*8+5+5*7);

// no-warning: The fields are nicely aligned to have the minimal amount of padding bytes.
struct Optimal {
  std::int64_t big1; // 8 bytes
  std::int64_t big2; // 8 bytes
  std::int64_t big3; // 8 bytes
  std::int64_t big4; // 8 bytes
  char c1;
  char c2;
  char c3;
  char c4;
  char c5;
  // 3 bytes of padding
};
static_assert(sizeof(Optimal) == 4*8+5+3);

// no-warning: Bit packing representation is also accepted by this checker, but
// it can significantly increase access times, so prefer reordering the fields.
#pragma pack(1)
struct BitPacked {
  char c1;
  std::int64_t big1; // 8 bytes
  char c2;
  std::int64_t big2; // 8 bytes
  char c3;
  std::int64_t big3; // 8 bytes
  char c4;
  std::int64_t big4; // 8 bytes
  char c5;
};
static_assert(sizeof(BitPacked) == 4*8+5);

```
The `AllowedPad` option can be used to specify a threshold for the number
padding bytes raising the warning. If the number of padding bytes of the struct
and the optimal number of padding bytes differ by more than the threshold value,
a warning will be raised.

By default, the `AllowedPad` threshold is 24 bytes.

To override this threshold to e.g. 4 bytes, use the`-analyzer-config optin.performance.Padding:AllowedPad=4` option.

#### 1.1.5.9. optin.portability.UnixAPI

Reports situations where 0 is passed as the “size” argument of various
allocation functions ( `calloc`, `malloc`, `realloc`, `reallocf`,`alloca`, `__builtin_alloca`, `__builtin_alloca_with_align`, `valloc`).

Note that similar functionality is also supported by [unix.Malloc (C)](#unix-malloc) which
reports code that *uses* memory allocated with size zero.

(The name of this checker is motivated by the fact that it was originally
introduced with the vague goal that it “Finds implementation-defined behavior
in UNIX/Posix functions.”)

### 1.1.6. optin.taint

Checkers implementing[taint analysis](https://en.wikipedia.org/wiki/Taint_checking).

#### 1.1.6.1. optin.taint.GenericTaint

Taint analysis identifies potential security vulnerabilities where the
attacker can inject malicious data to the program to execute an attack
(privilege escalation, command injection, SQL injection etc.).

The malicious data is injected at the taint source (e.g. `getenv()` call)
which is then propagated through function calls and being used as arguments of
sensitive operations, also called as taint sinks (e.g. `system()` call).

One can defend against this type of vulnerability by always checking and
sanitizing the potentially malicious, untrusted user input.

The goal of the checker is to discover and show to the user these potential
taint source-sink pairs and the propagation call chain.

The most notable examples of taint sources are:

> * data from network
> * files or standard input
> * environment variables
> * data from databases

Let us examine a practical example of a Command Injection attack.

```c
// Command Injection Vulnerability Example
int main(int argc, char** argv) {
  char cmd[2048] = "/bin/cat ";
  char filename[1024];
  printf("Filename:");
  scanf (" %1023[^n]", filename); // The attacker can inject a shell escape here
  strcat(cmd, filename);
  system(cmd); // Warning: Untrusted data is passed to a system call
}

```
The program prints the content of any user specified file.
Unfortunately the attacker can execute arbitrary commands
with shell escapes. For example with the following input the *ls* command is also
executed after the contents of */etc/shadow* is printed.*Input: /etc/shadow ; ls /*

The analysis implemented in this checker points out this problem.

One can protect against such attack by for example checking if the provided
input refers to a valid file and removing any invalid user input.

```c
// No vulnerability anymore, but we still get the warning
void sanitizeFileName(char* filename){
  if (access(filename,F_OK)){// Verifying user input
    printf("File does not existn");
    filename[0]='0';
    }
}
int main(int argc, char** argv) {
  char cmd[2048] = "/bin/cat ";
  char filename[1024];
  printf("Filename:");
  scanf (" %1023[^n]", filename); // The attacker can inject a shell escape here
  sanitizeFileName(filename);// filename is safe after this point
  if (!filename[0])
    return -1;
  strcat(cmd, filename);
  system(cmd); // Superfluous Warning: Untrusted data is passed to a system call
}
```
Unfortunately, the checker cannot discover automatically that the programmer
have performed data sanitation, so it still emits the warning.

One can get rid of this superfluous warning by telling by specifying the
sanitation functions in the taint configuration file (see[Taint Analysis Configuration](user-docs/TaintAnalysisConfiguration.html)).

```c
Filters:
- Name: sanitizeFileName
  Args: [0]
```
The clang invocation to pass the configuration file location:

```sh
clang  --analyze -Xclang -analyzer-config  -Xclang optin.taint.TaintPropagation:Config=`pwd`/taint_config.yml ...
```
If you are validating your inputs instead of sanitizing them, or don’t want to
mention each sanitizing function in our configuration,
you can use a more generic approach.

Introduce a generic no-op *csa_mark_sanitized(..)* function to
tell the Clang Static Analyzer
that the variable is safe to be used on that analysis path.

```c
// Marking sanitized variables safe.
// No vulnerability anymore, no warning.

// User csa_mark_sanitize function is for the analyzer only
#ifdef __clang_analyzer__
  void csa_mark_sanitized(const void *);
#endif

int main(int argc, char** argv) {
  char cmd[2048] = "/bin/cat ";
  char filename[1024];
  printf("Filename:");
  scanf (" %1023[^n]", filename);
  if (access(filename,F_OK)){// Verifying user input
    printf("File does not existn");
    return -1;
  }
  #ifdef __clang_analyzer__
    csa_mark_sanitized(filename); // Indicating to CSA that filename variable is safe to be used after this point
  #endif
  strcat(cmd, filename);
  system(cmd); // No warning
}

```
Similarly to the previous example, you need to
define a *Filter* function in a *YAML* configuration file
and add the *csa_mark_sanitized* function.

```c
Filters:
- Name: csa_mark_sanitized
  Args: [0]

```
Then calling *csa_mark_sanitized(X)* will tell the analyzer that *X* is safe to
be used after this point, because its contents are verified. It is the
responsibility of the programmer to ensure that this verification was indeed
correct. Please note that *csa_mark_sanitized* function is only declared and
used during Clang Static Analysis and skipped in (production) builds.

Further examples of injection vulnerabilities this checker can find.

```c
void test() {
  char x = getchar(); // 'x' marked as tainted
  system(&x); // warn: untrusted data is passed to a system call
}

// note: compiler internally checks if the second param to
// sprintf is a string literal or not.
// Use -Wno-format-security to suppress compiler warning.
void test() {
  char s[10], buf[10];
  fscanf(stdin, "%s", s); // 's' marked as tainted

  sprintf(buf, s); // warn: untrusted data used as a format string
}

```
There are built-in sources, propagations and sinks even if no external taint
configuration is provided.

Default sources:
:   `_IO_getc`, `fdopen`, `fopen`, `freopen`, `get_current_dir_name`,`getch`, `getchar`, `getchar_unlocked`, `getwd`, `getcwd`,`getgroups`, `gethostname`, `getlogin`, `getlogin_r`, `getnameinfo`,`gets`, `gets_s`, `getseuserbyname`, `readlink`, `readlinkat`,`scanf`, `scanf_s`, `socket`, `wgetch`

Default propagations rules:
:   `atoi`, `atol`, `atoll`, `basename`, `dirname`, `fgetc`,`fgetln`, `fgets`, `fnmatch`, `fread`, `fscanf`, `fscanf_s`,`index`, `inflate`, `isalnum`, `isalpha`, `isascii`, `isblank`,`iscntrl`, `isdigit`, `isgraph`, `islower`, `isprint`, `ispunct`,`isspace`, `isupper`, `isxdigit`, `memchr`, `memrchr`, `sscanf`,`getc`, `getc_unlocked`, `getdelim`, `getline`, `getw`, `memcmp`,`memcpy`, `memmem`, `memmove`, `mbtowc`, `pread`, `qsort`,`qsort_r`, `rawmemchr`, `read`, `recv`, `recvfrom`, `rindex`,`strcasestr`, `strchr`, `strchrnul`, `strcasecmp`, `strcmp`,`strcspn`, `strncasecmp`, `strncmp`, `strndup`,`strndupa`, `strpbrk`, `strrchr`, `strsep`, `strspn`,`strstr`, `strtol`, `strtoll`, `strtoul`, `strtoull`, `tolower`,`toupper`, `ttyname`, `ttyname_r`, `wctomb`, `wcwidth`

Default sinks:
:   `printf`, `setproctitle`, `system`, `popen`, `execl`, `execle`,`execlp`, `execv`, `execvp`, `execvP`, `execve`, `dlopen`

Please note that there are no built-in filter functions.

One can configure their own taint sources, sinks, and propagation rules by
providing a configuration file via checker option`optin.taint.TaintPropagation:Config`. The configuration file is in[YAML](http://llvm.org/docs/YamlIO.html#introduction-to-yaml) format. The
taint-related options defined in the config file extend but do not override the
built-in sources, rules, sinks. The format of the external taint configuration
file is not stable, and could change without any notice even in a non-backward
compatible way.

For a more detailed description of configuration options, please see the[Taint Analysis Configuration](user-docs/TaintAnalysisConfiguration.html). For an example see[Example configuration file](user-docs/TaintAnalysisConfiguration.html#clangsa-taint-configuration-example).

**Configuration**

* *Config* Specifies the name of the YAML configuration file. The user can
define their own taint sources and sinks.

**Related Guidelines**

* [CWE Data Neutralization Issues](https://cwe.mitre.org/data/definitions/137.html)
* [SEI Cert STR02-C. Sanitize data passed to complex subsystems](https://wiki.sei.cmu.edu/confluence/display/c/STR02-C.+Sanitize+data+passed+to+complex+subsystems)
* [SEI Cert ENV33-C. Do not call system()](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152177)
* [ENV03-C. Sanitize the environment when invoking external programs](https://wiki.sei.cmu.edu/confluence/display/c/ENV03-C.+Sanitize+the+environment+when+invoking+external+programs)

**Limitations**

* The taintedness property is not propagated through function calls which are
unknown (or too complex) to the analyzer, unless there is a specific
propagation rule built-in to the checker or given in the YAML configuration
file. This causes potential true positive findings to be lost.

#### 1.1.6.2. optin.taint.TaintedAlloc

This checker warns for cases when the `size` parameter of the `malloc` ,`calloc`, `realloc`, `alloca` or the size parameter of the
array new C++ operator is tainted (potentially attacker controlled).
If an attacker can inject a large value as the size parameter, memory exhaustion
denial of service attack can be carried out.

The analyzer emits warning only if it cannot prove that the size parameter is
within reasonable bounds (`<= SIZE_MAX/4`). This functionality partially
covers the SEI Cert coding standard rule [INT04-C](https://wiki.sei.cmu.edu/confluence/display/c/INT04-C.+Enforce+limits+on+integer+values+originating+from+tainted+sources).

You can silence this warning either by bound checking the `size` parameter, or
by explicitly marking the `size` parameter as sanitized. See the[optin.taint.GenericTaint (C, C++)](#optin-taint-generictaint) checker for an example.

Custom allocation/deallocation functions can be defined using[ownership attributes](../AttributeReference.html#analyzer-ownership-attrs).

```c
void vulnerable(void) {
  size_t size = 0;
  scanf("%zu", &size);
  int *p = malloc(size); // warn: malloc is called with a tainted (potentially attacker controlled) value
  free(p);
}

void not_vulnerable(void) {
  size_t size = 0;
  scanf("%zu", &size);
  if (1024 < size)
    return;
  int *p = malloc(size); // No warning expected as the the user input is bound
  free(p);
}

void vulnerable_cpp(void) {
  size_t size = 0;
  scanf("%zu", &size);
  int *ptr = new int[size];// warn: Memory allocation function is called with a tainted (potentially attacker controlled) value
  delete[] ptr;
}
```

#### 1.1.6.3. optin.taint.TaintedDiv

This checker warns when the denominator in a division
operation is a tainted (potentially attacker controlled) value.
If the attacker can set the denominator to 0, a runtime error can
be triggered. The checker warns when the denominator is a tainted
value and the analyzer cannot prove that it is not 0. This warning
is more pessimistic than the [core.DivideZero (C, C++, ObjC)](#core-dividezero) checker
which warns only when it can prove that the denominator is 0.

```c
int vulnerable(int n) {
  size_t size = 0;
  scanf("%zu", &size);
  return n / size; // warn: Division by a tainted value, possibly zero
}

int not_vulnerable(int n) {
  size_t size = 0;
  scanf("%zu", &size);
  if (!size)
    return 0;
  return n / size; // no warning
}
```

### 1.1.7. Security

Security related checkers.

#### 1.1.7.1. security.ArrayBound

Report out of bounds access to memory that is before the start or after the end
of the accessed region (array, heap-allocated region, string literal etc.).
This usually means incorrect indexing, but the checker also detects access via
the operators `*` and `->`.

```c
void test_underflow(int x) {
  int buf[100][100];
  if (x < 0)
    buf[0][x] = 1; // warn
}

void test_overflow() {
  int buf[100];
  int *p = buf + 100;
  *p = 1; // warn
}
```

If checkers like [unix.Malloc (C)](#unix-malloc) or [cplusplus.NewDelete (C++)](#cplusplus-newdelete) are enabled
to model the behavior of `malloc()`, `operator new` and similar
allocators), then this checker can also reports out of bounds access to
dynamically allocated memory:

```c
int *test_dynamic() {
  int *mem = new int[100];
  mem[-1] = 42; // warn
  return mem;
}
```

In uncertain situations (when the checker can neither prove nor disprove that
overflow occurs), the checker assumes that the the index (more precisely, the
memory offeset) is within bounds.

However, if [optin.taint.GenericTaint (C, C++)](#optin-taint-generictaint) is enabled and the index/offset is
tainted (i.e. it is influenced by an untrusted source), then this checker
reports the potential out of bounds access:

```c
void test_with_tainted_index() {
  char s[] = "abc";
  int x = getchar();
  char c = s[x]; // warn: potential out of bounds access with tainted index
}
```

Note

This checker is an improved and renamed version of the checker that was
previously known as `alpha.security.ArrayBoundV2`. The old checker`alpha.security.ArrayBound` was removed when the (previously
“experimental”) V2 variant became stable enough for regular use.

#### 1.1.7.2. security.cert.env.InvalidPtr

Corresponds to SEI CERT Rules [ENV31-C](https://wiki.sei.cmu.edu/confluence/display/c/ENV31-C.+Do+not+rely+on+an+environment+pointer+following+an+operation+that+may+invalidate+it) and [ENV34-C](https://wiki.sei.cmu.edu/confluence/display/c/ENV34-C.+Do+not+store+pointers+returned+by+certain+functions).

* **ENV31-C**:
Rule is about the possible problem with `main` function’s third argument, environment pointer,
“envp”. When environment array is modified using some modification function
such as `putenv`, `setenv` or others, It may happen that memory is reallocated,
however “envp” is not updated to reflect the changes and points to old memory
region.
* **ENV34-C**:
Some functions return a pointer to a statically allocated buffer.
Consequently, subsequent call of these functions will invalidate previous
pointer. These functions include: `getenv`, `localeconv`, `asctime`, `setlocale`, `strerror`

```c
int main(int argc, const char *argv[], const char *envp[]) {
  if (setenv("MY_NEW_VAR", "new_value", 1) != 0) {
    // setenv call may invalidate 'envp'
    /* Handle error */
  }
  if (envp != NULL) {
    for (size_t i = 0; envp[i] != NULL; ++i) {
      puts(envp[i]);
      // envp may no longer point to the current environment
      // this program has unanticipated behavior, since envp
      // does not reflect changes made by setenv function.
    }
  }
  return 0;
}

void previous_call_invalidation() {
  char *p, *pp;

  p = getenv("VAR");
  setenv("SOMEVAR", "VALUE", /*overwrite = */1);
  // call to 'setenv' may invalidate p

  *p;
  // dereferencing invalid pointer
}
```

The `InvalidatingGetEnv` option is available for treating `getenv` calls as
invalidating. When enabled, the checker issues a warning if `getenv` is called
multiple times and their results are used without first creating a copy.
This level of strictness might be considered overly pedantic for the commonly
used `getenv` implementations.

To enable this option, use:`-analyzer-config security.cert.env.InvalidPtr:InvalidatingGetEnv=true`.

By default, this option is set to *false*.

When this option is enabled, warnings will be generated for scenarios like the
following:

```c
char* p = getenv("VAR");
char* pp = getenv("VAR2"); // assumes this call can invalidate `env`
strlen(p); // warns about accessing invalid ptr
```

#### 1.1.7.3. security.FloatLoopCounter

Warn on using a floating point value as a loop counter (CERT: FLP30-C, FLP30-CPP).

```c
void test() {
  for (float x = 0.1f; x <= 1.0f; x += 0.1f) {} // warn
}

```

#### 1.1.7.4. security.insecureAPI.UncheckedReturn

Warn on uses of functions whose return values must be always checked.

```c
void test() {
  setuid(1); // warn
}
```

#### 1.1.7.5. security.insecureAPI.bcmp

Warn on uses of the ‘bcmp’ function.

```c
void test() {
  bcmp(ptr0, ptr1, n); // warn
}

```

#### 1.1.7.6. security.insecureAPI.bcopy

Warn on uses of the ‘bcopy’ function.

```c
void test() {
  bcopy(src, dst, n); // warn
}
```

#### 1.1.7.7. security.insecureAPI.bzero

Warn on uses of the ‘bzero’ function.

```c
void test() {
  bzero(ptr, n); // warn
}
```

#### 1.1.7.8. security.insecureAPI.getpw

Warn on uses of the ‘getpw’ function.

```c
void test() {
  char buff[1024];
  getpw(2, buff); // warn
}
```

#### 1.1.7.9. security.insecureAPI.gets

Warn on uses of the ‘gets’ function.

```c
void test() {
  char buff[1024];
  gets(buff); // warn
}
```

#### 1.1.7.10. security.insecureAPI.mkstemp

Warn when ‘mkstemp’ is passed fewer than 6 X’s in the format string.

```c
void test() {
  mkstemp("XX"); // warn
}
```

#### 1.1.7.11. security.insecureAPI.mktemp

Warn on uses of the `mktemp` function.

```c
void test() {
  char *x = mktemp("/tmp/zxcv"); // warn: insecure, use mkstemp
}
```

#### 1.1.7.12. security.insecureAPI.rand

Warn on uses of inferior random number generating functions (only if arc4random function is available):`drand48, erand48, jrand48, lcong48, lrand48, mrand48, nrand48, random, rand_r`.

```c
void test() {
  random(); // warn
}
```

#### 1.1.7.13. security.insecureAPI.strcpy

Warn on uses of the `strcpy` and `strcat` functions.

```c
void test() {
  char x[4];
  char *y = "abcd";

  strcpy(x, y); // warn
}
```

#### 1.1.7.14. security.insecureAPI.vfork

> Warn on uses of the ‘vfork’ function.

```c
void test() {
  vfork(); // warn
}
```

#### 1.1.7.15. security.insecureAPI.DeprecatedOrUnsafeBufferHandling

> Warn on occurrences of unsafe or deprecated buffer handling functions, which now have a secure variant: `sprintf, fprintf, vsprintf, scanf, wscanf, fscanf, fwscanf, vscanf, vwscanf, vfscanf, vfwscanf, sscanf, swscanf, vsscanf, vswscanf, swprintf, snprintf, vswprintf, vsnprintf, memcpy, memmove, strncpy, strncat, memset`

```c
void test() {
  char buf [5];
  strncpy(buf, "a", 1); // warn
}
```

#### 1.1.7.16. security.MmapWriteExec

Warn on `mmap()` calls with both writable and executable access.

```c
void test(int n) {
  void *c = mmap(NULL, 32, PROT_READ | PROT_WRITE | PROT_EXEC,
                 MAP_PRIVATE | MAP_ANON, -1, 0);
  // warn: Both PROT_WRITE and PROT_EXEC flags are set. This can lead to
  //       exploitable memory regions, which could be overwritten with malicious
  //       code
}
```

#### 1.1.7.17. security.PointerSub

Check for pointer subtractions on two pointers pointing to different memory
chunks. According to the C standard §6.5.6 only subtraction of pointers that
point into (or one past the end) the same array object is valid (for this
purpose non-array variables are like arrays of size 1). This checker only
searches for different memory objects at subtraction, but does not check if the
array index is correct. Furthermore, only cases are reported where
stack-allocated objects are involved (no warnings on pointers to memory
allocated by *malloc*).

```c
void test() {
  int a, b, c[10], d[10];
  int x = &c[3] - &c[1];
  x = &d[4] - &c[1]; // warn: 'c' and 'd' are different arrays
  x = (&a + 1) - &a;
  x = &b - &a; // warn: 'a' and 'b' are different variables
}

struct S {
  int x[10];
  int y[10];
};

void test1() {
  struct S a[10];
  struct S b;
  int d = &a[4] - &a[6];
  d = &a[0].x[3] - &a[0].x[1];
  d = a[0].y - a[0].x; // warn: 'S.b' and 'S.a' are different objects
  d = (char *)&b.y - (char *)&b.x; // warn: different members of the same object
  d = (char *)&b.y - (char *)&b; // warn: object of type S is not the same array as a member of it
}
```

There may be existing applications that use code like above for calculating
offsets of members in a structure, using pointer subtractions. This is still
undefined behavior according to the standard and code like this can be replaced
with the *offsetof* macro.

#### 1.1.7.18. security.PutenvStackArray

Finds calls to the `putenv` function which pass a pointer to a stack-allocated
(automatic) array as the argument. Function `putenv` does not copy the passed
string, only a pointer to the data is stored and this data can be read even by
other threads. Content of a stack-allocated array is likely to be overwritten
after exiting from the function.

The problem can be solved by using a static array variable or dynamically
allocated memory. Even better is to avoid using `putenv` (it has other
problems related to memory leaks) and use `setenv` instead.

The check corresponds to CERT rule[POS34-C. Do not call putenv() with a pointer to an automatic variable as the argument](https://wiki.sei.cmu.edu/confluence/display/c/POS34-C.+Do+not+call+putenv%28%29+with+a+pointer+to+an+automatic+variable+as+the+argument).

```c
int f() {
  char env[] = "NAME=value";
  return putenv(env); // putenv function should not be called with stack-allocated string
}
```
There is one case where the checker can report a false positive. This is when
the stack-allocated array is used at *putenv* in a function or code branch that
does not return (process is terminated on all execution paths).

Another special case is if the *putenv* is called from function *main*. Here
the stack is deallocated at the end of the program and it should be no problem
to use the stack-allocated string (a multi-threaded program may require more
attention). The checker does not warn for cases when stack space of *main* is
used at the *putenv* call.

#### 1.1.7.19. security.SetgidSetuidOrder

When dropping user-level and group-level privileges in a program by using`setuid` and `setgid` calls, it is important to reset the group-level
privileges (with `setgid`) first. Function `setgid` will likely fail if
the superuser privileges are already dropped.

The checker checks for sequences of `setuid(getuid())` and`setgid(getgid())` calls (in this order). If such a sequence is found and
there is no other privilege-changing function call (`seteuid`, `setreuid`,`setresuid` and the GID versions of these) in between, a warning is
generated. The checker finds only exactly `setuid(getuid())` calls (and the
GID versions), not for example if the result of `getuid()` is stored in a
variable.

```c
void test1() {
  // ...
  // end of section with elevated privileges
  // reset privileges (user and group) to normal user
  if (setuid(getuid()) != 0) {
    handle_error();
    return;
  }
  if (setgid(getgid()) != 0) { // warning: A 'setgid(getgid())' call following a 'setuid(getuid())' call is likely to fail
    handle_error();
    return;
  }
  // user-ID and group-ID are reset to normal user now
  // ...
}
```

In the code above the problem is that `setuid(getuid())` removes superuser
privileges before `setgid(getgid())` is called. To fix the problem the`setgid(getgid())` should be called first. Further attention is needed to
avoid code like `setgid(getuid())` (this checker does not detect bugs like
this) and always check the return value of these calls.

This check corresponds to SEI CERT Rule [POS36-C](https://wiki.sei.cmu.edu/confluence/display/c/POS36-C.+Observe+correct+revocation+order+while+relinquishing+privileges).

### 1.1.8. unix

POSIX/Unix checkers.

#### 1.1.8.1. unix.API

Check calls to various UNIX/Posix functions: `open, pthread_once, calloc, malloc, realloc, alloca`.

```c
// Currently the check is performed for apple targets only.
void test(const char *path) {
  int fd = open(path, O_CREAT);
    // warn: call to 'open' requires a third argument when the
    // 'O_CREAT' flag is set
}

void f();

void test() {
  pthread_once_t pred = {0x30B1BCBA, {0}};
  pthread_once(&pred, f);
    // warn: call to 'pthread_once' uses the local variable
}

void test() {
  void *p = malloc(0); // warn: allocation size of 0 bytes
}

void test() {
  void *p = calloc(0, 42); // warn: allocation size of 0 bytes
}

void test() {
  void *p = malloc(1);
  p = realloc(p, 0); // warn: allocation size of 0 bytes
}

void test() {
  void *p = alloca(0); // warn: allocation size of 0 bytes
}

void test() {
  void *p = valloc(0); // warn: allocation size of 0 bytes
}
```

#### 1.1.8.2. unix.BlockInCriticalSection

Check for calls to blocking functions inside a critical section.
Blocking functions detected by this checker: `sleep, getc, fgets, read, recv`.
Critical section handling functions modeled by this checker:`lock, unlock, pthread_mutex_lock, pthread_mutex_trylock, pthread_mutex_unlock, mtx_lock, mtx_timedlock, mtx_trylock, mtx_unlock, lock_guard, unique_lock`.

```c
void pthread_lock_example(pthread_mutex_t *m) {
  pthread_mutex_lock(m); // note: entering critical section here
  sleep(10); // warn: Call to blocking function 'sleep' inside of critical section
  pthread_mutex_unlock(m);
}
```

**Limitations**

* The `trylock` and `timedlock` versions of acquiring locks are currently assumed to always succeed.
This can lead to false positives.

```c
void trylock_example(pthread_mutex_t *m) {
  if (pthread_mutex_trylock(m) == 0) { // assume trylock always succeeds
    sleep(10); // warn: Call to blocking function 'sleep' inside of critical section
    pthread_mutex_unlock(m);
  } else {
    sleep(10); // false positive: Incorrect warning about blocking function inside critical section.
  }
}
```

#### 1.1.8.3. unix.Chroot

Check improper use of chroot described by SEI Cert C recommendation [POS05-C.
Limit access to files by creating a jail](https://wiki.sei.cmu.edu/confluence/display/c/POS05-C.+Limit+access+to+files+by+creating+a+jail).
The checker finds usage patterns where `chdir("/")` is not called immediately
after a call to `chroot(path)`.

```c
void f();

void test_bad() {
  chroot("/usr/local");
  f(); // warn: no call of chdir("/") immediately after chroot
}

 void test_bad_path() {
   chroot("/usr/local");
   chdir("/usr"); // warn: no call of chdir("/") immediately after chroot
   f();
 }

void test_good() {
  chroot("/usr/local");
  chdir("/"); // no warning
  f();
}
```

#### 1.1.8.4. unix.Errno

Check for improper use of `errno`.
This checker implements partially CERT rule[ERR30-C. Set errno to zero before calling a library function known to set errno,
and check errno only after the function returns a value indicating failure](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152351).
The checker can find the first read of `errno` after successful standard
function calls.

The C and POSIX standards often do not define if a standard library function
may change value of `errno` if the call does not fail.
Therefore, `errno` should only be used if it is known from the return value
of a function that the call has failed.
There are exceptions to this rule (for example `strtol`) but the affected
functions are not yet supported by the checker.
The return values for the failure cases are documented in the standard Linux man
pages of the functions and in the [POSIX standard](https://pubs.opengroup.org/onlinepubs/9699919799/).

```c
int unsafe_errno_read(int sock, void *data, int data_size) {
  if (send(sock, data, data_size, 0) != data_size) {
    // 'send' can be successful even if not all data was sent
    if (errno == 1) { // An undefined value may be read from 'errno'
      return 0;
    }
  }
  return 1;
}
```

The checker [unix.StdCLibraryFunctions (C)](#unix-stdclibraryfunctions) must be turned on to get the
warnings from this checker. The supported functions are the same as by[unix.StdCLibraryFunctions (C)](#unix-stdclibraryfunctions). The `ModelPOSIX` option of that
checker affects the set of checked functions.

**Parameters**

The `AllowErrnoReadOutsideConditionExpressions` option allows read of the
errno value if the value is not used in a condition (in `if` statements,
loops, conditional expressions, `switch` statements). For example `errno`can be stored into a variable without getting a warning by the checker.

```c
int unsafe_errno_read(int sock, void *data, int data_size) {
  if (send(sock, data, data_size, 0) != data_size) {
    int err = errno;
    // warning if 'AllowErrnoReadOutsideConditionExpressions' is false
    // no warning if 'AllowErrnoReadOutsideConditionExpressions' is true
  }
  return 1;
}
```

Default value of this option is `true`. This allows save of the errno value
for possible later error handling.

**Limitations**

> * Only the very first usage of `errno` is checked after an affected function
> call. Value of `errno` is not followed when it is stored into a variable
> or returned from a function.
> * Documentation of function `lseek` is not clear about what happens if the
> function returns different value than the expected file position but not -1.
> To avoid possible false-positives `errno` is allowed to be used in this
> case.

#### 1.1.8.5. unix.Malloc

Check for memory leaks, double free, and use-after-free problems. Traces memory managed by malloc()/free().

Custom allocation/deallocation functions can be defined using[ownership attributes](../AttributeReference.html#analyzer-ownership-attrs).

```c
void test() {
  int *p = malloc(1);
  free(p);
  free(p); // warn: attempt to release already released memory
}

void test() {
  int *p = malloc(sizeof(int));
  free(p);
  *p = 1; // warn: use after free
}

void test() {
  int *p = malloc(1);
  if (p)
    return; // warn: memory is never released
}

void test() {
  int a[] = { 1 };
  free(a); // warn: argument is not allocated by malloc
}

void test() {
  int *p = malloc(sizeof(char));
  p = p - 1;
  free(p); // warn: argument to free() is offset by -4 bytes
}
```

#### 1.1.8.6. unix.MallocSizeof

Check for dubious `malloc` arguments involving `sizeof`.

Custom allocation/deallocation functions can be defined using[ownership attributes](../AttributeReference.html#analyzer-ownership-attrs).

```c
void test() {
  long *p = malloc(sizeof(short));
    // warn: result is converted to 'long *', which is
    // incompatible with operand type 'short'
  free(p);
}
```

#### 1.1.8.7. unix.MismatchedDeallocator

Check for mismatched deallocators.

Custom allocation/deallocation functions can be defined using[ownership attributes](../AttributeReference.html#analyzer-ownership-attrs).

```c
void test() {
  int *p = (int *)malloc(sizeof(int));
  delete p; // warn
}

void __attribute((ownership_returns(malloc))) *user_malloc(size_t);
void __attribute((ownership_takes(malloc, 1))) *user_free(void *);

void __attribute((ownership_returns(malloc1))) *user_malloc1(size_t);
void __attribute((ownership_takes(malloc1, 1))) *user_free1(void *);

void test() {
  int *p = (int *)user_malloc(sizeof(int));
  delete p; // warn
}

void test() {
  int *p = new int;
  free(p); // warn
}

void test() {
  int *p = new int[1];
  realloc(p, sizeof(long)); // warn
}

void test() {
  int *p = user_malloc(10);
  user_free1(p); // warn
}

// C, C++
template <typename T>
struct SimpleSmartPointer {
  T *ptr;

  explicit SimpleSmartPointer(T *p = 0) : ptr(p) {}
  ~SimpleSmartPointer() {
    delete ptr; // warn
  }
};

void test() {
  SimpleSmartPointer<int> a((int *)malloc(4));
}

```

#### 1.1.8.8. unix.Vfork

Check for proper usage of `vfork`.

```c
int test(int x) {
  pid_t pid = vfork(); // warn
  if (pid != 0)
    return 0;

  switch (x) {
  case 0:
    pid = 1;
    execl("", "", 0);
    _exit(1);
    break;
  case 1:
    x = 0; // warn: this assignment is prohibited
    break;
  case 2:
    foo(); // warn: this function call is prohibited
    break;
  default:
    return 0; // warn: return is prohibited
  }

  while(1);
}
```

#### 1.1.8.9. unix.cstring.BadSizeArg

Check the size argument passed into C string functions for common erroneous patterns. Use `-Wno-strncat-size` compiler option to mute other `strncat`-related compiler warnings.

```c
void test() {
  char dest[3];
  strncat(dest, """""""""""""""""""""""""*", sizeof(dest));
    // warn: potential buffer overflow
}
```

#### [1.1.8.10. unix.cstring.NotNullTerminated (C)](#id114)[¶](#unix-cstring-notnullterminated-c "Link to this heading")

Check for arguments which are not null-terminated strings;
applies to the `strlen`, `strcpy`, `strcat`, `strcmp` family of functions.

Only very fundamental cases are detected where the passed memory block is
absolutely different from a null-terminated string. This checker does not
find if a memory buffer is passed where the terminating zero character
is missing.

```c
void test1() {
  int l = strlen((char *)&test1); // warn
}

void test2() {
label:
  int l = strlen((char *)&&label); // warn
}
```

#### 1.1.8.11. unix.cstring.NullArg

Check for null pointers being passed as arguments to C string functions:`strlen, strnlen, strcpy, strncpy, strcat, strncat, strcmp, strncmp, strcasecmp, strncasecmp, wcslen, wcsnlen`.

```c
int test() {
  return strlen(0); // warn
}
```

#### 1.1.8.12. unix.StdCLibraryFunctions

Check for calls of standard library functions that violate predefined argument
constraints. For example, according to the C standard the behavior of function`int isalnum(int ch)` is undefined if the value of `ch` is not representable
as `unsigned char` and is not equal to `EOF`.

You can think of this checker as defining restrictions (pre- and postconditions)
on standard library functions. Preconditions are checked, and when they are
violated, a warning is emitted. Postconditions are added to the analysis, e.g.
that the return value of a function is not greater than 255. Preconditions are
added to the analysis too, in the case when the affected values are not known
before the call.

For example, if an argument to a function must be in between 0 and 255, but the
value of the argument is unknown, the analyzer will assume that it is in this
interval. Similarly, if a function mustn’t be called with a null pointer and the
analyzer cannot prove that it is null, then it will assume that it is non-null.

These are the possible checks on the values passed as function arguments:
:   * The argument has an allowed range (or multiple ranges) of values. The checker
can detect if a passed value is outside of the allowed range and show the
actual and allowed values.
* The argument has pointer type and is not allowed to be null pointer. Many
(but not all) standard functions can produce undefined behavior if a null
pointer is passed, these cases can be detected by the checker.
* The argument is a pointer to a memory block and the minimal size of this
buffer is determined by another argument to the function, or by
multiplication of two arguments (like at function `fread`), or is a fixed
value (for example `asctime_r` requires at least a buffer of size 26). The
checker can detect if the buffer size is too small and in optimal case show
the size of the buffer and the values of the corresponding arguments.

```c
#define EOF -1
void test_alnum_concrete(int v) {
  int ret = isalnum(256); // 
  // warning: Function argument outside of allowed range
  (void)ret;
}

void buffer_size_violation(FILE *file) {
  enum { BUFFER_SIZE = 1024 };
  wchar_t wbuf[BUFFER_SIZE];

  const size_t size = sizeof(*wbuf);   // 4
  const size_t nitems = sizeof(wbuf);  // 4096

  // Below we receive a warning because the 3rd parameter should be the
  // number of elements to read, not the size in bytes. This case is a known
  // vulnerability described by the ARR38-C SEI-CERT rule.
  fread(wbuf, size, nitems, file);
}

int test_alnum_symbolic(int x) {
  int ret = isalnum(x);
  // after the call, ret is assumed to be in the range [-1, 255]

  if (ret > 255)      // impossible (infeasible branch)
    if (x == 0)
      return ret / x; // division by zero is not reported
  return ret;
}
```

Additionally to the argument and return value conditions, this checker also adds
state of the value `errno` if applicable to the analysis. Many system
functions set the `errno` value only if an error occurs (together with a
specific return value of the function), otherwise it becomes undefined. This
checker changes the analysis state to contain such information. This data is
used by other checkers, for example [unix.Errno (C)](#unix-errno).

**Limitations**

The checker can not always provide notes about the values of the arguments.
Without this information it is hard to confirm if the constraint is indeed
violated. The argument values are shown if they are known constants or the value
is determined by previous (not too complicated) assumptions.

The checker can produce false positives in cases such as if the program has
invariants not known to the analyzer engine or the bug report path contains
calls to unknown functions. In these cases the analyzer fails to detect the real
range of the argument.

**Parameters**

The `ModelPOSIX` option controls if functions from the POSIX standard are
recognized by the checker.

With `ModelPOSIX=true`, many POSIX functions are modeled according to the[POSIX standard](https://pubs.opengroup.org/onlinepubs/9699919799/). This includes ranges of parameters and possible return
values. Furthermore the behavior related to `errno` in the POSIX case is
often that `errno` is set only if a function call fails, and it becomes
undefined after a successful function call.

With `ModelPOSIX=false`, this checker follows the C99 language standard and
only models the functions that are described there. It is possible that the
same functions are modeled differently in the two cases because differences in
the standards. The C standard specifies less aspects of the functions, for
example exact `errno` behavior is often unspecified (and not modeled by the
checker).

Default value of the option is `true`.

#### 1.1.8.13. unix.Stream

Check C stream handling functions:`fopen, fdopen, freopen, tmpfile, fclose, fread, fwrite, fgetc, fgets, fputc, fputs, fprintf, fscanf, ungetc, getdelim, getline, fseek, fseeko, ftell, ftello, fflush, rewind, fgetpos, fsetpos, clearerr, feof, ferror, fileno`.

The checker maintains information about the C stream objects (`FILE *`) and
can detect error conditions related to use of streams. The following conditions
are detected:

* The `FILE *` pointer passed to the function is NULL (the single exception is`fflush` where NULL is allowed).
* Use of stream after close.
* Opened stream is not closed.
* Read from a stream after end-of-file. (This is not a fatal error but reported
by the checker. Stream remains in EOF state and the read operation fails.)
* Use of stream when the file position is indeterminate after a previous failed
operation. Some functions (like `ferror`, `clearerr`, `fseek`) are
allowed in this state.
* Invalid 3rd (”`whence`”) argument to `fseek`.

The stream operations are by this checker usually split into two cases, a success
and a failure case.
On the success case it also assumes that the current value of `stdout`,`stderr`, or `stdin` can’t be equal to the file pointer returned by `fopen`.
Operations performed on `stdout`, `stderr`, or `stdin` are not checked by
this checker in contrast to the streams opened by `fopen`.

In the case of write operations (like `fwrite`,`fprintf` and even `fsetpos`) this behavior could produce a large amount of
unwanted reports on projects that don’t have error checks around the write
operations, so by default the checker assumes that write operations always succeed.
This behavior can be controlled by the `Pedantic` flag: With`-analyzer-config unix.Stream:Pedantic=true` the checker will model the
cases where a write operation fails and report situations where this leads to
erroneous behavior. (The default is `Pedantic=false`, where write operations
are assumed to succeed.)

```c
void test1() {
  FILE *p = fopen("foo", "r");
} // warn: opened file is never closed

void test2() {
  FILE *p = fopen("foo", "r");
  fseek(p, 1, SEEK_SET); // warn: stream pointer might be NULL
  fclose(p);
}

void test3() {
  FILE *p = fopen("foo", "r");
  if (p) {
    fseek(p, 1, 3); // warn: third arg should be SEEK_SET, SEEK_END, or SEEK_CUR
    fclose(p);
  }
}

void test4() {
  FILE *p = fopen("foo", "r");
  if (!p)
    return;

  fclose(p);
  fclose(p); // warn: stream already closed
}

void test5() {
  FILE *p = fopen("foo", "r");
  if (!p)
    return;

  fgetc(p);
  if (!ferror(p))
    fgetc(p); // warn: possible read after end-of-file

  fclose(p);
}

void test6() {
  FILE *p = fopen("foo", "r");
  if (!p)
    return;

  fgetc(p);
  if (!feof(p))
    fgetc(p); // warn: file position may be indeterminate after I/O error

  fclose(p);
}
```

**Limitations**

The checker does not track the correspondence between integer file descriptors
and `FILE *` pointers.

© Copyright 2007-2025, The Clang Team.
 Created using [Sphinx](https://www.sphinx-doc.org/) 7.2.6. 
