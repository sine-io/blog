I’ve been working with Python bytecode recently, and wanted to share some of my experience working with it. To be more precise, I’ve been working exclusively on the bytecode for the CPython interpreter, and limited to versions 2.6 and 2.7.

Python is a dynamic language, and running it from the command line essentially triggers the following steps:

- The source is compiled the first time it is encountered (e.g., imported as a module or directly executed). This step generates the binary file, with a `pyc` or `pyo` extension depending on your system.
- The interpreter reads the binary file and executes the instructions (opcodes) one at a time.

The python interpreter is stack-based, and to understand the dataflow, we need to know what the stack effect is of each instruction (i.e., opcode and argument).

### Inspecting a Python binary file

The simplest way to get the bytecode of a binary file is to unmarshall the `CodeType` structure:

```python
import marshal
fd = open('path/to/my.pyc', 'rb')
magic = fd.read(4) # python version specific magic num
date = fd.read(4)  # compilation date
code_object = marshal.load(fd)
fd.close()
```

The `code_object` now contains a `CodeType` object which represents the entire module from the loaded file. To inspect all nested code objects from this module, meaning class declarations, methods, etc. we need to recursively inspect the const pool from the `CodeType`; that means doing something like this:

```python
import types

def inspect_code_object(co_obj, indent=''):
  print indent, "%s(lineno:%d)" % (co_obj.co_name, co_obj.co_firstlineno)
  for c in co_obj.co_consts:
    if isinstance(c, types.CodeType):
      inspect_code_object(c, indent + '  ')

inspect_code_object(code_object) # We resume from the previous snippet
```

In this case, we’ll print a tree of code objects nested under their respective parents. For the following simple code:

```python
class A:
  def __init__(self):
    pass
  def __repr__(self):
    return 'A()'
a = A()
print a
```

We’ll get the tree:

```shell
 <module>(lineno:2)
   A(lineno:2)
     __init__(lineno:3)
     __repr__(lineno:5)
```

For testing, we can get the code object from a string that contains the Python source code by using the `compile`directive:

```python
co_obj = compile(python_source_code, '<string>', 'exec')
```

