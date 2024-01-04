In a monorepo, where vendored packages contain `manifest = "../../Manifest.toml"` in their `Project.toml`, if any of the vendored packages depends on an unregistered package, all the other packages in the monorepo will fail to resolve their environment when we try to test them.

How to reproduce: In this repo, `julia --project`, activating the environment, and running `test` will succeed for `SubPackageWithUnregisteredDep` (which depends on an unregistered package "ObjectStore"), but fail for `OtherSubPackage`:

```julia
(Repro) pkg> test SubPackageWithUnregisteredDep
     Testing SubPackageWithUnregisteredDep
      Status `/private/var/folders/jq/n4hkdx3968z1qw60dlgzgmnr0000gn/T/jl_oLCIlZ/Project.toml`
  [1b5eed3d] ObjectStore v0.1.0 `https://github.com/RelationalAI/ObjectStore.jl#main`
  [781b6521] SubPackageWithUnregisteredDep v0.1.0 `~/repro/vendor/SubPackageWithUnregisteredDep`
  [8dfed614] Test
      Status `/private/var/folders/jq/n4hkdx3968z1qw60dlgzgmnr0000gn/T/jl_oLCIlZ/Manifest.toml`
  [1b5eed3d] ObjectStore v0.1.0 `https://github.com/RelationalAI/ObjectStore.jl#main`
  [781b6521] SubPackageWithUnregisteredDep v0.1.0 `~/repro/vendor/SubPackageWithUnregisteredDep`
  [2a0f44e3] Base64
  [b77e0a4c] InteractiveUtils
  [56ddb016] Logging
  [d6f4376e] Markdown
  [9a3f8284] Random
  [ea8e919c] SHA v0.7.0
  [9e88b42a] Serialization
  [8dfed614] Test
Precompiling project...
  1 dependency successfully precompiled in 1 seconds. 2 already precompiled.
     Testing Running tests...
     Testing SubPackageWithUnregisteredDep tests passed 
```


```julia
(Repro) pkg> test OtherSubPackage
     Testing OtherSubPackage
┌ Warning: Could not use exact versions of packages in manifest, re-resolving
└ @ Pkg.Operations ~/.julia/juliaup/julia-1.10.0+0.aarch64.apple.darwin14/share/julia/stdlib/v1.10/Pkg/src/Operations.jl:1813
ERROR: Unsatisfiable requirements detected for package ObjectStore [1b5eed3d]:
 ObjectStore [1b5eed3d] log:
 ├─ObjectStore [1b5eed3d] has no known versions!
 └─restricted to versions * by SubPackageWithUnregisteredDep [781b6521] — no versions left
   └─SubPackageWithUnregisteredDep [781b6521] log:
     ├─possible versions are: 0.1.0 or uninstalled
     └─SubPackageWithUnregisteredDep [781b6521] is fixed to version 0.1.0

```
