I tried to find out how it works in `go to definition`.
I _think_ the expanded macro is analysed in `DefDatabase::crate_def_map`
<details><summary>(see call stack)</summary>
<p>

```rust
ra_ide_db::defs::classify_name_ref
Semantics::resolve_path
Semantics::analyze
Semantics::analyze2
<VariantId as HasResolver>::resolver
<ModuleId as HasResolver>::resolver
DefDatabase::crate_def_map
ra_hir_def::db::crate_def_map_wait
DefDatabase::crate_def_map_query
CrateDefMap::crate_def_map_query
ra_hir_def::nameres::collector::collect_defs
DefCollector::collect
DefCollector::resolve_macros
DefCollector::collect_macro_expansion
DefDatabase::raw_items
RawItems::raw_items_query
AstDatabase::parse_or_expand
ra_hir_expand::db::parse_or_expand
```
<hr>
</p>
</details>

So `ra_hir_expand::db::parse_or_expand` is what turns `HirFileId(MacroFile(MacroFile {macro_call_id: LazyMacro(LazyMacroId(0))}))` into ```
MACRO_ITEMS@[0; 18)
  STRUCT_DEF@[0; 18)
    STRUCT_KW@[0; 6) "struct"
    NAME@[6; 9)
      IDENT@[6; 9) "Foo"
    TUPLE_FIELD_DEF_LIST@[9; 17)
      L_PAREN@[9; 10) "("
      TUPLE_FIELD_DEF@[10; 16)
        PATH_TYPE@[10; 16)
          PATH@[10; 16)
            PATH_SEGMENT@[10; 16)
              NAME_REF@[10; 16)
                IDENT@[10; 16) "String"
      R_PAREN@[16; 17) ")"
    SEMI@[17; 18) ";"
```

I'm a bit confused over the different approaches though:
`world_symbols` first builds data structures for all source files and then runs the query against these data structures.
`goto_definition` seems to have a more complex implementation with more special cases which seems more depth-first-search-like.

Is the difference caused by `goto_definition` assuming that the current path is already valid? Or is this a situation where one of the two functions should be changed at some point in the future? Off the top of my head, I don't see why `goto_definition` couldn't run on the same logic as `auto_import`.

