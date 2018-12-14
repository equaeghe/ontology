# Schema and meta-schema experiments

This folders contains a number of schemata (and a meta-schema) that are meant
to illustrate specific features of JSON schema. We discuss each of these
features below.

(No checks with validators were done yet, so it may be that validators are not
able to cope with some of these features.)

## Extending the JSON schema meta-schema

The JSON schema language does not define all the fields we might need. For
example, a ‘unit’ field would be informative if we could add it. The JSON
schema language is defined in the corresponding ‘meta-schema’. [It is possible
to create and extended version of this meta-schema][ext] and use that instead.
So to add a ‘unit’ field, we need to create such an extended meta-schema that
includes support for it. A proposal of such a meta-schema can be found in the
file `extended-json-schema.yaml`.

In that file, most of the lines are actually boilerplate needed because of some
technical reasons related to the recursive nature of such meta-schemata. That
should only concern us if we do more invasive extensions. For now, we can do
with simply defining the extra ‘unit’ field. This is done at the end of the
file and is very straightforward.

Once the extended meta-schema has been created, it can be used in schemata we
write. All the other files in this folder are built on the proposed extension.
In each of these files, this is done by replacing the original `$schema` field
value `"http://json-schema.org/draft-07/schema#"` with a reference to our
extension `"https://github.com/equaeghe/ontology/blob/schema-experiments/schemata_erik/extended-json-schema.yaml#"`.

Please have a look in these files and search for `unit:` to see how one can use
the unit field.

As a closing aside, it can be useful to share some thoughts about how we should
write the units. This should be done in a way that software can deal with it,
for whatever purposes. I have looked around a bit and one option seems to be to
use notation compatible with the ‘[udunits][udunits]’ software package, which,
e.g., for Python can be used via the ‘[CF-units][cfunits]’ package.

[ext]: https://json-schema.org/latest/json-schema-core.html#rfc.section.6.4
[udunits]: https://www.unidata.ucar.edu/software/udunits/
[cfunits]: https://github.com/SciTools/cf-units

## Internal referencing using `$id` and `$ref`

To avoid duplication within a schema, it is convenient to be able to link to
material defined elsewhere in the schema. A standard way to do this is to
create `definitions` part with sub-schemata each with their own name.

Let us assume that we have created a `thing` subschema under `definitions`.
Then we can reference it elsewhere using `$ref: "#/definitions/thing"`; i.e.,
the material under `thing` will then be included in replacement of that
reference. (In this reference, `#/definitions/thing` is an absolute JSON
pointer: `#` refers to the root of the schema and `/definitions/thing` is the
path to the `thing` subschema. The quotes are necessary because `#` is the
comment character in YAML.)

There is [another way in which one can reference other material][id]. By
including an identifier for the `thing` subschema, we can reference it directly
using that identifier. This is done by adding `$id: "#thingid"` under `thing`,
where `thingid` can be chosen freely as long as it is unique within the schema.
Then we can now reference it elsewhere using `$ref: "#thing"`.

This approach with `$id` and `$ref` perfectly mirrors the referencing built
into YAML. But this way the reference can be preserved even if we store the
schema as JSON or some other format. As an example, have a look at the
definition of `weibull_parameters` in the file `wind_resource-schema.yaml`
(lines 156—174).

[id]: https://json-schema.org/latest/json-schema-core.html#rfc.section.8.3.2

## Referencing (parts of) other schema files

Referencing other schema files or parts thereof is in principle done in the
same way as the internal references discussed above. The thing that changes is
that within the reference, the location of the other file must be made
explicit. In general this is done by using URIs, but for files that are located
in the same location (with the same base-uri) this can be done just by
prepending the file name.

In the set of schemata included in this folder, we have created the files
`location-schema.yaml` and `variables-schema.yaml` to be or contain the
material we reference from other files. The whole of `location-schema.yaml` is
referenced from `wind_resource-schema.yaml` on line 55. Parts of
`variables-schema.yaml` are referenced `wind_resource-schema.yaml` and
`turbine-schema.yaml`; search for ‘`$ref: "variables-schema.yaml#`’ to see some
instances.

## Referencing documents parts using (relative) JSON pointers

Whereas the mechanisms described above are about referencing within a schema,
we now consider referencing document parts from a schema. The way this is done
is *experimental* within JSON schema and therefore prone to syntax changes and
likely not supported by validators. However, it allows us to express
constraints in a very nice way.

The approach is to include a `$data` field that contains a
[JSON pointer][pointer] or a [relative JSON pointer][relpointer] to the
part of the document that one wishes to reference. (The syntax for these
pointers is straightforward as long as one avoids ‘`~`’ and ‘`/`’ in field
names.)

In the file `turbine-schema.yaml` we have used this approach to give bounds on
values. For example, on line 71 there, we require that the hub height provided
is strictly larger than the rotor radius. (A relative JSON pointer was used for
this here, but I think also an absolute pointer or an `$id` reference could be
used.)

JSON pointers can be used to reference within and between documents, as long as
their schema caters for this, i.e., [allows a JSON pointer][pointerformat] to
some material instead of directly including the material. ([`oneOf`][oneof]
could be used for this.) This is not elaborated here.

[pointer]: https://tools.ietf.org/html/rfc6901
[relpointer]: https://tools.ietf.org/html/draft-handrews-relative-json-pointer-01
[pointerformat]: https://json-schema.org/latest/json-schema-validation.html#rfc.section.7.3.7
[oneof]: https://json-schema.org/latest/json-schema-validation.html#rfc.section.6.7.3
