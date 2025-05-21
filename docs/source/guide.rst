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

To compile and install different versions of GCC and Clang, you can use the provided shell scripts below:

.. code-block:: bash

   #!/bin/sh

   export REPO_PATH=/home/compiler/gcc-source
   export INSTALL_PATH=$1
   export CC=gcc
   export CXX=g++
   rm -rf $INSTALL_PATH
   mkdir $INSTALL_PATH

   git clone https://github.com/gcc-mirror/gcc.git $REPO_PATH

   cd $REPO_PATH
   git checkout $2
   ./contrib/download_prerequisites
   cd ..
   rm -rf gcc_build
   mkdir gcc_build
   cd gcc_build
   ../gcc/configure --disable-multilib --disable-bootstrap --enable-languages=c,c++ --prefix=$INSTALL_PATH --enable-coverage --disable-werror --enable-checking=yes

   make -j$(($(nproc) - 2))
   make install

Save the script above as ``build_gcc.sh`` in the ``/home/compiler`` directory. It accepts two arguments: one for the GCC installation directory and another for the GCC release version. 
For example:

.. code-block:: console

   $ ./build_gcc.sh /home/software/gcc_14.2 releases/gcc-14.2.0

The GCC installation directory id ``/home/software/gcc_14.2``, the GCC release version is ``gcc-14.2.0``. After the script finishes running, you can use the ``/home/software/gcc_14.2/bin/gcc -v`` command to verify whether ``gcc`` was successfully installed.

.. code-block:: bash

   #!/bin/sh

   export REPO_PATH=/home/compiler/llvm-source
   export INSTALL_PATH=$1
   export CC=clang
   export CXX=clang++
   rm -rf $INSTALL_PATH
   mkdir $INSTALL_PATH

   git clone https://github.com/llvm/llvm-project.git $REPO_PATH

   cd $REPO_PATH
   cd ..

   rm -rf clang_build
   mkdir clang_build
   cd clang_build

   # git -C $REPO_PATH checkout llvmorg-18.0.0
   git -C $REPO_PATH checkout $2

   cmake $REPO_PATH/llvm -G Ninja -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS=clang -DLLVM_INCLUDE_BENCHMARKS=OFF -DLLVM_INCLUDE_TESTS=OFF -DLLVM_USE_NEWPM=ON -DLLVM_TARGETS_TO_BUILD=X86 -DCMAKE_INSTALL_PREFIX=$INSTALL_PATH -DLLVM_LINK_LLVM_DYLIB=ON -DLLVM_BUILD_LLVM_DYLIB=ON -DLLVM_BUILD_INSTRUMENTED_COVERAGE=ON

   ninja -j$(($(nproc) - 2)) install

Save the script above as ``build_clang.sh`` in the ``/home/compiler`` directory. It accepts two arguments: one for the LLVM installation directory and another for the LLVM release version. 
For example:

.. code-block:: console

   $ ./build_clang.sh /home/software/clang_19.1 llvmorg-19.1.0

The LLVM installation directory id ``/home/software/clang_19.1``, the LLVM release version is ``llvmorg-19.1.0``. After the script finishes running, you can use the ``/home/software/clang_19.1/bin/clang -v`` command to verify whether ``clang`` was successfully installed.

.. note::

   You need to grant execution permission to the shell file using ``chmod +x your_script.sh`` command.