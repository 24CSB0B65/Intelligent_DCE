# Intelligent Dead Code Elimination Using Machine Learning

## 1. Project Overview

**Intelligent Dead Code Elimination (DCE) Using Machine Learning** is a compiler optimization project that identifies unnecessary code in a program and helps eliminate it safely.

Dead code refers to code that does not contribute to the program's required output or behavior. Keeping unnecessary code can increase program size, reduce readability, and introduce avoidable maintenance overhead.

Traditional compiler optimization techniques use program analysis, such as liveness analysis, to identify dead code. This project combines a C++-based compiler analysis pipeline with a Python-based machine learning pipeline to classify variables as `LIVE` or `DEAD`, validate predictions, and compare machine learning results with classical analysis.

The project uses Abstract Syntax Tree (AST) construction, Control Flow Graph (CFG) generation, liveness analysis, feature extraction, dataset generation, machine learning, and safety validation.

**Project title:** Intelligent Dead Code Elimination Using Machine Learning

**Project type:** Compiler Design and Machine Learning

**Programming languages:** C++ and Python

**Tools and libraries:** Scikit-learn, Pandas, Matplotlib, CSV, and a C++ compiler

---

## 2. Problem Statement

Programs may contain variables and statements whose values are never used in any meaningful way. Such unnecessary code is commonly called dead code.

Consider the following example:

```cpp
int main() {
    int a = 10;
    int b = 20;
    int c = a + 5;

    return c;
}
```

In this example:

* `a` is used to calculate `c`, so its value contributes to the result.
* `c` is returned, so it is required.
* `b` is never used, so its declaration and assignment may be removable under the program's semantic and side-effect constraints.

The optimized code could be:

```cpp
int main() {
    int a = 10;
    int c = a + 5;

    return c;
}
```

The challenge is to detect unnecessary code without removing code that affects the program's output or behavior.

This project explores how classical compiler analysis and machine learning can work together to address this problem.

## 3. Objectives

The main objectives are:

1. Build a basic compiler-analysis pipeline using C++.
2. Tokenize source code using a lexer.
3. Construct an Abstract Syntax Tree to represent program statements.
4. Generate a Control Flow Graph to represent program structure and execution flow.
5. Perform liveness analysis to identify variables whose values are needed later.
6. Apply dead code elimination to removable declarations.
7. Extract numerical features from program structures and variable usage.
8. Generate a dataset containing features and `LIVE`/`DEAD` labels.
9. Train machine learning models to predict variable classifications.
10. Validate machine learning predictions using conservative safety rules.
11. Compare classical analysis and machine learning results.
12. Generate reports and visualizations to analyze the results.

## 4. Technologies Used

| Technology           | Purpose                                             |
| -------------------- | --------------------------------------------------- |
| C++                  | Lexer, parser, AST, CFG, liveness analysis, and DCE |
| Python               | Machine learning, validation, and result comparison |
| Pandas               | Loading and processing CSV datasets                 |
| Scikit-learn         | Training and evaluating machine learning models     |
| Matplotlib           | Generating comparison charts                        |
| CSV                  | Transferring extracted features and labeled data    |
| Pickle (`model.pkl`) | Saving and loading the trained model                |
| VS Code              | Development and execution environment               |

## 5. System Architecture

The project consists of two connected parts.

### Part A: C++ Compiler Analysis

The C++ component analyzes the source program and produces intermediate representations, liveness results, optimized code, extracted features, and a labeled dataset.

### Part B: Python Machine Learning

The Python component uses the generated dataset to train models, evaluate predictions, apply safety validation, and compare the results with classical analysis.

### Overall workflow

```text
                Input C++ Program
                        |
                        v
                    Lexer
                        |
                        v
                    Parser
                        |
                        v
                 Abstract Syntax Tree
                        |
                        v
             Control Flow Graph (CFG)
                        |
                        v
                Security Analysis
                        |
                        v
                 Liveness Analysis
                        |
                        v
              Dead Code Elimination
                        |
                        v
                 Feature Extraction
                        |
                        v
                Dataset Generation
                        |
                        v
                  features.csv
                  dataset.csv
                        |
                        v
             Python Machine Learning
                        |
             +----------+----------+
             |                     |
             v                     v
        Decision Tree        Gaussian Naive Bayes
             |                     |
             +----------+----------+
                        |
                        v
               Model Evaluation
                        |
                        v
               Safety Validation
                        |
                        v
             Classical vs. ML Comparison
                        |
                        v
              Reports and Chart
```

