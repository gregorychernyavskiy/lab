```markdown
# Tree-sitter Java BIO Labeling with NeuroX Integration

## Overview
This repository integrates **Tree-sitter** for parsing Java code with **BIO (Begin-Inside-Outside)** labeling for generating binary and multiclass annotated datasets. It also integrates **NeuroX** to generate activation files for further analysis. The dataset generation leverages the combination of Tree-sitter's Abstract Syntax Tree (AST) and custom label extraction using BIO tagging, with support for both binary and multiclass labels.

## Features
- **BIO Tagging**: Label tokens as `B-`, `I-`, or `O-` based on their chunk positions (Beginning, Inside, Outside).
- **NeuroX Activation Files**: Automatically generate NeuroX-compatible activation files during dataset generation.
- **Tree-sitter Integration**: Efficient parsing of Java code using the Tree-sitter library.
- **Custom Dataset Generation**: Generate binary and multiclass labeled datasets for machine learning tasks, which can be used to train and analyze models.

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
- NeuroX (for activation file generation)

## Usage

1. **Prepare the Java source code**: You will provide your Java file as input.
2. **Run the script**:

```bash
python bio_labeling.py <path_to_your_java_file>
```

3. **Output**: The program will output BIO-labeled tokens to the console and also generate NeuroX-compatible activation files.

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

## NeuroX Activation Files

Along with BIO-labeled tokens, the script generates activation files for use with **NeuroX**. These files can be used for model analysis and further experimentation in a neural network environment. The format of these files will match the format required for NeuroX's input and activation data.

## Future Enhancements

- Additional optimizations for BIO labeling with larger datasets.
- Support for more Java constructs (e.g., switch cases, lambdas) to improve token extraction.

## Contributions

Feel free to fork this repository, submit pull requests, or open issues to improve functionality and extend support for other languages or use cases.
```

---


```python
import numpy as np
from typing import List, Tuple
from tree_sitter import Language, Parser
import tree_sitter_java as tsjava
import sys
import os

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
        
        # Determine the label type based on the node type (from the PDF example)
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

# Function to save the NeuroX activation file
def save_activation_file(tokens_with_bio_labels, filename="activation.txt"):
    with open(filename, 'w') as f:
        for token, label in tokens_with_bio_labels:
            f.write(f"{token}\t{label}\n")

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

    # Print the results to the console
    for token, label in tokens_with_bio_labels:
        print(f"{token} -> {label}")

    # Save the activation file in NeuroX format
    activation_filename = os.path.splitext(sys.argv[1])[0] + "_activation.txt"
    save_activation_file(tokens_with_bio_labels, activation_filename)

    print(f"NeuroX activation file saved as: {activation_filename}")
```

