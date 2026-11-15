# Arrays: Full Mastery from Scratch to GATE CS Level

>  Legend & Color Coding Used in Notes:
>
> * **Red** ($\color{red}{\text{Red}}$): Critical pitfalls, GATE traps, and memory bounds.
>
> * **Green** ($\color{green}{\text{Green}}$): Base addresses, optimal complexities, and invariants.
>
> * **Blue** ($\color{blue}{\text{Blue}}$): Indices, offsets, and row coordinates.
>
> * **Orange** ($\color{orange}{\text{Orange}}$): Word size (element width in bytes) and column coordinates.

## 1. Fundamentals & Memory Architecture (From Scratch)

### 1.1 What is an Array?

An **Array** is a collection of **homogeneous** (same data type) elements stored in **contiguous** (adjacent and uninterrupted) memory locations, referenced by a common name and accessed via an index.

* **Homogeneity:** Every element occupies the exact same number of bytes, denoted as $\color{orange}{W}$ (Word size).

* **Contiguity:** If the first element is at byte address $\color{green}{B}$, the next is strictly at $\color{green}{B} + \color{orange}{W}$, with zero fragmentation between elements.

### 1.2 Visual Memory Layout (1D Array)

Suppose we declare `int A[5] = {10, 20, 30, 40, 50};` where $\color{green}{\text{Base Address } (B) = 1000}$ and $\color{orange}{\text{sizeof(int) } (W) = 4 \text{ bytes}}$.

```
Index (i):         0          1          2          3          4
              +----------+----------+----------+----------+----------+
Array (A):    |    10    |    20    |    30    |    40    |    50    |
              +----------+----------+----------+----------+----------+
Byte Address:  1000..1003 1004..1007 1008..1011 1012..1015 1016..1019
               ^          ^          ^          ^          ^
               B         B+1W       B+2W       B+3W       B+4W

```

### 1.3 Why is Array Access $O(1)$ (Random Access)?

Unlike a Linked List where you must traverse $i$ pointers to reach the $i^{\text{th}}$ node ($O(i)$ time), an array computes the exact physical memory address of `A[i]` using **one multiplication and one addition** in the CPU's Arithmetic Logic Unit (ALU):

$$
\text{Address}(A[\color{blue}{i}]) = \color{green}{B} + \color{blue}{i} \times \color{orange}{W}
$$

Because basic arithmetic on fixed-width integers (e.g., 64-bit registers) executes in a constant number of CPU clock cycles regardless of the value of $\color{blue}{i}$, reading or writing `A[i]` takes $\color{green}{\Theta(1)}$ **time**.

### 1.4 Hardware Synergy: Spatial Locality & Cache Lines

When the CPU requests `A[0]`, the memory controller does not fetch just $4$ bytes from RAM. Instead, it fetches an entire **Cache Line** (typically $64$ bytes) into the L1 Cache.

* Since an `int` is $4$ bytes, a single $64$-byte cache line loads `A[0]` through `A[15]` simultaneously!

* Consequently, sequential traversal of an array produces **1 Cache Miss** followed by **15 Cache Hits**, making arrays vastly faster in practice than pointer-based structures.

### 1.5 Dynamic Arrays & Amortized $O(1)$ Insertion Proof

Static arrays have a fixed size. **Dynamic Arrays** (`std::vector` in C++, `ArrayList` in Java) grow automatically by allocating a new array of **double the capacity** ($2N$), copying existing elements over, and freeing the old array.

#### Theorem: Inserting $n$ elements into an initially empty Dynamic Array using the doubling strategy takes $\color{green}{O(n)}$ total time, or $\color{green}{O(1)}$ **amortized time** per insertion.

#### Proof (Aggregate Method):

Let the initial capacity be $1$. A resize (copying elements) occurs only when the current insertion index $i$ exceeds a power of $2$ (i.e., when capacity increases from $1 \rightarrow 2 \rightarrow 4 \rightarrow 8 \dots \rightarrow 2^k$ where $2^k < n \le 2^{k+1}$).

The cost $c_i$ of the $i^{\text{th}}$ insertion is:

$$
c_i = \begin{cases} \color{red}{i} & \text{if } i - 1 \text{ is a power of } 2 \text{ (i.e., } i - 1 \text{ copies} + 1 \text{ insertion)} \\ \color{green}{1} & \text{otherwise (just } 1 \text{ direct write)} \end{cases}
$$

Summing the cost over $n$ insertions:

$$
\text{Total Cost } T(n) = \sum_{i=1}^{n} c_i \le \underbrace{\color{green}{n}}_{\text{actual writes}} + \underbrace{\color{red}{\sum_{j=0}^{\lfloor \log_2(n-1) \rfloor} 2^j}}_{\text{resize copy costs}}
$$

Using the geometric series sum $\sum_{j=0}^{k} 2^j = 2^{k+1} - 1$ where $k = \lfloor \log_2(n-1) \rfloor \implies 2^k \le n - 1$:

$$
T(n) \le \color{green}{n} + \color{red}{2(n - 1) - 1} = 3n - 3 < 3n
$$

$$
\text{Amortized Cost per Insertion} = \frac{T(n)}{n} \le \frac{3n}{n} = \color{green}{3 = O(1)} \quad \blacksquare
$$

> **GATE Trap:** What if we increase the capacity by a **constant** $C$ (e.g., $+10$ slots each time) instead of doubling?
> Then resize costs are $C + 2C + 3C + \dots + \lfloor n/C \rfloor C = \Theta(n^2)$ total time, which degrades to $\color{red}{\Theta(n)}$ amortized time per insertion!

