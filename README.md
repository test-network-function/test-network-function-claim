# test-network-function-claim

`test-network-function-claim` provides the definition for a
[test-network-function](https://github.com/redhat-nfvpe/test-network-function) claim.

A claim contains:
* A start time for the claim evaluation.
* A description of the Hardware System(s) Under Test
* All test configurations used by tests.
* All tests run, and their respective results.
* An end time for the claim evaluation.

`claim-schema.json` defines the claim schema using
[JSON Schema Draft-07](https://json-schema.org/draft-07/json-schema-release-notes.html).  JSON Schema serves as the
only definition language for a `test-network-function` claim.  In other words, even though other language bindings are
possible (and some provided), the ultimate claim definition is `claim-schema.json`.  In the unfortunate event of
ambiguity between `claim-schema.json` and a particular language binding, the former rules.

## Language Bindings

A GoLang implementation of for `claim` is exposed through `pkg/claim/schema.go`.  This is currently the only language
binding. 

## Modifying the claim schema

In order to make changes to `claim-schema.json`, please follow these steps.
 
### Edit the file as you see fit

Make the appropriate changes and save the `claim-schema.json`.

### Build the language bindings

After you are done editing the
file to a desired state, issue the following command:

```shell script
make
```

This will run the Makefile targets necessary to generate a new `pkg/claim/schema.go` GoLang schema language binding,
run the linter, run `gofmt` and execute all available tests.

One should take all effort to ensure changes to the schema are backwards compatible.  In the event that a breaking
change is necessary, please make sure to include that information in the commit message.  All breaking schema changes
require a semantic version major version bump upon release.

Due to the fact that [a-h/generate](https://github.com/a-h/generate) generates the GoLang language binding, some
generated code is not easily modifiable.  The library will occasionally output un-testable code, so 100% unit test is
not a requirement.  An example of un-testable code is:

```go
comma := false
// "EndTime" field is required
// only required object types supported for marshal checking (for now)
// Marshal the "endTime" field
if comma {
	buf.WriteString(",")
}
```

In the above example, the conditional will never evaluate as `true`.  This is a performance limitation of the tool that
is accepted, as the GoLang compiler will likely elide the conditional using Abstract Syntax Tree pruning anyway as a
compile-time optimization.

### Overriding type inferences

[a-h/generate](https://github.com/a-h/generate) is known to output JSON `object` types as GoLang `struct`s.  Although
this is normally desirable, we occasionally wish to allow arbitrary JSON instead.  In this case, we have no struct
definition for some facets of our claim, such as "lshw" and "results".  This is due to the fact that others define those
JSON structures, and we are not in the business of redefinition.  This is a rare case in which we want to allow GoLang
to represent arbitrary JSON data.

To remap a property type, please consult `override.json`, which provides an entrypoint for remapping types in GoLang
generated client code.

## Validating a JSON claim input file

### Requirements

[`jsonschema`](https://python-jsonschema.readthedocs.io/en/stable/) must be installed and made available in `$PATH`.

In order to install `jsonschema`, issue the following command:

```shell script
> pip3 install jsonschema
```

### Running the validation utility

In order to validate a given schema, issue the following command:

```shell script
> jsonschema -i <path_to_claim_file> ./claim-schema.json
```

For a successful validation of the input claim, one should expect no output and a `0` exit code.

```shell script
> jsonschema -i ./sample.json ./claim-schema.json
> echo $?
0
> 
```

If the claim does not adhere to the schema, `jsconschema` outputs an appropriate error message describing the issue.
