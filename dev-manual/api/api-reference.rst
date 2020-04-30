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
``POST``  **/api/transfer/start_transfer/**   *Initiates a transfer in Archive-
                                              matica.*
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
                   ``<location_uuid>:<relative_path>`` to be copied into the new
                   transfer. Location UUIDs should be associated with this pipe-
                   line, and relative path should be relative to the location.

``row_ids[]``      ID of the associated TransferMetadataSet for disk image
                   ingest. Can be provided as ``[""]``.

=============  ================================================================

Example request:

.. literalinclude:: _code/start_transfer.curl

Example response:

.. literalinclude:: _code/start_transfer_response.curl

List unapproved transfer(s)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

========  ==============================  ======================================

``GET``   **/api/transfer/unapproved/**    *Returns a list of transfers awaiting
                                           approval*

========  ==============================  ======================================

Response definitions:

==================   ================================   ========================
                          
``message``          "Fetched unapproved transfers 
                     successfully."

``results``          List of dicts with the following 
                     keys:     
                     
                     ``type``                           Transfer type. 

                                                        One of: standard,
                                                        unzipped bag, zipped 
                                                        bag, dspace.

                     ``directory``                      Directory the transfer 
                                                        is in currently.    

                     ``uuid``                           UUID of the transfer.      

==================   ================================   ========================

Example request:

.. literalinclude:: _code/list_transfer.curl

Example response:

.. literalinclude:: _code/list_transfer_response.curl

Approve transfer(s)
^^^^^^^^^^^^^^^^^^^^

========  ==========================  =========================================
``POST``  **/api/transfer/approve/**  *Approve a transfer awaiting initiation.*
========  ==========================  =========================================

Request body parameters:

=============  ================================================================

``type``           Type of the transfer (e.g. standard, unzipped bag,
                   zipped bag, dspace) to approve.

``directory``      Directory the transfer is in currently.

=============  ================================================================

Example request:

.. literalinclude:: _code/approve_transfer.curl

Response definitions:

==================   ==========================================================
                       
``message``          "Approval successful."

``uuid``             UUID of the approved transfer  

==================   ==========================================================

Example response:

.. literalinclude:: _code/approve_transfer_response.curl

Status
^^^^^^^

========  =================================   =================================
``GET``   **/api/transfer/status/<transfer    *Returns the status of a
          UUID>/**                            transfer* 
========  =================================   =================================

Response definitions:

=================       ============================     ======================

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

Example request:

.. literalinclude:: _code/status_request.curl

Example response:

.. literalinclude:: _code/status_response.curl

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

Response definitions:

==================   ==========================================================
                         
``message``          "Fetched completed transfers successfully."

``results``          List of UUIDs of completed Transfers. 

==================   ==========================================================

Example request:

.. literalinclude:: _code/completed_request.curl

Example response:

.. literalinclude:: _code/completed_response.curl

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
                    
``message``          "Approval successful."

``reingest_uuid``    UUID of the reingested transfer.

==================   ==========================================================

.. _ingest-resource.rst:

Ingest
-------

Ingest refers to the process by which digital objects are packaged into SIPs and
run through microservices that enable normalization, packaging into an 
AIP, and generation of a DIP.

Status
^^^^^^^

===========  =================================   ==============================
``GET``      **/ingest/status/<unit_UUID>/**     Returns the status of the SIP. 
===========  =================================   ==============================

Response definitions:

==================   ==========================================================
                    
``status``           One of FAILED, REJECTED, USER_INPUT, COMPLETE or PROCESSING.

``name``             Name of the SIP, e.g. "imgs".

``microservices``    Name of the current microservice

``directory``        Name of the directory, e.g. "imgs-52dd0c01-e803-423a-be5f
                     \ -b592b5d5d61c"

``path``             Full path to the transfer, e.g. "/var/archivematica/shared
                     Directory/currentlyProcessing/imgs-52dd0c01-e803-423a-be5f-
                     / b592b5d5d61c/".     

``message``          "Fetched status for <SIP UUID> successfully."

``type``             "SIP"

``uuid``             UUID of the SIP, e.g. "52dd0c01-e803-423a-be5f-b592b5d61c".

==================   ==========================================================

Example request:

.. literalinclude:: _code/status_request.curl

Example response (JSON):

.. literalinclude:: _code/status_response.curl

Hide
^^^^^

===========  =================================== ==============================
``DELETE``   **/api/ingest/<SIP UUID>/delete/**  Hide a SIP. 
===========  =================================== ==============================

Example request:

.. literalinclude:: _code/hide_request.curl

Example response:

.. literalinclude:: _code/hide_response.curl


List SIPS Waiting for User Input
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

=========  ========================  ===========================================
``GET``    **/api/ingest/waiting**   Returns a list of SIPs waiting for user 
                                     input.
=========  ========================  ===========================================

Response definitions:

==================   ==========================================================
                    
``message``          "Fetched units successfully."

``results:``         List of dicts with keys:

                     ``sip_directory``: Directory the SIP is in currently.

                     ``sip_uuid``: UUID OF THE SIP.

                     ``sip_name``: Name of the SIP.

                     ``microservice``: Name of the current microservice.

==================   ==========================================================

.. note:: 

   Despite the URL, this currently returns both SIPs & transfers that 
   are waiting for user input. In the future, a separate **/api/transfer/
   waiting** should be added for transfers.

Example request:

.. literalinclude:: _code/sips-waiting_request.curl

Example response (JSON):

.. literalinclude:: _code/sips-waiting_response.curl

Completed
^^^^^^^^^^

===========  ========================== ========================================
``GET``      **/api/ingest/completed/**  Return a list of completed SIPs.
===========  ========================== ========================================

Example request:

.. literalinclude:: _code/completed_ingest_request.curl

Example response (JSON):

.. literalinclude:: _code/completed_ingest_response.curl

Reingest
^^^^^^^^^

=========== ========================= ==========================================
``POST``    **/api/ingest/reingest**  Start a partial or metadata-only reingest.
=========== ========================= ==========================================

Request body parameters:

=============  ================================================================

``name``       Name of the AIP. The AIP should also be found
               at ``%sharedDirectory%/tmp/<name>``.

``uuid``       UUID of the AIP to reingest.

=============  ================================================================

Response definitions:

==================   ==========================================================
                    
``message``          "Approval successful."

``reingest_uuid``    UUID of the reingested transfer.

==================   ==========================================================

Example request:

.. literalinclude:: _code/am-ss_reingest_aip.curl

Example response (JSON):

.. literalinclude:: _code/reingest_aip_response.curl

Copy metadata
^^^^^^^^^^^^^^

=========== ==================================== ==============================
``POST``    **/api/ingest/copy_metadata_files/** Add metadata files to a SIP.
=========== ==================================== ==============================

.. TBD
   Example request:
   literalinclude:: _code/filename

Request body parameters:

==================  ============================================================

``sip_uuid``        UUID of the SIP to put files in.

``source_paths[]``  List of files to be copied, base64 encoded, in the format 
                    'source_location_uuid:full_path'

==================  ============================================================


.. TBD 
   Example response (JSON):
   literalinclude:: _code/filename


Response definitions:

==================   ==========================================================
                    
``message``          "Approval successful."

``reingest_uuid``    UUID of the reingested transfer.

==================   ==========================================================


.. _admin-resource.rst:

Administration
--------------

Administration enables you to configure various parts of the application and 
manage integrations and users.


Levels of description
^^^^^^^^^^^^^^^^^^^^^^

======= ===========================================  ===========================

``GET`` **/api/administration/dips/atom /levels/**   Return a JSON-encoded set 
                                                     of the configured levels 
                                                     of description.
      
======= ===========================================  ===========================


Example request:

.. literalinclude:: _code/admin_levels_of_desc_req.curl

Example response (JSON):

The following response includes a list of AtoM Levels of description with key 
'UUID' and value 'name of level of description'.

.. literalinclude:: _code/admin_levels_of_desc_response.curl


Fetch levels of description
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

======= =============================================== ========================

``GET`` **/api/administration/dips/atom/fetch_levels/**  Fetch all levels of 
                                                         description from an 
                                                         AtoM database,
                                                         replacing any previously 
                                                         existing.
      
======= =============================================== ========================

Example request:

.. literalinclude:: _code/fetch_levels.curl

Example response (JSON):

The following response includes an updated list of AtoM Levels of description 
with key 'UUID' and value 'name of level of description'.

.. literalinclude:: _code/fetch_levels_response.curl

.. _unit-resource.rst:


Unit
----

Unit refers to "a type of Package Description that is specialized to provide 
information about an Archival Information Unit for use by Access Aids." (OAIS, 
p.1-13, Section 1.7.2)

List jobs
^^^^^^^^^^

=============  =================================== =============================
``GET``         **/api/v2beta/jobs/<unit UUID>/**  Return a list of jobs for the
                                                   passed unit (transfer or
                                                   ingest).
=============  =================================== =============================

Request body parameters (optional):

================     ===========================================================

``microservice``     Name of the microservice the jobs belong to.

``link_uuid``        UUID of the job chain link.

``name``             Name of the job.

================     ===========================================================

Response definitions (from list of dicts):

==================   ==========================================================
                    
``uuid``             UUID of the job.

``name``             Name of the job.

``status``           One of USER_INPUT, PROCESSING, COMPLETE, FAILED or UNKNOWN.

``microservice``     Microservice to which the job belongs.

``link_uuid``        UUID of the job chain link.

``tasks``            List of dicts with information about the microservice's
                     tasks.

``uuid``             UUID of the task.

``exit_code``        Exit code of the task.

==================   ==========================================================

Example request:

.. literalinclude:: _code/list_transfer_unit_request.curl

Example response (JSON body):

.. literalinclude:: _code/list_transfer_unit_response.curl

Task
^^^^^

=========  =================================== =================================
``GET``    **/api/v2beta/task/<task UUID>/**   Return information about a task.
=========  =================================== =================================

Response definitions:

================   =============================================================
                    
``uuid``           UUID of the task.

``exit_code``      List of dicts with keys:

``file_uuid``      Exit code of the task.

``file_name``      UUID of the file used for the task.

``time_created``   File used for the task.

``time_started``   String (YYYY-MM-DD HH:MM:SS) representing when the task
                   was created.

``duration``       Task duration in seconds (integer). If the duration is less
                   than a second, this will be a < 1 string.

================   =============================================================

Example request:

.. literalinclude:: _code/task_request.curl

Example response (JSON):

.. literalinclude:: _code/task_response.curl

.. _other-resource.rst:

Other
------

Other is a generic resource category that includes path metadata associated 
with level of description, and it also includes name associated with any 
customized processing configuration.

Path metadata
^^^^^^^^^^^^^^

============= ============================= =================================
``GET, POST`` **/api/filesystem/metadata/** Fetch (GET) or update (POST)
                                            metadata for a path (currently 
                                            only level of description).   
============= ============================= =================================

Request parameter: 

.. tip:: The following parameter can be submitted as query string parameter 
         with GET or as a JSON object with a key-value pair in the request body
         with POST.

================     ===========================================================

``path``             Arranged path on which to get metadata.

================     ===========================================================

Response: the processing config file as a stream

Content type: text/xml

Example request:

.. literalinclude:: _code/task_request.curl

Example response (JSON):

.. literalinclude:: _code/task_response.curl

.. _beta.rst:

Beta
-----

API endpoints that are still in-flux and could potentially change.

Package
^^^^^^^^

============= ============================= =================================
``POST``      **/api/v2beta/package**       Start a new transfer type.   
============= ============================= =================================

Request parameters (mandatory):

================     ===========================================================

``name``             Transfer name.

``path``             Path relative, or absolute to a storage service transfer
                     source.

================     ===========================================================

Request parameters (optional):

=====================  =========================================================

``type``               Transfer name.

``processing_config``  Processing configuration, e.g. default, automated,
                       default: default

``accession``          Accession ID.

``access_system_id``   Access system ID (see `Docs <https://bit.ly/3c4yWt6>`_).

``metadata_set_id``    Used to link to metadata sets added via the user 
                       interface. It's safe to ignore this for now since 
                       metadata can't be associated to transfer via the API 
                       at the moment.

``auto_approve``       Boolean true or false to set the transfer to auto-approve, 
                       default: true.                     

=====================  =========================================================

Response definitions:

==================   ==========================================================
                    
``id``                Transfer UUID (Note: as the package endpoint allows the 
                      caller to interact with Archivematica asynchronously it 
                      does not guarantee a transfer has started. The caller must 
                      use the UUID in the response to verify it has begun or any
                      errors that were encountered initiating one.)

==================   ==========================================================

Examples:

Only a subset of these options might be needed for most use-cases. A fundamental
difference between the package endpoint and others from which a transfer can be
initiated is that a storage service transfer location UUID isnt always required.
In some cases that might still be ideal.

*Starting a transfer using an absolute path*::

   curl -v POST \
       -H "Authorization: ApiKey test:test" \
       -H "Content-Type: application/json" \
       -d "{\
           \"path\": \"$(echo -n '/home/archivematica/archivematica-sampledata
           /SampleTransfers/DemoTransfer' | base64 -w 0)\", \
           \"name\": \"demo_transfer_absolute\", \
           \"processing_config\": \"automated\", \
           \"type\": \"standard\" \
           }" \
      "http://<archivematica-uri>/api/v2beta/package"

*Starting a transfer using an relative path with a transfer source UUID*::

  curl -v POST \
       -H "Authorization: ApiKey test:test" \
       -H "Content-Type: application/json" \
       -d "{\
           \"path\": \"$(echo -n 'd1184f7f-d755-4c8d-831a-a3793b88f760:
           /archivematica/archivematica-sampledata/SampleTransfers/DemoTransfer' 
           | base64 -w 0)\", \
           \"name\": \"demo_transfer_relative\", \
           \"processing_config\": \"automated\", \
           \"type\": \"standard\" \
           }" \
      "http://<archivematica-uri>/api/v2beta/package"

Validate
^^^^^^^^

======================= ========================================================
**/api/v2beta/package** Available in Archivematica 1.10+. This endpoint can be 
                        used to validate CSVs against embedded sets of rules. 
                        Currently works with Avalon Media System Manifest files.

                        In the future, this endpoint can be extended to support 
                        validation for metadata.csv or rights.csv, or other 
                        institutionally-based rules.   
======================= ========================================================

Usage example: (assuming that avalon.csv exists)::

  curl http://127.0.0.1:62080/api/v2beta/validate/avalon \
  --data-binary "@avalon.csv" \
  --header "Authorization: ApiKey test:test" \
  --header "Content-Type: text/csv; charset=utf-8"

Response examples

200 OK::

  {
  "valid": true
  }

400 Bad Request (expect reason to include different validation errors)::

  {
  "valid": false,
  "reason": "Administrative data must include reference name and author."
  }

 404 Not Found::
 
  {
  "error": true,
  "message": "Unknown validator. Accepted values: avalon"
  } 

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

===========  =================================   ==============================
``POST``     **/api/v2/file/<UUID>/reingest/**   Initiate an AIP reingest.
===========  =================================   ==============================

Example request:

.. literalinclude:: _code/am-ss_reingest_aip.curl

Request body parameters:

======================  ========================================================

``pipeline``            UUID of the pipeline on which to reingest AIP.
               
``reingest_type``       One of `METADATA_ONLY` (metadata-only reingest), 
                        `OBJECTS` (partial reingest), `FULL` (full reingest).

``processing_config``   Optional. Name of the processing configuration to use
                        on full reingest.

======================  ========================================================

Example response:

.. literalinclude:: _code/reingest_aip_response.curl



