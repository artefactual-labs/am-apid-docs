.. _am-api-reference.rst:

==========
Reference
==========

Quickly review all available resources for the Archivematica API and Storage 
Service API with this reference.

Archivematica API
==================

The Archivematica API provides:

    * Coverage of some basic workflow functionality.
    * Proxy support to the Storage Service.
    * Exposure to unit status or processing configuration details.
    
Archivematica API resources and endpoints
==========================================

Endpoints for the Archivematica API have been organized into the following 
resource categories:

    * :ref:`transfer-resource.rst`
    * :ref:`ingest-resource.rst`
    * :ref:`admin-resource.rst`
    * :ref:`unit-resource.rst`
    * :ref:`other-resource.rst`
    * :ref:`beta.rst`

.. _transfer-resource.rst:

Transfer
---------

Transfer refers to the process of moving any set of digital objects 
into Archivematica and turning the materials into a Submission Information 
Package (SIP). The Transfer tab prepares your content for preservation in 
Archivematica.

Start transfer
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

List unapproved transfer(s)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

========  ==============================  ====================================
``GET``   **/api/transfer/unapproved/**    *Returns a list of transfers awaiting approval*
========  ==============================  ====================================

Example request:

.. literalinclude:: _code/list_transfer.curl

Example response:

.. literalinclude:: _code/list_transfer_response.curl

Response definitions:

==================   =========================================  ===============

**Response item**    **Description**                            

``message``          "Fetched unapproved transfers 
                     successfully."

``results``          List of dicts with the following keys:     |   
                     
                     ``type``                                   Transfer
                                                                type. 

                                                                One of: standard,     unzipped bag, zipped bag, dspace.

                     ``directory``                              Directory 
                                                                the 
                                                                transfer is in currently.    

                     ``uuid``                                   UUID of the 
                                                                transfer.      

==================   =========================================  ===============

Approve transfer(s)
^^^^^^^^^^^^^^^^^^^^

========  ==========================  =========================================
``POST``  **/api/transfer/approve/**  *Approve a transfer awaiting initiation.*
========  ==========================  =========================================

Example request:

.. literalinclude:: _code/approve_transfer.curl

Request body parameters:

=============  ================================================================

``type``           Type of the transfer (e.g. standard, unzipped bag,
                   zipped bag, dspace) to approve.

``directory``      Directory the transfer is in currently.

=============  ================================================================

Example response:

.. literalinclude:: _code/approve_transfer_response.curl

Response definitions:

==================   ==========================================================

**Response item**    **Description**                            

``message``          "Approval successful."

``uuid``             UUID of the approved transfer  

==================   ==========================================================

Status
^^^^^^^

========  =================================   =================================
``GET``   **/api/transfer/status/<transfer    *Returns the status of a
          UUID>/**                            transfer* 
========  =================================   =================================

Example request:

.. literalinclude:: _code/status_request.curl

Example response:

.. literalinclude:: _code/status_response.curl

Response definitions:

=================       ============================     ======================

**Response item**       **Description**                  **Data type**

``status``              One of FAILED, REJECTED,         string
                        USER_INPUT, COMPLETE or
                        PROCESSING                     

``name``                Name of the transfer,            string
                        e.g. "imgs"

``sip_uuid``            If status is COMPLETE,           string
                        this field will exist 
                        with either the UUID 
                        of the SIP or 'BACKLOG'

``microservice``        Name of the current              string  
                        microservice

``directory``           Name of the directory,           string      
                        e.g. "imgs-52dd0c01-e803-
                        423a-be5f-b592b5d5d61c"

``path``                Full path to the transfer,       string
                        e.g. "/var/archivematica/
                        sharedDirectory/watched
                        Directories/SIPCreation/
                        completedTransfers/imgs-
                        52dd0c01-e803-423a-
                        be5f-b592b5d5d61c/"

``message``             "Fetched status from             string
                        <transfer UUID>
                        successfully."      

``type``                "transfer"                       string

``uuid``                UUID of the transfer,            string
                        e.g. "52dd0c01-e803-423a-
                        be5f-b592b5d5d61c"

=================       ============================     ======================

..  note::

    For consumers of this endpoint, it is possible for Archivematica to return a
     status of COMPLETE without a sip_uuid. Consumers looking to use the UUID 
     of the AIP that will be created following Ingest should therefore test 
     for both a status of COMPLETE and the existence of sip_uuid that does not 
     also equal BACKLOG to ensure that they retrieve it. This might mean an 
     additional call to the status endpoint while this data becomes available.

Hide
^^^^^

===========  =================================   ==============================
``DELETE``   **/api/transfer/<transfer UUID>/    Hide a transfer
             delete/**                          
===========  =================================   ==============================

Example request:

.. literalinclude:: _code/hide_request.curl

Example response:

.. literalinclude:: _code/hide_response.curl

Completed
^^^^^^^^^^

===========  =================================   ==============================
``GET``      **/api/transfer/completed/**        Return list of Transfers that
                                                 are completed.
===========  =================================   ==============================

Example request:

.. literalinclude:: _code/completed_request.curl

Example response:

.. literalinclude:: _code/completed_response.curl

Response definitions:

==================   ==========================================================

**Response item**    **Description**                            

``message``          "Fetched completed transfers successfully."

``results``          List of UUIDs of completed Transfers. 

==================   ==========================================================

Start reingest
^^^^^^^^^^^^^^^

..  note::

    The Start reingest endpoint complements the :ref:`ss-reingest-aip.rst`
    endpoint on the Storage Service's end. When a reingest has been initiated, 
    Archivematica fetches the AIP from storage, extracts and runs fixity on it 
    to verify its integrity. Then Archivematica will set up the environment 
    as per the reingest type before the Storage Service endpoint calls the 
    Archivematica Start reingest endpoint.


===========  =================================   ==============================
``POST``      **/api/transfer/reingest**         Start a full reingest.
===========  =================================   ==============================

