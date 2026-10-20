## Feel It: Write-up

**Category:** Forensics / Audio  
**Flag:** `0SSEC{f33l_th3_m4agic_in_th3_4ir}`

### Challenge Overview

The challenge provides a single file, `main.wav`. Listening to it gives nothing useful, only a faint buzz. The title hints at the approach: don’t listen, inspect the samples as numbers.

### Step 1: Read the WAV file

python

```python
from scipy.io import wavfile

rate, data = wavfile.read("main.wav")
print(rate, len(data), data.min(), data.max())
```

- `rate` is the sample rate (8000 Hz) and `data` holds 26,400 samples, about 3.3 seconds.
- Every value is positive and falls roughly between 1000 and 9000. Real audio swings positive and negative, so this file is clearly artificial.

### Step 2: Plot the samples

python

```python
import matplotlib.pyplot as plt

plt.plot(data)
plt.show()
```

The plot is a staircase. The signal holds at one noisy level, then jumps to another. Each level lasts the same number of samples, and the levels sit about 500 apart.

### Step 3: Find the block size

Integer-divide the samples by 500 to remove the noise, then measure how long each level lasts:

python

```python
from itertools import groupby
from math import gcd
from functools import reduce

levels = [int(y) // 500 for y in data]
runs = [len(list(g)) for _, g in groupby(levels)]
print(reduce(gcd, runs))
```

This prints `400`. If two equal values appear in a row, their blocks merge into one longer run, so the gcd of all run lengths gives the real block size: **400 samples per value**.

### Step 4: Decode each block

Taking one sample per block, we reverse the encoding. Each level is `(value + 2) * 500` plus noise smaller than 500, so integer division by 500 recovers the value and subtracting 2 undoes the offset:

python

```python
nibbles = [int(y) // 500 - 2 for y in data[::400]]
```

This gives 66 numbers, each between 0 and 15. These are hexadecimal digits.

### Step 5: Convert to text

python

```python
import string

h = "".join(string.hexdigits[n] for n in nibbles)
print(bytes.fromhex(h).decode())
```

- `string.hexdigits` maps each number to its hex character.
- `bytes.fromhex()` pairs the characters into bytes.
- `.decode()` turns the bytes into readable text.

### Full solver

python

```python
from scipy.io import wavfile
import string

BLOCK = 400

rate, data = wavfile.read("main.wav")

nibbles = [int(y) // 500 - 2 for y in data[::BLOCK]]
h = "".join(string.hexdigits[n] for n in nibbles)
print(bytes.fromhex(h).decode())
```

### Flag

```
0SSEC{f33l_th3_m4agic_in_th3_4ir}
```

### Conclusion

The flag is converted to hex, and each hex digit is stored as an audio level held for 400 samples, with random noise added inside a 500-wide band so the signal looks natural. Integer division strips the noise and recovers each digit exactly. The key insight is to analyze the audio as data, not as sound.