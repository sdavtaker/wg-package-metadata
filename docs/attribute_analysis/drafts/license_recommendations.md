# License Metadata Recommendations

This document provides implementation guidance for package manager developers on how to represent license metadata in a consistent, machine-readable way. It is intended for implementers adding or improving license support in a package registry or build tool. For background on how current ecosystems handle license metadata, see [license_analysis.md](license_analysis.md).

Throughout this document, "the registry" refers to whatever system accepts and distributes packages — whether a centralized registry server or the build and packaging tooling in ecosystems without one.

## 1. General Recommendation

Expose a dedicated, singular `license` field (not `licenses`, not `licenseInfo`) whose value is a single **SPDX expression string**.

SPDX expressions handle all compound-license cases through built-in operators:

| Scenario | Expression |
|---|---|
| Single license | `MIT` |
| Either of two licenses (disjunctive) | `Apache-2.0 OR MIT` |
| Both licenses apply (conjunctive) | `GPL-2.0-only AND MIT` |
| License with exception | `GPL-2.0-only WITH Classpath-exception-2.0` |

Do **not** accept an array of strings for the field value. Arrays require consumers to guess whether the semantics are AND or OR, and they prevent the use of parenthesized sub-expressions or `WITH` clauses.

**References:**
- [SPDX License List](https://spdx.org/licenses/) — canonical identifiers for all common open-source licenses.
- [SPDX Expression Grammar](https://spdx.github.io/spdx-spec/v2.3/SPDX-license-expressions/) — formal grammar for compound expressions.

## 2. Escape Hatch: Custom Licenses

When a package uses a license that is not on the SPDX license list — for example, a bespoke proprietary license or a heavily modified OSI license — the package manager must support a way for package publishers to express that license. This requires two things from the package manager:

1. A syntactic mechanism to reference the custom license in the SPDX expression (the `LicenseRef-` prefix, defined by the SPDX specification).
2. A way to resolve what that reference points to — i.e., the actual license text — so that consumers and tools can read it.

Three approaches are valid. Choose the one that better fits your package manager's existing data model.

### Option A — Path as identifier

In this approach, the package manager interprets the `LicenseRef-` suffix as a relative file path from the package root. For example, a publisher would write:

- `LicenseRef-LICENSE` to reference the file at `/LICENSE`
- `LicenseRef-licenses/custom.txt` to reference the file at `/licenses/custom.txt`

A compound expression using this approach:

```
MIT AND LicenseRef-licenses/custom.txt
```

**Pros:** Machine-resolvable without any additional metadata; simpler to implement.

**Cons:** Expressions become longer and less readable as paths grow, and there is no short alias for the license.

### Option B — Separate mapping field

In this approach, the package manager exposes a separate field (e.g., `license_files`) that maps each `LicenseRef-` name to its relative path. Publishers use short symbolic names in the expression.

Expression:
```
MIT AND LicenseRef-MyLicense
```

Mapping field:
```json
{
  "license_files": {
    "LicenseRef-MyLicense": "licenses/custom.txt"
  }
}
```

**Pros:** Produces shorter, more readable expressions; allows a human-friendly alias.

**Cons:** Requires additional validation to ensure every `LicenseRef-` name used in the expression has a corresponding entry in the mapping.

### Option C — Convention-based directory (REUSE approach)

In this approach, the package manager defines a conventional directory (e.g., `LICENSES/`) where custom license files are stored. Publishers use short symbolic names in the expression and place the corresponding license text at `LICENSES/<LicenseRef-name>.txt`.

Expression:
```
MIT AND LicenseRef-MyLicense
```

File location:
```
LICENSES/LicenseRef-MyLicense.txt
```

No separate mapping field is needed. The resolution rule is: look for a file named `<LicenseRef-name>.txt` (or `.md`) inside the package's `LICENSES/` directory. This approach is defined by the [REUSE specification](https://reuse.software/spec/).

**Pros:** Short, readable expressions with no explicit mapping; resolution is predictable and convention-based.

**Cons:** Requires the package manager to agree on and document the conventional directory name; less flexible if the ecosystem already uses a different layout convention.

> **Note:** Package managers adopting Option A may also choose to standardise a base directory (e.g., always resolve paths relative to `LICENSES/`), effectively converging toward this convention.

> **Note:** In all three options, the referenced file must be present in the published package. Implementations should validate file presence regardless of which option is used.

### Scope of custom LicenseRefs

The package manager should document — and where feasible, enforce — that `LicenseRef-` is intended only for:
- Licenses not present in the SPDX license list.
- Proprietary or bespoke licenses.
- Materially modified versions of an OSI license that cannot be expressed with a `WITH` clause.

Discourage publishers from using `LicenseRef-` as a workaround for licenses that are on the SPDX list but whose identifier they are uncertain of.

## 3. Validation Guidance

Require that the `license` field value is parseable by a conformant SPDX expression library before the package is accepted.

Reference implementations:
- Python: [`license-expression`](https://pypi.org/project/license-expression/)
- JavaScript: [`spdx-expression-parse`](https://www.npmjs.com/package/spdx-expression-parse)
- PHP: [`composer/spdx-licenses`](https://packagist.org/packages/composer/spdx-licenses)
- Rust: [`spdx`](https://crates.io/crates/spdx)

Warn or reject when the value:
- Is a bare string that is not a valid SPDX identifier or expression (e.g., `"gpl"`, `"BSD"`, `"GPL v2"`).
- Contains multiple identifiers separated by commas, slashes, or spaces without SPDX operators.
- References a `LicenseRef-` name (in **Option B**) that has no corresponding entry in the mapping field. (In **Option C**, names are resolved by convention from the `LICENSES/` directory and do not require an explicit mapping.)
- Uses a deprecated SPDX identifier (e.g., `GPL-2.0` instead of `GPL-2.0-only`, `LGPL-2.1` instead of `LGPL-2.1-only`). Treat as a warning and surface the current equivalent to the publisher.
- Uses a `WITH` clause referencing an identifier that is not on the [SPDX exception list](https://spdx.org/licenses/exceptions-index.html). The expression parses successfully but is semantically invalid.
- Is exactly `NOASSERTION` or `NONE`. These are valid escape hatches in full SPDX documents but are not valid SPDX expression syntax. If your package manager chooses to accept them as special field values (meaning "license could not be determined" and "no license applies" respectively), validate and display them separately from SPDX expressions, and make their semantics visible to consumers in the registry UI and API.

Decide upfront whether validation failures are warnings or hard rejections, and communicate that policy clearly to package publishers. A warning-only policy is acceptable during a transition period; the long-term goal is hard rejection of invalid values.

For **distributed registries** where no central server gatekeeps publication, validation must be embedded in the build and packaging tooling rather than enforced at upload time. In these cases, roll out enforcement in two phases: first target the **publisher workflow** (e.g., the build tool, the package manifest linter, the publish command) so that package authors encounter validation before they distribute anything; only after publisher-side adoption is widespread should enforcement be added to the **consumer-side tooling** (e.g., the dependency resolver, the install command), where a hard failure would affect end users who have no control over the metadata of their dependencies.

## 4. Transition Guidance

For ecosystems that already have a license field in a non-SPDX or partially-SPDX format, a phased migration is recommended. The five stages below define a complete migration path. **All five stages are required.** Stopping at Stage 2 or 3 leaves a long tail of legacy-format packages in the registry and permanently weakens the value of the metadata.

Each stage requires a publicly committed date, set by the ecosystem maintainers. The specific dates are the ecosystem's responsibility to determine based on their community size, tooling maturity, and communication capacity. They are not set by these recommendations. Expect the full migration — from Stage 1 through Stage 5 — to span multiple years; rushing the timeline risks breaking publisher workflows and eroding trust in the registry.

---

### Stage 1 — Parallel support

The registry should accept both SPDX expressions and legacy formats in the `license` field. Do not reject packages for using the old format.

Actions:
- Publish documentation describing the SPDX format and the migration timeline.
- Integrate an SPDX expression parser and surface validation results to publishers (as warnings, not errors).
- Add SPDX-aware tooling to the registry UI and API so consumers can distinguish validated from unvalidated entries.

---

### Stage 2 — New packages require SPDX (from date X)

From the announced date, all **newly published** packages must supply a valid SPDX expression in the `license` field. Reject new package submissions that fail SPDX validation.

Existing packages remain on their current metadata until updated.

---

### Stage 3 — Updated packages require SPDX (from date Y)

From the announced date, the registry should reject any **update** to an existing package that does not include a valid SPDX `license` expression. This means publishers releasing a new version of a package that was previously using a legacy format will need to convert the field as part of the update.

---

### Stage 4 — Dormant packages (from date Z)

For packages that remain in use but have not been updated since Stage 2 or 3, the registry should:
- Attempt automatic conversion of the legacy `license` value where the mapping is unambiguous (e.g., `"BSD2clause"` → `"BSD-2-Clause"`, `"Apache 2.0"` → `"Apache-2.0"`).
- Flag packages where automatic conversion was not possible, and surface this status prominently in the registry UI and API.
- Notify maintainers of flagged packages.

Do not silently drop or hide unconverted packages; they should remain installable but marked as having unvalidated license metadata.

---

### Stage 5 — Phase-off (from date W)

Stop accepting the legacy format entirely. Remove any compatibility shims or dual-format parsing from the registry and associated tooling.

Communicate clearly that the migration window has closed. Any packages still carrying unconverted metadata should be treated as having unknown license information by downstream consumers and tooling.

**Completing this stage is as important as starting the migration.** An ecosystem that reaches Stage 3 but never removes legacy-format acceptance creates a permanent two-tier system where the SPDX field cannot be trusted unconditionally.

---

## 5. References

- [SPDX License List](https://spdx.org/licenses/)
- [SPDX Expression Grammar (SPDX 2.3)](https://spdx.github.io/spdx-spec/v2.3/SPDX-license-expressions/)
- [REUSE Specification](https://reuse.software/spec/) — convention for managing license information in projects
- [license_analysis.md](license_analysis.md) — current-state analysis across ecosystems
- [license_appendix.md](license_appendix.md) — supporting data and coverage statistics
