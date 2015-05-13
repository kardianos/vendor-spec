## The Go vendor file specification


### Specification
 * Description and purpose:
  - Copying third-party vendor packages into a local project structure requires
    a meta-data file describing the vendor's packages.
  - The meta-data file describing the vendor's packages will be referred to as
    the vendor file.
  - The vendor file is to be used by a vendor tool; the vendor file is not used
    for compiling source code.
  - The vendor file may be used to prevent duplicate packages from being
    included and store the revision of a package.
 * Vendor File:
  - The vendor file will be called "vendor.json".
  - The vendor file describes the vendor's packages under the vendor file.
  - The vendor file must be in or under a descendant of the "$GOPATH/src/" directory.
  - Additional fields may be added to the vendor file that is tool specific.
  - Each copied package entry is defined to match a single Go package. A
    vendor file Package entry does NOT match a package tree; a vendor file
	Package entry matches a single Go package.
  - A tool MUST persist any unknown fields when reading and writing out the
    vendor file. A tool may remove unknown fields on an explicit user request.
	This implies that a tool must not marshal or unmarshal from a known struct.

The following struct describes the minimum fields that must be present in
the json file:
```
struct {
	// Comment is free text for human use. Example "Revision abc123 introduced
	// changes that are not backwards compatible, so leave this as def876."
	Comment string `json:"comment,omitempty"`

	// Package represents a collection of vendor packages that have been copied
	// locally. Each entry represents a single Go package.
	Package []struct {
		// Canonical import path. Example "rsc.io/pdf".
		// go get <Canonical> should fetch the remote package.
		Canonical string `json:"canonical"`

		// Package path relative to the vendor file.
		// Examples: "vendor/rsc.io/pdf".
		//
		// Local should always use forward slashes and must not contain the
		// path elements "." or "..".
		Local string `json:"local"`

		// The revision of the package. This field must be persisted by all
		// tools, but not all tools will interpret this field.
		// The value of Revision should be a single value that can be used
		// to fetch the same or similar revision.
		// Examples: "abc104...438ade0", "v1.3.5"
		Revision string `json:"revision"`

		// RevisionTime is the time the revision was created. The time should be
		// parsed and written in the "time.RFC3339" format.
		RevisionTime string `json:"revisionString"`

		// Comment is free text for human use.
		Comment string `json:"comment,omitempty"`
	} `json:"package"`
}
```

### Example
*vendor file path: "$GOPATH/src/github.com/kardianos/mypkg/vendor.json"*

*first package copied to: "$GOPATH/src/github.com/kardianos/mypkg/vendor/rsc.io/pdf"*

```
{
	"comment": "Note the use of a non-standard crypto package.",
	"package": [
		{
			"canonical": "rsc.io/pdf",
			"local": "vendor/rsc.io/pdf",
			"revision": "3a3aeae79a3ec4f6d093a6b036c24698938158f3",
			"revisionTime": "2014-09-25T17:07:18Z-04:00",
			"comment": "located on disk at $GOPATH/src/github.com/kardianos/mypkg/vendor/rsc.io/pdf"
		},
		{
			"canonical": "crypto/tls",
			"local": "vendor/crypto/tls",
			"revision": "80a4e93853ca8af3e273ac9aa92b1708a0d75f3a",
			"revisionTime": "2015-04-07T09:07:157Z-07:00",
			"originPath": "github.com/MSOpenTech/azure-sdk-for-go/internal/crypto/tls",
			"originURL": "https://github.com/Azure/azure-sdk-for-go.git",
			"comment": "located on disk at $GOPATH/src/github.com/kardianos/mypkg/vendor/crypto/tls"
		},
		{
			"canonical": "github.com/coreos/etcd/raft",
			"local": "internal/github.com/coreos/etcd/raft",
			"revision": "25f1feceb5e13da68a35ee552069f86d18d63fee",
			"revisionTime": "2015-04-09T05:06:17Z-08:00",
			"comment": "located on disk at $GOPATH/src/github.com/kardianos/mypkg/internal/github.com/coreos/etcd/raft"
		},
		{
			"canonical": "golang.org/x/net/context",
			"local": "context",
			"revision": "25f1feceb5e13da68a35ee552069f86d18d63fee",
			"revisionTime": "2015-04-09T05:06:17Z-08:00",
			"originPath": "github.com/coreos/etcd/internal/golang.org/x/net/context",
			"comment": "Use the 25f1 version. Located on disk at $GOPATH/src/github.com/kardianos/mypkg/context"
		}
	]
}
```
*The above example would have the import paths as follows:*
```
import (
	"github.com/kardianos/mypkg/vendor/rsc.io/pdf"
	"github.com/kardianos/mypkg/vendor/crypto/tls"
	"github.com/kardianos/mypkg/internal/github.com/coreos/etcd/raft"
	"github.com/kardianos/mypkg/context"
)
```

### Canonical and Local fields
The Canonical field is the path that would be used to fetch the non-copied revision
from GOPATH, the "go get" path. The Local field is the path to the package
relative to the vendor file. The Local field may not have the path elements
"." or ".." and as such vendor packages must be under the vendor file.

While it is usually ideal to not vendor a package that also vendors packages,
there are cases where there are no other options. Sometimes useful packages
are developed in the context of an executable. Sometimes it is useful to make
modifications to the standard library and vendor them with a package. This is
what the azure sdk and heartbleed detectors do.

By separating out the remote (go get) path from the local internal path,
a tool can detect when a standard library package is vendored and take steps
to prevent duplicating imports. Without the Local field a tool will be unable
to re-write the import path to be shorter without losing information.

### Revision and RevisionTime fields
Both revision fields are optional. However tools must persist any information
present in them. The interpretation of both fields is dependent on the tool
itself. While the exact interpretation of the fields are tool specific, the
semantics are not. The Revision field must either be empty or contain a single
value that can be used to fetch a specific revision. The RevisionTime must be
empty or have a valid RFC3339 time string. If the Revision field is non-empty
the RevisionTime field must correlate with the Revision field. If the Revision
field is empty the meaning of the RevisionTime field is tool specific.

### Well known fields used by tools
package.originPath : path relative to GOPATH where files were copied from if
 different from the canonical path.

package.originURL : URL to fetch remote revision from.

### Alternate check to verify single package import
When a package is rewritten it loses the original import path. This can lead
the possibility of importing the same package multiple times. An additional
check a tool may imply is to use package canonical comments. This is a comment
very similar to the package import comment as they both contain the original
canonical import path.

For example any one of these files:
```
file example path A: company/third_party/context/doc.go
file example path B: github.com/user/mypkg/internal/v/context/doc.go
file example path C: github.com/user/mypkg/internal/golang.org/x/net/context/doc.go
```
Could have the following package canonical comment.
```
package context // canonical import "golang.org/x/net/context"
```

This comment will be ignored by the go build tool. A vendor tool may use
this comment as an additional check to prevent duplicate packages in cases
where the vendor packages are copied locally, but not under the local project.

If the package canonical comment and the vendor file are in conflict the tool must
stop and report the discrepancy. If there are two package with the same
package canonical comment the vendor tool must stop and report the issue. If
canonical package comment is the same as a used import path the vendor tool must
stop and report the issue.