## 2. 1D Array Addressing & C Pointer Arithmetic

### 2.1 Generalized 1D Array Addressing (Arbitrary Bounds)

In languages like Pascal, Fortran, or theoretical GATE questions, an array index does not necessarily start at $0$. Let an array be declared as $A[\color{red}{LB} \dots \color{red}{UB}]$, where:

* $\color{red}{LB}$ = Lower Bound (starting index)

* $\color{red}{UB}$ = Upper Bound (ending index)

#### 1. Number of Elements ($N$):

$$
N = \color{red}{UB} - \color{red}{LB} + 1
$$


*(Note: Subtracting bounds excludes the starting element, so we add* $1$ *to include both endpoints).*

#### 2. Address of Element $A[\color{blue}{i}]$:

To reach index $\color{blue}{i}$, we must skip $(\color{blue}{i} - \color{red}{LB})$ elements before it. Each element occupies $\color{orange}{W}$ bytes:

$$
\text{Address}(A[\color{blue}{i}]) = \color{green}{B} + (\color{blue}{i} - \color{red}{LB}) \times \color{orange}{W}
$$

### 2.2 Deep Dive: C Arrays & Pointer Arithmetic (High-Yield GATE Topic)

In C, an array name acts as a constant pointer to its first element in almost all expressions, **except** when used with the `sizeof` operator or the address-of `&` operator.

Consider the declaration: `int A[5] = {10, 20, 30, 40, 50};` at base address $\color{green}{1000}$, with `sizeof(int) = 4`.

```
graph TD
    subgraph MemoryBlock ["Contiguous Memory Block (1000 to 1019)"]
        E0["A[0] = 10<br/>Addr: 1000"]
        E1["A[1] = 20<br/>Addr: 1004"]
        E2["A[2] = 30<br/>Addr: 1008"]
        E3["A[3] = 40<br/>Addr: 1012"]
        E4["A[4] = 50<br/>Addr: 1016"]
    end
    P1["A or &A[0]<br/>(Type: int*)"] -->|Points to 4-byte int| E0
    P2["A + 1<br/>(1000 + 1*4 = 1004)"] -->|Steps by 4 bytes| E1
    P3["&A<br/>(Type: int (*)[5])"] -->|Points to whole 20-byte array| MemoryBlock
    P4["&A + 1<br/>(1000 + 1*20 = 1020)"] -->|Steps by 20 bytes| After["Addr: 1020<br/>(Just past array)"]

    style P1 fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    style P2 fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    style P3 fill:#fef3c7,stroke:#d97706,color:#78350f
    style P4 fill:#fee2e2,stroke:#dc2626,color:#7f1d1d

```

#### Critical Comparison Table: `A` vs `&A`

| Expression | Data Type | Numerical Value | Size of Pointed Object | Result of `+ 1` | 
| ----- | ----- | ----- | ----- | ----- | 
| `A` | `int *` | $\color{green}{1000}$ | $4$ bytes (`sizeof(int)`) | $\color{green}{1000} + \color{orange}{4} = \color{blue}{1004}$ | 
| `&A[0]` | `int *` | $\color{green}{1000}$ | $4$ bytes (`sizeof(int)`) | $\color{green}{1000} + \color{orange}{4} = \color{blue}{1004}$ | 
| `&A` | `int (*)[5]` | $\color{green}{1000}$ | $5 \times 4 = 20$ bytes | $\color{green}{1000} + \color{red}{20} = \color{red}{1020}$ | 
| `sizeof(A)` | `size_t` | $20$ | N/A (Total array bytes) | $21$ | 

> **Commutativity of Subscript Operator:**
> By definition in the C standard, `A[i]` is syntactic sugar for `*(A + i)`.
> Since addition is commutative, `*(A + i)` $\equiv$ `*(i + A)` $\equiv$ `i[A]`.
> Therefore, in C, **`A[3]` and `3[A]` evaluate to the exact same element!**

## 3. Multi-Dimensional Arrays & Mathematical Proofs

Physical RAM is strictly **1-Dimensional** (a linear sequence of byte addresses). To store a 2D matrix $A[M \times N]$ in 1D memory, we must linearize it using one of two canonical schemes:

1. **Row-Major Order (RMO):** Used by C, C++, Java, Python (NumPy default). Elements are stored row-by-row.

2. **Column-Major Order (CMO):** Used by Fortran, MATLAB, R, Julia. Elements are stored column-by-column.

```
Logical 2D Matrix (3x3):
      Col 0  Col 1  Col 2
Row 0 [ a00    a01    a02 ]
Row 1 [ a10    a11    a12 ]
Row 2 [ a20    a21    a22 ]

1D Memory in Row-Major Order (RMO):
+-----+-----+-----+-----+-----+-----+-----+-----+-----+
| a00 | a01 | a02 | a10 | a11 | a12 | a20 | a21 | a22 |
+-----+-----+-----+-----+-----+-----+-----+-----+-----+
| <--  Row 0  --> | <--  Row 1  --> | <--  Row 2  --> |

1D Memory in Column-Major Order (CMO):
+-----+-----+-----+-----+-----+-----+-----+-----+-----+
| a00 | a10 | a20 | a01 | a11 | a21 | a02 | a12 | a22 |
+-----+-----+-----+-----+-----+-----+-----+-----+-----+
| <--  Col 0  --> | <--  Col 1  --> | <--  Col 2  --> |

```

