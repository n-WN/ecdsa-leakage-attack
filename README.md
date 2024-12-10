# ECDSA Nonce Leakage Lattice-based Attack

This repository contains Python source code for attacking ECDSA with nonce leakage using lattice-based algorithms. The specific algorithm is detailed in the paper:

** Yiming Gao, Jinghui Wang, Honggang Hu and Binang He, Attacking ECDSA with Nonce Leakage by Lattice Sieving: Bridging the Gap with Fourier Analysis-based Attacks. ** 

In this [paper](https://eprint.iacr.org/2024/296), we aim to give a solution to an open question: Can lattice-based attacks be enhanced by utilizing more samples? Using this repository, we can break 160-bit ECDSA with 1-bit leakage using approximately $2^{25}$ ECDSA samples. In addition, our new algorithms
for solving the HNP are extended to address the case of erroneous input, increasing the robustness of lattice-based attacks.

> Multiple leaks can also use this Repo, thanks to the work of the original authors

## Version Information

- sage 10.4 (Python 3.12.4 | ARM)
- fplll 5.4.5
- fpylll 0.6.0
- gmpy2 2.2.0a1

## Quick Start

> Why use Sage because it integrates fplll, fpylll

``` shell
sage solveECDSAfromMSB.py -n 256 -s 4 -m1 $leak_count -m4 0 -t 16 -f1 lines.txt -f2 leak.txt -curve secp256k1 2
```

### input files format

#### `lines.txt`

```
[
    "e51cc9ac2d8bb2bcb5ad072d0bee5dfe97c061a657f6d1a9a2712a9fd48b5f6e f3d30f851d496a8b041c650c215520fb6cede263990dd7d07518f70ea740978ceeaa4eae256b288389a39ff5d13e9d775c9ded4515daeeb0775913f2f8418402",
    "e51cc9ac2d8bb2bcb5ad072d0bee5dfe97c061a657f6d1a9a2712a9fd48b5f6e 7b8aaa0a1e04e804b3e6fa964a160a0c5531ea5e197c891aa36a4a08f65be8b199e7b4303aa53b7a0d76bfd6eaf612439707b4f915d9428c6bd3043c716c3416",
    
    ...

    "e51cc9ac2d8bb2bcb5ad072d0bee5dfe97c061a657f6d1a9a2712a9fd48b5f6e 515456c53c9e3333326aa1f643c238dc882d90d6cadf4e7125a98f55a878f3bc829c40e3f1d42387c0fb99693a02c872c1020570299ac0ae4974540f7922e84a"
]

```

---

#### `leak.txt`

``` python
[1, 1, 1, 1, 1, 1, 0, 1, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 0, 0, 1, 1, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 0, 0, 0, 0, 1, 0, 0, 1, 0, 1, 0, 1, 1, 0, 0, 0, 1, 0, 1, 1, 1, 0, 1, 1, 1, 1, 0, 1, 0, 0, 1, 1, 1, 0, 0, 1]

```

## Fix

- Fix the bug in the `ecdsa-leakage-attack/HNPSolver.py` file. The error message is as follows:

``` python
Traceback (most recent call last):
  File "/ecdsa-leakage-attack/solveECDSAfromMSB.py", line 51, in <module>
    recovered_sk = Solver.recoverKey()
                   ^^^^^^^^^^^^^^^^^^^
  File "/ecdsa-leakage-attack/ECDSASolver.py", line 215, in recoverKey
    Solver.solve("eliminateAlpha")
  File "/ecdsa-leakage-attack/HNPSolver.py", line 184, in solve
    self.constructLatticeEliminateAlpha()
  File "/ecdsa-leakage-attack/HNPSolver.py", line 131, in constructLatticeEliminateAlpha
    self.lattice[i, i] = self.HNP_instance.q
    ~~~~~~~~~~~~^^^^^^
  File "src/fpylll/fplll/integer_matrix.pyx", line 960, in fpylll.fplll.integer_matrix.IntegerMatrix.__setitem__
  File "src/fpylll/fplll/integer_matrix.pyx", line 917, in fpylll.fplll.integer_matrix.IntegerMatrix._set
  File "src/fpylll/io.pyx", line 35, in fpylll.io.assign_Z_NR_mpz
  File "src/fpylll/io.pyx", line 62, in fpylll.io.assign_mpz
NotImplementedError: Type '<class 'gmpy2.mpz'>' not supported
```

### Diff

[self.lattice[i, i] #L131](https://github.com/JinghuiWW/ecdsa-leakage-attack/blob/a1eaf2627b6502cbfa0e4094d0f3b3384d363724/HNPSolver.py#L131)

[self.lattice[i, i] #L149](https://github.com/JinghuiWW/ecdsa-leakage-attack/blob/a1eaf2627b6502cbfa0e4094d0f3b3384d363724/HNPSolver.py#L149)

``` diff
-       self.lattice[i, i] = self.HNP_instance.q
+       self.lattice[i, i] = int(self.HNP_instance.q)
```

## Key Recovery of ECDSA with Nonce Leakage
You can perform the attack on an instance using multiple CPU cores. For example:
``` shell
python solveECDSAfromLSB.py -n 256 -s 4 -m 65 -t 4
```
This command will solve an ECDSA (256, 4) instance using 4 CPU threads.
- -n specifies the bit-size of modulus.
- -s indicates the leakage.
- -m is the number of samples for constructing lattice.
- -t defines the number of CPU threads to use.

To accelerate the attack using GPUs, specify the number of GPUs with the -g option. For detailed information on all parameters, run:
``` shell
python solveECDSAfromLSB.py -h
```

Another example, for solving an ECDSA(128, 1) instance, run:
 ``` shell
python solveECDSAfromLSB.py -n 128 -s 1 -m 117 -x 15 -t 24 -g 2 -f 0 -f1 "Instances/128_1/lines.txt" -f2 "Instances/128_1/lsb.txt" -f3 "Instances/128_1/sk.txt"
```

You can also perform attacks on public datasets such as [minerva](https://github.com/crocs-muni/minerva/tree/master/data). Note that the dataset indicates the number of leading-zero bits (i.e., most significant bits) of the nonce. For example:
``` shell
python solveECDSAfromMSB.py -n 256 -s 3 -m1 90 -m4 512 -t 16 -f1 "minerva-data/athena/256_3/lines.txt" -f2 "minerva-data/athena/256_3/msb.txt"
```
This command will solve an ECDSA instance from the Athena dataset, where:

- -n specifies the bit-size of modulus.
- -s indicates the leakage.
- -m1 is the number of samples for constructing lattice.
- -m4 is the number of samples for the linear predicate.
- -t defines the number of CPU threads to use.
- -f1 provides the path to the file containing the lines (including data and public key (r, s)).
- -f2 the path to the file containing the MSBs.

For convience, we provide two bash scripts, autoRunECDSALSB.sh and autoRunECDSAMSB.sh. Depending on the target of the attack, you could modify the parameters in the corresponding script file.

To execute the attack for the LSB situation, use the following command:

``` shell
bash autoRunECDSALSB.sh
```

To execute the attack for the MSB situation, use the following command:

``` shell
bash autoRunECDSAMSB.sh
```

## Environment
To conduct the attacks, ensure the following environment is properly set up on your machine:
- [FPLLL](https://github.com/fplll/fplll) and [FPyLLL](https://github.com/fplll/fpylll) for data structures and BKZ algorithm.
- [G6K](https://github.com/fplll/g6k) for lattice sieving.
- [G6K-GPU-Tensor](https://github.com/WvanWoerden/G6K-GPU-Tensor) for lattice sieving with GPUs accelerating.


## New Records of Lattice-based Attacks against ECDSA
We achieved several new records of lattice-based attacks against ECDSA. For reproducibility, the successfully broken instances have been stored in the Instances folder. These attacks were conducted using an Intel Xeon Platinum 8480+ CPU and four GeForce RTX 4090 GPUs. Specific details about time and memory consumption can be seen in the following table: 

**4-bit leakage**
| **Curve**        | **Leakage** | **d**   | **x**     | **Expected Sample Size** | **Wall time** | **Mem GiB** |
|------------------|-------------|---------|-----------|-------------|---------------|-------------|
| brainpoolp512r1   | 4           | 130     | 0         | $2^{10}$    | 96min         | 254         |

**1-bit leakage**

| **Curve**        | **Leakage** | **d**   | **x**     | **Expected Sample Size** | **Wall time** | **Mem GiB** |
|------------------|-------------|---------|-----------|-------------|---------------|-------------|
| secp128r1        | 1           | 131     | 0         | $2^8$       | 72min         | 294         |
| secp128r1        | 1           | 118     | 15  | $2^{26}$    | 8min          | 53          |
| secp160r1        | 1           | 144     | 14  | $2^{25}$    | 824min        | 1939        |
| secp160r1        | 1           | 138     | 25  | $2^{36}$    | 279min        | 850         |

**less than 1-bit leakage**

| **Curve**        | **Error rate** | **d**   | **x**       | **Expected Sample Size** | **Wall time** | **Mem GiB** |
|------------------|----------------|---------|-------------|-------------|---------------|-------------|
|secp128r1 |  0.1            | 140     | 20    | $2^{31}$    | 370min        | 1090        |
| secp160r1        | 0.02           | 144     | 14    | $2^{25}$    | 1009min       | 1960        |

Note that the definition of $x$ in this implementation differs slightly from that in the paper. Specifically, $2^{x}$ in the implementation corresponds to $x$ in the paper.

## Acknowledgements
This work was supported by National Natural Science Foundation of China (Grant No. 62472397) and Innovation Program for Quantum Science and Technology (Grant No. 2021ZD0302902).
