# v2.0.0 - yyyy-mm-dd


## Changes affecting backward compatibility
- `httpclient.contentLength` default to `-1` if the Content-Length header is not set in the response. It follows Apache HttpClient(Java), http(go) and .Net HttpWebResponse(C#) behavior. Previously it raised `ValueError`.

- `addr` is now available for all addressable locations,
  `unsafeAddr` is now deprecated and an alias for `addr`.

- Certain definitions from the default `system` module have been moved to
  the following new modules:

  - `std/syncio`
  - `std/assertions`
  - `std/formatfloat`
  - `std/objectdollar`
  - `std/widestrs`
  - `std/typedthreads`
  - `std/sysatomics`

  In the future, these definitions will be removed from the `system` module,
  and their respective modules will have to be imported to use them.
  Currently, to make these imports required, the `-d:nimPreviewSlimSystem` option
  may be used.

- Enabling `-d:nimPreviewSlimSystem` also removes the following deprecated
  symbols in the `system` module:
  - Aliases with `Error` suffix to exception types that have a `Defect` suffix
    (see [exceptions](https://nim-lang.org/docs/exceptions.html)):
    `ArithmeticError`, `DivByZeroError`, `OverflowError`,
    `AccessViolationError`, `AssertionError`, `OutOfMemError`, `IndexError`,
    `FieldError`, `RangeError`, `StackOverflowError`, `ReraiseError`,
    `ObjectAssignmentError`, `ObjectConversionError`, `FloatingPointError`,
    `FloatOverflowError`, `FloatUnderflowError`, `FloatInexactError`,
    `DeadThreadError`, `NilAccessError`
  - `addQuitProc`, replaced by `exitprocs.addExitProc`
  - Legacy unsigned conversion operations: `ze`, `ze64`, `toU8`, `toU16`, `toU32`
  - `TaintedString`, formerly a distinct alias to `string`
  - `PInt32`, `PInt64`, `PFloat32`, `PFloat64`, aliases to
    `ptr int32`, `ptr int64`, `ptr float32`, `ptr float64`

- Enabling `-d:nimPreviewSlimSystem` removes the import of `channels_builtin` in
  in the `system` module.

- Enabling `-d:nimPreviewCstringConversion`, `ptr char`, `ptr array[N, char]` and `ptr UncheckedArray[N, char]` don't support conversion to cstring anymore.

- The `gc:v2` option is removed.

- The `mainmodule` and `m` options are removed.

- The `threads:on` option is now the default.

- Optional parameters in combination with `: body` syntax (RFC #405) are now opt-in via
  `experimental:flexibleOptionalParams`.

- Automatic dereferencing (experimental feature) is removed.

- The `Math.trunc` polyfill for targeting Internet Explorer was
  previously included in most JavaScript output files.
  Now, it is only included with `-d:nimJsMathTruncPolyfill`.
  If you are targeting Internet Explorer, you may choose to enable this option
  or define your own `Math.trunc` polyfill using the [`emit` pragma](https://nim-lang.org/docs/manual.html#implementation-specific-pragmas-emit-pragma).
  Nim uses `Math.trunc` for the division and modulo operators for integers.

- `shallowCopy` and `shallow` are removed for ARC/ORC. Use `move` when possible or combine assignment and
`sink` for optimization purposes.

- The experimental `nimPreviewDotLikeOps` switch is going to be removed or deprecated because it didn't fullfill its promises.

- The `{.this.}` pragma, deprecated since 0.19, has been removed.
- `nil` literals can no longer be directly assigned to variables or fields of `distinct` pointer types. They must be converted instead.
  ```nim
  type Foo = distinct ptr int

  # Before:
  var x: Foo = nil
  # After:
  var x: Foo = Foo(nil)
  ```
- Removed two type pragma syntaxes deprecated since 0.20, namely
  `type Foo = object {.final.}`, and `type Foo {.final.} [T] = object`.

- `foo a = b` now means `foo(a = b)` rather than `foo(a) = b`. This is consistent
  with the existing behavior of `foo a, b = c` meaning `foo(a, b = c)`.
  This decision was made with the assumption that the old syntax was used rarely;
  if your code used the old syntax, please be aware of this change.

- [Overloadable enums](https://nim-lang.github.io/Nim/manual.html#overloadable-enum-value-names) and Unicode Operators
  are no longer experimental.

- Removed the `nimIncrSeqV3` define.

- `macros.getImpl` for `const` symbols now returns the full definition node
  (as `nnkConstDef`) rather than the AST of the constant value.

- Lock levels are deprecated, now a noop.

- ORC is now the default memory management strategy. Use
  `--mm:refc` for a transition period.

- `strictEffects` are no longer experimental.
  Use `legacy:laxEffects` to keep backward compatibility.

- The `gorge`/`staticExec` calls will now return a descriptive message in the output
  if the execution fails for whatever reason. To get back legacy behaviour use `-d:nimLegacyGorgeErrors`.

- Pointer to `cstring` conversion now triggers a `[PtrToCstringConv]` warning.
  This warning will become an error in future versions! Use a `cast` operation
  like `cast[cstring](x)` instead.

- `logging` will default to flushing all log level messages. To get the legacy behaviour of only flushing Error and Fatal messages, use `-d:nimV1LogFlushBehavior`.

- Redefining templates with the same signature was previously
  allowed to support certain macro code. To do this explicitly, the
  `{.redefine.}` pragma has been added. Note that this is only for templates.
  Implicit redefinition of templates is now deprecated and will give an error in the future.

- Using an unnamed break in a block is deprecated. This warning will become an error in future versions! Use a named block with a named break instead.

- Several Standard libraries are moved to nimble packages, use `nimble` to install them:
  - `std/punycode` => `punycode`
  - `std/asyncftpclient` => `asyncftpclient`
  - `std/smtp` => `smtp`
  - `std/db_common` => `db_connector/db_common`
  - `std/db_sqlite` => `db_connector/db_sqlite`
  - `std/db_mysql` => `db_connector/db_mysql`
  - `std/db_postgres` => `db_connector/db_postgres`
  - `std/db_odbc` => `db_connector/db_odbc`

- Previously, calls like `foo(a, b): ...` or `foo(a, b) do: ...` where the final argument of
  `foo` had type `proc ()` were assumed by the compiler to mean `foo(a, b, proc () = ...)`.
  This behavior is now deprecated. Use `foo(a, b) do (): ...` or `foo(a, b, proc () = ...)` instead.

- If no exception or any exception deriving from Exception but not Defect or CatchableError given in except, a `warnBareExcept` warning will be triggered.

- The experimental strictFuncs feature now disallows a store to the heap via a `ref` or `ptr` indirection.

- - Added the `--legacy:verboseTypeMismatch` switch to get legacy type mismatch error messages.

## Standard library additions and changes

[//]: # "Changes:"
- OpenSSL 3 is now supported.
- `macros.parseExpr` and `macros.parseStmt` now accept an optional
  filename argument for more informative errors.
- Module `colors` expanded with missing colors from the CSS color standard.
  `colPaleVioletRed` and `colMediumPurple` have also been changed to match the CSS color standard.
- Fixed `lists.SinglyLinkedList` being broken after removing the last node ([#19353](https://github.com/nim-lang/Nim/pull/19353)).
- The `md5` module now works at compile time and in JavaScript.
- Changed `mimedb` to use an `OrderedTable` instead of `OrderedTableRef` to support `const` tables.
- `strutils.find` now uses and defaults to `last = -1` for whole string searches,
  making limiting it to just the first char (`last = 0`) valid.
- `random.rand` now works with `Ordinal`s.
- Undeprecated `os.isvalidfilename`.
- `std/oids` now uses `int64` to store time internally (before it was int32).
- `std/uri.Uri` dollar `$` improved, precalculates the `string` result length from the `Uri`.
- `std/uri.Uri.isIpv6` is now exported.
- `std/logging.ConsoleLogger` and `FileLogger` now have a `flushThreshold` attribute to set what log message levels are automatically flushed. For Nim v1 use `-d:nimFlushAllLogs` to automatically flush all message levels. Flushing all logs is the default behavior for Nim v2.


- `std/net.IpAddress` dollar `$` improved, uses a fixed capacity for the `string` result based from the `IpAddressFamily`.
- `std/jsfetch.newFetchOptions` now has default values for all parameters
- `std/jsformdata` now accepts `Blob` data type.

- `std/sharedlist` and `std/sharedtables` are now deprecated, see RFC [#433](https://github.com/nim-lang/RFCs/issues/433).

- New compile flag (`-d:nimNoGetRandom`) when building `std/sysrand` to remove dependency on linux `getrandom` syscall.

  This compile flag only affects linux builds and is necessary if either compiling on a linux kernel version < 3.17, or if code built will be executing on kernel < 3.17.

  On linux kernels < 3.17 (such as kernel 3.10 in RHEL7 and CentOS7), the `getrandom` syscall was not yet introduced. Without this, the `std/sysrand` module will not build properly, and if code is built on a kernel >= 3.17 without the flag, any usage of the `std/sysrand` module will fail to execute on a kernel < 3.17 (since it attempts to perform a syscall to `getrandom`, which isn't present in the current kernel). A compile flag has been added to force the `std/sysrand` module to use /dev/urandom (available since linux kernel 1.3.30), rather than the `getrandom` syscall. This allows for use of a cryptographically secure PRNG, regardless of kernel support for the `getrandom` syscall.

  When building for RHEL7/CentOS7 for example, the entire build process for nim from a source package would then be:
  ```sh
  $ yum install devtoolset-8 # Install GCC version 8 vs the standard 4.8.5 on RHEL7/CentOS7. Alternatively use -d:nimEmulateOverflowChecks. See issue #13692 for details
  $ scl enable devtoolset-8 bash # Run bash shell with default toolchain of gcc 8
  $ sh build.sh  # per unix install instructions
  $ bin/nim c koch  # per unix install instructions
  $ ./koch boot -d:release  # per unix install instructions
  $ ./koch tools -d:nimNoGetRandom  # pass the nimNoGetRandom flag to compile std/sysrand without support for getrandom syscall
  ```

  This is necessary to pass when building nim on kernel versions < 3.17 in particular to avoid an error of "SYS_getrandom undeclared" during the build process for stdlib (sysrand in particular).

[//]: # "Additions:"
- Added ISO 8601 week date utilities in `times`:
  - Added `IsoWeekRange`, a range type for weeks in a week-based year.
  - Added `IsoYear`, a distinct type for a week-based year in contrast to a regular year.
  - Added a `initDateTime` overload to create a datetime from an ISO week date.
  - Added `getIsoWeekAndYear` to get an ISO week number and week-based year from a datetime.
  - Added `getIsoWeeksInYear` to return the number of weeks in a week-based year.
- Added new modules which were part of `std/os`:
  - Added `std/oserrors` for OS error reporting. Added `std/envvars` for environment variables handling.
  - Added `std/paths`, `std/dirs`, `std/files`, `std/symlinks` and `std/appdirs`.
  - Added `std/cmdline` for reading command line parameters.
- Added `sep` parameter in `std/uri` to specify the query separator.
- Added bindings to [`Array.shift`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/shift)
  and [`queueMicrotask`](https://developer.mozilla.org/en-US/docs/Web/API/queueMicrotask)
  in `jscore` for JavaScript targets.
- Added `UppercaseLetters`, `LowercaseLetters`, `PunctuationChars`, `PrintableChars` sets to `std/strutils`.
- Added `complex.sgn` for obtaining the phase of complex numbers.
- Added `insertAdjacentText`, `insertAdjacentElement`, `insertAdjacentHTML`,
  `after`, `before`, `closest`, `append`, `hasAttributeNS`, `removeAttributeNS`,
  `hasPointerCapture`, `releasePointerCapture`, `requestPointerLock`,
  `replaceChildren`, `replaceWith`, `scrollIntoViewIfNeeded`, `setHTML`,
  `toggleAttribute`, and `matches` to `std/dom`.
- Added [`jsre.hasIndices`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/hasIndices)
- Added `capacity` for `string` and `seq` to return the current capacity, see https://github.com/nim-lang/RFCs/issues/460
- Added `openArray[char]` overloads for `std/parseutils` allowing more code reuse.
- Added `openArray[char]` overloads for `std/unicode` allowing more code reuse.
- Added `safe` parameter to `base64.encodeMime`.

[//]: # "Deprecations:"
- Deprecated `selfExe` for Nimscript.
- Deprecated `std/sums`.
- Deprecated `std/base64.encode` for collections of arbitrary integer element type.
  Now only `byte` and `char` are supported.

[//]: # "Removals:"
- Removed deprecated module `parseopt2`.
- Removed deprecated module `sharedstrings`.
- Removed deprecated module `dom_extensions`.
- Removed deprecated module `LockFreeHash`.
- Removed deprecated module `events`.
- Removed deprecated `oids.oidToString`.
- Removed define `nimExperimentalAsyncjsThen` for `std/asyncjs.then` and `std/jsfetch`.
- Removed deprecated `jsre.test` and `jsre.toString`.
- Removed deprecated `math.c_frexp`.
- Removed deprecated `` httpcore.`==` ``.
- Removed deprecated `std/posix.CMSG_SPACE` and `std/posix.CMSG_LEN` that takes wrong argument types.
- Removed deprecated `osproc.poDemon`, symbol with typo.
- Removed deprecated `tables.rightSize`.


- Removed deprecated `posix.CLONE_STOPPED`.


## Language changes

- [Tag tracking](https://nim-lang.github.io/Nim/manual.html#effect-system-tag-tracking) supports the definition of forbidden tags by the `.forbids` pragma
  which can be used to disable certain effects in proc types.
- [Case statement macros](https://nim-lang.github.io/Nim/manual.html#macros-case-statement-macros) are no longer experimental,
  meaning you no longer need to enable the experimental switch `caseStmtMacros` to use them.
- Full command syntax and block arguments i.e. `foo a, b: c` are now allowed
  for the right-hand side of type definitions in type sections. Previously
  they would error with "invalid indentation".

- Compile-time define changes:
  - `defined` now accepts identifiers separated by dots, i.e. `defined(a.b.c)`.
    In the command line, this is defined as `-d:a.b.c`. Older versions can
    use accents as in ``defined(`a.b.c`)`` to access such defines.
  - [Define pragmas for constants](https://nim-lang.github.io/Nim/manual.html#implementation-specific-pragmas-compileminustime-define-pragmas)
    now support a string argument for qualified define names.

    ```nim
    # -d:package.FooBar=42
    const FooBar {.intdefine: "package.FooBar".}: int = 5
    echo FooBar # 42
    ```

    This was added to help disambiguate similar define names for different packages.
    In older versions, this could only be achieved with something like the following:

    ```nim
    const FooBar = block:
      const `package.FooBar` {.intdefine.}: int = 5
      `package.FooBar`
    ```
  - A generic `define` pragma for constants has been added that interprets
    the value of the define based on the type of the constant value.
    See the [experimental manual](https://nim-lang.github.io/Nim/manual_experimental.html#generic-define-pragma)
    for a list of supported types.

- [Macro pragmas](https://nim-lang.github.io/Nim/manual.html#userminusdefined-pragmas-macro-pragmas) changes:
  - Templates now accept macro pragmas.
  - Macro pragmas for var/let/const sections have been redesigned in a way that works
    similarly to routine macro pragmas. The new behavior is documented in the
    [experimental manual](https://nim-lang.github.io/Nim/manual_experimental.html#extended-macro-pragmas).
  - Pragma macros on type definitions can now return `nnkTypeSection` nodes as well as `nnkTypeDef`,
    allowing multiple type definitions to be injected in place of the original type definition.

    ```nim
    import macros
    macro multiply(amount: static int, s: untyped): untyped =
      let name = $s[0].basename
      result = newNimNode(nnkTypeSection)
      for i in 1 .. amount:
        result.add(newTree(nnkTypeDef, ident(name & $i), s[1], s[2]))
    type
      Foo = object
      Bar {.multiply: 3.} = object
        x, y, z: int
      Baz = object
    # becomes
    type
      Foo = object
      Bar1 = object
        x, y, z: int
      Bar2 = object
        x, y, z: int
      Bar3 = object
        x, y, z: int
      Baz = object
    ```

- A new form of type inference called [top-down inference](https://nim-lang.github.io/Nim/manual_experimental.html#topminusdown-type-inference)
  has been implemented for a variety of basic cases. For example, code like the following now compiles:

  ```nim
  let foo: seq[(float, byte, cstring)] = @[(1, 2, "abc")]
  ```

- `cstring` is now accepted as a selector in `case` statements, removing the
  need to convert to `string`. On the JS backend, this is translated directly
  to a `switch` statement.

- Nim now supports `out` parameters and ["strict definitions"](https://nim-lang.github.io/Nim/manual_experimental.html#strict-definitions-and-nimout-parameters).
- Nim now offers a [strict mode](https://nim-lang.github.io/Nim/manual_experimental.html#strict-case-objects) for `case objects`.


## Compiler changes

- The `gc` switch has been renamed to `mm` ("memory management") in order to reflect the
  reality better. (Nim moved away from all techniques based on "tracing".)

- Defines the `gcRefc` symbol which allows writing specific code for the refc GC.

- `nim` can now compile version 1.4.0 as follows: `nim c --lib:lib --stylecheck:off compiler/nim`,
  without requiring `-d:nimVersion140` which is now a noop.

- `--styleCheck`, `--hintAsError` and `--warningAsError` now only apply to the current package.

- The switch `--nimMainPrefix:prefix` has been added to add a prefix to the names of `NimMain` and
  related functions produced on the backend. This prevents conflicts with other Nim
  static libraries.

- When compiling for Release the flag `-fno-math-errno` is used for GCC.


## Tool changes

- Nim now ships Nimble version 0.14 which added support for lock-files. Libraries are stored in `$nimbleDir/pkgs2` (it was `$nimbleDir/pkgs`).
