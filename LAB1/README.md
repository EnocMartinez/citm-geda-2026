# GeDa Lab 1 — Variables, Lists, Functions & Files

---

**Course**: GeDa - Gestió de Dades: Comunicacions, Programació i Simulació  
**Program**: Ciències i Tecnologies del Mar  
**Author**: Enoc Martínez  
**Department**: Departament d'Enginyeria Electrònica (EEL)  
**Contact**: enoc.martinez@upc.edu

<p align="center">
  <img height="100" src="https://github.com/EnocMartinez/citm-geda-2026/blob/main/resources/banner.png?raw=true" alt="infographic">
</p>

---

## Introduction

Welcome to your first hands-on session in GeDa. By the end of this lab you will have written a program that reads a real CTD profile from a file, computes the speed of sound at every depth, saves the result, and plots it — and you will be able to point at the **SOFAR channel** in your own figure.

We build it one piece at a time. Each task adds exactly one new idea, and each one exists because the previous task made you want it.

Where you see `____`, that is a blank for you to fill in. 📝 marks a question to answer in your report.

### Objectives

* Variables, basic types and arithmetic in Python 3
* Lists, `for` loops and `if` conditionals
* Writing and reusing your own functions
* Reading and writing text files
* Understanding why the speed of sound in the ocean is not constant

### What to hand in

Deliver to Atenea the following files:

* A report in PDF format with snapshots of your code explaining every task.
* In some tasks there are questions, marked with 📝, that need to be answered in your report.
* The report must include an `AI usage` section describing which tools you used and why.
* The final Python script containing all the tasks in a single file named `LAB1.py`.

### Getting the files

You need `ctd_profile.csv` from this folder. Either clone the whole repository:

```bash
git clone https://github.com/EnocMartinez/citm-geda-2026.git
cd citm-geda-2026/LAB1
```

or download the single file from the GitHub web page (open it, then click **Download raw file**).

> ⚠️ Keep `LAB1.py` and `ctd_profile.csv` **in the same folder**, otherwise Python will not find the data file.

---

## Setup — before the first task

### 1. Install Python 3 and an IDE

You need two separate things, and it is worth understanding the difference:

* **Python 3** is the *interpreter* — the program that actually runs your code.
* **An IDE** is the *editor* — where you write your code comfortably. It does not run anything by itself; it asks Python to do it.