The diagram describes the logical pipeline. Depending on the current execution setup, the C++ executable and Python scripts may need to be run as separate steps.

## 6. Project Directory Structure

The project files are organized approximately as follows:

```text
Intelligent-DCE/
|
|-- Lexer.cpp
|-- Lexer.h
|
|-- Parser.cpp
|-- Parser.h
|-- ASTNode.h
|
|-- CFG.cpp
|-- CFG.h
|
|-- Security.cpp
|-- Security.h
|
|-- Analysis.cpp
|-- Analysis.h
|
|-- FeatureExtractor.cpp
|-- FeatureExtractor.h
|
|-- DatasetBuilder.cpp
|-- DatasetBuilder.h
|
|-- main.cpp
|
|-- examples/
|   |-- sample_program.txt
|
|-- week11_ml_model.py
|-- week12_safety_validation.py
|-- week13_integration_comparison.py
|
|-- features.csv
|-- dataset.csv
|-- liveness_output.txt
|-- model.pkl
|-- comparison_chart.png
|-- README.md
```

Some files are generated during execution. The exact generated files may vary depending on the implementation and current configuration.

## 7. Detailed Module Explanation

### 7.1 Lexer

**Files:** `Lexer.cpp`, `Lexer.h`

The lexer is the first stage of the compiler pipeline. It reads the source program and converts its characters into tokens.

Examples of tokens include:

* Keywords such as `int` and `return`
* Identifiers such as `a`, `b`, and `c`
* Numeric literals such as `10` and `20`
* Operators such as `=`, `+`, and `-`
* Punctuation such as `;`, `(`, and `)`

For example:

```cpp
int a = 10;
```

The lexer identifies tokens representing the keyword `int`, identifier `a`, assignment operator `=`, number `10`, and statement terminator `;`.

**Purpose:** To convert raw source code into a structured token sequence that the parser can process.

### 7.2 Parser and Abstract Syntax Tree

**Files:** `Parser.cpp`, `Parser.h`, `ASTNode.h`

The parser processes the tokens generated by the lexer and builds an Abstract Syntax Tree.

An AST represents the logical structure of a program as a hierarchy of nodes.

For example:

```cpp
int a = 10;
return a;
```

A simplified AST could look like this:

```text
PROGRAM
|
|-- DECL
|   |-- IDENTIFIER: a
|   |-- VALUE: 10
|
|-- RETURN
    |-- IDENTIFIER: a
```

Here:

* `PROGRAM` is the root node.
* `DECL` represents a variable declaration.
* `IDENTIFIER` represents a variable name.
* `VALUE` represents a literal value.
* `RETURN` represents a return statement.

The AST makes it easier to analyze declarations, expressions, loops, and variable usage without repeatedly processing raw source text.

**Purpose:** To represent the program in a structured form suitable for subsequent analysis.

**Current scope:** The parser handles the subset of C++ constructs implemented in the project; it should not be considered a complete C++ language parser.

### 7.3 Control Flow Graph

**Files:** `CFG.cpp`, `CFG.h`

A Control Flow Graph represents the possible execution flow between program statements or blocks.

Its nodes represent program components, while its edges represent possible transfers of control.

A simplified example is:

```text
        Declaration
             |
             v
        Computation
             |
             v
          Return
```

CFG information is useful when analyzing how statements relate to each other and when extracting structural features.

The project also uses CFG-related information to calculate properties such as the number of graph nodes and the graph depth.

**Purpose:** To provide structural information about the program for analysis and feature extraction.

The accuracy and completeness of the CFG depend on which control-flow constructs are implemented in the current version.

### 7.4 Security Analysis

**Files:** `Security.cpp`, `Security.h`

The security-analysis stage examines code for conditions that may make automatic removal unsafe.

For example, a statement may have an observable side effect, such as writing to a file or printing output. Removing such a statement could change the program's behavior even if a variable assigned by that statement is never used.

The project includes security-related tagging and side-effect information to support conservative optimization decisions.

