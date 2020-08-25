# Sailing Administrative Interface Layer (SAIL)
This project is to outline the specification for a RESTful interface for to serve the administrative needs of managing data relating to sail racing under the Racing Rules of Sailing. Its mandate is to provide an open interoperative interface to support collaborative innovation with the goal of making race administration easier and more accessible as a grass-roots sport. 
It is conceived to operate in a combination of server-to-server and server-to-client configurations to provide various functions of race administration. 
## Nexus
A SAIL-Nexus provides the core data backend for the service, clients can connect to it to perform race administration functions, and a Nexus can connect to a peer Nexus to exchange information (such as assembling a multi-club series, or collect boat rating data). The nexus will support flat-file exports and import of SAIL specified data structures.
## Clients
SAIL compatible clients will be developed to meet the specific designs of their creators. They will connect to one or more SAIL-Nexus providers and deliver functionality such as:
* Competitor Registration
* Race Committee Recording
* Race Scoring
* Race Result Presentation
## Interface Specification
SAIL is a RESTful interface relying on JSON structured objects. The HTTP request will contain the following fields:
* Payload (mandatory): A SAIL compliant data structure appropriate to the endpoint being called. This is a URL encoded, serialized JSON representation. It may be compressed.
* Compression (mandatory): Default to none. Supported compression standards can be negotiated between client and server.
* Checksum (mandatory): the MD5 Checksum for the uncompressed Payload field
* CacheValid (optional): an assertion of when the data was prepared. ISO 8601 formatted with seconds time in UTC
The SAIL standard does not specify provision for endpoint security, rather it relies on bearer tokens for authorization control within a Nexus. Endpoint security may be implemented using appropriate industry standards. It is also strongly recommended to disallow unencrypted traffic to your endpoints, and to support contemporary secure cypher suites.
When appropriate, data keys will be structured using a UUID for record specification @ separated from the nexus’s domain prepended with the data class as a sub-domain (e.g. 123e4567-e89b-12d3-a456-426614174000@race.foobar.net).
## Data Specifications
The SAIL service includes services for the following data types. Services can be independently versioned to expand data storage options and modify service functions. 

### Series
A series collects multiple races for aggregated scoring and indirect authorization.

|Field|	Status|	Type|	Notes|
|---|---|---|---|
|UUID|	Required|	String||	
|Name	|Optional|	String|	Friendly name for the series|
|Method	|Required|	String|	Either “Reference” or “Full”|
|Races	|Required|	Multi-valued, unkeyed, Strings|	If method is “Reference” it is the full reference to the race record, if “Full” it is a serialized string of the full race record.|
|Authorized	|Required|	Multi-valued, unkeyed String|	Access tokens authorized on the series, will be applied to all attached races.|

### Race
A race is the core structure for the primary intent of this initiative. 

|Field|	Status|	Type|	Notes|
|---|---|---|---|
|UUID|	Required|	String||	
|Name	|Optional|	String|	Friendly name for the race.|
|Start|	Required|	String, ISO 8601	||
|Type|	Required|	|	e.g. W/L Fixed, Pursuit|
|Scoring|	Required||		Scoring Method Flag e.g. PHRF-LO|
|Competitors|	Required|	Multi-valued, unkeyed objects of Boat	||
|Results|	Optional|	Keyed Object, Results data|	Using access tokens as keys, results data structure.|
|Scores|	Optional|	Keyed Object, Score data|	Using access tokens as keys, score data structure.|
|Authorized|	Optional|	Multi-valued, unkeyed String|	Access tokens authorized on the race|

### Results
The results data structure is contributed per-reporting official. When more than one is submitted client interfaces should facilitate cross-referenced reviews.
|Field|	Status|	Type|	Notes|
|---|---|---|---|
|Record|	Optional|	String|	Indications of where a recording of the finish is available. Either with official or a URL/path to the file. File types as supported by implementation.|
|Finishes|	Required|	Multi-value Object|	List of finish records using the sub-structure of “Finish Record”|

A sub-structure of “Finish Record” is embedded within the results.
|Field|	Status|	Type|	Notes|
|---|---|---|---|
|BoatKey|	Required|	String|	The field from the Boat structure in the Race competitor list being used as a key|
|Key|	Required|	String|	The value of the competitor’s key field|
|Status|	Required|	String|	Code indicating their finish status (e.g. DNE, DSQ, Finished)|
|FinishOrder|	Optional|	Int|	Sequential order of finish|
|FinishTime|	Optional|	String, ISO 8601|	Time of day when finish occurred|

### Scores
The Scores data structure is contributed per-scoring official. 
|Field|	Status|	Type|	Notes|
|---|---|---|---|
|BoatKey|	Required|	String|	The field from the Boat structure in the Race competitor list being used as a key|
|Key|	Required|	String|	The value of the competitor’s key field|
|Placing|	Required|	Int|	The determined order of finish|
|ElapsedTime|	Optional|	String	||
|CorrectedTime|	Optional|	String||	
|Points|	Required|	Int||	

### Boat
A structure for scoring information specific to a boat. This structure is not subject to data keys, but should include descriptive candidate keys which can be used for record discovery.
|Field|	Status|	Type|	Notes|
|---|---|---|---|
|SailNo|	Optional, Candidate Key|	Int	||
|HullNo|	Optional, Candidate Key|	String||	
|Name|	Optional|	String||	
|SupportedRatings|	Required|	Multi-valued String|	List the keys for the Ratings field. One design ratings prefixed with “OD-“|
|Ratings|	Required|	Keyed Object of Ratings|	Keyed set of objects specific to the rating methods.|

### Ratings
Where possible ratings structures syntax should be brought under the control of the ratings authority maintainers of this standard should work to include their contributions in the standard maintenance processes. In the absence of a single authority, maintainers should follow community consensus. For error handling purposes all fields should be considered optional. Suggested fields:
|Field|	Status|	Type|	Note|
|---|---|---|---|
|name|	Optional|	String|	The common name of the rating standard|
|version|	Optional|	String||	
|checksum|	Optional|	String, MD5 checksum|	Checksum for record less checksum field.|
|repudiated|	Optional|	Multi-valued String	|List of checksums superseded by the current record|
|coefficient|	Optional|	Double|	The scoring coefficient for the given method. It may be renamed and there may be additional fields in the record.|

Example: One Design
{“OD-J80”:{“name”:“J80”, “version”:“0.1”}}

Example: PHRF-LO
{“PHRF-LO”:{“TOT”:0.34, “TOD”:0.36”, “version”: “0.1”}}