For more inspection of the code object, we can have a look at the [co_* fields](https://docs.python.org/2/library/inspect.html#types-and-members) from the Python documentation.

### First look into the bytecode

Once we get the code objects, we can actually start looking at the disassembly of it (in the `co_code` field). Parsing the bytecode to make sense out of it means:

- Interpreting what the opcode means
- Dereference any argument

The [disassemble function in the dis module](https://hg.python.org/cpython/file/2.7/Lib/dis.py#l61) shows how to do that. It will actually provide the following output from our previous code example:

```shell
2   0 LOAD_CONST        0 ('A')
    3 LOAD_CONST        3 (())
    6 LOAD_CONST        1 (<code object A at 0x42424242, file "<string>", line 2>)
    9 MAKE_FUNCTION     0
   12 CALL_FUNCTION     0
   15 BUILD_CLASS
   16 STORE_NAME        0 (A)

8  19 LOAD_NAME         0 (A)
   22 CALL_FUNCTION     0
   25 STORE_NAME        1 (a)

9  28 LOAD_NAME         1 (a)
   31 PRINT_ITEM
   32 PRINT_NEWLINE
   33 LOAD_CONST        2 (None)
   36 RETURN_VALUE
```

Where we get:

- The line number (when it changed)
- The index of the instruction
- The opcode of the current instruction
- The oparg, which is what the opcode takes to resolve to the actual argument, it knows where to look based on the opcode. For example, with a `LOAD_NAME` opcode, the oparg will point to the index in the `co_names` tuple.
- The resolved argument in parentheses

As we can see at the index 6, the `LOAD_CONST` opcode takes an oparg that points to which object should be loaded from the `co_consts` tuple. Here, it points to the type declaration of `A`. Recursively, we can go and decompile all code objects to get the full bytecode of the module.

The first part of the bytecode (index 0 to 16) relates to the type declaration of `A` while the rest represents the code where we instantiate an `A` and print it. Even in this code, there are constructs that are not relevant unless you plan on modifying the bytecode and changing types, etc.

### Interesting bytecode constructs

The [overall opcodes](https://docs.python.org/2/library/dis.html#python-bytecode-instructions) are fairly straight forward, but a few cases seem weird as they might come from:

- Compiler optimizations
- Interpreter optimizations (therefore leading to extra opcodes)

##### Variables assignment with sequences

In the first category, we can have a look at what happens when the source assign sequences of variables:

```
(1) a, b = 1, '2'
(2) a, b = 1, e
(3) a, b, c = 1, 2, e
(4) a, b, c, d = 1, 2, 3, e
```

These 4 statements produce quite a different bytecode.

The first case is the simplest one since the right-hand side (RHS) of the assignment contains only constants. In that case, CPython can create the tuple `(1, '2')`, use `UNPACK_SEQUENCE` to put 2 elements on the stack, and create a `STORE_FAST` for each variable `a` and `b`:

```
0 LOAD_CONST               5 ((1, '2'))
3 UNPACK_SEQUENCE          2
6 STORE_FAST               0 (a)
9 STORE_FAST               1 (b)
```

The second case however introduce a variable on the RHS, so the generic case is called where an expression is fetched (here, a simple one with a `LOAD_GLOBAL`). The compiler however does not need to create a new tuple from the values on the stack (at index 18) and use an `UNPACK_SEQUENCE`; it’s sufficient to call the `ROT_TWO` which swaps the 2 top elements from the stack (it might have been enough to switch 19 and 22 though):

```
12 LOAD_CONST               1 (1)
15 LOAD_GLOBAL              0 (e)
18 ROT_TWO
19 STORE_FAST               0 (a)
22 STORE_FAST               1 (b)
```

The third case is where it becomes really strange. Putting the expressions on the stack is exactly the same mechanism as in the previous case, but after it first swap the 3 top elements, then swap again the 2 top elements:

```
25 LOAD_CONST               1 (1)
28 LOAD_CONST               3 (2)
31 LOAD_GLOBAL              0 (e)
34 ROT_THREE
35 ROT_TWO
36 STORE_FAST               0 (a)
39 STORE_FAST               1 (b)
42 STORE_FAST               2 (c)
```

The final one represents the generic case, where no more `ROT_*`-play seems possible and a tuple is created and then a call to `UNPACK_SEQUENCE` to put them on the stack:

```
45 LOAD_CONST               1 (1)
48 LOAD_CONST               3 (2)
51 LOAD_CONST               4 (3)
54 LOAD_GLOBAL              0 (e)
57 BUILD_TUPLE              4
60 UNPACK_SEQUENCE          4
63 STORE_FAST               0 (a)
66 STORE_FAST               1 (b)
69 STORE_FAST               2 (c)
72 STORE_FAST               3 (d)
```

##### Call constructs

The last set of interesting examples are around the call constructs and the [4 different opcodes](https://docs.python.org/2/library/dis.html#opcode-CALL_FUNCTION) to create calls. I suppose the number of opcodes is to optimize the interpreter code, since it’s not like in Java where it makes sense to have one of the `invokedynamic`, `invokeinterface`, `invokespecial`, `invokestatic`, or `invokevirtual`.

In Java, `invokeinterface`, `invokespecial` and `invokevirtual` are originally coming from the static typing of the language (and `invokespecial` is only used for calling constructors and superclasses AFAIK). `invokestatic`is self describing (no need to put the receiver on the stack) and there is no such concept (down to the interpreter and not through decorators) in Python. In short, Python calls could always be translated with an `invokedynamic`.

The different `CALL_*` opcodes in Python are indeed not here because of typing, static methods, or the need to have a special access for constructors. They are all targeting on how a method call can be specified in Python; from the grammar:

```
  Call(expr func, expr* args, keyword* keywords,
       expr? starargs, expr? kwargs)
```

The calls structure allow for code like this:

```
func(arg1, arg2, keyword=SOME_VALUE, *unpack_list, **unpack_dict)
```

The keyword arguments allow for passing formal parameters by name and not just position, the `*` puts all elements from the iterable as arguments (inlined, not in a tuple), and the `**` expects a dictionary of keywords with values.

This example actually uses all possible features of the call site construction:

- Variables argument list passing (`_VAR`): `CALL_FUNCTION_VAR`, `CALL_FUNCTION_VAR_KW`
- Keyword based dict passing (`_KW`): `CALL_FUNCTION_KW`, `CALL_FUNCTION_VAR_KW`

The bytecode looks like this:

```
 0 LOAD_NAME                0 (func)
 3 LOAD_NAME                1 (arg1)
 6 LOAD_NAME                2 (arg2)
 9 LOAD_CONST               0 ('keyword')
12 LOAD_NAME                3 (SOME_VALUE)
15 LOAD_NAME                4 (unpack_list)
18 LOAD_NAME                5 (unpack_dict)
21 CALL_FUNCTION_VAR_KW   258
```

Usually, a `CALL_FUNCTION` takes as oparg the number of arguments for the function. Here however, more information is encoded. The first byte (`0xff` mask) carries the number of arguments and the second one (`(value >> 8) & 0xff`) the number of keyword arguments passed. To compute the number of elements to pop from the stack, we then need to get:

```
na = arg & 0xff         # num args
nk = (arg >> 8) & 0xff  # num keywords
n_to_pop = na + 2 * nk + CALL_EXTRA_ARG_OFFSET[op]
```

where `CALL_EXTRA_ARG_OFFSET` contains an offset specific to the call opcode (2 for `CALL_FUNCTION_VAR_KW`). Here, that gives us 6, the number of elements to pop before accessing the function name.

To relate to other `CALL_*` keywords, it then all depends if the code is either using the list passing or dictionary passing argument; it’s all about combination here!

### Building a minimal CFG

For understanding how the code actually works, it’s interesting to build a control-flow graph (CFG) so we can follow which unconditional sequences of opcodes (basic blocks) will be executed, and under what conditions.

Even if the bytecode is a fairly small language, building a reliable CFG requires more details than this blog post can allow, so for an actual implementation of a CFG construction, you can have a look at [equip](https://github.com/neuroo/equip/blob/master/equip/analysis/flow.py#L170).

Here, we’ll focus on loop/exception free code, where the control flow only depends on if statements.

There are a handful of opcodes that carry a jump address (for non-loop/exceptions); they are:

- `JUMP_FORWARD`: Relative jump in the bytecode. Takes the amount of bytes to skip.
- `JUMP_IF_FALSE_OR_POP`, `JUMP_IF_TRUE_OR_POP`, `JUMP_ABSOLUTE`, `POP_JUMP_IF_FALSE`, and `POP_JUMP_IF_TRUE` all take absolute index in the bytecode.

Building the CFG for a function means creating basic blocks (sequence of opcodes that have unconditional execution — except when an exception can occur), and connecting them in a graph that contains conditions on branches. In our case, we only have `True`, `False`, and `Unconditional` branches.

Let’s consider the following code example (which should never be used in practice):

```
def factorial(n):
  if n <= 1:
    return 1
  elif n == 2:
    return 2
  return n * factorial(n - 1)
```

As mentioned before, we get the code object for the `factorial` method:

```
module_co = compile(python_source, '<string>', 'exec')
meth_co = module_co.co_consts[0]
```

The disassembly looks like this (minus my annotations):

```
3           0 LOAD_FAST                0 (n)
            3 LOAD_CONST               1 (1)
            6 COMPARE_OP               1 (<=)
            9 POP_JUMP_IF_FALSE       16              <<< control flow

4          12 LOAD_CONST               1 (1)
           15 RETURN_VALUE                            <<< control flow

5     >>   16 LOAD_FAST                0 (n)
           19 LOAD_CONST               2 (2)
           22 COMPARE_OP               2 (==)
           25 POP_JUMP_IF_FALSE       32              <<< control flow

6          28 LOAD_CONST               2 (2)
           31 RETURN_VALUE                            <<< control flow

7     >>   32 LOAD_FAST                0 (n)
           35 LOAD_GLOBAL              0 (factorial)
           38 LOAD_FAST                0 (n)
           41 LOAD_CONST               1 (1)
           44 BINARY_SUBTRACT
           45 CALL_FUNCTION            1
           48 BINARY_MULTIPLY
           49 RETURN_VALUE                            <<< control flow
```

In this bytecode, we have 5 instructions that change the structure of the CFG (so adds constraints or allows for quick exit):

- `POP_JUMP_IF_FALSE`: Jump to the absolute index 16 and 32,
- `RETURN_VALUE`: Pop one element from the stack and returns it.

Extracting the basic blocks becomes easy since these instructions that change the control flow are the only one we’re interested in detecting. In our case, we don’t have jumps that impose no fall-through, but `JUMP_FORWARD` or `JUMP_ABSOLUTE` do that.

Example code to extract such structure:

```
import opcode
RETURN_VALUE = 83
JUMP_FORWARD, JUMP_ABSOLUTE = 110, 113
FALSE_BRANCH_JUMPS = (111, 114) # JUMP_IF_FALSE_OR_POP, POP_JUMP_IF_FALSE

def find_blocks(meth_co):
  blocks = {}
  code = meth_co.co_code
  finger_start_block = 0
  i, length = 0, len(code)
  while i < length:
    op = ord(code[i])
    i += 1
    if op == RETURN_VALUE: # We force finishing the block after the return,
                           # dead code might still exist after though...
      blocks[finger_start_block] = {
        'length': i - finger_start_block - 1,
        'exit': True
      }
      finger_start_block = i
    elif op >= opcode.HAVE_ARGUMENT:
      oparg = ord(code[i]) + (ord(code[i+1]) << 8)
      i += 2
      if op in opcode.hasjabs: # Absolute jump to oparg
        blocks[finger_start_block] = {
          'length': i - finger_start_block
        }
        if op == JUMP_ABSOLUTE: # Only uncond absolute jump
          blocks[finger_start_block]['conditions'] = {
            'uncond': oparg
          }
        else:
          false_index, true_index = (oparg, i) if op in FALSE_BRANCH_JUMPS else (i, oparg)
          blocks[finger_start_block]['conditions'] = {
            'true': true_index,
            'false': false_index
          }
        finger_start_block = i
      elif op in opcode.hasjrel:
        # Essentially do the same...
        pass

  return blocks
```

And we get the following basic blocks:

```
Block  0: {'length': 12, 'conditions': {'false': 16, 'true': 12}}
Block 12: {'length': 3, 'exit': True}
Block 16: {'length': 12, 'conditions': {'false': 32, 'true': 28}}
Block 28: {'length': 3, 'exit': True}
Block 32: {'length': 17, 'exit': True}
```

With the current structure of the blocks:

```
Basic blocks
  start_block_index :=
     length     := size of instructions
     condition  := true | false | uncond -> target_index
     exit*      := true
```

we have our control flow graph (minus the entry and implicit return blocks), and we can for example convert it to dot for visualization:

```
def to_dot(blocks):
  cache = {}

  def get_node_id(idx, buf):
    if idx not in cache:
      cache[idx] = 'node_%d' % idx
      buf.append('%s [label="Block Index %d"];' % (cache[idx], idx))
    return cache[idx]

  buffer = ['digraph CFG {']
  buffer.append('entry [label="CFG Entry"]; ')
  buffer.append('exit  [label="CFG Implicit Return"]; ')

  for block_idx in blocks:
    node_id = get_node_id(block_idx, buffer)
    if block_idx == 0:
      buffer.append('entry -> %s;' % node_id)
    if 'conditions' in blocks[block_idx]:
      for cond_kind in blocks[block_idx]['conditions']:
        target_id = get_node_id(blocks[block_idx]['conditions'][cond_kind], buffer)
        buffer.append('%s -> %s [label="%s"];' % (node_id, target_id, cond_kind))
    if 'exit' in blocks[block_idx]:
      buffer.append('%s -> exit;' % node_id)

  buffer.append('}')
  return '\n'.join(buffer)
```

### Why bother?

It’s indeed fairly rare to only have access to the Python bytecode, but I’ve had this case a few times in the past. Hopefully, this information can help someone starting a reverse engineering project on Python.

Right now however, I’ve been investigating the ability to instrument Python code, and especially its bytecode since there are no facilities for doing so in Python (and instrumenting source code often leaves with very inefficient instrumentation code with decorators, etc.). That’s where [equip](https://github.com/neuroo/equip) comes from.