### 3.1 Rigorous Proof of 2D Array Addressing

Let a 2D array be declared with arbitrary bounds:

$$
A[\color{red}{L_1} \dots \color{red}{U_1}][\color{red}{L_2} \dots \color{red}{U_2}]
$$

* Number of Rows ($M$) = $\color{red}{U_1} - \color{red}{L_1} + 1$

* Number of Columns ($N$) = $\color{red}{U_2} - \color{red}{L_2} + 1$

* Base Address = $\color{green}{B}$, Element Size = $\color{orange}{W}$ bytes.

* We want the address of element $A[\color{blue}{i}][\color{orange}{j}]$, where $\color{red}{L_1} \le \color{blue}{i} \le \color{red}{U_1}$ and $\color{red}{L_2} \le \color{orange}{j} \le \color{red}{U_2}$.

#### Case 1: Row-Major Order (RMO) Derivation

In RMO, to reach element $A[\color{blue}{i}][\color{orange}{j}]$, we must completely skip:

1. All full rows before row $\color{blue}{i}$, which are rows $\color{red}{L_1}, \color{red}{L_1}+1, \dots, \color{blue}{i}-1$.

   * Number of full rows skipped = $(\color{blue}{i} - \color{red}{L_1})$.

   * Since each row contains $N = (\color{red}{U_2} - \color{red}{L_2} + 1)$ elements, total elements skipped in prior rows = $(\color{blue}{i} - \color{red}{L_1}) \times N$.

2. Inside the target row $\color{blue}{i}$, we must skip elements in columns $\color{red}{L_2}, \color{red}{L_2}+1, \dots, \color{orange}{j}-1$.

   * Number of elements skipped in row $\color{blue}{i}$ = $(\color{orange}{j} - \color{red}{L_2})$.

$$
\text{Total Elements Skipped} = (\color{blue}{i} - \color{red}{L_1}) \cdot N + (\color{orange}{j} - \color{red}{L_2})
$$

Multiplying by $\color{orange}{W}$ bytes per element and adding Base Address $\color{green}{B}$:

$$
\boxed{\text{Addr}_{\text{RMO}}(A[\color{blue}{i}][\color{orange}{j}]) = \color{green}{B} + \Big[ (\color{blue}{i} - \color{red}{L_1}) \cdot (\color{red}{U_2} - \color{red}{L_2} + 1) + (\color{orange}{j} - \color{red}{L_2}) \Big] \times \color{orange}{W}} \quad \blacksquare
$$

*(For* $0$*-based indexing where* $\color{red}{L_1} = \color{red}{L_2} = 0$*:* $\text{Addr}_{\text{RMO}}(A[\color{blue}{i}][\color{orange}{j}]) = \color{green}{B} + (\color{blue}{i} \cdot N + \color{orange}{j}) \times \color{orange}{W}$*).*

#### Case 2: Column-Major Order (CMO) Derivation

In CMO, memory is laid out column-by-column. To reach $A[\color{blue}{i}][\color{orange}{j}]$, we must completely skip:

1. All full columns before column $\color{orange}{j}$, which are columns $\color{red}{L_2}, \color{red}{L_2}+1, \dots, \color{orange}{j}-1$.

   * Number of full columns skipped = $(\color{orange}{j} - \color{red}{L_2})$.

   * Since each column contains $M = (\color{red}{U_1} - \color{red}{L_1} + 1)$ elements, total elements skipped in prior columns = $(\color{orange}{j} - \color{red}{L_2}) \times M$.

2. Inside the target column $\color{orange}{j}$, we must skip elements in rows $\color{red}{L_1}, \color{red}{L_1}+1, \dots, \color{blue}{i}-1$.

   * Number of elements skipped in column $\color{orange}{j}$ = $(\color{blue}{i} - \color{red}{L_1})$.

$$
\boxed{\text{Addr}_{\text{CMO}}(A[\color{blue}{i}][\color{orange}{j}]) = \color{green}{B} + \Big[ (\color{orange}{j} - \color{red}{L_2}) \cdot (\color{red}{U_1} - \color{red}{L_1} + 1) + (\color{blue}{i} - \color{red}{L_1}) \Big] \times \color{orange}{W}} \quad \blacksquare
$$

*(For* $0$*-based indexing where* $\color{red}{L_1} = \color{red}{L_2} = 0$*:* $\text{Addr}_{\text{CMO}}(A[\color{blue}{i}][\color{orange}{j}]) = \color{green}{B} + (\color{orange}{j} \cdot M + \color{blue}{i}) \times \color{orange}{W}$*).*

### 3.2 3D Array Addressing ($A[D_1][D_2][D_3]$)

Let a 3D array have dimensions $N_1 \times N_2 \times N_3$ with $0$-based offsets $e_1 = (\color{blue}{i} - L_1)$, $e_2 = (\color{orange}{j} - L_2)$, $e_3 = (\color{purple}{k} - L_3)$:

* **Row-Major Order (Rightmost index varies fastest):**
  

  $$
  \text{Addr}_{\text{RMO}}(A[\color{blue}{i}][\color{orange}{j}][\color{purple}{k}]) = \color{green}{B} + \Big[ (\color{blue}{e_1} \cdot N_2 + \color{orange}{e_2}) \cdot N_3 + \color{purple}{e_3} \Big] \times \color{orange}{W}
  $$

