# RSA - 3- Writeup

## Challenge Overview

In this challenge, an unconventional RSA setup is used that "entangles" the prime factors among three different moduli. The parameters are constructed as follows:

- **n₁ = p · q**
- **n₂ = K₁ · p · r**
- **n₃ = K₂ · r · s**

Here, *p*, *q*, *r*, and *s* are large primes, while *K₁* and *K₂* are small prime constants (for example, 3 and 5). The public exponent is **e = 65537**.

The flag (formatted as `tarang{...}`) is encrypted through three layers:
1. Compute m₁ = (flag)ᵉ mod n₁.
2. Compute m₂ = (m₁)ᵉ mod n₂.
3. Compute c = (m₂)ᵉ mod n₃.

You are provided with the values of n₁, n₂, n₃, K₁, K₂, e, and the final ciphertext c. The challenge is to reverse this multi-layered encryption by carefully exploiting the relationships between the moduli.

## Approach

1. **Recover p:**  
   Since n₁ = p · q and n₂ = K₁ · p · r, compute p as gcd(n₁, n₂).

2. **Factor n₁:**  
   With p known, q can be obtained as q = n₁ / p.

3. **Recover r:**  
   Knowing n₂ and p, compute r = n₂ / (K₁ · p).

4. **Factor n₃:**  
   Given n₃ = K₂ · r · s and the recovered r, obtain s = n₃ / (K₂ · r).

5. **Compute Euler’s Totients:**  
   - φ(n₁) = (p – 1) · (q – 1)
   - φ(n₂) = (K₁ – 1) · (p – 1) · (r – 1)
   - φ(n₃) = (K₂ – 1) · (r – 1) · (s – 1)

6. **Determine Private Exponents:**  
   For each modulus, calculate the private exponent:
   - d₁ = inverse(e, φ(n₁))
   - d₂ = inverse(e, φ(n₂))
   - d₃ = inverse(e, φ(n₃))

7. **Layered Decryption:**  
   Reverse the encryption layers:
   - Compute m₂ = c^(d₃) mod n₃.
   - Compute m₁ = m₂^(d₂) mod n₂.
   - Compute m₀ = m₁^(d₁) mod n₁.
   - Convert m₀ from an integer to bytes to reveal the flag.

## Solution

```python
from Crypto.Util.number import inverse, long_to_bytes
import math

n1  = 54626907649670364532084564524281589816803562094924069168884326115008447189498970169034885532606173081906251235177462497071879931331564177298834751734829475438261752248844512024582627115623717872936064767827395535930896975320927132129270924978098419084012688544397622689479512730561358570020861797396200138809
n2 = 153834065775178295825952009955039799457967602823880901377454172818760467728926990892782698749614306675015824705837370633975704713021674435884374655558327515327686964850554490639691754991421517410408320393747979183697746308655218107455649718030684307276729225097067925779115707829754353674613068633044120715099
n3 = 406269481776474234515061301555857336826671188246682464699461371654715032129558031398000505478333811101402881994011933062348880476815994992624769015353157292296665892161555472973500954380478050568773016486727491208717216066451246938715112200151294011455263764977775208595512953400607663683247037079807676898585
K1 = 3
K2 = 5
e = 65537
c  = 248957704074507076045084306698061572170856911758170377539312700194312418748850806939582167184083790586650765675364000995052894118450142118285188304881245913982557552660609408762532881805309632812424797364328386611425364623526986763054703854290355637170446255226812537390883665723455065143797534738049398152839

p = math.gcd(n1, n2)
print("Recovered p =", p)
q = n1 // p
print("Recovered q =", q)
r = n2 // (K1 * p)
print("Recovered r =", r)
s = n3 // (K2 * r)
print("Recovered s =", s)
phi_n1 = (p - 1) * (q - 1)
phi_n2 = (K1 - 1) * (p - 1) * (r - 1)
phi_n3 = (K2 - 1) * (r - 1) * (s - 1)
d1 = inverse(e, phi_n1)
d2 = inverse(e, phi_n2)
d3 = inverse(e, phi_n3)
m2 = pow(c, d3, n3)
m1 = pow(m2, d2, n2)
m0 = pow(m1, d1, n1)
flag = long_to_bytes(m0)
print("Recovered flag:", flag.decode())
```
##Flag

- **tarang{il0vevikas}**