Install Python 3 from [python.org](https://www.python.org/downloads/) (tick **"Add Python to PATH"** on Windows), and then either [PyCharm Community](https://www.jetbrains.com/pycharm/download/) or [Visual Studio Code](https://code.visualstudio.com/).

### 2. Your first script

Create a file called `hello_world.py` containing one line:

```python
print("Hello, ocean!")
```

### 3. Run it two different ways

**From the IDE:** press the green ▶ Run button.

**From the terminal:** open a terminal, navigate to the folder, and type:

```bash
python3 hello_world.py       # macOS / Linux
python hello_world.py        # Windows
```

> 📝 You should get exactly the same output both times. Explain in your report why that is — what is the IDE actually doing when you press ▶?

### 4. Install matplotlib

You will need it in the last task:

```bash
pip3 install matplotlib      # macOS / Linux
pip install matplotlib       # Windows
```

---

## Before you start: five programming ideas you will use today

### Variables and types

A variable is a name that stores a value so you can reuse it without retyping it — a labelled box.

```python
depth = 25.4          # float   -> a number with decimals
n_samples = 12        # int     -> a whole number
station = "OBSEA"     # str     -> text, always in quotes
is_valid = True       # bool    -> True or False
```

Python works out the type by itself. You can ask it:

```python
print(type(depth))        # <class 'float'>
print(type(n_samples))    # <class 'int'>
print(type(station))      # <class 'str'>
```

Types matter, because Python refuses to mix them:

```python
print(n_samples + depth)      # 37.4   -> fine, both are numbers
print(station + " station")   # OBSEA station  -> fine, both are text
print(station + n_samples)    # TypeError!     -> text plus number makes no sense
```

To mix them on purpose, convert first:

```python
print(station + " has " + str(n_samples) + " samples")   # str() turns a number into text
print(float("25.4") + 1)                                 # float() turns text into a number -> 26.4
```

### Lists

Most of the time we do not have one value but a whole sequence — one measurement per depth, for example. A **list** stores an ordered sequence, written with square brackets:

```python
temperatures = [24.1, 20.4, 15.6, 12.1]

print(temperatures[0])     # first element  -> 24.1   (Python counts from 0!)
print(temperatures[1])     # second element -> 20.4
print(temperatures[-1])    # last element   -> 12.1
print(len(temperatures))   # how many       -> 4
```

Lists start empty and grow with `append`:

```python
speeds = []                # an empty list
speeds.append(1533.8)      # add one value at the end
speeds.append(1525.2)
print(speeds)              # [1533.8, 1525.2]
```

### `for` loops

A `for` loop repeats the same block of code once per element in a list. Note the colon and the indentation — Python uses indentation instead of brackets, and it is not optional.

```python
temperatures = [24.1, 20.4, 15.6]

for t in temperatures:
    print("Temperature:", t)
```

```
Temperature: 24.1
Temperature: 20.4
Temperature: 15.6
```

Very often you need the *position* as well as the value, so you can look up the same index in a second list. `range(len(...))` gives you 0, 1, 2, ...:

```python
depths       = [0, 50, 100]
temperatures = [24.1, 20.4, 15.6]

for i in range(len(depths)):
    print("At", depths[i], "m the temperature is", temperatures[i], "C")
```

```
At 0 m the temperature is 24.1 C
At 50 m the temperature is 20.4 C
At 100 m the temperature is 15.6 C
```

### `if` conditionals

An `if` runs a block only when a condition is true:

```python
temperature = 1.8

if temperature < 2:
    print("Warning: this is very cold water")
elif temperature > 30:
    print("Warning: this is suspiciously warm")
else:
    print("Temperature looks normal")
```

```
Warning: this is very cold water
```

Conditions can be combined with `or` and `and`:

```python
if temperature < 2 or temperature > 30:
    print("Outside the valid range")
```

### Functions

A function is a named, reusable recipe: you give it inputs, it does some work, and hands back a result with `return`. Write the calculation once, use it as many times as you like.

```python
def add_five(x):
    return x + 5

print(add_five(10))    # -> 15
print(add_five(2.5))   # -> 7.5
```

As many arguments as you need:

```python
def rectangle_area(width, height):
    return width * height

print(rectangle_area(3, 4))     # -> 12
print(rectangle_area(10, 2.5))  # -> 25.0
```

### Reading and writing files

To read a text file, open it and loop over its lines. `with` guarantees the file is closed again even if something goes wrong:

```python
with open("ctd_profile.csv", "r") as f:      # "r" = read
    for line in f:
        print(line)
```

Each `line` arrives as **text**, including the invisible newline character at the end. Two tools clean it up:

```python
line = "0,24.10,36.52\n"

clean = line.strip()          # removes the newline -> "0,24.10,36.52"
parts = clean.split(",")      # splits on commas    -> ['0', '24.10', '36.52']

print(parts[0])               # '0'      <- still text!
print(float(parts[0]))        # 0.0      <- now a number
```

Writing works the same way, with `"w"` instead of `"r"`. `\n` is the newline character — without it everything ends up on one line:

```python
with open("results.csv", "w") as f:          # "w" = write (overwrites the file!)
    f.write("depth_m,sound_speed_ms\n")
    f.write("0,1533.78\n")
```

---

## The science bit: why the speed of sound is not constant

In air, sound travels at roughly 340 m/s. In seawater it is about **1500 m/s** — but not exactly, and that "not exactly" is what makes underwater acoustics interesting.

The speed of sound in seawater increases with all three of:

| Property | Effect |
|---|---|
| **Temperature** | strongest effect near the surface — warm water is faster |
| **Salinity** | weakest effect — saltier water is slightly faster |
| **Pressure (depth)** | dominant in the deep ocean — deeper water is faster |

These pull in opposite directions as you descend. Temperature drops fast through the thermocline, slowing sound down; but pressure keeps rising, speeding it up. Somewhere in between there is a **minimum**, and sound that enters that layer gets refracted back into it and trapped. That waveguide is the **SOFAR channel**, and it is why a whale call can travel thousands of kilometres.

You are going to find it in real data today.

### The Mackenzie equation

Mackenzie (1981) fitted a nine-term polynomial to measurements:

**c = 1448.96 + 4.591·T − 5.304×10⁻²·T² + 2.374×10⁻⁴·T³ + 1.340·(S−35) + 1.630×10⁻²·D + 1.675×10⁻⁷·D² − 1.025×10⁻²·T·(S−35) − 7.139×10⁻¹³·T·D³**

where **T** is temperature in °C, **S** is salinity in PSU, **D** is depth in metres, and **c** comes out in m/s.

It is only valid for **2 ≤ T ≤ 30 °C**, **25 ≤ S ≤ 40 PSU** and **0 ≤ D ≤ 8000 m**. Remember that — it matters in Task 5.

In Python, `10⁻²` is written `1e-2`, and `T²` is written `t**2`.

---

# Lab Assignment

**Goal:** by the end you will have one script (`LAB1.py`) that reads `ctd_profile.csv`, computes the speed of sound at every depth, writes the result to a new file, and plots the profile.

---

## Task 1 — Hello, ocean

**Goal:** get comfortable with variables, types and `print()` before any oceanography enters the picture.  
**Report**: add a snapshot of the code and its output.

```python
name = "____"                       # TODO: put your name (or your pair's names) here
print("Hello, my name is:", name)

temperature = 24.1                  # degrees Celsius
salinity = 36.5                     # PSU
depth = 0                           # metres

print("Temperature:", temperature, "C")
print("Salinity:", salinity, "PSU")
print("Depth:", depth, "m")

print("Type of temperature:", ____(temperature))   # TODO: which function reports the type?
print("Type of depth:", ____(depth))               # TODO: same here
```

> 📝 `temperature` and `depth` are both numbers, but Python reports two different types for them. Which two, and what is the difference?

Now try this line, and then **delete it** once you have seen what happens:

```python
print("Depth is " + depth)     # this crashes on purpose
```

> 📝 Copy the error message into your report. What is Python complaining about, and what would you change to make it work?

---

## Task 2 — The speed of sound, the hard way

**Goal:** compute the speed of sound in three different water masses.  
**Report**: add a snapshot of the code and the three results.

Here is the Mackenzie equation for the first water mass. Fill in the two blanks:

```python
# 1. Surface Mediterranean water in summer
t = 24.1
s = 36.5
d = 0

c = (1448.96
     + 4.591 * t
     - 5.304e-2 * t**2
     + 2.374e-4 * ____            # TODO: this term needs T cubed
     + 1.340 * (s - 35)
     + 1.630e-2 * d
     + 1.675e-7 * d**2
     - 1.025e-2 * t * (s - 35)
     - 7.139e-13 * t * ____)      # TODO: this term needs D cubed

print("Surface Mediterranean:", round(c, 2), "m/s")
```

Expected output:

```
Surface Mediterranean: 1533.76 m/s
```

Now do the same for two more water masses. **Copy and paste** the whole block twice and change only the three input values:

```python
# 2. Levantine Intermediate Water
t = 13.5
s = 38.7
d = 400

# 3. Deep Atlantic water
t = 2.5
s = 34.9
d = 3000
```

You should get:

```
Levantine Intermediate: 1512.85 m/s
Deep Atlantic:          1510.34 m/s
```

> 📝 Levantine Intermediate Water is **11 °C warmer** than the deep Atlantic water, yet the two sound speeds are almost identical. Explain why, using the table of effects above.

> 📝 You have now written the same nine-term formula three times. Suppose you found a typo in one term. How many places would you have to fix it, and how confident are you that you would catch all of them?

---

## Task 3 — Write the function

**Goal:** write the formula **once**, and never again.  
**Report**: add a snapshot of the code and the self-check output.

That last question is the whole point of functions. Wrap the formula up, give it a name, and let it take the three values as arguments:

```python
def sound_speed(t, s, d):
    """Speed of sound in seawater (Mackenzie 1981), in m/s."""
    c = (1448.96
         + 4.591 * t
         - 5.304e-2 * t**2
         + 2.374e-4 * t**3
         + 1.340 * (s - 35)
         + 1.630e-2 * d
         + 1.675e-7 * d**2
         - 1.025e-2 * t * (s - 35)
         - 7.139e-13 * t * d**3)
    return ____                    # TODO: what should the function hand back?
```

Check it reproduces Task 2, in three lines instead of thirty:

```python
print(round(sound_speed(24.1, 36.5, 0), 2))       # -> 1533.76
print(round(sound_speed(13.5, 38.7, 400), 2))     # -> 1512.85
print(round(sound_speed(____, ____, ____), 2))    # TODO: the deep Atlantic values -> 1510.34
```

**Self-check.** The value everyone uses to verify a Mackenzie implementation is T = 25 °C, S = 35 PSU, D = 1000 m:

```python
print(round(sound_speed(25, 35, 1000), 3))    # must print exactly 1550.744
```

> ⚠️ If you do not get `1550.744`, you have a typo in the formula. Fix it now — every remaining task depends on this function being right.

---

## Task 4 — Read the CTD profile from a file

**Goal:** load 25 real depth / temperature / salinity records into three lists.  
**Report**: add a snapshot of the code and the printed summary.

`ctd_profile.csv` is a real-shaped CTD cast from the open ocean. Its first three lines look like this:

```
depth_m,temperature_c,salinity_psu
0,24.10,36.52
10,24.05,36.52
```

The first line is a **header** — it names the columns, it is not data, and we must skip it. `f.readline()` reads exactly one line and moves on, which is precisely what we need:

```python
depths = []
temperatures = []
salinities = []

with open("ctd_profile.csv", "r") as f:
    header = f.readline()               # read the header line and set it aside
    for line in f:                      # the loop now starts at the first data line
        parts = line.strip().split(____)      # TODO: which character separates the columns?
        depths.append(float(parts[0]))
        temperatures.append(float(parts[____]))    # TODO: which column is temperature?
        salinities.append(float(parts[____]))      # TODO: which column is salinity?
```

Check what you loaded:

```python
print("Header was:", header.strip())
print("Number of records:", ____(depths))       # TODO: how many items are in a list?
print("Shallowest:", depths[0], "m ->", temperatures[0], "C,", salinities[0], "PSU")
print("Deepest:", depths[____], "m ->", temperatures[____], "C,", salinities[____], "PSU")
```

Expected output:

```
Header was: depth_m,temperature_c,salinity_psu
Number of records: 25
Shallowest: 0.0 m -> 24.1 C, 36.52 PSU
Deepest: 4000.0 m -> 1.8 C, 34.92 PSU
```

> 📝 Why do we need `float()` here? What would `depths[0] + depths[1]` produce if we left the values as text? Try it and report what happens.

---

## Task 5 — Compute the whole profile, and check your inputs

**Goal:** call your function once per depth, and refuse to trust values outside the equation's valid range.  
**Report**: add a snapshot of the code and the warnings it prints.

You have a function and three lists. Now put them together. Because we need to read the same position `i` from all three lists at once, we loop over `range(len(depths))`:

```python
speeds = []

for i in range(len(depths)):
    t = temperatures[i]
    s = salinities[i]
    d = depths[i]

    speeds.append(sound_speed(t, s, d))

print("Computed", len(speeds), "sound speeds")
print("At the surface:", round(speeds[0], 2), "m/s")
```

Now add the validation. Remember Mackenzie is only valid for 2 ≤ T ≤ 30 °C. Insert this **inside the loop**, just before the `append`:

```python
    if t < ____ or t > ____:            # TODO: the valid temperature range
        print("WARNING: temperature", t, "C at", d, "m is outside the valid range (2-30 C)")
```

Expected output:

```
WARNING: temperature 1.95 C at 3500.0 m is outside the valid range (2-30 C)
WARNING: temperature 1.8 C at 4000.0 m is outside the valid range (2-30 C)
Computed 25 sound speeds
At the surface: 1533.78 m/s
```

> 📝 Two records triggered the warning. Are these measurement errors, or is this real water? Look up the typical temperature of Antarctic Bottom Water before you answer.

> 📝 We printed a warning but still computed a value for those two depths. Was that the right decision? What else could we have done, and what would we lose in each case?

---

## Task 6 — Save your results, then read them back

**Goal:** write a new CSV file, and prove it worked by loading it again.  
**Report**: add a snapshot of the code and the first few lines of your output file.

Computing something is useless if it disappears when the script ends. Write the results out:

```python
with open("sound_speed_profile.csv", "____") as f:      # TODO: "r" to read, "w" to write?
    f.write("depth_m,sound_speed_ms\n")
    for i in range(len(depths)):
        f.write(str(depths[i]) + "," + str(round(speeds[i], 2)) + "____")   # TODO: end the line
```

Open `sound_speed_profile.csv` in your IDE. It should start like this:

```
depth_m,sound_speed_ms
0.0,1533.78
10.0,1533.82
20.0,1533.63
```

and end like this:

```
4000.0,1524.75
```

Now read it back — never trust a file you have not re-opened:

```python
check_depths = []
check_speeds = []

with open("sound_speed_profile.csv", "r") as f:
    f.readline()                        # skip the header
    for line in f:
        parts = line.strip().split(",")
        check_depths.append(float(parts[0]))
        check_speeds.append(float(parts[1]))

print("Read back", len(check_speeds), "values")
print("First value matches:", round(check_speeds[0], 2) == round(speeds[0], 2))
```

> 📝 What happens if you run the script twice? Does the file grow, or stay the same size? Explain what `"w"` does to a file that already exists.

---

## Task 7 — Plot the profile and find the SOFAR channel

**Goal:** see the physics in your own data.  
**Report**: include the figure and answer the questions.

We will not explain how plotting works today — that is T03. Just copy this in and run it.

Two conventions worth noticing: oceanographers put **depth on the vertical axis**, and they **flip it** so that deep is down, the way the ocean actually is.

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(5, 7))