* **Column-Major Order (Leftmost index varies fastest):**
  

  $$
  \text{Addr}_{\text{CMO}}(A[\color{blue}{i}][\color{orange}{j}][\color{purple}{k}]) = \color{green}{B} + \Big[ (\color{purple}{e_3} \cdot N_2 + \color{orange}{e_2}) \cdot N_1 + \color{blue}{e_1} \Big] \times \color{orange}{W}
  $$

### 3.3 2D Array Pointer Hierarchy in C

Consider `int A[3][4];` with Base Address $\color{green}{1000}$ and `sizeof(int) = 4`.
In C, a 2D array is strictly a **1D array of 1D arrays**: `A` contains $3$ elements, where each element `A[i]` is itself a $4$-element integer array (`int[4]`, size $16$ bytes).

```
Level 0 (Pointer to whole 3x4 matrix: int (*)[3][4], step = 48B):
  &A  --------------------------------------------------------+
                                                              |
Level 1 (Pointer to 1D row of 4 ints: int (*)[4], step = 16B):|
  A or &A[0] ---> [ Row 0 (16B) ] [ Row 1 (16B) ] [ Row 2 (16B) ]
                  1000            1016            1032
                  |
Level 2 (Pointer to single int: int *, step = 4B):
  *A or A[0] or &A[0][0] ---> [1000][1004][1008][1012]
                                |
Level 3 (Actual integer value: int):
  **A or *A[0] or A[0][0] ---> Value at 1000

```

#### Golden Equivalence Rules for 2D Arrays in C:

1. `A[i]` $\equiv$ `*(A + i)` $\rightarrow$ Address of the start of row $i$ (Type: `int *`, value: $\color{green}{B} + 16i$).

2. **`*A` and `A` have the exact same numerical address (**$\color{green}{1000}$**), NOT the value at** $1000$**!** Why? Because `A` is a pointer to Row 0 (`int (*)[4]`), so dereferencing it once (`*A` $\equiv$ `A[0]`) produces the 1D array representing Row 0, which decays into an `int *` pointing to `A[0][0]`. You must dereference **twice** (`**A`) to get the integer `A[0][0]`.

3. `A[i][j]` $\equiv$ `*(A[i] + j)` $\equiv$ `*(*(A + i) + j)`.

## 4. Special Matrices & Space Optimization (GATE Favorite)

When a square matrix $A[n \times n]$ has a large proportion of zero elements in a predictable pattern, storing all $n^2$ elements wastes memory. Instead, we pack only the **non-zero** elements into a **1D Array** $M[\dots]$ and derive a mathematical mapping function $f(\color{blue}{i}, \color{orange}{j})$ to locate $A[\color{blue}{i}][\color{orange}{j}]$ in $M$.

*(Assume* $1$***-based indexing*** $1 \le \color{blue}{i}, \color{orange}{j} \le n$ *for the derivations below, as standard in GATE questions, stored in a* $0$*-indexed 1D array* $M$ *with base address* $\color{green}{B}$*).*

### 4.1 Lower Triangular Matrix (LTM)

An $n \times n$ matrix where $A[\color{blue}{i}][\color{orange}{j}] = 0$ for all $\color{blue}{i} < \color{orange}{j}$ (all elements strictly above the main diagonal are zero).

```
      j=1   j=2   j=3   j=4
i=1 [ a11    0     0     0  ]  --> 1 non-zero element
i=2 [ a21   a22    0     0  ]  --> 2 non-zero elements
i=3 [ a31   a32   a33    0  ]  --> 3 non-zero elements
i=4 [ a41   a42   a43   a44 ]  --> 4 non-zero elements

```

* **Total Non-Zero Elements:** $1 + 2 + 3 + \dots + n = \color{green}{\frac{n(n+1)}{2}}$

* **Zero Elements Saved:** $n^2 - \frac{n(n+1)}{2} = \frac{n(n-1)}{2}$

#### Proof 1: LTM in Row-Major Order (RMO)

To find the index of $A[\color{blue}{i}][\color{orange}{j}]$ (where $\color{blue}{i} \ge \color{orange}{j}$) in 1D array $M$:

1. Count non-zero elements in rows $1$ through $\color{blue}{i}-1$:
   Row $1$ has $1$, Row $2$ has $2$, $\dots$, Row $(\color{blue}{i}-1)$ has $(\color{blue}{i}-1)$ elements.
   

   $$
   \text{Elements in prior rows} = \sum_{r=1}^{\color{blue}{i}-1} r = \frac{(\color{blue}{i}-1)\color{blue}{i}}{2}
   $$

2. Count elements before $A[\color{blue}{i}][\color{orange}{j}]$ in Row $\color{blue}{i}$ (columns $1$ to $\color{orange}{j}-1$):
   

   $$
   \text{Elements in row } \color{blue}{i} = \color{orange}{j} - 1
   $$

$$
\boxed{\text{Addr}_{\text{LTM-RMO}}(A[\color{blue}{i}][\color{orange}{j}]) = \color{green}{B} + \left[ \frac{\color{blue}{i}(\color{blue}{i}-1)}{2} + (\color{orange}{j} - 1) \right] \times \color{orange}{W}} \quad \blacksquare
$$

#### Proof 2: LTM in Column-Major Order (CMO)

Look at the columns of the LTM diagram above:

* Column $1$ has $n$ elements ($a_{11}$ to $a_{n1}$).

* Column $2$ has $n-1$ elements ($a_{22}$ to $a_{n2}$).

