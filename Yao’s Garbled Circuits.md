# Yao's Garbled Circuits

### Introduction
The fundamental problem in secure multiparty computation is to compute a function $F(x_1,x_2,\ldots)$ where the input $x_i$ is known only to party $P_i$, in a way that no party should learn anything that they couldn't have deduced from their own input and the value of $F(x_1, x_2, \ldots)$.
For now, we assume there are only two parties $P_1, P_2$.

1.  Assume both $P_1$ and $P_2$ mutually agree upon a circuit $C$ that represents $F$.
2. Without loss of generality, assume circuit $C$ only contains XOR, NOT, and AND gates.
3.  P1 has access to a random oracle or a pseudorandom function.

### Setup
Here, $P_1$ generates a garbled circuit and sends it to $P_2$ to compute.
Suppose that the circuit $C$ consisted of a single logic gate $G$. $G(x,y)$ is a boolean function of the type $Bool * Bool \rightarrow Bool$. Since a boolean variable can only take 2 values, the domain of $G$ has size 4. We can just represent $G$ as a lookup table of size 4. For example, the lookup table of an AND gate would be:


| x | y | AND(x,y) |
| -------- | -------- | -------- |
| 0     | 0     | 0     |
| 0     | 1     | 0     |
| 1     | 0     | 0     |
| 1     | 1     | 1     |


In fact, this is exactly what $P_1$ does for the NOT, XOR, and AND gates. It creates a lookup table. 
The next step is to 'garble' the circuit. For each possible value of $x$ and $y$, $v_x$ and $v_y$, $P_1$ generates random keys $k_x^0$, $k_x^1$, $k_y^0$,  and $k_y^1$. Then it encrypts the values $G(v_x, v_y)$ with both $k_y^{v_y}$ and $k_x^{v_x}$ such that it can only be decrypted with both keys but not with any one. Thus, the new table looks as follows:



| x | y | Enc(AND(x,y)) |
| -------- | -------- | -------- |
| 0     | 0     | $Enc_{k_x^{0}, k_y^{0}}(0)$     |
| 0     | 1     | $Enc_{k_x^{0}, k_y^{1}}(0)$      |
| 1     | 0     | $Enc_{k_x^{1}, k_y^{0}}(0)$      |
| 1     | 1     | $Enc_{k_x^{1}, k_y^{1}}(1)$      |


Keep in mind that every possible output of $G$ is encrypted with a different pair of keys. Now, to compute $G$ together, $P_1$ and $P_2$ must know both the keys without knowing each other's inputs. 
This is accomplished in the following manner:
1. $P_1$ sends the last column of the table (containing only the garbled values) to $P_2$.
2. Since $P_1$ knows the value of $x$, she sends $k_x^{v_x}$ to $P_2$.
3. $P_2$ obtains $k_y^{v_y}$ by Oblivious Transfer from $P_1$. Thus, $P_1$ doesn't learn $v_y$.
4. $P_2$ can now use both $k_x^{v_x}$ and $k_y^{v_y}$ to decrypt the corresponding entry in the table.

A subtle point here is that $P_2$ must know which entry to decrypt and decryption must not reveal the $v_x$. To accomplish  the latter, $P_1$ permutes the table. This can be done by generating random tags for each entry in the table and then sorting the table by the random tags. For the former, we ensure that the last bit of each key is a pointer to the permuted table telling $P_2$ which entry to decrypt. This method is called point-and-permute.


### Composing Gates

The circuit $C$ which represents $F$ will typically contain more than one gate. If we recursively apply the same procedure above to each layer of the circuit $C$, then the intermediate values of the circuit will be revealed to both parties, which undermines the security of Yao's protocol. Thus, the secure computation of these gates must be composed in a way that preserves the secrecy of intermediate outputs. Yao's protocol accomplishes this in a very clever way. 

1. For every wire $w_i$ in $C$, $P_1$ creates keys for both values that $w_i$ can take.
2. For every gate $G\epsilon C$, with input wires $x,y$ and output wire $w$, the permuted lookup table now stores $Enc_{k_x, k_y}(k_w)$.
3. $P_2$ computes each input gate as explained above, using Oblivious Transfer.
4. For any intermediate gate, $P_2$ knows both the keys (but not the values) because it has decrypted the keys from the lookup table.
5. $P_2$ recursively computes every subsequent gate/layer in the circuit until it has obtained the key of the output wire.
6. $P_1$ shares a lookup table with $P_2$ which maps the keys of the output wire to the corresponding values. This table could also be shared in the initial phase of the protocol to avoid another round of communication.

### Security
Yao's protocol is secure in the semi-honest model.
$P_1$ never communicates with $P_2$ after sending the garbled circuit and output decoding table. The only communication that happens besides this is the oblivious transfer, which is already known to be secure. Even if $P_2$ is corrupt, the protocol is secure because $P_2$ never sees 2 keys for the same wire. Hence, the protocol is secure.


### Analysis
Yao's protocol runs in constant rounds but the number of Oblivious transfers required is linear in the number of input gates in the circuit. Thus, the communication complexity can be high.