**Purpose:** To help prevent unsafe code removal.

Security tagging alone does not prove that a transformation preserves semantics. A production-quality optimizer would require more comprehensive side-effect analysis and semantic validation.

### 7.5 Liveness Analysis

**Files:** `Analysis.cpp`, `Analysis.h`

Liveness analysis is a classical compiler data-flow analysis that determines whether a variable's current value may be needed in the future.

The analysis works backward through program statements.

A variable is considered live at a program point if its current value may be used along a future execution path before being overwritten.

Example:

```cpp
int a = 10;
int b = 20;
int c = a + 5;

return c;
```

The analysis identifies:

* `a` as live because its value is used to calculate `c`.
* `c` as live because its value is returned.
* `b` as dead because its value is not used later.

The implementation processes statements such as declarations, returns, and loops. Loop handling requires repeated analysis until the live-variable information stabilizes.

**Purpose:** To provide a classical baseline for identifying unnecessary variable declarations and supporting dead code elimination.

**Important distinction:** Liveness is a property of a value at a particular program point. A complete optimizer must also consider control flow, variable redefinitions, and observable side effects.

### 7.6 Dead Code Elimination

**Files:** `Analysis.cpp`, `Analysis.h`

Dead Code Elimination removes statements that can be proven unnecessary without changing required program behavior.

The current implementation uses liveness information to decide which variable declarations can be removed.

For example:

```cpp
int a = 10;
int b = 20;
return a;
```

A simplified optimized version is:

```cpp
int a = 10;
return a;
```

The declaration of `b` can be removed because its value is never used, assuming the initializer has no observable side effects and the transformation is otherwise safe.

The implementation treats returns and loops conservatively rather than indiscriminately deleting them.

**Purpose:** To produce a program with unnecessary declarations removed while preserving required statements.

### 7.7 Feature Extraction

**Files:** `FeatureExtractor.cpp`, `FeatureExtractor.h`

Feature extraction converts program-analysis information into numerical values that can be supplied to machine learning algorithms.

The project extracts features related to variable declarations, variable usage, program size, control-flow structure, side effects, and usage ratio.

The eight documented feature names are:

| Feature             | Meaning                                                       |
| ------------------- | ------------------------------------------------------------- |
| `var_decl_count`    | Number of variable declarations counted by the extractor      |
| `var_use_count`     | Number of variable uses counted by the extractor              |
| `total_statements`  | Number of statements counted by the extractor                 |
| `total_variables`   | Number of variables counted by the extractor                  |
| `cfg_depth`         | Depth of the control-flow structure                           |
| `cfg_nodes`         | Number of CFG nodes                                           |
| `side_effect_count` | Number of side effects detected by the implemented analysis   |
| `usage_ratio`       | Ratio derived from variable usage and declaration information |

The precise counting rules depend on the implemented traversal logic.

The extracted features are written to `features.csv`.

**Purpose:** To represent the program in a numerical format suitable for machine learning.

### 7.8 Dataset Generation

**Files:** `DatasetBuilder.cpp`, `DatasetBuilder.h`

The dataset builder generates labeled examples for machine learning.

It collects declared variables, analyzes their liveness, and associates each variable with extracted program features and a `LIVE` or `DEAD` label.

A simplified dataset may look like this:

| Variable | Declaration count | Usage count | Label |
| -------- | ----------------: | ----------: | ----- |
| `a`      |                 1 |           1 | LIVE  |
| `b`      |                 1 |           0 | DEAD  |
| `c`      |                 1 |           1 | LIVE  |

These rows are illustrative, not a guarantee of the exact output format of the current implementation.

The actual dataset contains the feature columns required by the Python scripts and a target label.

The project uses classical liveness analysis to establish the labels. Machine learning is then trained to learn patterns associated with these labels.

The generated dataset is stored in `dataset.csv`.

**Purpose:** To create training and evaluation data from the compiler-analysis pipeline.

**Limitation:** If labels are derived from classical analysis, the model learns to approximate that analysis. The model is not automatically discovering a universally correct definition of dead code.

## 8. Machine Learning Pipeline

The Python component is organized into three stages.

### 8.1 Week 11 — Model Training and Evaluation

**File:** `week11_ml_model.py`

