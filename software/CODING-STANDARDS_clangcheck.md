# Clang Static Analyzer – Default Checkers pour le langage C
# Sommaire

- [Clang Static Analyzer – Default Checkers pour le langage C](#clang-static-analyzer--default-checkers-pour-le-langage-c)
- [Sommaire](#sommaire)
  - [1.1.1. core](#111-core)
  - [1.1.5. optin](#115-optin)
  - [1.1.6. optin.taint](#116-optintaint)
  - [1.1.7. security](#117-security)
  - [1.1.8. unix](#118-unix)


Ce document extrait et résume les checkers par défaut de Clang Static Analyzer qui concernent spécifiquement le langage C.
## 1.1.1. core

- **core.BitwiseShift (C, C++)**  
	Détecte les comportements indéfinis liés aux opérateurs de décalage bit à bit sur les types entiers.
	```c
	void basic_examples(int a, int b) {
		if (b < 0) {
			b = a << b; // warn: right operand is negative in left shift
		} else if (b >= 32) {
			b = a >> b; // warn: right shift overflows the capacity of 'int'
		}
	}
	```

- **core.CallAndMessage (C, C++, ObjC)**  
	Détecte les erreurs logiques lors des appels de fonctions, comme l'utilisation de pointeurs de fonction nuls.
	```c
	void test() {
		 void (*foo)(void);
		 foo = 0;
		 foo(); // warn: function pointer is null
	}
	```

- **core.DivideZero (C, C++, ObjC)**  
	Détecte les divisions par zéro.
	```c
	void test(int z) {
		if (z == 0)
			int x = 1 / z; // warn
	}
	```

- **core.FixedAddressDereference (C, C++, ObjC)**  
	Détecte la déréférenciation de pointeurs à adresse fixe.
	```c
	void test1() {
		int *p = (int *)0x020;
		int x = p[0]; // warn
	}
	```

- **core.NonNullParamChecker (C, C++, ObjC)**  
	Détecte le passage de pointeurs nuls à des fonctions qui attendent des arguments non nuls.
	```c
	int f(int *p) __attribute__((nonnull));
	void test(int *p) {
		if (!p)
			f(p); // warn
	}
	```

- **core.NullDereference (C, C++, ObjC)**  
	Détecte la déréférenciation de pointeurs nuls.
	```c
	void test(int *p) {
		if (p)
			return;
		int x = p[0]; // warn
	}
	```

- **core.StackAddressEscape (C)**  
	Vérifie que les adresses de la pile ne s'échappent pas de la fonction.
	```c
	char const *p;
	void test() {
		char const str[] = "string";
		p = str; // warn
	}
	```

- **core.UndefinedBinaryOperatorResult (C)**  
	Détecte les résultats indéfinis des opérateurs binaires.
	```c
	void test() {
		int x;
		int y = x + 1; // warn: left operand is garbage
	}
	```

- **core.VLASize (C)**  
	Vérifie la déclaration de tableaux de taille variable (VLA) de taille indéfinie, nulle ou négative.
	```c
	void test() {
		int x;
		int vla1[x]; // warn: garbage as size
	}
	```

- **core.uninitialized.ArraySubscript (C)**  
	Détecte l'utilisation de valeurs non initialisées comme indices de tableau.
	```c
	void test() {
		int i, a[10];
		int x = a[i]; // warn: array subscript is undefined
	}
	```

- **core.uninitialized.Assign (C)**  
	Détecte l'affectation de valeurs non initialisées.
	```c
	void test() {
		int x;
		x |= 1; // warn: left expression is uninitialized
	}
	```

- **core.uninitialized.Branch (C)**  
	Détecte l'utilisation de valeurs non initialisées dans les conditions de branchement.
	```c
	void test() {
		int x;
		if (x) // warn
			return;
	}
	```

- **core.uninitialized.CapturedBlockVariable (C)**  
	Détecte les blocs qui capturent des valeurs non initialisées.
	```c
	void test() {
		int x;
		^{ int y = x; }(); // warn
	}
	```

- **core.uninitialized.UndefReturn (C)**  
	Détecte le retour de valeurs non initialisées.
	```c
	int test() {
		int x;
		return x; // warn
	}
	```

## 1.1.5. optin

- **optin.mpi.MPI-Checker (C)**  
	Vérifie l'utilisation correcte des appels MPI non bloquants.
	```c
	void test() {
		double buf = 0;
		MPI_Request sendReq1;
		MPI_Ireduce(MPI_IN_PLACE, &buf, 1, MPI_DOUBLE, MPI_SUM,
				0, MPI_COMM_WORLD, &sendReq1);
	} // warn: request 'sendReq1' has no matching wait.
	```

- **optin.performance.Padding (C, C++, ObjC)**  
	Détecte les structures avec un padding excessif.

- **optin.portability.UnixAPI (C)**  
	Signale les situations où 0 est passé comme argument "size" à des fonctions d'allocation.
	```c
	void test() {
		void *p = malloc(0); // warn: allocation size of 0 bytes
	}
	```

## 1.1.6. optin.taint

- **optin.taint.GenericTaint (C, C++)**  
	Analyse de la propagation de données potentiellement dangereuses (taint analysis).
	```c
	int main(int argc, char** argv) {
		char cmd[2048] = "/bin/cat ";
		char filename[1024];
		printf("Filename:");
		scanf (" %1023[^\n]", filename); // L'attaquant peut injecter ici
		strcat(cmd, filename);
		system(cmd); // Warning: Untrusted data is passed to a system call
	}
	```

- **optin.taint.TaintedAlloc (C, C++)**  
	Alerte si la taille passée à malloc/calloc/realloc/allocation dynamique provient d'une source non fiable.
	```c
	void vulnerable(void) {
		size_t size = 0;
		scanf("%zu", &size);
		int *p = malloc(size); // warn
		free(p);
	}
	```

## 1.1.7. security

- **security.ArrayBound (C, C++)**  
	Détecte les accès hors limites sur les tableaux.
	```c
	void test_underflow(int x) {
		int buf[100][100];
		if (x < 0)
			buf[0][x] = 1; // warn
	}
	```

- **security.FloatLoopCounter (C)**  
	Alerte sur l'utilisation de flottants comme compteur de boucle.
	```c
	void test() {
		for (float x = 0.1f; x <= 1.0f; x += 0.1f) {} // warn
	}
	```

- **security.insecureAPI.UncheckedReturn (C)**  
	Alerte sur l'utilisation de fonctions dont la valeur de retour doit être vérifiée.
	```c
	void test() {
		setuid(1); // warn
	}
	```

- **security.insecureAPI.bcmp (C)**  
	Alerte sur l'utilisation de la fonction 'bcmp'.
	```c
	void test() {
		bcmp(ptr0, ptr1, n); // warn
	}
	```

- **security.insecureAPI.bcopy (C)**  
	Alerte sur l'utilisation de la fonction 'bcopy'.
	```c
	void test() {
		bcopy(src, dst, n); // warn
	}
	```

- **security.insecureAPI.bzero (C)**  
	Alerte sur l'utilisation de la fonction 'bzero'.
	```c
	void test() {
		bzero(ptr, n); // warn
	}
	```

- **security.insecureAPI.getpw (C)**  
	Alerte sur l'utilisation de la fonction 'getpw'.
	```c
	void test() {
		char buff[1024];
		getpw(2, buff); // warn
	}
	```

- **security.insecureAPI.gets (C)**  
	Alerte sur l'utilisation de la fonction 'gets'.
	```c
	void test() {
		char buff[1024];
		gets(buff); // warn
	}
	```

- **security.insecureAPI.mkstemp (C)**  
	Alerte si 'mkstemp' est appelé avec moins de 6 X dans le format.
	```c
	void test() {
		mkstemp("XX"); // warn
	}
	```

- **security.insecureAPI.mktemp (C)**  
	Alerte sur l'utilisation de la fonction 'mktemp'.
	```c
	void test() {
		char *x = mktemp("/tmp/zxcv"); // warn
	}
	```

- **security.insecureAPI.rand (C)**  
	Alerte sur l'utilisation de fonctions de génération de nombres aléatoires inférieures.
	```c
	void test() {
		random(); // warn
	}
	```

- **security.insecureAPI.strcpy (C)**  
	Alerte sur l'utilisation de 'strcpy' et 'strcat'.
	```c
	void test() {
		char x[4];
		char *y = "abcd";
		strcpy(x, y); // warn
	}
	```

- **security.insecureAPI.vfork (C)**  
	Alerte sur l'utilisation de la fonction 'vfork'.
	```c
	void test() {
		vfork(); // warn
	}
	```

- **security.insecureAPI.DeprecatedOrUnsafeBufferHandling (C)**  
	Alerte sur l'utilisation de fonctions de gestion de buffer obsolètes ou non sûres.
	```c
	void test() {
		char buf [5];
		strncpy(buf, "a", 1); // warn
	}
	```

- **security.MmapWriteExec (C)**  
	Alerte sur les appels à mmap() avec accès en écriture et exécution.
	```c
	void test(int n) {
		void *c = mmap(NULL, 32, PROT_READ | PROT_WRITE | PROT_EXEC,
									 MAP_PRIVATE | MAP_ANON, -1, 0);
		// warn
	}
	```

- **security.PointerSub (C)**  
	Détecte les soustractions de pointeurs sur des zones mémoire différentes.
	```c
	void test() {
		int a, b, c[10], d[10];
		int x = &c[3] - &c[1];
		x = &d[4] - &c[1]; // warn
	}
	```

- **security.PutenvStackArray (C)**  
	Détecte les appels à putenv() avec un tableau alloué sur la pile.
	```c
	int f() {
		char env[] = "NAME=value";
		return putenv(env); // warn
	}
	```

- **security.SetgidSetuidOrder (C)**  
	Vérifie l'ordre correct des appels à setgid et setuid lors de la révocation des privilèges.
	```c
	void test1() {
		if (setuid(getuid()) != 0) {
			handle_error();
			return;
		}
		if (setgid(getgid()) != 0) { // warning
			handle_error();
			return;
		}
	}
	```

## 1.1.8. unix

- **unix.API (C)**  
	Vérifie les appels à diverses fonctions UNIX/POSIX.
	```c
	void test(const char *path) {
		int fd = open(path, O_CREAT);
			// warn: call to 'open' requires a third argument when the 'O_CREAT' flag is set
	}
	```

- **unix.Malloc (C)**  
	Détecte les fuites mémoire, double free et use-after-free avec malloc/free.
	```c
	void test() {
		int *p = malloc(1);
		free(p);
		free(p); // warn
	}
	```

- **unix.MallocSizeof (C)**  
	Détecte les arguments douteux à malloc impliquant sizeof.
	```c
	void test() {
		long *p = malloc(sizeof(short)); // warn
		free(p);
	}
	```

- **unix.MismatchedDeallocator (C, C++)**  
	Détecte les désallocations non appariées (delete/free).
	```c
	void test() {
		int *p = (int *)malloc(sizeof(int));
		delete p; // warn
	}
	```

- **unix.cstring.BadSizeArg (C)**  
	Vérifie la taille passée aux fonctions de manipulation de chaînes C.
	```c
	void test() {
		char dest[3];
		strncat(dest, "*", sizeof(dest)); // warn
	}
	```

- **unix.cstring.NotNullTerminated (C)**  
	Vérifie que les arguments sont bien des chaînes null-terminées.
	```c
	void test1() {
		int l = strlen((char *)&test1); // warn
	}
	```

- **unix.cstring.NullArg (C)**  
	Vérifie les pointeurs nuls passés aux fonctions de chaînes C.
	```c
	int test() {
		return strlen(0); // warn
	}
	```

- **unix.StdCLibraryFunctions (C)**  
	Vérifie les appels aux fonctions standard C qui violent les contraintes d'arguments prédéfinies.
	```c
	void test_alnum_concrete(int v) {
		int ret = isalnum(256); // warning
	}
	```

- **unix.Stream (C)**  
	Vérifie la gestion correcte des flux C (fopen, fclose, etc.).
	```c
	void test1() {
		FILE *p = fopen("foo", "r");
	} // warn: opened file is never closed
	```

---

Pour plus de détails, voir la [documentation officielle](https://clang.llvm.org/docs/analyzer/checkers.html#default-checkers).
