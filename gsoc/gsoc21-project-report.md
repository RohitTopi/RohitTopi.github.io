# Google Summer of Code 2021: Project Report

### CernVM-FS preload capability  
**Contributor:** Rohit Topi  
**Organization:** CERN-HSF  
**Member organization:** CERN  
**Mentors:** Jakob Blomer, Enrico Bocchi  

---
## Background
CernVM-FS (CVMFS) is a service for fast and reliable software distribution on a global scale. It is capable of delivering the scientific software used by the High Energy Physics (HEP) community to tens of thousands of client nodes worldwide.
Data is organized in repositories that are mounted as a POSIX read-only file system by the clients. Files and metadata are downloaded on-demand by means of HTTP requests and take advantage of several layers of caches.
 
## Project description
The current per-file granularity of distribution and caching leads to poor performance especially in the case of cold caches. In some cases, however, it is known upfront that all files of a certain set are required if any of the files is accessed. The loading of interdependent files and libraries required for certain applications can be improved. The ability to load all required files together would improve performance in the case of cold caches.  
The goal of this project is to introduce the concept of bundles and related support, which would improve the startup performance of applications.
 
## Project development
### Summary 
**Bundles** are the new content type introduced in the repository. A bundle has the binary format of the object packs used by the gateway as a form to concatenate multiple objects in a single BLOB. A **bundle specification file** called `.cvmfsbundles` describes the content to be included in each bundle. This file will thus, allow the repository owners to create file bundles.  
To keep information on the bundles and the files belonging to bundles, the catalog table is extended so that an optional **bundle id** can be stored for each regular file. A new file catalog table stores the information about individual bundles.  
When the client has a cache miss for a file that is part of a bundle, it can then decide based on the cache contents whether to download the individual requested file or the entire bundle, into the local cache.  
The development towards the project was done through Github pull requests. The corresponding CERN JIRA issue is **[CVM-1886](https://sft.its.cern.ch/jira/browse/CVM-1886)**. The overall work was divided into four PRs.  
### Development 
#### [PR #2736](https://github.com/cvmfs/cvmfs/pull/2736) | Add support for .cvfmsbundles file | [Status: merged]
A bundle specification file, `.cvmfsbundles`, allows the creation of such bundles on the server-side. This file is created and maintained by the repository owners. During publication, the bundles are created and stored based on the contents of this file.
Implemented functionality to:
* Allow recognition of the bundle specification files on the server-side.
* Enforce the requirement that such files must be regular and not a hard link.
* Ensure that the file is in the root directory of a repository.
* Reserve the filename, `.cvmfsbundles`, for the bundle specification files.
Once the bundle specification file is determined to be of a valid type, it is parsed so that the structure can be validated. Further, bundles are created based on the list of files in the file.
 
#### [PR #2741](https://github.com/cvmfs/cvmfs/pull/2741) | Introduce bundles content type in the repository | [Status: under review]
To support the creation of bundles:
* Created a new `Bundle` class
* Added a function to allow parsing of the `.cvmfsbundles` file.
* Added a function to create a bundle
For testing:
* Added a test for parsing bundle specification
* Added a test for the creation of a bundle
Once the bundle is created, it needs to be further processed so that it can be written to the storage. After writing the bundle to the storage, an entry for it is added to the catalog. To facilitate this, the following changes were implemented:
* Defined a new `BundleEntry` struct, which will facilitate passing information about the bundle 
* Added relevant functions to insert bundle entries to the catalog
 
#### [PR #2742](https://github.com/cvmfs/cvmfs/pull/2742)  | Extend catalog table to support bundle files | [Status: under review]
To support keeping track of bundles and the files belonging to the bundles:
* Extended the catalog table in the file catalogs to create an additional column, bundleid, that specifies a bundle id that can be optionally associated with a regular file.
* Added a file catalog table called `bundles`. This table registers, for each bundle, its bundle id, hash, and the size
 
#### [PR #2743](https://github.com/cvmfs/cvmfs/pull/2743) | Add client-side support for bundle files | [Status: under review]
On the client-side, initial access to a file initiates the fetch and stores it in the local cache. Access to the file with a valid `bundleid` will lead to the prefetch of the corresponding bundle if it doesn’t already exist in the local cache. The changes made to support this were:
* Checking if the file being opened is regular 
* Fetching the corresponding bundle id for the file 
* Fetching the bundle entry for the fetched bundle id 
* Pre-load the bundle to the local cache if it doesn't already exist there
 
## Acknowledgment
I am extremely grateful to my mentors Jakob Blomer and Enrico Bocchi for guiding me and helping me out at various stages throughout the project. Sincere gratitude to Google and CERN-HSF for offering me this great opportunity.
 
