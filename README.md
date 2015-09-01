## The Go vendor file specification

### Specification
 * Description and purpose:
  - Copying third-party vendor packages into a local project structure requires
    a meta-data file describing the vendor's packages.
  - The meta-data file describing the vendor's packages will be referred to as
    the vendor file.
  - The vendor file is to be used by a vendor tool or the developer; the vendor
    file is not used for compiling source code.
 * Vendor File:
  - The vendor file will be called "vendor.json" and be placed in a folder
    named "vendor".
  - The vendor file describes the packages in the vendor folder the vendor file is in.
  - The vendor file does not describe any nested vendor folders; nested vendor
    folders must contain their own vendor file.
  - The vendor file must be in or under a descendant of the "$GOPATH/src/" directory.
  - Additional fields may be added to the vendor file that is tool specific.
  - Each copied package entry is defined to match a single Go package. A
    vendor file Package entry does NOT match a package tree; a vendor file
	Package entry matches a single Go package.
  - A tool MUST persist any unknown fields when reading and writing out the
    vendor file. A tool may remove unknown fields on an explicit user request.
	This implies that a tool must not marshal or unmarshal from a known struct.

The following struct describes well-known fields:
```go
struct {
	// Comment is free text for human use. Example "Revision abc123 introduced
	// changes that are not backwards compatible, so leave this as def876."
	Comment string `json:"comment,omitempty"`

	// Package represents a collection of vendor packages that have been copied
	// locally. Each entry represents a single Go package.
	Package []struct {
		// Import path. Example "rsc.io/pdf".
		// go get <Path> should fetch the remote package.
		Path string `json:"path"`

		// Origin is an import path where it was copied from. This import path
		// may contain "vendor" segments.
		// 
		// If empty or missing origin is assumed to be the same as the Path field.
		Origin string `json:"origin"`

		// The revision of the package. This field must be persisted by all
		// tools, but not all tools will interpret this field.
		// The value of Revision should be a single value that can be used
		// to fetch the same or similar revision.
		// Examples: "abc104...438ade0", "v1.3.5"
		Revision string `json:"revision"`

		// RevisionTime is the time the revision was created. The time should be
		// parsed and written in the "time.RFC3339" format.
		RevisionTime string `json:"revisionTime"`

		// Comment is free text for human use.
		Comment string `json:"comment,omitempty"`
	} `json:"package"`
}
```

### Example
*vendor file path: "$GOPATH/src/github.com/kardianos/mypkg/vendor/vendor.json"*

*first package copied to: "$GOPATH/src/github.com/kardianos/mypkg/vendor/rsc.io/pdf"*

```json
{
	"comment": "Note the use of a non-standard crypto package.",
	"package": [
		{
			"path": "rsc.io/pdf",
			"revision": "3a3aeae79a3ec4f6d093a6b036c24698938158f3",
			"revisionTime": "2014-09-25T17:07:18-04:00",
			"comment": "located on disk at $GOPATH/src/github.com/kardianos/mypkg/vendor/rsc.io/pdf"
		},
		{
			"origin": "github.com/MSOpenTech/azure-sdk-for-go/vendor/crypto/tls",
			"path": "crypto/tls",
			"revision": "80a4e93853ca8af3e273ac9aa92b1708a0d75f3a",
			"revisionTime": "2015-04-07T09:07:15-07:00",
			"comment": "located on disk at $GOPATH/src/github.com/kardianos/mypkg/vendor/crypto/tls"
		},
		{
			"path": "github.com/coreos/etcd/raft",
			"revision": "25f1feceb5e13da68a35ee552069f86d18d63fee",
			"revisionTime": "2015-04-09T05:06:17-08:00",
			"comment": "located on disk at $GOPATH/src/github.com/kardianos/mypkg/vendor/github.com/coreos/etcd/raft"
		},
		{
			"path": "golang.org/x/net/context",
			"revision": "25f1feceb5e13da68a35ee552069f86d18d63fee",
			"revisionTime": "2015-04-09T05:06:17-08:00",
			"comment": "Use the 25f1 version. Located on disk at $GOPATH/src/github.com/kardianos/mypkg/vendor/golang.org/x/net/context"
		}
	]
}
```
*The above example would have the import paths as follows:*
```go
import (
	"rsc.io/pdf"
	"crypto/tls"
	"github.com/coreos/etcd/raft"
	"golang.org/x/net/context"
)
```

### Package.Path and Package.Origin fields
The Path field identifies the package with the import path. The package it
describes will be rooted in the same vendor folder as the vendor file and be
located at "Path" relative to the vendor folder.

The Origin field contains the import path of the package it was copied from.
If this field is empty or not present it is assumed to be the same as the Path
field. This field is useful when updating dependencies.

### Revision and RevisionTime fields
Both revision and revisionTime fields are optional. However tools must persist any information
present in them. The interpretation of both fields is dependent on the tool
itself. While the exact interpretation of the fields are tool specific, the
semantics are not. The Revision field must either be empty or contain a single
value that can be used to fetch a specific revision. The RevisionTime must be
empty or have a valid RFC3339 time string. If the Revision field is non-empty
the RevisionTime field must correlate with the Revision field. If the Revision
field is empty the meaning of the RevisionTime field is tool specific.
