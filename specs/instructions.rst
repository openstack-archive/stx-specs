..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License.

  http://creativecommons.org/licenses/by/3.0/legalcode

.. index::
   single: instructions
   single: getting started

.. _instructions:

============
Instructions
============

- Use the template.rst as the basis of your specification.
- Attempt to detail each applicable section.
- If a section does not apply, use N/A, and optionally provide
  a short explanation.
- New specs for review should be placed in the ``approved`` subfolder, where
  they will undergo review and approval in Gerrit_.
- Specs that have finished implementation should be moved to the
  ``implemented`` subfolder.

Indexing and Categorization
---------------------------

Use of the `index`_ directive in reStructuredText for each document provides
the ability to generate indexes to more easily find items later. Authors are
encouraged to use index entries for their documents to help with discovery.

Naming
------

Document naming standards help readers find specs. For the StarlingX repository,
the following document naming is recommended. The categories listed here are
likely incomplete, and may need expansion to cover new cases. It is preferrable
to deviate (and hopefully amend the list) than force document names into
nonsense categories. Prefer using categories that have previously been used or
that are listed here over new categories, but don't force the category into
something that doesn't make sense.

Document names should follow a pattern as follows::

  [category]_title.rst

Use the following guidelines to determine the category to use for a document:

1) For new functionality and features, the best choice for a category is to
   match a functional duty of StarlingX.

2) For specs that are not feature focused, the component of the system may
   be the best choice for a category, e.g. ``sysinv``, ``armada`` etc...
   When there are multiple components involved, or the concern is cross
   cutting, use of ``starlingx`` is an acceptable category.

3) If the spec is related to the ecosystem StarlingX is maintained within, an
   appropriate category would be related to the aspect it is impacting, e.g.:
   ``git``, ``docker``, ``zuul``, etc...

.. _index: http://www.sphinx-doc.org/en/stable/markup/misc.html#directive-index
.. _Gerrit: https://review.openstack.org/#/q/project:openstack/stx-specs
