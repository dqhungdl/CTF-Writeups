# BogoAttack
## Statement
The problem contains the source code only.
```
import random
NUMS = list(range(10**4))
random.shuffle(NUMS)
tries = 15
while True:
    try:
        n = int(input('Enter (1) to steal and (2) to guess: '))
        if n == 1:
            if tries==0:
                print('You ran out of tries. Bye!')
                break
            l = map(int,input('Enter numbers to steal: ').split(' '))
            output = []
            for i in l:
                assert 0<= i < len(NUMS)
                output.append(NUMS[i])
            random.shuffle(output)
            print('Stolen:',output)
            tries-=1
        elif n == 2:
            l = list(map(int,input('What is the list: ').split(' ')))
            if l == NUMS:
                print(open('flag.txt','r').read())
                break
            else:
                print('NOPE')
                break
        else:
            print('Not a choice.')
    except:
        print('Error. Nice Try...')

```
After analyzing the source code, I shortly describe the problem below.
## Short description
Given an array of **10000 elements** from 0-9999 that is random shuffled (or we can call that is a permutation). Our target is finding out that array. If we answer the correct array, the server will return the flag. But we can ask the server **at most 15 queries** with the following format:
```
x1 x2 ... xk
```
The server will response the values of all asked positions `x1, x2,...xk` and again, those values are shuffled, so we cannot know the correct positions of those returned values.

## Solution
So, how can we use only 15 queries to find out all the values of an array? I will give you an easy example.
Assume that, we only need to find an array of `8` elements, for example: `A = [4, 3, 2, 5, 7, 0, 6, 1]`.
Then, I just need to ask 3 questions to find out all the elements:
```
Query 0: 1 3 5 7 (Bitmask at position 0 is equal 1) --> The set of responsed value are (0, 1, 3, 5)
Query 1: 2 3 5 6 (Bitmask at position 1 is equal 1) --> The set of responsed value are (1, 2, 5, 6)
Query 2: 4 5 6 7 (Bitmask at position 2 is equal 1) --> The set of responsed value are (0, 1, 6, 7)
```
After that, I can find the position for each value, for example:
* For value `0`, it appears in `Query 0` and `Query 2`, so the position of value `0` is `2^0 + 2^2 = 5`.
* For value `1`, it appears in `Query 0`, `Query 1` and `Query 2`, so the position of value `1` is `2^0 + 2^1 + 2^2 = 7`.
* For value `2`, it appears in `Query 1`, so the position of value `2` is `2^1 = 2`.
* For value `3`, it appears in `Query 0`, so the position of value `3` is `2^0 = 1`.
* For value `4`, it doesn't appear in any query, so the position of value `4` is `0`.
* For value `5`, it appears in `Query 0` and `Query 1`, so the position of value `5` is `2^0 + 2^1 = 3`.
* For value `6`, it appears in `Query 1` and `Query 2`, so the position of value `6` is `2^1 + 2^2 = 6`.
* For value `7`, it appears in `Query 2`, so the position of value `7` is `2^2 = 4`.

Here is the general algorithm to solve with 10000 elements:
* Step 1: Prepare **13 queries** (because numbers in `[0-9999]` has at most 13 bits without leading zeros in the binary form), where the i-th query contains all the positions with the i-th bit is turned on.
* Step 2: For the i-th query, if the value `x` appears, add `2^i` to the array `P[x]`. After 13 queries, `P[x]` is the position of value `x`.
* Step 3: Send the answer to the server and receive the flag.

## Code
Here is the code, this code can be used for both **BogoAttack** and **BogoSolve** problems.
```
from pwn import *

# Connect to the remote server
r = remote('ctf.k3rn3l4rmy.com', 2247)


# Get the k-th bit of number x
def get_bit(x, k):
    return (x >> k) & 1


# Parse the array receives from the server
def get_array(receive):
    receive = receive.split(b'[')[1].split(b']')[0]
    receive = receive.split(b', ')
    receive = [int(value) for value in receive]
    return receive


# Ask 13 queries
P = [0] * 10000  # P[i] = Position for the value i in the original array
for i in range(14):
    print(f'Query #{i}')
    query = []

    # Prepare i-th query
    for mask in range(10000):
        if get_bit(mask, i):
            query.append(mask)

    # Ask the server and receive set of values
    r.sendline(b'1')
    r.recv()
    r.sendline(' '.join(str(value) for value in query).encode())
    ans = get_array(r.recvline())

    # Calculate position
    for value in ans:
        P[value] += 1 << i

# Answer and receive the flag
rs = [0] * 10000
for i in range(len(P)):
    rs[P[i]] = i
r.sendline(b'2')
r.recv()
r.sendline(' '.join(str(value) for value in rs).encode())
print(r.recvline())

```

**The flag is `flag{m0d1f13d_b1n4ry_s34rch!}`**
