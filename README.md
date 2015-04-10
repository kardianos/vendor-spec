## The Go vendor file specification.

 * Copying third-party vendor packages into a local project structure requires
    a meta-data file describing the vendor's packages.
 * The vendor packages should be placed in an "internal" directory.
 * The meta-data file describing the vendor's packages will be referred to as
    the vendor file.
 * The vendor file will be in the "internal" folder called "vendor.json".
 * The vendor file describes the vendor's packages rooted in the same "internal"
    folder.
 * The vendor file must contain a minimum set of fields.
 * Additional fields may be added to the vendor file that is tool specific.
 * The vendor file is to be used by a vendor tool; the vendor file is not used
    for building.
 * The following struct describes the minimum fields present in the json file:

```
struct {
	Tool string // Free text field describing the tool that last wrote this file.
	
	List []struct {
		// Import path. Example "rsc.io/pdf".
		// go get <Import> should fetch the remote package.
		//
		// If Import ends in "/..." the tool should manage all packages below 
		// the import as well.
		Import string
		
		// Package path relative to "internal" folder.
		// Examples: "rsc.io/pdf", "pdf".
		// If Local is an empty string, the tool should assume the path is
		// relative to GOPATH and the package is not currently copied
		// locally.
		// 
		// Local should not contain a trailing "/...".
		// Local should always use forward slashes.
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

*vendor package copied to: "$GOPATH/src/github.com/kardianos/mypkg/internal/pdf"*

```
{
	"Tool": "go vendor",
	"List": [
		{
			"Import": "rsc.io/pdf",
			"Local": "pdf",
			"Version": "3a3aeae79a3ec4f6d093a6b036c24698938158f3",
			"VersionTime": "2014-09-25T17:07:18Z-04:00"
		}
	]
}
```

### FAQ
 * Q: Why include a "Local" field? Isn't that just extra information?
    Shouldn't we always use fully qualified names?
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
    - A: The copied package will likely change when the imports are
	 rewritten. Tools are free to add a hash, but it should not be standard.
 * Q: Why not just re-use godeps meta-data file?
    - A: The godeps meta-data file includes the revision number in with
	 the revision hash.
	 It includes the go version number which is detramental in a team,
	 escpecially when one team member is tasked with getting a product
	 ready for the next Go version before that Go version is released.
	 It also lacks a Local field for adequate re-write
	 support and doesn't have a time of revision.
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
	 (the "Import") and what they can be found once copied (the "Local").
	 Version information is just common extra information.
