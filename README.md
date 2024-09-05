```markdown
# Tree-sitter Java BIO Labeling with NeuroX Integration

### RAID Tool: Rapid Automatic Interpretability Datasets

This tool aims to quickly generate code datasets for interpretability studies. It provides functionality for parsing Java code using Tree-sitter, labeling the code with BIO tags, and generating activation files for NeuroX.

### How the Tool Works

1. **Tree-sitter Overview**:
   Tree-sitter is a parser generator tool and an incremental parsing library. It builds a concrete syntax tree for a source file and efficiently updates the syntax tree as the source file is edited.

2. **Token Extraction**:
   After parsing the code into an Abstract Syntax Tree (AST), the tool traverses the AST to extract tokens, which represent meaningful elements such as keywords, identifiers, and literals.

3. **BIO Labeling**:
   Each token is labeled with **BIO tags** (Begin, Inside, Outside) to indicate its position within various constructs (e.g., loops, conditions, functions).

4. **NeuroX Activation Files**:
   The tool automatically generates **NeuroX-compatible activation files** that can be used for probing or alignment studies.

---

## Installation

1. **Clone the repository**:
    ```bash
    git clone https://github.com/your-username/tree-sitter-java-bio-labeling.git
    cd tree-sitter-java-bio-labeling
    ```

2. **Install the required dependencies**:
    ```bash
    pip install -r requirements.txt
    ```

3. **Additional libraries**:
    Ensure you have the necessary dependencies:
    ```bash
    pip install tree-sitter-java
    pip install graphviz
    ```

---

## Code Walkthrough

### Step 1: Parsing Java Code and Extracting Tokens

The following code parses a Java file, extracts tokens using Tree-sitter, and labels them with BIO tags.

```python
import numpy as np
import re
import collections
from typing import Pattern, Callable, Dict, List
from tree_sitter import Language, Parser
import tree_sitter_java as tsjava
import graphviz

# Load Tree-sitter Java language
JAVA_LANGUAGE = Language(tsjava.language())
parser = Parser(JAVA_LANGUAGE)

# Sample Java code for testing
source_code = b'''
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
'''

# Parse the source code
tree = parser.parse(source_code)
root_node = tree.root_node

# Extract tokens from the Tree-sitter AST
def extract_tokens(node):
    tokens = []
    if node.is_named:
        tokens.append((node.type, node.text.decode('utf-8')))
    for child in node.children:
        tokens.extend(extract_tokens(child))
    return tokens

tokens = extract_tokens(root_node)

# Print tokens to inspect their types and contents
print("Extracted Tokens and Types:")
for token_type, token_text in tokens:
    print(f"Type: {token_type}, Text: {token_text}")
```

### Step 2: BIO Labeling

Once the tokens are extracted, they are labeled with **BIO tags** based on their position in the AST. The code assigns `B-`, `I-`, and `O-` labels based on the token's role in constructs like functions, loops, and conditions.

```python
def bio_labeling(node, prev_label=None) -> List[Tuple[str, str]]:
    tokens = []
    if node.is_named:
        token_text = node.text.decode('utf-8')
        
        # Determine label based on the node type
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
        
        # Assign BIO tag
        if prev_label == label:
            tokens.append(('I-' + label, token_text))
        else:
            tokens.append(('B-' + label, token_text))
        
        prev_label = label
    else:
        tokens.append(('O-other', node.text.decode('utf-8')))
    
    for child in node.children:
        tokens.extend(bio_labeling(child, prev_label))
    
    return tokens

# Apply BIO labeling to the tokens
bio_tokens = bio_labeling(root_node)
```

### Step 3: Visualizing the AST

To understand the structure of the parsed code, you can visualize the AST using **Graphviz**. The following code generates a graphical representation of the AST:

```python
# Visualize the AST using Graphviz
def visualize_ast(node, graph, parent_id=None):
    node_id = str(id(node))
    label = f"{node.type} [{node.start_point}-{node.end_point}]"
    graph.node(node_id, label)
    if parent_id:
        graph.edge(parent_id, node_id)
    for child in node.children:
        visualize_ast(child, graph, node_id)

graph = graphviz.Digraph(format="png")
visualize_ast(root_node, graph)
graph.render("java_ast")
```

### Step 4: Multiclass Dataset Generation

You can also use this tool to generate **multiclass datasets** based on tokens and activations. The following code defines filters for different classes (e.g., keywords, function names, etc.) and assigns labels accordingly.

```python
def _create_multiclass_data(tokens, activations, class_filters, balance_data=False):
    """
    Given a list of tokens, their activations, and class_filters, create a multi-class labeled dataset.
    """
    class_words = collections.defaultdict(list)
    class_activations = collections.defaultdict(list)

    def get_class(word):
        for class_name, filter_fn in class_filters.items():
            if isinstance(filter_fn, set) and word in filter_fn:
                return class_name
            elif isinstance(filter_fn, Pattern) and filter_fn.match(word):
                return class_name
            elif callable(filter_fn) and filter_fn(word):
                return class_name
        return "negative"

    for token_type, token_text in tokens:
        class_name = get_class(token_text)
        class_words[class_name].append(token_text)

    words = []
    labels = []

    for class_name, word_list in class_words.items():
        words.extend(word_list)
        labels.extend([class_name] * len(word_list))

    return words, labels

# Define filters for creating multiclass data
class_filters = {
    "function_name": lambda x: re.match(r'\w+', x),
    "keyword": set(["public", "class", "static", "void", "System", "out", "println"]),
}

# Simulated activations
activations = [np.random.rand(len(tokens), 3, 5).tolist()]

# Annotate the data
words, labels = _create_multiclass_data(tokens, activations, class_filters)

print("\nGenerated Multiclass Labels:")
for word, label in zip(words, labels):
    print(f"Word: {word}, Label: {label}")
```

---

## Example Filters for Multiclass Dataset

```python
class_filters = {
    "method_name": lambda t, x: t == "identifier" and re.match(r'\w+', x),
    "keyword": set(["public", "class", "static", "void", "System", "out", "println"]),
    "string_literal": lambda t, x: t == "string_literal",
}
```
---
Conclusion
This tool combines Tree-sitter's ability to efficiently parse Java source code with BIO labeling and the generation of NeuroX-compatible activation files. With this setup, you can easily extract tokens from Java code, label them with meaningful BIO tags, and prepare datasets for interpretability studies.

To fully test and explore the tool’s capabilities, you can provide real input/output files for testing. Additionally, the tool automatically generates activation files, which will be useful for further analysis, but the detailed handling of these files can be added as needed.
