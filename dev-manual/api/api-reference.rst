.. _am-api-reference.rst:

==========
Reference
==========

Quickly review all available resources for the Archivematica API and Storage Service API with this reference.

Archivematica API
==================

The Archivematica API provides:

    * Coverage of some basic workflow functionality.
    * Proxy support to the Storage Service.
    * Exposure to unit status or processing configuration details.


Endpoints for the Archivematica API have been organized into the following resource categories:

    * :ref:`transfer-resource.rst`
    * :ref:`ingest-resource.rst`
    * :ref:`admin-resource.rst`
    * :ref:`unit-resource.rst`
    * :ref:`other-resource.rst`
    * :ref:`beta.rst`
    
Endpoints for Archivematica API
================================

.. _transfer-resource.rst:

Transfer
---------

Transfer refers to the process of moving any set of digital objects into Archivematica and turning the materials into a Submission Information Package (SIP). The Transfer tab prepares your content for preservation in Archivematica.

Start Transfer
^^^^^^^^^^^^^^

========  ==================================  =================================
``POST``  **/api/transfer/start_transfer/**   *Initiates a transfer in Archivematica.*
========  ==================================  =================================

Request body parameters:

.. tip:: The parameters in the following table can be submitted as a JSON
   object with key-value pairs in the request body.

=============  ================================================================

``name``           Name of transfer

``type``           Type of the new transfer (e.g. standard, unzipped bag,
                   zipped bag, dspace).

``accession``      Accession number of new transfer.

``paths[]``        List of base64-encoded
                   ``<location_uuid>:<relative_path>`` to be copied into the new transfer. Location UUIDs should be associated with this pipeline, and relative path should be relative to the location.

``row_ids[]``      ID of the associated TransferMetadataSet for disk image
                   ingest. Can be provided as ``[""]``.

=============  ================================================================

Example request:

.. literalinclude:: _code/start_transfer.curl

Example response:

.. literalinclude:: _code/start_transfer_response.curl

.. _ingest-resource.rst:

Ingest
-------

Ingest refers to the process by which digital objects are packaged into SIPs and run through several microservices, including normalization, packaging into an AIP and generation of a DIP.

.. _admin-resource.rst:

Administration
--------------

Administration enables you to configure various parts of the application and manage integrations and users.

.. _unit-resource.rst:

Unit
----

Unit refers to "a type of Package Description that is specialized to provide information about an Archival Information Unit for use by Access Aids." (OAIS, p.1-13, Section 1.7.2)

.. _other-resource.rst:

Other
------

Other is a generic resource category that includes path metadata associated with level of description, and it also includes name associated with any customized processing configuration.

.. _beta.rst:

Beta
-----

Beta refers to functionality for which the API endpoints are still in-flux and that could potentially change.

Storage Service API
====================

The Storage Service API provides access to the following resources via endpoints:

    * :ref:`ss-pipeline.rst`
    * :ref:`ss-space.rst`
    * :ref:`ss-location.rst`
    * :ref:`ss-package.rst`

.. _ss-pipeline.rst:

Pipeline
---------

A pipeline is a representation of an Archivematica installation that is assigned a unique universal identifier (UUID) when it is registered with the Storage Service. The UUID provides information about your server and all associated clients. 

.. _ss-space.rst:

Space
-----

A storage space contains all the information necessary to connect to the physical storage. It is where the files are stored. Protocol-specific information, like an NFS export path and hostname, or the username of a system accessible only via SSH, is stored here. All locations must be contained in a space.

.. _ss-location.rst:

Location
--------

A location is a subdivision of a space. Each location is assigned a specific purpose, such as AIP storage, DIP storage, transfer source or transfer backlog, in order to provide an organized way to structure content within a space.

.. _ss-package.rst:

Package
-------

A package is a bundle of one or more files transferred from an external service; for example, a package may be an AIP, a backlogged transfer, or a DIP. Each package is stored in a location.
