Python Internals
****************

:date: 2014-12-19 19:48
:update: 2014-12-20 9:52

.. code-block:: python

    In [18]: import dis
    In [19]: dis.dis(lambda x: x + 2)
    1           0 LOAD_FAST                0 (x)
                3 LOAD_CONST               1 (2)
                6 BINARY_ADD
                7 RETURN_VALUE

Search for `BINARY_ADD` opcode in `ceval.c`

.. code-block:: bash

    $ cpython  ack --with-filename BINARY_ADD Python/ceval.c
    Python/ceval.c
    1543:        TARGET(BINARY_ADD) {


The `BINARY_ADD` in `Python/ceval.c`.

.. code-block:: c

    :emphasize-lines: 3,5
    :linenos:

    TARGET(BINARY_ADD) {
        PyObject *right = POP();
        PyObject *left = TOP();
        PyObject *sum;
        if (PyUnicode_CheckExact(left) &&
                    PyUnicode_CheckExact(right)) {
            sum = unicode_concatenate(left, right, f, next_instr);
            /* unicode_concatenate consumed the ref to v */
        }
        else {
            sum = PyNumber_Add(left, right);
            Py_DECREF(left);
        }
        Py_DECREF(right);
        SET_TOP(sum);
        if (sum == NULL)
            goto error;
        DISPATCH();
    }

