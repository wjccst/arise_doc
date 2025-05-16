Guide
==========

.. note::

   All the following code are executed in the docker container.

You can use the following command to enter the container:

.. code-block:: console

   $ docker exec -it arise /bin/bash


Once inside the container, navigate to the appropriate directory and execute the following commands:


.. code-block:: console

   $ cd /home/code

The files in the ``code`` directory are as follows:

.. code-block:: text

    code/
    ├── InsertAttr/
    │   ├── clang_attributes.py
    │   ├── gcc_attributes.py
    │   ├── insert_attributes_clang.py # Used to insert clang related attributes
    │   ├── insert_attributes_gcc.py   # Used to insert gcc related attributes
    │   ├── optimization.py
    │   ├── sanitize.py
    │   └── utils.py
    ├── some_tool/
    │   ├── test.c                     # Used to test the tools
    │   ├── tool_for_clang.c           # Used to parse program's ast for clang
    │   └── tool_for_gcc.c             # Used to parse program's ast for gcc
    └── ──

First, you need a tool to parse the AST (Abstract Syntax Tree) of C programs. You can compile this tool using the following command:

.. code-block:: console

   $ clang-16 tool_for_clang.c  -L/usr/lib/llvm-16/lib -lclang -I/usr/lib/llvm-16/include -o tool_clang  #for clang
   $ clang-16 tool_for_gcc.c  -L/usr/lib/llvm-16/lib -lclang -I/usr/lib/llvm-16/include -o tool_gcc      #for gcc

After compilation, you will obtain the AST parsing tool.

To parse a C program using this tool, take ``test.c`` in the ``some_tool`` directory as an example. Run the following command:

.. code-block:: console

   $ ./tool_clang test.c

The output will include: 

.. code-block:: text
    
    1. Type information of variables
    2. Source locations of variables
    3. Source locations of functions
    4. Return types of functions
    5. Function parameter lists
    6. Source locations of loops
    7. ...

Next, run the following command to insert attributes into the program and test it using a given compiler

.. code-block:: console
   $ export LD_LIBRARY_PATH=/home/software/gcc-trunk/lib64:$LD_LIBRARY_PATH     #set up the sanitizer environment variables
   $ python3 insert_attributes_gcc.py --compiler gcc --source 0 --multi 0       #for gcc
   $ python3 insert_attributes_clang.py --compiler clang --source 0 --multi 0   #for clang

The instrumented program can be accessed in the ``/home/code/InsertAttr/work`` directory. The ``work`` directory contains many subfolders, each named with a randomly generated 8-character string. Each folder contains both the original program and the instrumented mutants.

Once the process starts, a results directory named in the format ``Arise-<timestamp>`` will be generated in the current working directory. The contents of this folder are organized as follows:

.. code-block:: text

    Arise-<timestamp>/
    ├── bug/        # contains test cases that triggered misoptimization bugs
    ├── crash/      # contains test cases that triggered compiler crashes
    ├── BUG.log     # describes each discovered bug
    ├── ERR.log     # logs errors encountered during execution
    └── INFO.log    # records compilation details, including the number of successfully compiled variants for each test case

.. note::

   If any errors occur while running the Python code, you can find the source code files from the [`Zenodo <https://doi.org/10.5281/zenodo.14645190>`_] link and replace all Python files in directory ``code`` with the ones from the link, then rerun the Python code.