* Column $c$ has $(n - c + 1)$ elements.

To reach $A[\color{blue}{i}][\color{orange}{j}]$ (where $\color{blue}{i} \ge \color{orange}{j}$):

1. Elements in prior columns $1$ through $\color{orange}{j}-1$:
   

   $$
   \sum_{c=1}^{\color{orange}{j}-1} (n - c + 1) = n(\color{orange}{j}-1) - \frac{(\color{orange}{j}-1)(\color{orange}{j}-2)}{2}
   $$

2. In column $\color{orange}{j}$, non-zero elements start at row $\color{orange}{j}$ ($a_{\color{orange}{j},\color{orange}{j}}$). So to reach row $\color{blue}{i}$, we skip $(\color{blue}{i} - \color{orange}{j})$ elements.

$$
\boxed{\text{Addr}_{\text{LTM-CMO}}(A[\color{blue}{i}][\color{orange}{j}]) = \color{green}{B} + \left[ n(\color{orange}{j}-1) - \frac{(\color{orange}{j}-1)(\color{orange}{j}-2)}{2} + (\color{blue}{i} - \color{orange}{j}) \right] \times \color{orange}{W}} \quad \blacksquare
$$

### 4.2 Upper Triangular Matrix (UTM)

An $n \times n$ matrix where $A[\color{blue}{i}][\color{orange}{j}] = 0$ for all $\color{blue}{i} > \color{orange}{j}$ (all elements strictly below the main diagonal are zero). Notice that **UTM is the transpose of LTM**: swapping $\color{blue}{i} \leftrightarrow \color{orange}{j}$ and RMO $\leftrightarrow$ CMO transforms LTM formulas directly into UTM formulas!

* **UTM in Row-Major Order (for** $\color{blue}{i} \le \color{orange}{j}$**):**
  

  $$
  \boxed{\text{Addr}_{\text{UTM-RMO}}(A[\color{blue}{i}][\color{orange}{j}]) = \color{green}{B} + \left[ n(\color{blue}{i}-1) - \frac{(\color{blue}{i}-1)(\color{blue}{i}-2)}{2} + (\color{orange}{j} - \color{blue}{i}) \right] \times \color{orange}{W}}
  $$

* **UTM in Column-Major Order (for** $\color{blue}{i} \le \color{orange}{j}$**):**
  

  $$
  \boxed{\text{Addr}_{\text{UTM-CMO}}(A[\color{blue}{i}][\color{orange}{j}]) = \color{green}{B} + \left[ \frac{\color{orange}{j}(\color{orange}{j}-1)}{2} + (\color{blue}{i} - 1) \right] \times \color{orange}{W}}
  $$

### 4.3 Tridiagonal Matrix

A square matrix where non-zero elements exist **only** on the main diagonal ($\color{blue}{i} = \color{orange}{j}$), the sub-diagonal ($\color{blue}{i} = \color{orange}{j} + 1$), and the super-diagonal ($\color{blue}{i} = \color{orange}{j} - 1$). That is, $A[\color{blue}{i}][\color{orange}{j}] \ne 0 \iff |\color{blue}{i} - \color{orange}{j}| \le 1$.

```
      j=1   j=2   j=3   j=4
i=1 [ a11   a12    0     0  ]  --> 2 elements
i=2 [ a21   a22   a23    0  ]  --> 3 elements
i=3 [  0    a32   a33   a34 ]  --> 3 elements
i=4 [  0     0    a43   a44 ]  --> 2 elements

```

* **Total Non-Zero Elements:** $n \text{ (main)} + (n-1) \text{ (super)} + (n-1) \text{ (sub)} = \color{green}{3n - 2}$.

* **RMO Formula Derivation (**$1$**-based):**

  * Row $1$ has $2$ elements.

  * Rows $2$ to $\color{blue}{i}-1$ (total $\color{blue}{i}-2$ rows) each have $3$ elements $\implies 2 + 3(\color{blue}{i}-2) = 3\color{blue}{i} - 4$ elements before row $\color{blue}{i}$.

  * In row $\color{blue}{i}$, the first non-zero element is at column $\color{blue}{i}-1$. So to reach column $\color{orange}{j}$, we skip $\color{orange}{j} - (\color{blue}{i} - 1) = \color{orange}{j} - \color{blue}{i} + 1$ elements.

  * Total index ($0$-based offset): $(3\color{blue}{i} - 4) + (\color{orange}{j} - \color{blue}{i} + 1) = \color{green}{2\color{blue}{i} + \color{orange}{j} - 3}$.
    

    $$
    \boxed{\text{Addr}_{\text{Tridiag-RMO}}(A[\color{blue}{i}][\color{orange}{j}]) = \color{green}{B} + (2\color{blue}{i} + \color{orange}{j} - 3) \times \color{orange}{W}}
    $$

## 5. Algorithmic Paradigms on Arrays (DSA & GATE)

### 5.1 Prefix Sum & Difference Arrays

1. **Prefix Sum Array (**$P$**):** Precomputes cumulative sums in $O(n)$ so any range sum query $\text{Sum}(L, R)$ answers in $\color{green}{O(1)}$ time:
   

   $$
   P[i] = \sum_{k=0}^{i} A[k] \implies \text{Sum}(L, R) = P[R] - P[L-1]
   $$

