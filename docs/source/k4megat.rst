    :Author: yong

.. contents::



1 A Quick Tour
--------------

1.1 Write and run a job
~~~~~~~~~~~~~~~~~~~~~~~

Jobs are steered by a python script called ``job option`` file in Gaudi's jargon.
Users need to write their own job option to customize the job process chain.
The overall picture is simple:

1. ``import`` the services, tools and algorithms needed

2. configure each imported objects by setting their properties

3. append them to the ``AppMgr`` instance

4. finally, run this file with ``k4run your_job_file``

1.1.1 AppMgr (the entrance point)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1.2 Write a new algorithm
~~~~~~~~~~~~~~~~~~~~~~~~~

Specify following:

1.2.1 Configuration
^^^^^^^^^^^^^^^^^^^

- Gaudi\:\:Property

1.2.2 Event Data access
^^^^^^^^^^^^^^^^^^^^^^^

- Gaudi\:\:DataHandle

1.2.2.1 input
:::::::::::::

1.2.2.2 Output
::::::::::::::

1.2.3 Service access
^^^^^^^^^^^^^^^^^^^^

- Gaudi\:\:ServiceHandle

1.2.4 Processing
^^^^^^^^^^^^^^^^

- initialize()

- execute()

- finalize()

2 Gaudi Basics
--------------

2.0.1 Overview
^^^^^^^^^^^^^^

2.0.1.1 Key Concepts
::::::::::::::::::::

.. image:: gaudi_components.png

from `EIC Software Infrastructure Review <https://indico.bnl.gov/event/15644/contributions/65452/attachments/41840/70083/2022.06.29-Experience%20with%20Gaudi-2.pdf>`_

2.0.1.2 Algorithm, Service & Tool
:::::::::::::::::::::::::::::::::

2.0.2 Transient Event Store
^^^^^^^^^^^^^^^^^^^^^^^^^^^

2.0.3 Services
^^^^^^^^^^^^^^

2.0.4 Component-based programming
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

2.1 How to use
~~~~~~~~~~~~~~

2.2 Service Access
~~~~~~~~~~~~~~~~~~

Gaudi provides two API for accessing a service:

SmartIF
    general-purpose, low-level, bare-bone

ServiceHandle
    better control on Gaudi usage protocols such as

    - auto create/fetch the underlying service (lazily)

    - data race protection

    - metadata management: typeinfo, python, printing

    - exception handling

The usage is similar and both are:

- Resource Handle in general sense

- smart pointers with reference counting idiom

- easy to use and can be mixed (better following one)

Recommendation: use ServiceHandle unless there is a reason

2.2.1 Method1
^^^^^^^^^^^^^

.. code:: c++

    // declare a member in class definition
    ServiceHandle<ITargetSvc> m_svc;

    // intialize in constructor: (className, parentName)
    // actual acquisition of the service happens lazily in the check step
    MyClass::MyClass(const std::string &aName, ISvcLocator *aSvcLoc)
    : GaudiAlgorithm(aName, aSvcLoc),
      m_svc("SvcType", aName) {}

    // check validity in initialize()
    if (!m_svc) {
      error() << "some error message" << endmsg;
      return StatusCode::FAILURE;
    }

    // ... use m_svc as a pointer

.. image:: ServiceHandle.png

2.2.2 Method2
^^^^^^^^^^^^^

.. code:: c++

    // declare a member in class definition
    SmartIF<ITargetSvc> m_svc;

    // create/fetch the service
    // and check validity in initialize()
    m_svc = service("SvcType");
    if (!m_svc) {
      error() << "some error message" << endmsg;
      return StatusCode::FAILURE;
     }

    // ... use m_svc as a pointer

SmartIF has no inheritance.

2.3 Data Access
~~~~~~~~~~~~~~~

2.3.1 Data Access Checklist
^^^^^^^^^^^^^^^^^^^^^^^^^^^

· Do not delete objects that you have registered.
· Do not delete objects that are contained within an object that you have registered.
· Do not register local objects, i.e. objects NOT created with the new operator.
· Do not delete objects which you got from the store via findObject() or retrieveObject().
· Do delete objects which you create on the heap, i.e. by a call to new, and which you do not register into
a store.

2.3.2 Object Key
^^^^^^^^^^^^^^^^

- Default RootName: '/Event'

- PodioInput put collections under: '/Event', it's hardcoded

- RootNode is special

Write Mode: corret name/Path:

.. table::

    +-----------+----------------+-----------+
    | name/Path | internal       | ROOT file |
    +-----------+----------------+-----------+
    | XXX/YYY   | /Event/XXX/YYY | YYY       |
    +-----------+----------------+-----------+
    | /XXX/YYY  | /XXX/YYY       | YYY       |
    +-----------+----------------+-----------+
    | /XXX      | invalid        | \         |
    +-----------+----------------+-----------+

