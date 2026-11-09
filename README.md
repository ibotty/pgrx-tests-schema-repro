# pgrx-tests-schema-repro

`top_ext` `requires` `dep_ext`; both carry the standard `#[pg_schema] mod tests`.
Running `dep_ext`'s harness and then `top_ext`'s on the same cluster fails.

```
cd dep_ext && cargo pgrx test pg18            # passes; installs dep_ext with `CREATE SCHEMA IF NOT EXISTS tests`
cd ../top_ext && cargo pgrx test pg18         # FAILS
#   Could not initialize test framework: There was an issue creating the extension 'top_ext' in Postgres:
#   ERROR SQLSTATE[55000]: schema tests is not a member of extension "top_ext"
#   query: CREATE EXTENSION top_ext CASCADE;

# Workaround: reinstall the dependency without pg_test, then top_ext passes.
cd ../dep_ext && cargo pgrx install --pg-config "$(cargo pgrx info pg-config pg18)" --no-default-features --features pg18
cd ../top_ext && cargo pgrx test pg18         # passes

# Renaming top_ext's module (`mod top_ext_tests`) does not help: pgrx-tests hardcodes the schema.
#   ERROR: function tests.top_works() does not exist
```

pgrx 0.19.2 from crates.io, PostgreSQL 18.