Request body parameters:

=============  ================================================================

``name``       Name of the AIP. The AIP should also be found
               at ``%sharedDirectory%/tmp/<name>``.

``uuid``       UUID of the AIP to reingest.

=============  ================================================================

Response definitions:

==================   ==========================================================

**Response item**    **Description**                            

``message``          "Approval successful."

``reingest_uuid``    UUID of the reingested transfer.

==================   ==========================================================

.. _ingest-resource.rst:

Ingest
-------

Ingest refers to the process by which digital objects are packaged into SIPs and
 run through several microservices, including normalization, packaging into an 
 AIP and generation of a DIP.

.. _admin-resource.rst:

Administration
--------------

Administration enables you to configure various parts of the application and 
manage integrations and users.

.. _unit-resource.rst:

Unit
----

Unit refers to "a type of Package Description that is specialized to provide 
information about an Archival Information Unit for use by Access Aids." (OAIS, 
p.1-13, Section 1.7.2)

.. _other-resource.rst:

Other
------

Other is a generic resource category that includes path metadata associated 
with level of description, and it also includes name associated with any 
customized processing configuration.

.. _beta.rst:

Beta
-----

Beta refers to functionality for which the API endpoints are still in-flux and 
that could potentially change.

Storage Service API
====================

The Storage Service API enables access to storage areas to which the Storage 
Service has access.

Storage Service API resources and endpoints
============================================

Endpoints for the Storage Service API have been organized into the following 
resource categories:

    * :ref:`ss-pipeline.rst`
    * :ref:`ss-space.rst`
    * :ref:`ss-location.rst`
    * :ref:`ss-package.rst`

.. _ss-pipeline.rst:

Pipeline
---------

A pipeline is a representation of an Archivematica installation that is assigned
 a unique universal identifier (UUID) when it is registered with the Storage 
 Service. The UUID provides information about your server and all associated 
 clients. 

.. _ss-space.rst:

Space
-----

A storage space contains all the information necessary to connect to the 
physical storage. It is where the files are stored. Protocol-specific 
information, like an NFS export path and hostname, or the username of a system 
accessible only via SSH, is stored here. All locations must be contained in 
a space.

.. _ss-location.rst:

Location
--------

A location is a subdivision of a space. Each location is assigned a specific 
purpose, such as AIP storage, DIP storage, transfer source or transfer backlog, 
in order to provide an organized way to structure content within a space.

.. _ss-package.rst:

Package
-------

A package is a bundle of one or more files transferred from an external service;
 for example, a package may be an AIP, a backlogged transfer, or a DIP. Each 
 package is stored in a location.

.. _ss-reingest-aip.rst:

Reingest AIP
^^^^^^^^^^^^

Example request:

.. literalinclude:: _code/am-ss_reingest_aip.curl