2. **Difference Array (**$D$**):** Dual of Prefix Sum. Supports adding $X$ to all elements in range $[L, R]$ in $\color{green}{O(1)}$ time per update!

   * Define $D[0] = A[0]$ and $D[i] = A[i] - A[i-1]$.

   * Notice that the **Prefix Sum of** $D$ **reconstructs** $A$: $\sum_{k=0}^{i} D[k] = A[i]$.

   * **Range Update** $\text{Add}(L, R, X)$**:** Simply execute `D[L] += X` and `D[R + 1] -= X`. When taking the prefix sum at the end, $+X$ propagates from index $L$ to $R$, and $-X$ cancels it from $R+1$ onward!

### 5.2 Kadane's Algorithm (Maximum Subarray Sum) & Proof

**Problem:** Given an array $A[0 \dots n-1]$ containing positive and negative numbers, find a contiguous subarray with the maximum possible sum in $\color{green}{O(n)}$ time and $\color{green}{O(1)}$ space.

#### Recurrence Relation (Dynamic Programming State):

Let $S[i]$ be the maximum subarray sum **ending strictly at index** $i$.
To form a contiguous subarray ending at $i$, we have only two choices:

1. Extend the best subarray ending at $i-1$ by adding $A[i]$ (giving $S[i-1] + A[i]$).

2. Start a fresh subarray at index $i$ alone (giving $A[i]$).

$$
S[i] = \max\big(\color{blue}{A[i]}, \; \color{green}{S[i-1]} + \color{blue}{A[i]}\big)
$$

```
flowchart LR
    Start["Check S[i-1]"] --> Cond{"Is S[i-1] < 0?"}
    Cond -- "Yes (Dragging sum down)" --> Reset["Start Fresh at i:<br/>S[i] = A[i]"]
    Cond -- "No (Helping or neutral)" --> Extend["Extend Previous:<br/>S[i] = S[i-1] + A[i]"]
    Reset --> UpdateMax["GlobalMax = max(GlobalMax, S[i])"]
    Extend --> UpdateMax

    style Reset fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    style Extend fill:#dcfce7,stroke:#16a34a,color:#14532d
    style UpdateMax fill:#dbeafe,stroke:#2563eb,color:#1e3a8a

```

#### Correctness Proof by Loop Invariant:

* **Loop Invariant:** At the end of iteration $i$, `curr_sum` holds $S[i]$ (the maximum sum of any subarray ending at $i$), and `max_so_far` holds $\max_{0 \le k \le i} S[k]$.

* **Base Case (**$i = 0$**):** Only one subarray ends at $0$, which is $A[0 \dots 0]$. Both `curr_sum` and `max_so_far` are initialized to $A[0]$. Invariant holds.

* **Inductive Step:** Suppose the invariant holds at $i - 1$. Any contiguous subarray ending at $i$ is either of length $1$ (just $A[i]$) or length $\ge 2$ (some subarray ending at $i-1$ followed by $A[i]$). By the inductive hypothesis, the maximum sum of any subarray ending at $i-1$ is $S[i-1]$. Since adding $A[i]$ preserves the inequality $X \le S[i-1] \implies X + A[i] \le S[i-1] + A[i]$, the best subarray ending at $i$ must have sum $\max(A[i], S[i-1] + A[i]) = S[i]$. Updating `max_so_far = max(max_so_far, S[i])` maintains the global maximum across all endpoints $0 \dots i$. $\blacksquare$

### 5.3 Boyer-Moore Majority Vote Algorithm & Cancellation Proof

**Problem:** Find the element that appears **strictly more than** $\lfloor n/2 \rfloor$ **times** in $A[0 \dots n-1]$ in $\color{green}{O(n)}$ time and $\color{green}{O(1)}$ auxiliary space.

#### Algorithm Intuition:

Maintain a `candidate` and a `count = 0`. Traverse the array:

* If `count == 0`, set `candidate = A[i]` and `count = 1`.

* Else if `A[i] == candidate`, increment `count++`.

* Else (`A[i] != candidate`), decrement `count--` (pair up and **cancel** one instance of `candidate` with one distinct element).

#### Proof of Correctness:

Suppose a majority element $M$ exists, occurring $k > \lfloor n/2 \rfloor$ times. All non-$M$ elements combined occur $n - k < n/2$ times.

* Every time `count` is decremented, **two distinct elements** are paired and eliminated from consideration.

* Even in the worst case where every single non-$M$ element pairs up against an instance of $M$, at most $(n - k)$ instances of $M$ can be cancelled.

* Alternatively, if two non-$M$ elements cancel each other, even fewer instances of $M$ are lost.

* Therefore, at least $k - (n - k) = 2k - n > 2(n/2) - n = \color{green}{0}$ instances of $M$ remain uncancelled at the end, guaranteeing `candidate == M`! $\blacksquare$

> **GATE Trap:** If the problem does **not** guarantee that a majority element exists, you **must** perform a second $O(n)$ pass to count the frequency of `candidate` and verify that $\text{freq} > \lfloor n/2 \rfloor$.

### 5.4 Array Rotation by $d$ Positions (Reversal Algorithm)

To left-rotate $A[0 \dots n-1]$ by $d$ positions in $\color{green}{O(n)}$ time and $\color{green}{O(1)}$ space without extra buffers:
Let the array be partitioned into two blocks $A = \color{blue}{X}\color{orange}{Y}$, where $\color{blue}{X} = A[0 \dots d-1]$ and $\color{orange}{Y} = A[d \dots n-1]$. We want to transform $\color{blue}{X}\color{orange}{Y} \rightarrow \color{orange}{Y}\color{blue}{X}$.