This stage loads the generated dataset and prepares the features and target labels.

The project uses two classification algorithms.

#### Decision Tree Classifier

A Decision Tree learns a sequence of decision rules based on the input features.

For example, a simple learned rule might classify a variable with no counted uses as `DEAD`. The actual rule depends on the training data.

Advantages:

* Easy to interpret
* Can learn nonlinear decision rules
* Useful for understanding which features influence predictions

#### Gaussian Naive Bayes

Gaussian Naive Bayes is a probabilistic classification algorithm that models numerical features using Gaussian distributions, subject to its conditional-independence assumption.

It provides a second model for comparing predictions against the Decision Tree.

#### Evaluation metrics

The training stage may evaluate the models using:

* **Accuracy:** Fraction of predictions that are correct.
* **Precision:** Fraction of predicted dead-code cases that are actually dead according to the reference labels.
* **Recall:** Fraction of reference dead-code cases identified by the model.
* **F1-score:** Harmonic mean of precision and recall.
* **Cross-validation:** Evaluation across different train/test splits.

The implementation uses leave-one-out cross-validation (LOOCV) as an evaluation method.

The trained model is saved as `model.pkl` for subsequent use.

**Purpose:** To train and evaluate models that classify variable-related examples as `LIVE` or `DEAD`.

**Important limitation:** The initial dataset discussed during development contained only five rows. Perfect scores on such a small dataset do not demonstrate that the model will generalize to unseen, real-world programs. A larger and more diverse dataset is necessary for meaningful evaluation.

### 8.2 Week 12 — Safety Validation

**File:** `week12_safety_validation.py`

The safety-validation stage loads the trained model and checks its predictions against additional constraints.

Potentially important checks include:

* Whether the variable is used later
* Whether the code has observable side effects
* Whether classical analysis indicates that the value is live
* Whether a predicted removal would violate conservative safety rules

The intended design is to avoid allowing a machine learning prediction alone to authorize unsafe deletion.

For example, if the model predicts `DEAD` but the safety checks identify the value as live or the statement as side-effecting, the candidate should not be removed automatically.

**Purpose:** To add conservative checks around machine learning predictions and reduce the risk of unsafe optimization.

The exact guarantees depend on the implemented validation rules. These checks should not be interpreted as a formal proof of semantic equivalence.

### 8.3 Week 13 — Integration and Comparison

**File:** `week13_integration_comparison.py`

This stage brings together the results of classical analysis and machine learning.

It compares their classifications, summarizes the results, and produces a visual representation of the comparison.

The comparison can help identify:

* Cases where both approaches agree
* Cases where the approaches disagree
* Potential false-positive classifications
* Potential false-negative classifications

A **false positive** in dead-code detection occurs when code that is actually needed is classified as dead. This is particularly dangerous because removing it may change program behavior.

A **false negative** occurs when removable code is classified as live. This reduces optimization effectiveness but is generally less dangerous than incorrectly removing necessary code.

The project generates a comparison chart, `comparison_chart.png`, and a text report if those outputs are enabled by the current script configuration.

**Purpose:** To evaluate the agreement between classical analysis and machine learning and summarize the experimental results.

---

## 9. Input and Output Files

### Input

The sample source program is stored in:

```text
examples/sample_program.txt
```

The C++ pipeline reads this file, processes the program, and generates analysis results.

### Intermediate and final outputs

| File                   | Purpose                                           |
| ---------------------- | ------------------------------------------------- |
| `liveness_output.txt`  | Records liveness-analysis results                 |
| `features.csv`         | Stores extracted numerical features               |
| `dataset.csv`          | Stores feature examples and classification labels |
| `model.pkl`            | Stores a serialized trained model                 |
| `comparison_chart.png` | Visualizes comparison results                     |
| Text comparison report | Summarizes results, where generated by the script |

Some files must be generated before later stages can run. For example, the Week 12 validation script depends on the trained model produced during Week 11.

## 10. Installation and Requirements

### Prerequisites

Install the following:

1. A C++ compiler such as GCC or MinGW.
2. Python 3.
3. VS Code or another suitable editor.
4. The Python libraries required by the scripts.

Verify the installations in a terminal:

```bash
g++ --version
python --version
```

On Windows, the Python launcher may also be available:

