# License Metadata Recommendations

This document provides implementation guidance for package manager developers on how to represent license metadata in a consistent, machine-readable way. It is intended for implementers adding or improving license support in a package registry or build tool. For background on how current ecosystems handle license metadata, see [license_analysis.md](license_analysis.md).

## 1. General Recommendation

Use a dedicated, singular `license` field (not `licenses`, not `licenseInfo`) whose value is a single **SPDX expression string**.

SPDX expressions handle all compound-license cases through built-in operators:

| Scenario | Expression |
|---|---|
| Single license | `MIT` |
| Either of two licenses (disjunctive) | `Apache-2.0 OR MIT` |
| Both licenses apply (conjunctive) | `GPL-2.0-only AND MIT` |
| License with exception | `GPL-2.0-only WITH Classpath-exception-2.0` |

Do **not** use an array of strings for the field value. Arrays require consumers to guess whether the semantics are AND or OR, and they prevent the use of parenthesized sub-expressions or `WITH` clauses.

**References:**
- [SPDX License List](https://spdx.org/licenses/) — canonical identifiers for all common open-source licenses.
- [SPDX Expression Grammar](https://spdx.github.io/spdx-spec/v2.3/SPDX-license-expressions/) — formal grammar for compound expressions.

## 2. Escape Hatch: Custom Licenses

When a package uses a license that is not on the SPDX license list — for example, a bespoke proprietary license or a heavily modified OSI license — use the `LicenseRef-` prefix defined by the SPDX specification.

Two approaches are valid. Choose the one that better fits your package manager's existing data model.

### Option A — Path as identifier

Embed the relative file path directly in the `LicenseRef-` name. The path is resolved from the package root.

```
LicenseRef-LICENSE
LicenseRef-licenses/custom.txt
```

A compound expression using this approach:

```
MIT AND LicenseRef-licenses/custom.txt
```

**Tradeoffs:** Machine-resolvable without any additional metadata; simpler to implement. Expressions become longer and less readable as paths grow, and there is no short alias for the license.

### Option B — Separate mapping field

Use a short symbolic name in the expression and expose a separate field (e.g., `license_files`) that maps each `LicenseRef-` name to its relative path.

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

**Tradeoffs:** Produces shorter, more readable expressions. Requires additional validation to ensure every `LicenseRef-` name used in the expression has a corresponding entry in the mapping.

> **Note:** In both options, the referenced file must be present in the published package. Implementations should validate file presence regardless of which option is used.

### When to use custom LicenseRefs

Use `LicenseRef-` only when:
- The license is not present in the SPDX license list.
- The license is proprietary or bespoke.
- The license is a materially modified version of an OSI license that cannot be expressed with a `WITH` clause.

Do not use `LicenseRef-` as a workaround for licenses that are on the SPDX list but whose identifier you are uncertain of.

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
- References a `LicenseRef-` name (in Option B) that has no corresponding entry in the mapping field.

Decide upfront whether validation failures are warnings or hard rejections, and communicate that policy clearly to package publishers. A warning-only policy is acceptable during a transition period; the long-term goal is hard rejection of invalid values.

For **distributed registries** where no central server gatekeeps publication, validation must be embedded in the build and packaging tooling rather than enforced at upload time. In these cases, roll out enforcement in two phases: first target the **publisher workflow** (e.g., the build tool, the package manifest linter, the publish command) so that package authors encounter validation before they distribute anything; only after publisher-side adoption is widespread should enforcement be added to the **consumer-side tooling** (e.g., the dependency resolver, the install command), where a hard failure would affect end users who have no control over the metadata of their dependencies.

## 4. Transition Guidance

For ecosystems that already have a license field in a non-SPDX or partially-SPDX format, a phased migration is recommended. The five stages below define a complete migration path. **All five stages are required.** Stopping at Stage 2 or 3 leaves a long tail of legacy-format packages in the registry and permanently weakens the value of the metadata.

Each stage requires a publicly committed date, set by the ecosystem maintainers. The specific dates are the ecosystem's responsibility to determine based on their community size, tooling maturity, and communication capacity. They are not set by these recommendations. Expect the full migration — from Stage 1 through Stage 5 — to span multiple years; rushing the timeline risks breaking publisher workflows and eroding trust in the registry.

---

### Stage 1 — Parallel support

Accept both SPDX expressions and legacy formats in the `license` field. Do not reject packages for using the old format.

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

From the announced date, any **update** to an existing package must include a valid SPDX `license` expression. Publishers who release a new version of a package that was previously using a legacy format must convert the field as part of the update.

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
- [license_analysis.md](license_analysis.md) — current-state analysis across ecosystems
- [license_appendix.md](license_appendix.md) — supporting data and coverage statistics