Using string reversal properties, notice that:

$$
\left( \color{blue}{X}^R \color{orange}{Y}^R \right)^R = (\color{orange}{Y}^R)^R (\color{blue}{X}^R)^R = \color{orange}{Y}\color{blue}{X}
$$

1. Reverse $\color{blue}{X}$ ($A[0 \dots d-1]$): takes $d/2$ swaps.

2. Reverse $\color{orange}{Y}$ ($A[d \dots n-1]$): takes $(n-d)/2$ swaps.

3. Reverse the entire array ($A[0 \dots n-1]$): takes $n/2$ swaps.

* **Total Swaps:** $\frac{d}{2} + \frac{n-d}{2} + \frac{n}{2} = \color{green}{n \text{ swaps}} = \Theta(n)$ time and $O(1)$ space!

## 6. Classic GATE-Level Solved Problems & Derivations

### Problem 1: RMO vs. CMO Fixed-Point Elements

**Question:** Consider a 2D array $A[0 \dots M-1][0 \dots N-1]$. For which elements $A[\color{blue}{i}][\color{orange}{j}]$ is the memory address in **Row-Major Order** identical to the memory address in **Column-Major Order**?

#### Solution & Derivation:

Equate the $0$-based RMO and CMO address formulas:

$$
\text{Addr}_{\text{RMO}}(A[\color{blue}{i}][\color{orange}{j}]) = \text{Addr}_{\text{CMO}}(A[\color{blue}{i}][\color{orange}{j}])
$$

$$
\color{green}{B} + (\color{blue}{i} \cdot N + \color{orange}{j}) \times \color{orange}{W} = \color{green}{B} + (\color{orange}{j} \cdot M + \color{blue}{i}) \times \color{orange}{W}
$$

Subtract $\color{green}{B}$ and divide by $\color{orange}{W} > 0$:

$$
\color{blue}{i} \cdot N + \color{orange}{j} = \color{orange}{j} \cdot M + \color{blue}{i}
$$

$$
\boxed{\color{blue}{i}(N - 1) = \color{orange}{j}(M - 1)}
$$

* **Special Case 1 (Square Matrix** $M = N$**):**
  Since $M - 1 = N - 1$, dividing both sides gives $\color{blue}{i} = \color{orange}{j}$. Thus, for any square matrix ($M = N > 1$), **all** $N$ **main diagonal elements** ($A[0][0], A[1][1], \dots, A[N-1][N-1]$) have the exact same address in RMO and CMO!

* **Special Case 2 (Rectangular Matrix** $M \ne N$**):**
  

  $$
  \frac{\color{blue}{i}}{\color{orange}{j}} = \frac{M - 1}{N - 1}
  $$

  
  The total number of solutions for $0 \le \color{blue}{i} \le M-1$ and $0 \le \color{orange}{j} \le N-1$ is given by:
  

  $$
  \text{Number of Fixed-Point Elements} = \color{green}{\gcd(M - 1, \; N - 1) + 1}
  $$

  
  *(Always includes at least two elements: the first element* $A[0][0]$ *and the last element* $A[M-1][N-1]$*).*

### Problem 2: Optimal Simultaneous Minimum and Maximum

**Question:** Given an unsorted array $A[1 \dots n]$, what is the minimum number of element-to-element comparisons required to find **both** the minimum and maximum elements?

#### Proof of the $\lceil 3n/2 \rceil - 2$ Tournament Pair Method:

* **Naive Approach:** Find $\min$ in $n - 1$ comparisons, then find $\max$ among the remaining $n - 1$ elements in $n - 2$ comparisons. Total = $2n - 3$ comparisons.

* **Optimal Pairwise Approach:** Process elements in **pairs** $(A[2k], A[2k+1])$:

  1. Compare the two elements in the pair against each other ($1$ **comparison**).

  2. Compare the **smaller** of the pair with `current_min` ($1$ **comparison**).

  3. Compare the **larger** of the pair with `current_max` ($1$ **comparison**).

  * This processes $2$ **elements using only** $3$ **comparisons** (rate of $1.5$ comparisons per element)!

#### Exact Count Breakdown:

* **If** $n$ **is Even:**

  * Initialize `min` and `max` by comparing $A[1]$ and $A[2]$: $1$ **comparison**.

  * Remaining $(n - 2)$ elements form $\frac{n - 2}{2}$ pairs, each costing $3$ comparisons:
    

    $$
    \text{Total Comparisons} = 1 + 3\left(\frac{n - 2}{2}\right) = \color{green}{\frac{3n}{2} - 2}
    $$

* **If** $n$ **is Odd:**

  * Initialize `min = max = A[1]`: $0$ **comparisons**.

  * Remaining $(n - 1)$ elements form $\frac{n - 1}{2}$ pairs, each costing $3$ comparisons:
    

    $$
    \text{Total Comparisons} = 3\left(\frac{n - 1}{2}\right) = \color{green}{\frac{3n - 3}{2}}
    $$

* Unified formula for all $n \ge 1$: $\boxed{\left\lceil \frac{3n}{2} \right\rceil - 2}$.

### Problem 3: GATE C Pointer & 2D Array Dereferencing Trap

**Question:** What is the output of the following C program?

```
#include <stdio.h>
int main() {
    int a[3][3] = {{1, 2, 3}, {4, 5, 6}, {7, 8, 9}};
    int *p = (int *)(&a + 1);
    printf("%d, %d\n", **(a + 1), *(p - 1));
    return 0;
}

```