```bash
py --version
```

### Install Python dependencies

Run:

```bash
python -m pip install pandas scikit-learn matplotlib
```

If your system uses the Python launcher, use:

```bash
py -m pip install pandas scikit-learn matplotlib
```

Use the same Python environment for installing dependencies and running the project scripts.

## 11. How to Run the Project

Run the stages in order so that each stage has the input files it needs.

### Step 1: Prepare the input

Place the source program in:

```text
examples/sample_program.txt
```

Use syntax supported by the current lexer and parser.

### Step 2: Compile the C++ pipeline

From the project root, compile the C++ source files. For example, if all the listed source files are located in the root directory:

```bash
g++ main.cpp Lexer.cpp Parser.cpp CFG.cpp Security.cpp Analysis.cpp FeatureExtractor.cpp DatasetBuilder.cpp -o dce_pipeline
```

If the project uses additional source files, different include paths, or a separate build configuration, adjust the command accordingly.

### Step 3: Run the C++ executable

On Windows:

```powershell
.\dce_pipeline.exe
```

On Linux:

```bash
./dce_pipeline
```

This stage should execute the C++ pipeline and generate the intermediate analysis outputs, including the feature and dataset CSV files, according to the current implementation.

### Step 4: Train the machine learning models

Run:

```bash
python week11_ml_model.py
```

This stage reads the generated dataset, trains and evaluates the configured classifiers, and saves `model.pkl` if training completes successfully.

### Step 5: Perform safety validation

After the model has been generated, run:

```bash
python week12_safety_validation.py
```

This stage loads the trained model and performs the configured validation checks.

### Step 6: Compare the results

Run:

```bash
python week13_integration_comparison.py
```

This stage performs the configured comparison and generates the report and chart.

**Execution note:** The commands above describe a sequential workflow. If your current `main.cpp` already invokes the Python scripts automatically, do not run the same stages a second time unnecessarily. Confirm the integration logic in your actual source files.

## 12. Expected Workflow Example

Consider this sample program:

```cpp
int main() {
    int x = 10;
    int y = 20;
    int z = x + 5;

    return z;
}
```

The pipeline is intended to process it as follows:

1. The lexer converts the source code into tokens.
2. The parser constructs an AST for the supported statements.
3. The CFG stage builds a representation of program flow.
4. Security analysis identifies relevant safety information.
5. Liveness analysis identifies values needed later.
6. DCE identifies removable declarations.
7. Feature extraction generates numerical program features.
8. Dataset generation associates examples with reference labels.
9. Python trains the configured classifiers.
10. Safety validation checks predictions against conservative rules.
11. The comparison stage summarizes the results.

In this example, `x` contributes to the returned value, and `z` is returned. The value assigned to `y` is unused, so its declaration is a candidate for removal, provided that its initializer has no observable side effects and the implemented safety checks permit the transformation.

## 13. Safety Considerations

Dead code elimination must preserve the program's observable behavior.

A variable being unused does not automatically mean that the statement assigning it can be removed.

For example:

```cpp
int x = performOperation();
return 0;
```

Even if `x` is never used, `performOperation()` might print output, modify a file, update shared state, or perform another observable operation.

Therefore, an optimizer must distinguish between an unused variable value and a statement that can safely be eliminated.

This project incorporates security tagging and safety validation to support conservative decisions. However, its current analysis is limited to the language constructs and checks implemented in the source code.

A machine learning classifier should be treated as a decision-support component, not as a replacement for reliable compiler correctness checks.

## 14. Testing and Evaluation

The project can be evaluated using programs containing different types of code.

Suggested test cases include:

1. A program containing a variable that is never used.
2. A program in which every declared variable contributes to the result.
3. A program containing multiple assignments to the same variable.
4. A program containing loops.
5. A program containing return statements.
6. A program containing side-effecting statements, if supported by the parser and analysis.
7. Programs containing multiple variables with different usage patterns.

For each test case, check:

* Whether the lexer and parser process the supported syntax correctly.
* Whether the AST represents the expected statements.
* Whether the CFG structure is reasonable.
* Whether liveness analysis identifies the required values.
* Whether DCE preserves statements that must remain.
* Whether the feature and dataset CSV files contain the expected rows and columns.
* Whether both machine learning models execute successfully.
* Whether safety validation catches potentially unsafe predictions.
* Whether the comparison report and chart are generated correctly.

