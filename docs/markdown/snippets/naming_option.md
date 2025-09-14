## Added new `namingscheme` option

Traditionally Meson has named output targets so that they don't clash
with each other. This has meant, among other things, that on Windows
Meson uses a nonstandard `.a` suffix for static libraries because both
static libraries and import libraries have the suffix `.lib`.

Starting from this release Meson uses a more platform native naming
scheme that replicates what Rust does. That is, shared libraries on
Windows get a suffix `.dll`, static libraries get `.lib` and import
libraries have the name `.dll.lib`.

Setting the `namingscheme` option name to `classic` restores the old
behaviour.