READ Mode: corret name/Path:

.. table::

    +-----------+----------------+-----------+
    | name/Path | internal       | ROOT file |
    +-----------+----------------+-----------+
    | XXX       | /Event/XXX     | XXX       |
    +-----------+----------------+-----------+
    | XXX/YYY   | /Event/XXX/YYY | invalid   |
    +-----------+----------------+-----------+
    | /XXX/YYY  | /Event/YYY     | YYY       |
    +-----------+----------------+-----------+
    | /XXX      | invalid        | \         |
    +-----------+----------------+-----------+

3 Event Data Model
------------------

3.1 Extension of EDM4hep
~~~~~~~~~~~~~~~~~~~~~~~~

- TPC may need special data model not provided by edm4hep

- Possible to define new data class reusing edm4hep classes

- Proposed by EIC community and `EDM4eic <https://github.com/eic/EDM4eic>`_ is a nice reference

.. image:: edm4hep_extension.png

3.2 What's in ROOT file
~~~~~~~~~~~~~~~~~~~~~~~

3.3 A Summary of PODIO/EDM4hep
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. image:: podio_edm4hep_summary.png

- `EIC Software Infrastructure Review: Data Model <https://indico.bnl.gov/event/16676/contributions/66942/attachments/42858/71974/Slides%20-%20Data%20Model.pdf>`_

- some articles by M. Frank

- Podio official doc

3.4 Future development
~~~~~~~~~~~~~~~~~~~~~~

- Current ``k4FWCore`` is limited, no MT support.

- Major updates waiting for podio\:\:Frame

- To be updated to Gaudi\:\:Functional

- Multi-threaded Event Data Service

4 Geometry model
----------------

4.1 TPC
~~~~~~~

- drift distance based on Helper surface

- readout segmentation based on Sensitive surface

  - readout pcb is forced to attach to a Sensitive surface

  - drift anode surface by default is Helper, but can be changed to Sensitive (in xml)
    if no readout pcb defined

- Multi readout PCB for pixel segmentation is supported

- Only single readout PCB allowed for strip segmentation

5 Resources
-----------

5.1 Reference projects
~~~~~~~~~~~~~~~~~~~~~~

These projects can be used as an example of using ``Key4hep`` components
and in general of how to build a NHEP experiment software.

5.1.1 EIC
^^^^^^^^^

This a gold mine, personal recommendation. Actively developed with modern C++.
The project members are also contributors to several ``Key4hep`` component package.

- NPDet

- joggler

5.1.2 FCC
^^^^^^^^^

The official demo project recommended by ``key4hep``.
The community develops ``k4FWCore`` and ``k4SimGeant4``.
Its code bases are kept in pace with latest development of ``key4hep``.

5.1.3 OpenDetector
^^^^^^^^^^^^^^^^^^

A experiment neutral detector aims to be used as a testbed for ``ACTS``.
It's built upon ``DD4hep`` and is kept in pace with the two packages latest development.

5.2 About Gaudi
~~~~~~~~~~~~~~~

The `official documentation <https://gaudi-framework.readthedocs.io/en/latest/>`_ is a combination of legacy compatibility and latest development.
But it provides a very nice overview of the architecture design and key building blocks.
Not needed for end user, recommend for average developer, a must read for software builder/maintainer.

LHCb provides `some tutorial for Gaudi & Modern C++ <https://lhcb.github.io/DevelopKit/>`_

5.3 Others
~~~~~~~~~~

5.3.1 Software build
^^^^^^^^^^^^^^^^^^^^

- `modern cmake <https://cliutils.gitlab.io/modern-cmake/chapters/install/exporting.html>`_

- git

5.3.2 Modern C++
^^^^^^^^^^^^^^^^

- `Cheatsheat of using modern C++ <https://github.com/BartVandewoestyne/Effective-Modern-Cpp>`_

  - 中文学习笔记以及示例代码，作者其它的笔记也值得一读

- `现代C++教程：高速上手C++11/14/17/20 <https://changkun.de/modern-cpp/>`_

- `C++ Rvalue References Explained <http://thbecker.net/articles/rvalue_references/section_01.html>`_

  - what, how, why of move and perfect forward

  - `more examples <https://www.artima.com/articles/a-brief-introduction-to-rvalue-references>`_

- `Cpp Guideline <http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines>`_

  - all the best practices of using modern C++

  - Maybe the ultimate source of reference?

- `More C++ Idioms <https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms>`_
