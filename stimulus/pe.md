# Stimulus Based Testing of the PE

## Unit Tests

https://github.com/StanfordAHA/CGRAGenerator/tree/master/tests/test_pe

Currently most of the tests are on the `more-unit-tests` branch
https://github.com/StanfordAHA/CGRAGenerator/tree/more-unit-tests/tests/test_pe

To units are currently under test, organized/named by their Verilog module
* `test_pe_comp_unq1` - the PE compute unit
* `test_pe_unq1` - the PE tile

### Compute Unit
https://github.com/StanfordAHA/CGRAGenerator/blob/more-unit-tests/tests/test_pe/test_compute_unit.py
#### Tested operations
1. `or`
2. `and`
3. `xor`
4. `lshr`
5. `lshl`
6. `add`
7. `sub`
8. `mul0`
9. `mul1`
10. `mul2`
11. `abs`
12. `sel`
13. `ge`
14. `le`

#### Testing Strategies
* complete
  ```python
  width = 4
  N = 1 << width
  ("op_a", range(0, N) if not signed else range(- N // 2, N // 2)),
  ("op_b", range(0, N) if not signed else range(- N // 2, N // 2)),
  ("op_d_p", range(0, 2))
  ```
* random
  ```python
  num_tests = 256
  width = 16
  N = 1 << width
  ("op_a", lambda : randint(0, N) if not signed else randint(- N // 2, N // 2 - 1)),
  ("op_b", lambda : randint(0, N) if not signed else randint(- N // 2, N // 2 - 1)),
  ("op_d_p", lambda : randint(0, 1))
  ```

### PE Tile
https://github.com/StanfordAHA/CGRAGenerator/blob/more-unit-tests/tests/test_pe/test_pe.py
#### Tested compute unit operations
1. `or`
2. `and`
3. `xor`
4. `lshr`
5. `lshl`
6. `add`
7. `sub`
8. `mul0`
9. `mul1`
10. `mul2`
11. `abs`
12. `sel`
13. `ge`
14. `le`

#### Other tested entities
1. LUT
2. Input modes (CONST, BYPASS, DELAY, VALID)
3. 1-bit Flags
4. IRQ mode

#### In progress
1. acc-en mode

#### Testing Strategies
* complete
  ```python
  width = 4
  N = 1 << width
  ("op_a", range(0, N) if not signed else range(- N // 2, N // 2)),
  ("op_b", range(0, N) if not signed else range(- N // 2, N // 2)),
  ("op_d_p", range(0, 2))
  ```
* random
  ```python
  num_tests = 256
  width = 16
  N = 1 << width
  ("op_a", lambda : randint(0, N) if not signed else randint(- N // 2, N // 2 - 1)),
  ("op_b", lambda : randint(0, N) if not signed else randint(- N // 2, N // 2 - 1)),
  ("op_d_p", lambda : randint(0, 1))
  ```

#### Testing Strategies
* complete
  ```python
  width = 4
  N = 1 << width
  ("op_a", range(0, N) if not signed else range(- N // 2, N // 2)),
  ("op_b", range(0, N) if not signed else range(- N // 2, N // 2)),
  ("op_d_p", range(0, 2))
  ```
* random
  ```python
  num_tests = 256
  width = 16
  N = 1 << width
  ("op_a", lambda : randint(0, N) if not signed else randint(- N // 2, N // 2 - 1)),
  ("op_b", lambda : randint(0, N) if not signed else randint(- N // 2, N // 2 - 1)),
  ("op_d_p", lambda : randint(0, 1))
  ```

**TODO: ** List strategies for testing LUT, IRQ, Register Input Modes, and acc_en
