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
		
		// The version of the package. This field must be persisted by all
		// tools, but not all tools will interpret this field.
		// The value of Version should be a single value that can be used
		// to fetch the same or similar version.
		// Examples: "abc104...438ade0", "v1.3.5"
		Version string
		
		// VersionTime is the time the version was created. The time should be
		// parsed and written in the "time.RFC3339" format.
		VersionTime string
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
			"Version": "3a3aeae79a3ec4f6d093a6b036c24698938158f3",
			"VersionTime": "2014-09-25T17:07:18Z-04:00"
		},
		{
			"Vendor": "github.com/MSOpenTech/azure-sdk-for-go/internal/crypto/tls",
			"Local": "github.com/kardianos/mypkg/internal/crypto/tls",
			"Version": "80a4e93853ca8af3e273ac9aa92b1708a0d75f3a",
			"VersionTime": "2015-04-07T09:07:157Z-07:00"
		},
		{
			"Vendor": "github.com/coreos/etcd/raft",
			"Local": "github.com/kardianos/mypkg/internal/github.com/coreos/etcd/raft",
			"Version": "25f1feceb5e13da68a35ee552069f86d18d63fee",
			"VersionTime": "2015-04-09T05:06:17Z-08:00"
		},
		{
			"Vendor": "github.com/coreos/etcd/internal/golang.org/x/net/context",
			"Local": "github.com/kardianos/mypkg/internal/golang.org/x/net/context",
			"Version": "25f1feceb5e13da68a35ee552069f86d18d63fee",
			"VersionTime": "2015-04-09T05:06:17Z-08:00"
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

### Tool field
A vendor tool is allowed to write a superset of fields in the file. To know
how to interpret these fields the name of the tool must be known.

### Vendor and Local fields
The Vendor field is the path that would be used to fetch the non-copied version
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

### Version and VersionTime fields
Both version fields are optional. However tools must persist any information
present in them. The interpretation of both fields is dependent on the tool
itself. While the exact interpretation of the fields are tool specific, the
semantics are not. The Version field must either be empty or contain a single
value that can be used to fetch a specific version. The VersionTime must be
empty or have a valid RFC3339 time string. If the Version field is non-empty
the VersionTime field must correlate with the Version field. If the Version
field is empty the meaning of the VersionTime field is tool specific.

### Open Questions
 * JSON documents do not support comments and are less friendly for a human
    to read and write then something like toml. However toml is not supported
	in the standard library, nor is the specification complete.
	We could define a format just for this, perhaps similar to a ini file
	or custom comments in go code, but that is not the problem being solved.
	That being said, I could see a sub-set of toml defined and used.
	Only support string values, tables, and comments.

```
Tool = "github.com/kardianos/vendor"
# Read the content of the PDF from the azure service.
[[Package]]
Vendor = "rsc.io/pdf"
Local = "github.com/kardianos/mypkg/internal/rsc.io/pdf"
Version = "3a3aeae79a3ec4f6d093a6b036c24698938158f3"
VersionTime = "2014-09-25T17:07:18Z-04:00"
[[Package]]
Vendor = "github.com/MSOpenTech/azure-sdk-for-go/internal/crypto/tls"
Local = "github.com/kardianos/mypkg/internal/crypto/tls"
Version = "80a4e93853ca8af3e273ac9aa92b1708a0d75f3a"
VersionTime = "2015-04-07T09:07:157Z-07:00"
```

* Should the VersionTime field be present? It isn't needed for any mechanical
    process, but a maintainer 5 years later may find it useful to pin-point
    what time period that code was from.

* Is it practical to support vendoring std library packages? Sometimes done
    to work around a bug or limitation in the currently released std package.
    Examples include forking crypto/tls to connect to azure, dwarf for debugging,
    crypto/tls to test for the heart bleed attack.

### A Rational for Package Copying and Import Path Re-writing
It is a desired trait to build code after fetching or updating the source
without an additional command. It has been observed that for non-trivial
packages appending the GOPATH with an internal
vendor root is problematic. The Go maintainers will not re-define or add to
the method used to build packages. It is a firm requirement for many projects,
especially commercial projects, to keep a local copy of everything that is
used to build the product and use that copy whenever the product is built.

There are many users of Go who do not agree with the above statements or do
not require all the source to be kept with the product. For example, some
users only require a path to the remote repository and a version to use.
The presence of users with more relaxed needs does not alleviate the needs
outlined in the first paragraph.

The only solution that meets all the needs in the first paragraph is copying
the source to the project and re-writing the import paths. Not liking the
solution does not make the problem disappear or make the needs change.

### FAQ
 * Q: Why include a "Local" field? We should just use fully qualified names.
     - A: Cases sometimes arise where the top level main package must import
	 a package that also vendors packages. If the top level main package
	 also uses these packages then the import paths become extra long.
	 It would be beneficial to be able to move the second level vendor
	 packages to be first level vendor packages while still keeping the
	 original source. The fact that this is unideal to begin with
	 doesn't help the underlying problem.
	 Some tools may prefer to follow a set of rules that shortens the
	 vendor package's paths.
 * Q: Why omit the revision number?
    - A: revision numbers are not guaranteed to be sequential in distributed
	 version control systems. Centralized systems should just use Version.
 * Q: Why not contain a hash of each package to ensure the package
    doesn't change?
    - A: The copied package will change when the imports are
	 rewritten. Tools are free to add a hash, but it is not standard.
 * Q: Why not just re-use godeps meta-data file?
    - A: Godep (github.com/tools/godep) is a great tool with a nice meta-data
	 file. It can serve as a good base for a standard.
	 The go version is not helpful and can be harmful as different developers
	 may be on different versions, changing it each time.
	 It was designed to support a local GOPATH, so it lacks the Local field.
 * Q: Why record the name of the last tool to write out the vendor file?
    - A: If there is a malformed vendor file or re-writes that are found
	 to be problematic, the tool used can then be traced back and corrected.
	 It will also be useful for statistics.
 * Q: Why not include the go version?
    - A: It isn't required when recording vendor packages. It can change
	 needlessly based on who ran the tool last if different team members
	 are on different go versions. It makes little sense if one team member
	 is testing on the Go master branch.
 * Q: Why not separate out repositories from packages?
    - A: In practice vendor tools should work with packages, not repositories.
	 If a vendor tool is able to sniff out the source control version and time
	 then that's extra information for a human to manage the package.
	 Machines only need to know what a package used to be called
	 (the "Vendor") and what they can be found once copied (the "Local").
  * Q: Seriously, can't we rely on github to stay around and just pin the version?
     - A: I've maintained line of business applications that are over 15 years
	    old. In one case many of the dependent third-party components
		were no longer available. A re-write eventually took nearly a year to
		complete. Companies that are more then a few years old will
		have legacy programs. Libraries will drop off the face of the Earth.
 * Q: What if we just copied the vendor code to the repository, then copy them to
    the GOPATH locations after a fetch? I don't like the idea of import re-writes.
    - A: If you setup one project per GOPATH it would mostly work ("go get"
		wouldn't work). However, if two projects that shared the same GOPATH
		relied on the same vendor package, but different versions, then one vendor package
		would clobber the other vendor package. This approach doesn't work when scaling
		a company code base.
 * Q: What about putting the hash or similar version in each package folder path?
    - A: I'd be interested to see the tool that took this approach. I'm not
	    convinced that it would be practical or that it could be done without
		modifying the "go" command.
