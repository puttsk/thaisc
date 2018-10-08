=============
Contributing
=============

First off, thank you for considering contributing to our documentation. Please read the following sections in order to know how to ask questions and how to work on something.

Contributing to User Guides and Tutorials
==========================================

If you are using any specific software on any ThaiSC platform, you might already developed

*  a set of scripts to efficiently run the software
*  an effient workflow to work with the software
*  troubleshooting steps for your software


Then your experiece will be valuable for the other users and we would appreciate your help to complete this document with new topics/entries.

To do that, you first need to :ref:`install <installation>` this repository to your local machine.


.. _installation: 

Installation 
=============

Here is a step by step plan on how to install this repository and generate your local copy of this document.

First, obtain `Python 3.6 <http://www.python.org/>`_ and `virtualenv <http://pypi.python.org/pypi/virtualenv>`_ if you do not already have them. Using a virtual environment will make the installation easier, and will help to avoid clutter in your system-wide libraries. You will also need `Git <http://git-scm.com/>`_ in order to clone the repository. 

First, you need to clone the repository using following command 

.. code:: bash

    git clone https://github.com/puttsk/thaisc.git
    cd thaisc

Next, you will need to verify that your ``pip`` version is higher by using

.. code:: bash

    pip --version

If the version is lesser than 18, you should upgrade ``pip`` before continuing.

.. code:: bash

    pip install --upgrade pip

Once you have these, create a virtual environment inside the directory, then activate it:

.. code:: bash

    virtualenv venv
    source venv/bin/activate

Next, install the dependencies using ``pip``

.. code:: bash
    
    pip install -r requirements.txt

The source code of the document is in the ``/docs`` directory. To build your local document

.. code:: bash

    cd docs
    make html           # For building HTML document
    make latexpdf       # For building PDF document

The HTML document will be in ``./docs/_build/html/index.html`` directory and the PDF document will be in ``./docs/_build/latex/ThaiSCDocumentation.pdf``.







