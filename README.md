# cGrading - Student Grading System

A fast, interactive student grading system and table management CLI built on top of the **cDB** database library. It operates as a finite state machine (FSM) to calculate grades, compute descriptive statistics and distributions, search and filter student records, and export results.

cDB github repository: https://github.com/NattakrisThongsakulpanich/cDB

---

## What It Is

`cGrading` is an interactive grading management application designed for instructors and students. It provides:

- **Multiple Input Formats**:
  - Reads `.csv` files (e.g., header `id,name,score`).
  - Reads native `.cdb` binary database files.
- **Dual Grading Models**:
  - **Criterion-Referenced (Fixed Cut-offs)**: Standard grading scale:
    - `A >= 80`, `B+ >= 75`, `B >= 70`, `C+ >= 65`, `C >= 60`, `D+ >= 55`, `D >= 50`, `F < 50`
  - **Norm-Referenced (T-Score)**: Curve grading using normalized T-scores:
    $$T = 50 + 10 \times \frac{\text{Score} - \text{Mean}}{\text{SD}}$$
    - `A >= 65`, `B+ >= 60`, `B >= 55`, `C+ >= 50`, `C >= 45`, `D+ >= 40`, `D >= 35`, `F < 35`
- **Statistical Analysis & Visualizations**:
  - Computes Count, Mean, Standard Deviation (SD), Min, and Max score.
  - Generates an ASCII terminal bar chart showing the grade distribution.
- **Interactive Queries & Filtering**:
  - Instant lookup by Student ID using cDB's record index.
  - Filter and display all students who received a specific letter grade.
- **Dynamic Sorting**:
  - Transient view sorting (Sort by Name A-Z/Z-A, Score Low-High/High-Low).
  - Permanent file reordering and disk synchronization using `cdb_swapRecord`.
- **Automatic Persistence & Export**:
  - When a `.csv` is loaded, it generates an active working `.cdb` database that automatically saves incremental updates (like adding new students).
  - Exports clean CSV files (`RECORD` [Student ID], `Name`, `Grade`).

---

## Architecture: Finite State Machine (FSM)

The program runs as an explicit finite state machine where every state is represented by a dedicated handler function:

```
[S_LOAD] ──> [S_MENU] ──┬──> [S_SHOW]     (View/Sort table & choose grading scale)
                        ├──> [S_STATS]    (Mean, SD, min/max, ASCII histogram)
                        ├──> [S_QUERY]    (Search student by ID)
                        ├──> [S_WRITE]    (Export to RFC 4180 CSV)
                        ├──> [S_GRADE]    (Filter by specific letter grade)
                        ├──> [S_ADD]      (Interactive entry with auto-regrade)
                        ├──> [S_SORTFILE] (Permanent table rearrangement)
                        └──> [S_END]      (Clean database shutdown & exit)
```

---

## How to Build

### Dependencies
- C99-compliant C compiler (`gcc`, `clang`, or MSVC)
- Math library (`-lm` for POSIX systems)
- `cDB` library (`cDB.h` and `cDB.lib` / compiled source files)

### Compiling with CMake
From the repository root:

```bash
mkdir build
cd build
cmake ..
cmake --build . --config Release
```

### Compiling Directly with GCC / Clang
```bash
gcc -std=c99 -O2 examples/example3.c src/all_cDB_Library.c -Iinclude -o cGrading -lm
```

---

## How to Use

### 1. Launching the Program

You can supply an input file directly via CLI argument or prompt on startup:

```bash
# Direct launch with a file
./cGrading students.csv

# Or with an existing cDB database
./cGrading class.cdb

# Interactive launch (prompts for filename)
./cGrading
```

### 2. Input File Formats

#### CSV Input (`.csv`)
Requires a header line with `id,name,score` (order is flexible, case-insensitive):
```csv
id,name,score
6501001,Alice Smith,88.5
6501002,Bob Jones,72.0
6501003,Charlie Brown,59.0
```

#### Native Database (`.cdb`)
Directly loads `.cdb` binary database tables containing `name`, `score`, and optional `id` attributes.

---

## Menu Options Walkthrough

When loaded, `cGrading` displays the main interactive menu:

```text
+----------------------------------------------------------------+
|                            cGrading                            |
+----------------------------------------------------------------+
  1  Show all students and grades
  2  Statistics
  3  Query by student ID
  4  Write output file (.csv)
  5  Students with a grade
  6  Add students
  7  Sort entire file (permanent)
  0  Exit

  Choose [0-7] > 
```

### Option 1: Show all students and grades
- Prompts whether to switch between **Fixed cut-offs** or **Norm-referenced (T-scores)**.
- Prompts for display ordering (As stored, Name A-Z, Name Z-A, Score high-low, Score low-high).
- Renders an ASCII formatted table projection directly to the terminal.

### Option 2: Statistics & Distribution
- Displays class metrics: Count, Mean, SD, Min, Max.
- Generates an ASCII frequency histogram of grade distribution:
```text
--- Statistics ---------------------------------------------------
  Students : 3
  Mean     : 73.17
  SD       : 12.08
  Min      : 59.00
  Max      : 88.50

  Grade distribution
  A  | ############## 1
  B+ | 0
  B  | ############## 1
  C+ | 0
  C  | 0
  D+ | ############## 1
  D  | 0
  F  | 0
```

### Option 3: Query by student ID
- Performs an $O(1)$ alias lookup on the student ID.
- Displays the matching student record or indicates if not found.

### Option 4: Write output file (.csv)
- Exports the graded table to an RFC 4180-compliant `.csv` file with columns `RECORD` (ID), `Name`, and `Grade`.

### Option 5: Students with a grade
- Selects and displays only the students who achieved a specific letter grade (e.g., `A`, `B+`, `F`).

### Option 6: Add students
- Allows manual entry of student records in `ID,Name,Score` format.
- Recalculates Mean, SD, T-scores, and grades automatically.
- Automatically saves changes to the working `.cdb` file.

### Option 7: Sort entire file (permanent)
- Permanently rearranges the underlying physical table records using `cdb_swapRecord` based on Name or Score.
- Saves the permanently sorted table back to the active `.cdb` file.
