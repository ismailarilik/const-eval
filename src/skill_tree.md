# Skill tree for const eval features

```skill-tree
[[group]]
name = "cell_ref"
label = "references to interior mutability\nfeature:const_cell_refs"
href = "https://github.com/rust-lang/rust/issues/79703"
items = []

[[group]]
name = "mut_ref"
label = "mutable references in\nconst fn\nfeature:const_mut_refs"
href = "https://github.com/rust-lang/rust/issues/57349"
items = []

[[group]]
name = "const_mut_ref"
label = "mutable references in\ninitializers of const items"
href = "https://github.com/rust-lang/rust/issues/71212"
requires = ["mut_ref"]
items = []

[[group]]
name = "file_output"
label = "write to files\nfrom constants"
href = "https://github.com/rust-lang/const-eval/issues/25"
items = []

[[group]]
name = "final_heap"
label = "heap allocations\nin the final value of constants"
requires = ["heap"]
href = "https://github.com/rust-lang/const-eval/issues/20"
items = []

[[group]]
name = "heap"
label = "heap allocations"
requires = ["mut_ref", "cell_ref", "trait_impl"]
items = []
href = "https://github.com/rust-lang/const-eval/issues/20"

[[group]]
name = "iterators"
label = "iterators"
requires = ["mut_ref", "trait_impl"]
items = []

[[group]]
name = "for"
label = "for loops"
requires = ["trait_impl", "mut_ref"]
items = []

[[group]]
name = "slice_eq"
label = "[T]::eq"
requires = ["for", "fuzzy-ptr-comparisons", "trait_impl"]
items = []

[[group]]
name = "box_new"
label = "Box::new"
requires = ["heap", "trait_bound_opt_out", "drop"]
items = []

[[group]]
name = "ptr-is-null"
label = "<*T>::is_null\nfeature:const_ptr_is_null"
requires = ["fuzzy-ptr-comparisons"]
href = "https://github.com/rust-lang/rust/issues/74939"
items = []

[[group]]
name = "fuzzy-ptr-comparisons"
label = "guaranteed_eq and\nguaranteed_ne\nfeature:const_compare_raw_pointers"
href = "https://github.com/rust-lang/rust/issues/53020"
requires = []
items = []

[[group]]
name = "unconst_rules"
label = "unconst operations"
href = "https://github.com/rust-lang/const-eval/issues/14"
items = []

[[group]]
name = "raw_ptr_deref"
label = "feature:const_raw_ptr_deref"
items = []
href = "https://github.com/rust-lang/rust/issues/51911"

[[group]]
name = "offset_of"
label = "offset_of"
items = []
requires = [
  "raw_ptr_deref",
  "raw_ref_macros",
  "maybe_uninit_as_ptr",
  "offset_from",
]

[[group]]
name = "offset"
label = "offset\nfeature:const_ptr_offset"
href = "https://github.com/rust-lang/rust/issues/71499"
items = []

[[group]]
name = "offset_from"
label = "offset_from\nfeature:ptr_offset_from"
href = "https://github.com/rust-lang/rust/issues/41079"
items = []

[[group]]
name = "raw_ref_macros"
label = "raw_ref maros\nfeature:raw_ref_macros"
href = "https://github.com/rust-lang/rust/issues/73394"
items = []

[[group]]
name = "maybe_uninit_as_ptr"
label = "MaybeUninit::as_ptr\nfeature:const_maybe_uninit_as_ptr"
href = "https://github.com/rust-lang/rust/issues/75251"
items = []

[[group]]
name = "question_mark"
label = "using ? in const"
href = "https://github.com/rust-lang/rust/issues/74935"
requires = ["trait_impl"]
items = []

[[group]]
name = "mutex_new"
label = "Mutex::new"
href = "https://github.com/rust-lang/rust/issues/66806"
items = []
requires = ["parking_lot"]

[[group]]
name = "parking_lot"
label = "`parking_lot` in `std`"
href = "https://github.com/rust-lang/rust/issues/73714"
items = []

[[group]]
name = "const_fn_in_patterns"
label = "const fn callable in patterns"
href = "https://github.com/rust-lang/rust/issues/57240"
requires = ["const-blocks"]
items = []

[[group]]
name = "from_str"
label = "FromStr"
href = "https://github.com/rust-lang/rust/issues/59133"
requires = ["trait_impl"]
items = []

[[group]]
name = "float"
label = "floats in const fn"
href = "https://github.com/rust-lang/rust/issues/57241"
items = [
    { label = "from_bits" },
    { label = "to_bits" },
    { label = "float math, arguments and return types" },
]

[[group]]
name = "float_classify"
label = "feature:const_float_classify"
href = "https://github.com/rust-lang/rust/issues/72505"
items = []
requires = ["float_bits_conv"]

[[group]]
name = "float_bits_conv"
label = "feature:const_float_bits_conv"
href = "https://github.com/rust-lang/rust/issues/72447"
items = []
requires = ["float"]

[[group]]
name = "const-assert-eq"
label = "assert_eq!"
requires = ["trait_impl", "panic_fmt"]
href = "https://github.com/rust-lang/rust/issues/74925"
items = []

[[group]]
name = "const-blocks"
label = "inline const"
href = "https://github.com/rust-lang/rust/issues/76001"
items = []

[[group]]
label = "panic! with formatting"
name = "panic_fmt"
requires = ["format_args"]
href = "https://github.com/rust-lang/rust/issues/51999"
items = []

[[group]]
label = "feature:const_discriminant"
name = "discriminant"
href = "https://github.com/rust-lang/rust/pull/69825"
items = []

[[group]]
label = "feature:const_trait_bound_opt_out"
name = "trait_bound_opt_out"
href = "https://github.com/rust-lang/rust/issues/67794"
items = []

[[group]]
label = "feature:const_trait_impl"
name = "trait_impl"
href="https://github.com/rust-lang/rust/issues/67792"
items = []
requires = ["trait_bound_opt_out"]

[[group]]
label = "feature:const_raw_ptr_to_usize_cast"
name = "raw_ptr_to_usize_cast"
href="https://github.com/rust-lang/rust/issues/51910"
items = []
requires = ["unconst_rules"]

[[group]]
label = "feature:const_extern_fn"
name = "extern_const_fn"
href = "https://github.com/rust-lang/rust/issues/64926"
items = []

[[group]]
label = "const function pointers"
name = "const_fn_ptr"
href = "https://github.com/rust-lang/rust/issues/63997"
items = []
requires = ["trait_impl", "trait_bound_opt_out"]

[[group]]
label = "format!"
name = "format"
items = []
requires = ["string", "format_args"]

[[group]]
label = "format_args!"
name = "format_args"
items = []
requires = ["trait_impl", "fuzzy-ptr-comparisons"]

[[group]]
label = "String operations"
name = "string"
items = []
requires = ["vec"]

[[group]]
label = "Vec operations"
name = "vec"
items = []
requires = ["mut_ref", "heap", "trait_impl", "drop", "raw_ptr_deref"]

[[group]]
label = "Drop"
name = "drop"
items = []
requires = ["mut_ref", "trait_impl"]

[[group]]
label = "ptr::copy_nonoverlapping\nfeature:const_intrinsic_copy"
name = "copy_nonoverlapping"
items = []
href = "https://github.com/rust-lang/rust/issues/80697"
requires = ["raw_ptr_deref", "mut_ref"]

[[group]]
label = "async functions\nand blocks"
name = "async"
items = []
href = "https://github.com/rust-lang/rust/issues/69431"
requires = ["trait_impl"]
```
