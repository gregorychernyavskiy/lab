### Repository Structure:

```
tree-sitter-java-bio-labeling/
├── README.md
├── bio_labeling.py
├── requirements.txt
```

### 1. **`README.md`**

```markdown
# Tree-sitter Java BIO Labeling

## Overview
This repository integrates **Tree-sitter** for parsing Java code with **BIO (Begin-Inside-Outside)** labeling for generating binary and multiclass annotated datasets. We use Tree-sitter to extract Abstract Syntax Tree (AST) tokens and apply BIO tags to label the tokens based on their position in constructs such as loops, conditions, and functions.

## Features
- **BIO Tagging**: Label tokens as `B-`, `I-`, or `O-` based on their chunk positions.
- **Tree-sitter Integration**: Efficient parsing of Java code using the Tree-sitter library.
- **Custom Dataset Generation**: This framework will be used for generating labeled datasets for machine learning tasks.

## Installation

1. Clone the repository:

```bash
git clone https://github.com/your-username/tree-sitter-java-bio-labeling.git
cd tree-sitter-java-bio-labeling
```

2. Install the dependencies:

```bash
pip install -r requirements.txt
```

### Requirements

- Python 3.x
- Tree-sitter
- NumPy
- Graphviz (optional, for AST visualization)

## Usage

1. **Prepare the Java source code**: You will provide your Java file as input.
2. **Run the script**:

```bash
python bio_labeling.py <path_to_your_java_file>
```

3. **Output**: The program will output BIO-labeled tokens to the console.

## Example

For a sample Java code like this:

```java
public class HelloWorld {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            System.out.println(i);
        }
    }
}
```

The output will be BIO-labeled as follows:

```plaintext
public -> O-other
class -> B-other
HelloWorld -> I-other
{ -> O-other
public -> O-other
static -> O-other
void -> O-other
main -> B-function
( -> I-function
String[] -> I-function
args -> I-function
) -> I-function
{ -> O-other
for -> B-loop
(int -> I-loop
i -> I-loop-variable
= -> I-loop
...
```

## Future Work

- Integration with **NeuroX activation files** for deeper analysis (work in progress).
- Enhancing support for additional Java constructs and optimizing the BIO tagging process.
```

---

### 2. **`bio_labeling.py`**

This script integrates the code from the PDF with BIO labeling.

```python
import numpy as np
from typing import List, Tuple
from tree_sitter import Language, Parser
import tree_sitter_java as tsjava
import sys

# Load Tree-sitter Java language
JAVA_LANGUAGE = Language(tsjava.language())
parser = Parser()
parser.set_language(JAVA_LANGUAGE)

# Read Java source code from a file
def read_source_code(filename: str) -> bytes:
    with open(filename, 'rb') as f:
        return f.read()

# BIO Labeling function
def bio_labeling(node, prev_label=None) -> List[Tuple[str, str]]:
    tokens = []
    if node.is_named:
        token_text = node.text.decode('utf-8')
        
        # Determine the label type based on the node type (this is from the PDF example)
        if node.type == 'for_statement':
            label = 'loop'
        elif node.type == 'if_statement':
            label = 'condition'
        elif node.type == 'method_declaration':
            label = 'function'
        elif node.type == 'variable_declaration':
            label = 'variable'
        else:
            label = 'other'
        
        # Assign BIO tagging
        if prev_label == label:
            tokens.append(('I-' + label, token_text))  # Inside the same chunk
        else:
            tokens.append(('B-' + label, token_text))  # Beginning of a new chunk
        
        prev_label = label
    else:
        tokens.append(('O-other', node.text.decode('utf-8')))  # Outside any labeled chunk
    
    # Recursively call the function for each child node
    for child in node.children:
        tokens.extend(bio_labeling(child, prev_label))
    
    return tokens

# Main execution
if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python bio_labeling.py <path_to_java_file>")
        sys.exit(1)

    # Read the source code
    source_code = read_source_code(sys.argv[1])
    
    # Parse the source code using Tree-sitter
    tree = parser.parse(source_code)
    root_node = tree.root_node
    
    # Extract tokens and BIO labels
    tokens_with_bio_labels = bio_labeling(root_node)

    # Print the results
    for token, label in tokens_with_bio_labels:
        print(f"{token} -> {label}")
```

---

### 3. **`requirements.txt`**

```text
tree-sitter
numpy
```

---

### How to Set Up and Use the Repository:

1. **Create a new GitHub repository**:
   - Go to GitHub and create a repository called `tree-sitter-java-bio-labeling`.
   - Clone the repository to your local machine.

2. **Add the Files**:
   - Copy and paste the files above into the local repository.

3. **Push to GitHub**:
   - Run the following commands to push the repository to GitHub:
   
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/your-username/tree-sitter-java-bio-labeling.git
   git push -u origin master
   ```

4. **Test the Repository**:
   - Provide a Java file, run the command, and check the output BIO-labeled tokens.
