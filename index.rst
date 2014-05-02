.. Panels G4 documentation master file, created by
   sphinx-quickstart on Thu May  1 10:33:04 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


.. figure:: _static/uno_controller.png
   :align:  center

Introduction
=====================================

The fourth generation (G4) Panels system, currently in prototype stage, is a modular
display for designed for insect vision research. The G4 design is largely inspired
by the earlier Panels G3 design, however there are a few key differences: 

* smaller 20mm x 20mm LED matrices vs the larger 32mm x 32mm matrices used with the G3 Panels.
* SPI vs I2C for communications between the controller and panels for higher pattern transfer speeds. 
* synchronous controller driven dislay updates (patterns are only shown when the controller is sending pattern data).

Table of Contents
=====================================

.. toctree::
   :maxdepth: 2

   components.rst
   communications.rst


The source for this documentation can be found `here <http://bitbucket.org/iorodeo/panels_g4_docs/src>`_.

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

