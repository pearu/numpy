======================
Advanced F2PY usages
======================

Adding self-written functions to F2PY generated modules
=======================================================

Self-written Python C/API functions can be defined inside
signature files using ``usercode`` and ``pymethoddef`` statements
(they must be used inside the ``python module`` block). For
example, the following signature file ``spam.pyf``

.. include:: spam.pyf
   :literal:

wraps the C library function ``system()``::

  f2py -c spam.pyf

In Python:

.. include:: spam_session.dat
   :literal:

Modifying the dictionary of a F2PY generated module
===================================================

The following example illustrates how to add user-defined
variables to a F2PY generated extension module. Given the following
signature file

.. include:: var.pyf
  :literal:

compile it as ``f2py -c var.pyf``.

Notice that the second ``usercode`` statement must be defined inside
an ``interface`` block and where the module dictionary is available through
the variable ``d`` (see ``f2py var.pyf``-generated ``varmodule.c`` for
additional details).

In Python:

.. include:: var_session.dat
  :literal:


Dealing with KIND specifiers
============================

Currently, F2PY can handle only ``<type spec>(kind=<kindselector>)``
declarations where ``<kindselector>`` is a numeric integer (e.g. 1, 2,
4,...), but not a function call ``KIND(..)`` or any other
expression. F2PY needs to know what would be the corresponding C type
and a general solution for that would be too complicated to implement.

However, F2PY provides a hook to overcome this difficulty, namely,
users can define their own <Fortran type> to <C type> maps. For
example, if Fortran 90 code contains::

    REAL(kind=KIND(0.0D0)) ...

then create a mapping file containing a Python dictionary::

    {'real': {'KIND(0.0D0)': 'double'}}

for instance.

Use the ``--f2cmap`` command-line option to pass the file name to F2PY.
By default, F2PY assumes file name is ``.f2py_f2cmap`` in the current
working directory.

Or more generally, the f2cmap file must contain a dictionary
with items::

    <Fortran typespec> : {<selector_expr>:<C type>}

that defines mapping between Fortran type::

    <Fortran typespec>([kind=]<selector_expr>)

and the corresponding <C type>. <C type> can be one of the following::

    char
    signed_char
    short
    int
    long_long
    float
    double
    long_double
    complex_float
    complex_double
    complex_long_double
    string

For more information, see F2Py source code ``numpy/f2py/capi_maps.py``.

.. _Character strings:

Character strings
=================

Assumed length chararacter strings
-----------------------------------

In Fortran, assumed length character string arguments are declared as
``character*(*)`` or ``character(len=*)``, that is, the length of such
arguments are determined by the actual string arguments in runtime.
For ``intent(in)`` arguments, this lack of length information poses no
problems for f2py to construct functional wrapper functions. However,
for ``intent(out)`` arguments, the lack of length information is
problematic for f2py generated wrappers because there is no size
information available for creating memory buffers for such arguments
and F2PY assumes the length is 0.  Depending on how the length of
assumed length character strings can be specified, there exists ways
to workaround this problem, as exemplified below.

If the length of ``character*(*)`` output argument is determined by
the state of other input arguments, the required connection can be
established in a signature file or within a f2py-comment by adding
extra declaration for the corresponding argument that specifies the
length in character selector part. For example, consider a Fortran
file ``asterisk1.f90``:

.. include:: asterisk1.f90
  :literal:

Compile it with ``f2py -c asterisk1.f90 -m asterisk1`` and then in Python:

.. include:: asterisk1_session.dat
  :literal:

Notice that the extra declaration ``character(f2py_len=12) s`` is
interpreted only by f2py and in the ``f2py_len=`` specification one
can use C-expressions as a length value.

In the following example:

.. include:: asterisk2.f90
  :literal:

the lenght of output assumed length string depends on an input
argument ``n``, after wrapping with F2PY, in Python:

.. include:: asterisk2_session.dat
  :literal:
