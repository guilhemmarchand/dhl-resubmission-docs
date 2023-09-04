.. dhl-resubmission documentation master file

Welcome to the DHL resubmission documentation
=============================================

**The DHL resubmission solution provides services for:**

- Publishing on demand any volume of messages to IBM MQ managers or SOAR APIs via HTTPS, in a scalabale, resilient and distributed manner using various Splunk components
- Handling the message life cycle at any point in time with RBAC features
- Hanlding temporary and permanent failures with automated retries based on a life cycle concept
- High availability with automated fail-over using multiple Splunk Heavy Fowarders working in a HA group concept

*In a nutshell:*

- For each technlology (MQ and SOAP), there are frontend and backend Splunk applications which are deployed as needed
- Users interact with the solution with a pre-defined workflow involving different components to create and maintain records in a Splunk distributed KVstore.
- The KVstore records contain the message payloads in addition with various Metadata such as the application name, the region, MQ/SOAP related information (managers, queues. Etc), statuses, user requesting information.
- By nature, the KVstore is automatically replicated amongst the SHC members and is available on all DHL deployed sites.
- A number of Splunk Heavy Forwarders are deployed on a per site basis and leverage the Splunk API to retrieve and maintain the KVstore records with configurable filtering rules which allow dedicating a single Heavy Forwarder to consume the records it is responsible for.
- Accounts are created in the MQ and SOAP Addons, an account defines various technical information such as the target or the authentication
- A concept of automatic retries and eviction policy is part of the application to handle the life cycle of the records, from their initial submission requests (pending state), to their publication (success), or temporary or definitive failures. (temporary_failure, permanent_failure)
- Messages are submitted to the MQ third party using a low level high performance third party (q command) which allow batching single line messages for the better performances (which single line messages represent around 70% of the total volume of messages), while multiline messages are handled with the same q command but a different workflow handling the records individually at a lower rate.

Overview:
=========

.. toctree::
   :maxdepth: 2

   compatibility
   requirements

Deployment and configuration:
=============================

.. toctree::
   :maxdepth: 2

   deployment
   configuration_http
   configuration_mq
   high_availability
   rbac
   scaling

User guide:
===========

.. toctree::
   :maxdepth: 2

   userguide

Versions, build history and development:
========================================

.. toctree::
   :maxdepth: 1

   releasenotes
   development
