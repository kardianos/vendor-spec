## The Go vendor file specification

The tool that implements this specification can be installed by running:
```
go get -u github.com/kardianos/vendor
```

### Specification
 * Copying third-party vendor packages into a local project structure requires
    a meta-data file describing the vendor's packages.
 * The vendor packages should be placed in an "internal" directory.
 * The meta-data file describing the vendor's packages will be referred to as
    the vendor file.
 * The vendor file will be in the "internal" folder and be called "vendor.json".
 * The vendor file describes the vendor's packages rooted in the same "internal"
    folder.
 * The vendor file must contain a minimum set of fields.
 * Additional fields may be added to the vendor file that is tool specific.
 * The vendor file is to be used by a vendor tool; the vendor file is not used
    for compiling source code.
 * Each copied package entry is defined to match a single Go package. A
    vendor file Package entry does NOT match a package tree; a vendor file
	Package entry maches a single Go package.
 * The following struct describes the minimum fields that must be present in
    the json file:

```
struct {
	// The import path of the tool used to write this file.
	// Examples: "github.com/kardianos/vendor" or "golang.org/x/tools/cmd/vendor".
	Tool string
	
	// Comment is free text for human use.
	Comment string
	
	Package []struct {
		// Vendor import path. Example "rsc.io/pdf".
		// go get <Vendor> should fetch the remote vendor package.
		Vendor string
		
		// Package path as found in GOPATH.
		// Examples: "path/to/mypkg/internal/rsc.io/pdf".
		// If Local is an empty string, the tool should assume the
		// package is not currently copied locally.
		// 
		// Local should always use forward slashes and must not contain the
		// path elements "." or "..".
		Local string
		
		// The revision of the package. This field must be persisted by all
		// tools, but not all tools will interpret this field.
		// The value of Revision should be a single value that can be used
		// to fetch the same or similar revision.
		// Examples: "abc104...438ade0", "v1.3.5"
		Revision string
		
		// RevisionTime is the time the revision was created. The time should be
		// parsed and written in the "time.RFC3339" format.
		RevisionTime string
		
		// Comment is free text for human use.
		Comment string
	}
}
```

### Example
*vendor file path: "$GOPATH/src/github.com/kardianos/mypkg/internal/vendor.json"*

*first package copied to: "$GOPATH/src/github.com/kardianos/mypkg/internal/rsc.io/pdf"*

```
{
	"Tool": "github.com/kardianos/vendor",
	"Package": [
		{
			"Vendor": "rsc.io/pdf",
			"Local": "github.com/kardianos/mypkg/internal/rsc.io/pdf",
			"Revision": "3a3aeae79a3ec4f6d093a6b036c24698938158f3",
			"RevisionTime": "2014-09-25T17:07:18Z-04:00"
		},
		{
			"Vendor": "github.com/MSOpenTech/azure-sdk-for-go/internal/crypto/tls",
			"Local": "github.com/kardianos/mypkg/internal/crypto/tls",
			"Revision": "80a4e93853ca8af3e273ac9aa92b1708a0d75f3a",
			"RevisionTime": "2015-04-07T09:07:157Z-07:00"
		},
		{
			"Vendor": "github.com/coreos/etcd/raft",
			"Local": "github.com/kardianos/mypkg/internal/github.com/coreos/etcd/raft",
			"Revision": "25f1feceb5e13da68a35ee552069f86d18d63fee",
			"RevisionTime": "2015-04-09T05:06:17Z-08:00"
		},
		{
			"Vendor": "github.com/coreos/etcd/internal/golang.org/x/net/context",
			"Local": "github.com/kardianos/mypkg/internal/golang.org/x/net/context",
			"Revision": "25f1feceb5e13da68a35ee552069f86d18d63fee",
			"RevisionTime": "2015-04-09T05:06:17Z-08:00"
		}
	]
}
```
*The above example would have the import paths as follows:*
```
import (
	"github.com/kardianos/mypkg/internal/rsc.io/pdf"
	"github.com/kardianos/mypkg/internal/crypto/tls"
	"github.com/kardianos/mypkg/internal/github.com/coreos/etcd/raft"
	"github.com/kardianos/mypkg/internal/golang.org/x/net/context"
)
```

### Proposed addition to support revision pinning and GOPATH modifications
In each Package struct, add the field "ImportAs string". Valid values are
"vendor" and "local". If the value is "vendor", then the import path is expected
to be the value of the "Vendor" field. If the value is "local", the import
path is expected to be the "Local" field. If the "ImportAs" value is omitted,
the default will be treated as "vendor".

Revision pinning example (Package struct item only):
```
{
	"Vendor": "github.com/coreos/etcd/raft",
	"Local": "github.com/coreos/etcd/raft",
	"ImportAs": "vendor",
	"Revision": "25f1feceb5e13da68a35ee552069f86d18d63fee",
	"RevisionTime": "2015-04-09T05:06:17Z-08:00"
}
```

GOPATH modify (godep save) example (Package struct item only):
```
{
	"Vendor": "github.com/coreos/etcd/raft",
	"Local": "github.com/kardianos/mypkg/internal/src/github.com/coreos/etcd/raft",
	"ImportAs": "vendor",
	"Revision": "25f1feceb5e13da68a35ee552069f86d18d63fee",
	"RevisionTime": "2015-04-09T05:06:17Z-08:00"
}
```

Import path rewrite example (Package struct item only):
```
{
	"Vendor": "github.com/coreos/etcd/raft",
	"Local": "github.com/kardianos/mypkg/internal/github.com/coreos/etcd/raft",
	"ImportAs": "local",
	"Revision": "25f1feceb5e13da68a35ee552069f86d18d63fee",
	"RevisionTime": "2015-04-09T05:06:17Z-08:00"
}
```

In all cases the "Local" field points to where the package sits on disk within
the GOPATH. The "Vendor" field always gives the original vendor's import path.

### Tool field
A vendor tool is allowed to write a superset of fields in the file. To know
how to interpret these fields the name of the tool must be known.

### Vendor and Local fields
The Vendor field is the path that would be used to fetch the non-copied revision
from GOPATH, the "go get" path. The Local field is the path to the package
in GOPATH. The Local field may not have the path elements
"." or "..".

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
