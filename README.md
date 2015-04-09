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
		// Local should never contain a trailing "/..."
		// Local should always use forward slashes.
		Local string
		
		// The version of the package. This field must be persisted by all
		// tools, but not all tools will interpret this field.
		// The value of Version should be a single value that can be used
		// to fetch the same or similar version.
		// Examples: "abc104...438ade0", "v1.3.5"
		Version string
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
			"Version": ""
		}
	]
}
```