For stronger correctness testing, compile and execute both the original and optimized programs on the same inputs and compare their observable behavior.

## 15. Current Limitations

The current project is an educational prototype rather than a complete production compiler.

Its main limitations include:

* **Limited language support:** The parser handles only the C++ constructs implemented in the project.
* **Dataset size:** The initial dataset is very small, so reported metrics may not generalize.
* **Reference labels:** The machine learning labels depend on the classical analysis used to generate the dataset.
* **Control-flow coverage:** The completeness of the CFG and liveness analysis depends on the implemented statement types and control-flow rules.
* **Side-effect analysis:** The implemented security checks may not cover every possible side effect.
* **Model reliability:** A trained classifier may make incorrect predictions on unseen code.
* **Semantic correctness:** Safety rules and prediction agreement do not, by themselves, formally prove that an optimized program is equivalent to the original.
* **Generalization:** Results from a small collection of examples should not be presented as evidence of production-level optimization performance.

These limitations provide opportunities for future development.

## 16. Future Enhancements

Possible future improvements include:

1. **Expand the dataset:** Generate more diverse programs with independently verified labels.
2. **Improve language support:** Extend the parser to handle additional C++ constructs and expressions.
3. **Build a more complete CFG:** Represent branches, loop back edges, and other control-flow paths explicitly.
4. **Improve data-flow analysis:** Handle variable redefinitions, multiple paths, and more complex program structures.
5. **Strengthen side-effect detection:** Track function calls, I/O, memory updates, and other observable operations.
6. **Improve model evaluation:** Use separate training, validation, and test datasets and report class-specific metrics.
7. **Add more models:** Compare additional classifiers only when sufficient data is available.
8. **Improve semantic validation:** Use execution-based testing and, where feasible, formal methods to verify transformations.
9. **Automate the pipeline:** Provide a single command that runs C++ analysis, model training, validation, and comparison in the correct order.
10. **Improve reporting:** Generate detailed logs, confusion matrices, feature summaries, and reproducible experiment reports.
11. **Integrate with a real compiler framework:** Explore LLVM-based intermediate representations and optimization passes.
12. **Improve reproducibility:** Reset generated files before each fresh run and record model parameters and dataset versions.

## 17. Learning Outcomes

Through this project, the following concepts can be explored in practice:

* Compiler front-end design
* Lexical analysis and parsing
* Abstract Syntax Trees
* Control Flow Graphs
* Liveness analysis and data-flow analysis
* Dead Code Elimination
* Feature engineering
* Dataset preparation and labeling
* Supervised machine learning
* Decision Tree classification
* Gaussian Naive Bayes classification
* Model evaluation and cross-validation
* Safety checks for automated decisions
* Data exchange between C++ and Python
* CSV processing and report generation

## 18. Conclusion

The Intelligent Dead Code Elimination Using Machine Learning project combines traditional compiler analysis with supervised machine learning to investigate how unnecessary variable declarations can be detected and removed.

The C++ pipeline handles source-code processing, AST and CFG construction, liveness analysis, dead code elimination, feature extraction, and dataset generation. The Python pipeline trains Decision Tree and Gaussian Naive Bayes models, evaluates their predictions, applies safety validation, and compares the results with classical analysis.

The project demonstrates how machine learning can be integrated into a compiler-related workflow while highlighting an important engineering principle: **optimization must prioritize program correctness over the removal of code.**

The current implementation provides a foundation for experimentation and learning. Larger datasets, stronger control-flow and side-effect analysis, and more rigorous semantic validation would be required to establish reliable performance on complex real-world programs.

---

## Author

**Project:** Intelligent Dead Code Elimination Using Machine Learning

**Domain:** Compiler Design, Static Analysis, and Machine Learning

**Languages:** C++ and Python

**Libraries:** Scikit-learn, Pandas, and Matplotlib

## Keywords

Dead Code Elimination, Compiler Optimization, Abstract Syntax Tree, Control Flow Graph, Liveness Analysis, Feature Engineering, Machine Learning, Decision Tree, Gaussian Naive Bayes, Safety Validation, Static Analysis.