#### Step-by-Step Trace:

1. **`**(a + 1)`:**

   * `a` points to Row $0$ (`{1, 2, 3}`).

   * `a + 1` points to Row $1$ (`{4, 5, 6}`).

   * `*(a + 1)` is `a[1]`, which decays to a pointer to `a[1][0]` (address of `4`).

   * `**(a + 1)` dereferences that pointer, yielding **`4`**.

2. **`*(p - 1)`:**

   * `&a` has type `int (*)[3][3]` (pointer to the entire $3 \times 3$ matrix of $9$ integers).

   * `&a + 1` jumps past the entire $3 \times 3$ array, pointing immediately after the last element `a[2][2]` (`9`).

   * Cast to `int *` and stored in `p`, `p - 1` steps backward by **one `int` (**$4$ **bytes)**, landing directly on `a[2][2]`.

   * Dereferencing `*(p - 1)` yields **`9`**.

* **Output:** `4, 9`

## 7. Master Revision Cheat Sheet

| Concept / Matrix Type | Condition / Structure | Total Elements | 1D Index Formula ($0$-based offset from $\color{green}{B}$) | 
| ----- | ----- | ----- | ----- | 
| **1D Array** $A[LB \dots UB]$ | Linear | $N = UB - LB + 1$ | $(\color{blue}{i} - LB)$ | 
| **2D Array (RMO)** | Row-by-Row | $M \times N$ | $(\color{blue}{i} - L_1) \cdot N + (\color{orange}{j} - L_2)$ | 
| **2D Array (CMO)** | Col-by-Col | $M \times N$ | $(\color{orange}{j} - L_2) \cdot M + (\color{blue}{i} - L_1)$ | 
| **Lower Triangular (RMO)** | $A[\color{blue}{i}][\color{orange}{j}] = 0$ if $\color{blue}{i} < \color{orange}{j}$ ($1$-based) | $\frac{n(n+1)}{2}$ | $\frac{\color{blue}{i}(\color{blue}{i}-1)}{2} + (\color{orange}{j} - 1)$ | 
| **Lower Triangular (CMO)** | $A[\color{blue}{i}][\color{orange}{j}] = 0$ if $\color{blue}{i} < \color{orange}{j}$ ($1$-based) | $\frac{n(n+1)}{2}$ | $n(\color{orange}{j}-1) - \frac{(\color{orange}{j}-1)(\color{orange}{j}-2)}{2} + (\color{blue}{i} - \color{orange}{j})$ | 
| **Upper Triangular (RMO)** | $A[\color{blue}{i}][\color{orange}{j}] = 0$ if $\color{blue}{i} > \color{orange}{j}$ ($1$-based) | $\frac{n(n+1)}{2}$ | $n(\color{blue}{i}-1) - \frac{(\color{blue}{i}-1)(\color{blue}{i}-2)}{2} + (\color{orange}{j} - \color{blue}{i})$ | 
| **Upper Triangular (CMO)** | $A[\color{blue}{i}][\color{orange}{j}] = 0$ if $\color{blue}{i} > \color{orange}{j}$ ($1$-based) | $\frac{n(n+1)}{2}$ | $\frac{\color{orange}{j}(\color{orange}{j}-1)}{2} + (\color{blue}{i} - 1)$ | 
| **Symmetric Matrix** | $A[\color{blue}{i}][\color{orange}{j}] = A[\color{orange}{j}][\color{blue}{i}]$ | $\frac{n(n+1)}{2}$ | Store as LTM; if $\color{blue}{i} < \color{orange}{j}$, look up $A[\color{orange}{j}][\color{blue}{i}]$ | 
| **Tridiagonal (RMO)** | $A[\color{blue}{i}][\color{orange}{j}] = 0$ if $\vert{}\color{blue}{i}-\color{orange}{j}\vert{} > 1$ | $3n - 2$ | $2\color{blue}{i} + \color{orange}{j} - 3$ *(for* $1$*-based* $\color{blue}{i}, \color{orange}{j}$*)* | 

### Time & Space Complexity Summary

| Operation / Algorithm | Static Array | Dynamic Array | Optimal Paradigm / Notes | 
| ----- | ----- | ----- | ----- | 
| **Random Access `A[i]`** | $\color{green}{\Theta(1)}$ | $\color{green}{\Theta(1)}$ | Direct ALU address calculation | 
| **Insert / Delete at End** | N/A | $\color{green}{\Theta(1)}$ amortized | Worst-case $\color{red}{O(n)}$ on resize | 
| **Insert / Delete at Index** $i$ | $\color{red}{\Theta(n - i)}$ | $\color{red}{\Theta(n - i)}$ | Requires shifting elements | 
| **Range Sum Query** ($Q$ queries) | $\color{red}{O(n)}$ / query | $\color{green}{O(1)}$ / query | With $O(n)$ Prefix Sum preprocessing | 
| **Range Add Update** ($U$ updates) | $\color{red}{O(n)}$ / update | $\color{green}{O(1)}$ / update | With Difference Array $D[L] \mathrel{+}= X, D[R+1] \mathrel{-}= X$ | 
| **Max Subarray Sum** | $\color{green}{\Theta(n)}$ time | $\color{green}{\Theta(1)}$ space | Kadane's DP Invariant | 
| **Simultaneous Min & Max** | $\color{green}{\lceil 3n/2 \rceil - 2}$ | $\color{green}{\Theta(1)}$ space | Tournament Pair Comparison | 

