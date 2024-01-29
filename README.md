# Toy_Kyber
Post-Quantum Cryptography

## Part 1
Consider a toy-version of the Kyber Post-Quantum Public-Key Cryptosystem that uses polynomials from
Z97/(X4+1). That is the set of polynomials mod X4+1 with coefficients mod 97. Using naive polynomial
multiplication, implement the toy cryptosystem in C. Start by adding some macro definitions in your code:
//toy.h:
//toy Post-Quantum Public-Key Cryptosystem
#define TK_K 3
#define TK_N 4
//toy.c:
#define TK_Q 97
a. Implement some static helper functions in toy.c, for example:
//polynomial multiplication in Z97[X]/(X^4+1)
static void toy_polmul_naive(
short *dst,
const short *a,
const short *b,
int add_to_dst //if true: dst += a*b; if false: dst = a*b;
);
Pseudocode to multiply polynomials in Zq[X]/(X^4+1):
dst[0] = (a[0]*b[0] - a[3]*b[1] - a[2]*b[2] - a[1]*b[3]) mod q
dst[1] = (a[1]*b[0] + a[0]*b[1] - a[3]*b[2] - a[2]*b[3]) mod q
dst[2] = (a[2]*b[0] + a[1]*b[1] + a[0]*b[2] - a[3]*b[3]) mod q
dst[3] = (a[3]*b[0] + a[2]*b[1] + a[1]*b[2] + a[0]*b[3]) mod q
There is no need to implement the Number Theoretic Transform just yet. Implement other helper
functions if needed.

b. Then implement the main API:
void toy_gen(short *A, short *t, short *s);
void toy_enc(const short *A, const short *t, int plain, short *u, short *v);
int toy_dec(const short *s, const short *u, const short *v);

Where:
plain: Contains 4 bits.
A: Is a K*K matrix of polynomials with N coefficients.
t, s, u: Are K-vectors of polynomials with N coefficients.
v: Is one polynomial with N coefficients.
All stored consequently in memory as flat arrays.
Pseudocode for the main operations:
Gen():
Fill K*K-matrix A with uniformly random numbers mod q
Fill K-vectors s & e with small random numbers mod q, defined as
val = rand()&3 //Uniform distribution
buffer[i] = (val&1) – (val>>1&1) //Binomial distribution
t = A.s + e //matrix-vector multiplication in Zq[X]/(X^n+1)
(A, t) is the public key
While (s) is the private key
Enc(PU=(A, t), plain):
Fill K-vectors r & e1, and the scalar (one-polynomial) e2 with small
random numbers mod q
u = A_transpose . r + e1
msg_bits[i] = plain>>i&1
v = t dot r + e2 + msg_bits * q/2
(u, v) is the ciphertext
Dec(PR=s, c=(u, v)):
p = v – s dot u
plain=0
for i=0 to n:
val = p[i]
if(val>q/2)
val -= q
bit = abs(val)>q/4
plain |= bit<<i
return plain
