# Package manager metadata analysis

## Summary

This document provides a comprehensive comparison table of metadata handling across different package managers. The analysis covers how various package managers store, validate, and expose package metadata, highlighting both commonalities and differences in their approaches.

The table below summarizes key metadata attributes and their support across major package managers, serving as a quick reference for understanding compatibility and reliability considerations. More detailed analyses are available in separate documents:

- **Per-attribute analysis**: Detailed examination of how each metadata field is handled across package managers
- **Per-package-manager analysis**: Comprehensive coverage of individual package manager metadata schemas and practices

This comparative analysis is intended to support the Package Metadata Working Group's efforts in understanding current practices and identifying opportunities for standardization and interoperability.

## License Field Comparison

| Ecosystem | Package Manager | Licenses Field Present | License Field Name/Path | Spec-Ref |
|---|---|---|---|---|
| C++ | Conan | Yes | `package['license']` | [Conan Package Reference](https://docs.conan.io/2/reference/conanfile/attributes.html) |
| C++ | Vcpkg | Yes | `package['License']` | [vcpkg.json Reference](https://learn.microsoft.com/en-us/vcpkg/reference/vcpkg-json) |
| Clojure | Clojars:Leinengen | Yes | `licenses(version_xml).join(",")` | [project.clj Format](https://github.com/technomancy/leiningen/blob/master/doc/TUTORIAL.md) |
| Container | Docker | No | - | [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/) |
| Dart | Pub | No | - | [pubspec.yaml](https://dart.dev/tools/pub/pubspec) |
| Elm | Elm | Yes | `package['license']` | [elm.json](https://github.com/elm/compiler/blob/master/docs/elm.json/package.md) |
| Emacs | Elpa | No | - | [Package.el Format](https://cgit.git.savannah.gnu.org/cgit/emacs/elpa.git/plain/README) |
| Elixir | Mix | Yes | `repo_fallback(package["meta"].fetch("licenses", []).join(",")` | [mix.exs](https://hex.pm/docs/publish) |
| FreeBSD | Ports | Yes | `pkg_metadata["LICENSE"]` | [Porter's Handbook](https://docs.freebsd.org/en/books/porters-handbook/) |
| Go | Go | Yes | `package[:html].css('*[data-test-id="UnitHeader-license"]').map(&:text).join(",")` | [go.mod](https://go.dev/ref/mod#go-mod-file) |
| Haskell | Cabal | Yes | `find_attribute(package[:page], "License")` | [Cabal Specification](https://cabal.readthedocs.io/en/3.4/cabal-package.html) |
| HPC | Spack | Yes | `[]` | [Spack Package Schema](https://spack.readthedocs.io/en/latest/packaging_guide.html) |
| Java | Maven | Yes | `licenses(version_xml).join(",")` | [POM Reference](https://maven.apache.org/pom.html) |
| JavaScript | Npm | Yes | `licenses(latest_version)` | [package.json](https://docs.npmjs.com/cli/v8/configuring-npm/package-json) |
| JavaScript | Yarn | Yes | `licenses(latest_version)` | [Yarn Package.json](https://yarnpkg.com/configuration/manifest) |
| Julia | Julia | Yes | `json['license']` | [Project.toml](https://pkgdocs.julialang.org/v1/creating-packages/) |
| Linux | apk | Yes | `pkg_metadata["L"]` | [APKBUILD Reference](https://wiki.alpinelinux.org/wiki/APKBUILD_Reference) |
| Linux | dpkg | Yes | `pkg_metadata["License"]` | [Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-controlfields.html) |
| Linux | rpm | Yes | `pkg_metadata["License"]` | [RPM Package Manager](https://rpm.org/docs/4.20.x/manual/spec.html) |
| macOS | Homebrew | Yes | `package['license']` | [Formula Cookbook](https://docs.brew.sh/Formula-Cookbook) |
| .NET | Nuget | Yes | `item["licenseExpression"]` | [nuspec Reference](https://learn.microsoft.com/en-us/nuget/reference/nuspec) |
| Perl | Cpan | Yes | `package.fetch("license", []).join(",")` | [CPAN::Meta::Spec](https://metacpan.org/pod/CPAN::Meta::Spec) |
| PHP | Composer | Yes | `package['license']` | [composer.json](https://getcomposer.org/doc/04-schema.md) |
| pkg-config | pkg-config | No | - | [pkg-config Files](https://people.freedesktop.org/~dbn/pkg-config-guide.html) |
| Puppet | Puppet | Yes | `metadata["license"]` |  |
| Python | Conda | Yes | `pkg_metadata["about"]["license"]` | [meta.yaml](https://docs.conda.io/projects/conda-build/en/latest/resources/define-metadata.html) |
| Python | Pypi | Yes | `licenses(package)` | [Dependency Specifiers](https://packaging.python.org/en/latest/specifications/dependency-specifiers/#dependency-specifiers) |
| R | Bioconductor | Yes | `package[:properties]["License"]` | [Packager documentation](https://bioconductor.org/developers/) |
| R | Cran | Yes | `package[:properties]["License:"]` | [DESCRIPTION File](https://cran.r-project.org/doc/manuals/R-exts.html#The-DESCRIPTION-file) |
| Racket | Racket | No | - | [info.rkt](https://docs.racket-lang.org/pkg/Package_Concepts.html) |
| Ruby | Rubygems | Yes | `pkg_metadata.fetch("licenses", []).try(:join, ",")` | [gemspec Reference](https://guides.rubygems.org/specification-reference/) |
| Rust | Cargo | Yes | `latest_version["license"]` | [Cargo.toml](https://doc.rust-lang.org/cargo/reference/manifest.html) |
| Swift | Carthage | Yes | `package['license']` | [Cartfile](https://github.com/Carthage/Carthage/blob/master/Documentation/Artifacts.md) |
| Swift | Cocoapods | Yes | `parse_license(package["license"])` | [Podspec](https://guides.cocoapods.org/syntax/podspec.html) |
| Swift | Swiftpm | Yes | `package['license']` | [Package.swift](https://swift.org/package-manager/) |

