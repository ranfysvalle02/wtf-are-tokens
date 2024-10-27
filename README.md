# wtf-are-tokens
Tokens are not just for LLMs. They are about capturing meaning. Tokens in NLP: Tokens are fundamental units in natural language processing (NLP). They are the smallest meaningful units of language, such as words, parts of words (subwords), or even characters. 

# Understanding Tokenization Beyond LLMs: Building a Simple Query Interpreter in Python

Tokenization isn't a concept exclusive to Large Language Models (LLMs) like GPT-4; it's a foundational element in computer science, particularly in the realms of compilers, interpreters, and data parsing. In this blog post, we'll explore how tokenization plays a critical role in interpreting and executing queries by dissecting a Python script that builds a simple query interpreter from scratch.

## Table of Contents

1. [Introduction](#introduction)
2. [The Importance of Tokenization](#the-importance-of-tokenization)
3. [Overview of the Script](#overview-of-the-script)
4. [Tokenization Process](#tokenization-process)
5. [Abstract Syntax Tree (AST)](#abstract-syntax-tree-ast)
6. [Parsing the Tokens](#parsing-the-tokens)
7. [Interpreting the AST](#interpreting-the-ast)
8. [Putting It All Together](#putting-it-all-together)
9. [Conclusion](#conclusion)
10. [Further Reading](#further-reading)

## Introduction

Tokenization is the process of breaking down a stream of text into meaningful units called tokens. While it's a critical step in training and using LLMs, tokenization has been a fundamental concept in computing long before the advent of AI models. This blog post aims to demystify tokenization by walking through a Python script that interprets simple database queries.

## The Importance of Tokenization

In the context of programming languages and data parsing, tokenization serves as the first step in understanding and processing input data. By converting raw text into tokens, we enable programs to:

- **Understand Syntax**: Recognize keywords, operators, and identifiers.
- **Build Structure**: Facilitate the creation of an Abstract Syntax Tree (AST).
- **Execute Logic**: Allow interpreters or compilers to execute code based on the parsed tokens.

## Overview of the Script

The Python script we're examining demonstrates how to:

- **Tokenize** a simple query language.
- **Parse** the tokens into an AST.
- **Interpret** the AST to execute the query against a sample dataset.

Here's a high-level view of the script components:

- **Dataset**: A list of dictionaries representing documents.
- **Tokens**: Definitions for recognizing different parts of the query.
- **Tokenizer**: A function to break the query into tokens.
- **AST Nodes**: Classes representing different parts of the parsed query.
- **Parser**: Converts tokens into an AST.
- **Interpreter**: Evaluates the AST against the dataset.
- **Main Function**: Orchestrates the tokenization, parsing, and interpretation.

## Tokenization Process

### Defining Tokens

Tokens are defined using regular expressions:

```python
TOKENS = [
    ('DB',         r'db'),
    ('DOT',        r'\.'),
    ('IDENTIFIER', r'[a-zA-Z_][a-zA-Z0-9_]*'),
    ('LPAREN',     r'\('),
    ('RPAREN',     r'\)'),
    ('SKIP',       r'[ \t\n]+'),
    ('FILTER',     r'.+'),  # Capture the rest of the line as the filter
]
```

Each tuple in `TOKENS` consists of a token name and its corresponding regex pattern.

### The Tokenizer Function

The `tokenize` function uses these patterns to break the input query into tokens:

```python
def tokenize(query):
    token_regex = '|'.join('(?P<%s>%s)' % pair for pair in TOKENS)
    get_token = re.compile(token_regex).match
    # Tokenization logic...
```

- **Compiling Regex**: The patterns are combined into a single regex using named groups.
- **Matching Tokens**: The function iteratively matches tokens from the input query.
- **Handling Unrecognized Characters**: If an unexpected character is encountered, a `SyntaxError` is raised.

### Example

Given the query:

```python
query = 'db.collection.find({"age": {"$gt": 25}, "city": "New York"})'
```

The tokenizer would produce tokens like:

```
[('DB', 'db'), ('DOT', '.'), ('IDENTIFIER', 'collection'), ('DOT', '.'), ('IDENTIFIER', 'find'), ('LPAREN', '('), ('FILTER', '{"age": {"$gt": 25}, "city": "New York"})')]
```

## Abstract Syntax Tree (AST)

An AST is a tree representation of the syntactic structure of the source code. In our script, we define AST nodes:

```python
class ASTNode:
    pass

class Query(ASTNode):
    def __init__(self, collection, filter_expr):
        self.collection = collection
        self.filter_expr = filter_expr  # Dictionary parsed from JSON
```

The `Query` node represents a database query with a collection name and a filter expression.

## Parsing the Tokens

The parser takes the list of tokens and builds the AST.

### The Parser Class

```python
class Parser:
    def __init__(self, tokens):
        self.tokens = tokens
        self.pos = 0

    # Parsing methods...
```

### Parsing Logic

- **Current Token**: Retrieves the token at the current position.
- **Eating Tokens**: The `eat` method advances the position if the expected token is found.
- **Building the AST**: The `parse_query` method constructs the `Query` node.

### Handling the Filter Expression

The filter expression is expected to be valid JSON:

```python
filter_str = token_value.rstrip(')')
try:
    filter_expr = json.loads(filter_str)
except json.JSONDecodeError as e:
    raise SyntaxError(f"Invalid JSON filter: {e}")
```

If the JSON is invalid, a `SyntaxError` is raised.

## Interpreting the AST

The interpreter evaluates the AST against the dataset.

### The Interpreter Class

```python
class Interpreter:
    def __init__(self, dataset):
        self.dataset = dataset

    def evaluate(self, node):
        if isinstance(node, Query):
            return self.evaluate_query(node)
        else:
            raise ValueError(f'Unknown node type: {type(node)}')
```

### Evaluation Logic

- **evaluate_query**: Filters the dataset based on the filter expression.
- **match_filter**: Checks if a document matches the filter conditions.
- **match_condition**: Handles field-level conditions, including logical operators like `$gt`, `$lt`, `$and`, `$or`, and `$not`.

### Supported Operators

- Comparison: `$gt`, `$lt`, `$gte`, `$lte`, `$eq`, `$ne`
- Logical: `$and`, `$or`, `$not`

### Example Evaluation

For the filter:

```python
{"age": {"$gt": 25}, "city": "New York"}
```

The interpreter would return documents where `age > 25` and `city` is `"New York"`.

## Putting It All Together

### The Main Function

```python
def main():
    # Example query
    query = 'db.collection.find({"age": {"$gt": 25}, "city": "New York"})'
    tokens = tokenize(query)
    parser = Parser(tokens)
    # Parsing and interpreting logic...
```

### Execution Flow

1. **Tokenization**: Breaks the query into tokens.
2. **Parsing**: Builds the AST from the tokens.
3. **Interpretation**: Evaluates the AST against the dataset.
4. **Output**: Prints the query results.

### Sample Output

```json
Query Results:
{
    "_id": 1,
    "name": "Alice",
    "age": 30,
    "city": "New York"
}
```

## Conclusion

This script illustrates how tokenization and parsing work hand-in-hand to interpret and execute queries. Tokenization breaks down the input into manageable pieces, which are then structured into an AST by the parser. The interpreter executes the logic defined by the AST against the dataset.

**Key Takeaways**:

- **Tokenization is Fundamental**: It's a critical step in understanding and processing any structured input, not just in LLMs.
- **AST Simplifies Execution**: By representing the query as a tree, we can easily navigate and evaluate complex expressions.
- **Extensibility**: The script can be extended to support more operators and complex queries.

## Further Reading

- **Compiler Design Books**: For a deeper understanding of tokenization and parsing.
- **Python `re` Module Documentation**: To learn more about regular expressions in Python.
- **Abstract Syntax Trees**: Explore how ASTs are used in various programming languages.

---

By appreciating the role of tokenization beyond the realm of LLMs, we gain a deeper understanding of how computers interpret and execute instructions—a foundational concept that underpins much of computer science.