ax.plot(speeds, depths, marker="o")
ax.invert_yaxis()                       # deep at the bottom, like the real ocean
ax.set_xlabel("sound speed (m/s)")
ax.set_ylabel("depth (m)")
ax.set_title("Sound speed profile")
ax.grid(True)

plt.show()
```

Now find the minimum. Python has a built-in for this, and `.index()` tells you *where* a value sits in a list:

```python
c_min = min(speeds)
i_min = speeds.index(c_min)

print("Minimum sound speed:", round(c_min, 2), "m/s")
print("Found at depth:", depths[i_min], "m")
```

Expected output:

```
Minimum sound speed: 1490.46 m/s
Found at depth: 1200.0 m
```

> 📝 Include your figure in the report and mark the SOFAR axis on it. At what depth is it?

> 📝 The sound speed at the surface is 1533.78 m/s and at 4000 m it is 1524.75 m/s — nearly the same value. But the profile in between is very far from a straight line. Explain the shape in terms of the three competing effects.

> 📝 A whale calls from 1200 m depth. Why does its call travel further than the same call made at 50 m?

### Bonus

Plot temperature and salinity next to the sound speed, and see which one drives the shape:

```python
fig, axes = plt.subplots(1, 3, figsize=(12, 6), sharey=True)

axes[0].plot(temperatures, depths, color="red")
axes[0].set_xlabel("temperature (C)")
axes[0].set_ylabel("depth (m)")

axes[1].plot(salinities, depths, color="green")
axes[1].set_xlabel("salinity (PSU)")

axes[2].plot(speeds, depths, color="blue")
axes[2].set_xlabel("sound speed (m/s)")

axes[0].invert_yaxis()
plt.show()
```

> 📝 Above 1000 m, which of the two — temperature or salinity — explains the sound speed curve? Below 2000 m, neither of them does. What is driving it there?

---

## Wrap-up

Today you learned how to:

- store values in **variables**, and why Python cares about their **type**
- keep sequences of values in **lists**, and walk through them with **`for`** loops
- make decisions with **`if`**, and use it to validate data before trusting it
- write a **function** once and reuse it, instead of copy-pasting a formula nine terms long
- **read** a real data file, split each line into columns, and convert text into numbers
- **write** your results back out to a new file, and verify them by reading them again
- turn a table of numbers into a figure that shows the **SOFAR channel**

The script you just wrote is a complete, if small, piece of oceanographic software: it ingests data, validates it, processes it, stores the result and visualises it. Every lab from here on is a variation on those same five steps.
