# SBOM Generation for C/++

## Overview

SBOM generation has become essential for detecting vulnerabilities, ensuring compliance, and addressing license issues. Although it may seem straightforward, the quality of results varies based on ecosystems, generators, formats, and specific project files.

By utilizing this repository, a fork from [egendron93/cpp_library_template](https://github.com/egendron93/cpp_library_template), which is an open-source C++ library with conan as package manager, we aim to demonstrate different outputs and guide you through the SBOM generation process and its expected results. This will assist you in reproducing the process in your own projects with Manifest.

<aside>
⚠️ This repository assumes the use of Conan. To this date, other package managers and C/++ projects without package managers are not supported by any OSS generator.

</aside>

## Prerequisites

- Install **Manifest CLI** by following the [installation guide](https://github.com/manifest-cyber/cli).
- **conan** **v1** is installed and configured. (v2 is required for conan extensions):
- (optional) Node, NPM installed (**cdxgen** only)

## Conan installation

Conan is a python-based software, installation should be as easy as running:

```
pip install conan

```

This will install v2 by default, to install a specific version run:

```
pip install "conan==1.64.0"
```

if using `pipx`, you can run:

```
pipx ensurepath && pipx install "conan==1.64.0"
```

For more information, go to the [official installation guide](https://docs.conan.io/2/installation.html).

## SBOM Quality

A quality SBOM possesses key traits that ensure comprehensiveness and accuracy. It must detect all direct dependencies and identify transitive dependencies to capture the full hierarchy of components. Each component should include detailed license data and precise identifiers such as Package URLs (purl), CPEs, and SHA for integrity and traceability.

Additionally, a quality SBOM features a clear dependency tree, illustrating the interconnections among components. Accurate metadata for each component, including the root component, is crucial for providing a reliable and comprehensive resource to understand the software’s composition and manage potential risks.

## Generate

In this example, we would compare the results of **syft**, **trivy,** and **cdxgen.**

<aside>
⚠️ Even though other generators exists, they do not provide support for C/++ and conan projects.

</aside>

<aside>
⚠️ Important! We are currently using conan v1 because not all features have been ported and breaking changes fixed for v2. If you prefer to use v2, feel free to skip this part.

</aside>

### Installing Generators

The **manifest-cli** cannot function without the proper installation of the underlying generators. Fortunately, you can install them using the following command.

```bash
manifest-cli install -g cdxgen
manifest-cli install -g syft
manifest-cli install -g trivy
```

### Project files

Conan projects have different configurations such as `conanfile.txt` , `conaninfo.txt` , `conanfile.py` and `conanfile.lock` .

This project `main` branch uses [`conanfile.py`](http://conanfile.py) which is the more robust and recommended way by conan. see `legacy` for example with `conanfile.txt` and `conaninfo.txt` .

### Generation with Manifest

The example project uses a mix of those, lets attempt to generate an SBOM and compare the results.

**Syft**

```tsx
manifest-cli sbom --name=cpp --version=0.0.0-dev --generator=syft -f test-syft.json .

```

The output should look like this:

```tsx
{
  "$schema": "http://cyclonedx.org/schema/bom-1.5.schema.json",
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "version": 1,
  "metadata": {
    "tools": {
      "components": [
        {
          "type": "application",
          "author": "anchore",
          "name": "syft",
          "version": "1.3.0"
        }
      ]
    },
    "component": {
      "bom-ref": "cpp@0.0.0-dev",
      "type": "application",
      "name": "cpp",
      "version": "0.0.0-dev"
    }
  },
  "components": [
    {
      "bom-ref": ".@:af63bd4c8601b7f1",
      "type": "file",
      "name": "."
    }
  ],
  "dependencies": [
    {
      "ref": "cpp@0.0.0-dev",
      "dependsOn": [
        ".@:af63bd4c8601b7f1"
      ]
    }
  ]
}

```

The empty SBOM is expected as **syft** doesn’t support [`conanfile.py`](http://conanfile.py) , however, it supports `conan.lock` file, to generate it run:

```groovy
conan lock create conanfile.py
```

If for some reason the build fails, try running within a linux container, an additional .devcontainer is configured for this project.

re-run generation:

```tsx
{
  "$schema": "http://cyclonedx.org/schema/bom-1.5.schema.json",
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "version": 1,
  "metadata": {
    "tools": {
      "components": [
        {
          "type": "application",
          "author": "anchore",
          "name": "syft",
          "version": "1.3.0"
        }
      ]
    },
    "component": {
      "bom-ref": "cpp@0.0.0-dev",
      "type": "application",
      "name": "cpp",
      "version": "0.0.0-dev"
    }
  },
  "components": [
    {
      "bom-ref": ".@:af63bd4c8601b7f1",
      "type": "file",
      "name": ".",
      "components": [
        {
          "bom-ref": ".@:pkg:conan/gtest@1.14.0?package-id=774c846603720ad7",
          "type": "library",
          "name": "gtest",
          "version": "1.14.0",
          "cpe": "cpe:2.3:a:gtest:gtest:1.14.0:*:*:*:*:*:*:*",
          "purl": "pkg:conan/gtest@1.14.0",
          "properties": [
            {
              "name": "syft:package:foundBy",
              "value": "conan-cataloger"
            },
            {
              "name": "syft:package:language",
              "value": "c++"
            },
            {
              "name": "syft:package:type",
              "value": "conan"
            },
            {
              "name": "syft:package:metadataType",
              "value": "c-conan-lock-entry"
            },
            {
              "name": "syft:location:0:path",
              "value": "/conan.lock"
            }
          ]
        },
        {
          "bom-ref": ".@:pkg:conan/hello_world@1.0.0?package-id=1c0e4535a9fb07fc",
          "type": "library",
          "name": "hello_world",
          "version": "1.0.0",
          "cpe": "cpe:2.3:a:hello-world:hello-world:1.0.0:*:*:*:*:*:*:*",
          "purl": "pkg:conan/hello_world@1.0.0",
          "properties": [
            {
              "name": "syft:package:foundBy",
              "value": "conan-cataloger"
            },
            {
              "name": "syft:package:language",
              "value": "c++"
            },
            {
              "name": "syft:package:type",
              "value": "conan"
            },
            {
              "name": "syft:package:metadataType",
              "value": "c-conan-lock-entry"
            },
            {
              "name": "syft:cpe23",
              "value": "cpe:2.3:a:hello-world:hello_world:1.0.0:*:*:*:*:*:*:*"
            },
            {
              "name": "syft:cpe23",
              "value": "cpe:2.3:a:hello_world:hello-world:1.0.0:*:*:*:*:*:*:*"
            },
            {
              "name": "syft:cpe23",
              "value": "cpe:2.3:a:hello_world:hello_world:1.0.0:*:*:*:*:*:*:*"
            },
            {
              "name": "syft:cpe23",
              "value": "cpe:2.3:a:hello:hello-world:1.0.0:*:*:*:*:*:*:*"
            },
            {
              "name": "syft:cpe23",
              "value": "cpe:2.3:a:hello:hello_world:1.0.0:*:*:*:*:*:*:*"
            },
            {
              "name": "syft:location:0:path",
              "value": "/conan.lock"
            }
          ]
        },
        {
          "bom-ref": ".@:pkg:conan/mathter@1.1.1?package-id=6a9872a2438c3cba",
          "type": "library",
          "name": "mathter",
          "version": "1.1.1",
          "cpe": "cpe:2.3:a:mathter:mathter:1.1.1:*:*:*:*:*:*:*",
          "purl": "pkg:conan/mathter@1.1.1",
          "properties": [
            {
              "name": "syft:package:foundBy",
              "value": "conan-cataloger"
            },
            {
              "name": "syft:package:language",
              "value": "c++"
            },
            {
              "name": "syft:package:type",
              "value": "conan"
            },
            {
              "name": "syft:package:metadataType",
              "value": "c-conan-lock-entry"
            },
            {
              "name": "syft:location:0:path",
              "value": "/conan.lock"
            }
          ]
        },
        {
          "bom-ref": ".@:pkg:conan/ms-gsl@4.0.0?package-id=8561fa79ef7b4a03",
          "type": "library",
          "name": "ms-gsl",
          "version": "4.0.0",
          "cpe": "cpe:2.3:a:ms-gsl:ms-gsl:4.0.0:*:*:*:*:*:*:*",
          "purl": "pkg:conan/ms-gsl@4.0.0",
          "properties": [
            {
              "name": "syft:package:foundBy",
              "value": "conan-cataloger"
            },
            {
              "name": "syft:package:language",
              "value": "c++"
            },
            {
              "name": "syft:package:type",
              "value": "conan"
            },
            {
              "name": "syft:package:metadataType",
              "value": "c-conan-lock-entry"
            },
            {
              "name": "syft:cpe23",
              "value": "cpe:2.3:a:ms-gsl:ms_gsl:4.0.0:*:*:*:*:*:*:*"
            },
            {
              "name": "syft:cpe23",
              "value": "cpe:2.3:a:ms_gsl:ms-gsl:4.0.0:*:*:*:*:*:*:*"
            },
            {
              "name": "syft:cpe23",
              "value": "cpe:2.3:a:ms_gsl:ms_gsl:4.0.0:*:*:*:*:*:*:*"
            },
            {
              "name": "syft:cpe23",
              "value": "cpe:2.3:a:ms:ms-gsl:4.0.0:*:*:*:*:*:*:*"
            },
            {
              "name": "syft:cpe23",
              "value": "cpe:2.3:a:ms:ms_gsl:4.0.0:*:*:*:*:*:*:*"
            },
            {
              "name": "syft:location:0:path",
              "value": "/conan.lock"
            }
          ]
        },
        {
          "bom-ref": ".@:pkg:conan/xsimd@11.1.0?package-id=82c175e62a3194ad",
          "type": "library",
          "name": "xsimd",
          "version": "11.1.0",
          "cpe": "cpe:2.3:a:xsimd:xsimd:11.1.0:*:*:*:*:*:*:*",
          "purl": "pkg:conan/xsimd@11.1.0",
          "properties": [
            {
              "name": "syft:package:foundBy",
              "value": "conan-cataloger"
            },
            {
              "name": "syft:package:language",
              "value": "c++"
            },
            {
              "name": "syft:package:type",
              "value": "conan"
            },
            {
              "name": "syft:package:metadataType",
              "value": "c-conan-lock-entry"
            },
            {
              "name": "syft:location:0:path",
              "value": "/conan.lock"
            }
          ]
        }
      ]
    }
  ],
  "dependencies": [
    {
      "ref": ".@:pkg:conan/hello_world@1.0.0?package-id=1c0e4535a9fb07fc",
      "dependsOn": [
        ".@:pkg:conan/mathter@1.1.1?package-id=6a9872a2438c3cba",
        ".@:pkg:conan/ms-gsl@4.0.0?package-id=8561fa79ef7b4a03"
      ]
    },
    {
      "ref": ".@:pkg:conan/mathter@1.1.1?package-id=6a9872a2438c3cba",
      "dependsOn": [
        ".@:pkg:conan/xsimd@11.1.0?package-id=82c175e62a3194ad"
      ]
    },
    {
      "ref": "cpp@0.0.0-dev",
      "dependsOn": [
        ".@:af63bd4c8601b7f1"
      ]
    }
  ]
}

```

The results appear to include all build and runtime dependencies with an accurate dependency tree; however, they lack license data. This type of information is not contained in the `conan.lock` file. syft also generates CPEs which are helpful for license and vulnerability matching.

**Cdxgen**

```tsx
manifest-cli sbom --name=cpp --version=0.0.0-dev --generator=cdxgen -vvv -f test-cdx.json .
```

Unlike syft, **cdxgen** supports many project C/++ project files, such as [`conanfile.py`](http://conanfile.py) and `CMakeLists.txt` alongside `conan.lock` .

```tsx
{
  "$schema": "http://cyclonedx.org/schema/bom-1.5.schema.json",
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "version": 1,
  "metadata": {
    "tools": {
      "components": [
        {
          "bom-ref": "pkg:npm/@cyclonedx/cdxgen@10.6.1",
          "type": "application",
          "author": "OWASP Foundation",
          "publisher": "OWASP Foundation",
          "group": "@cyclonedx",
          "name": "cdxgen",
          "version": "10.6.1",
          "purl": "pkg:npm/%40cyclonedx/cdxgen@10.6.1"
        }
      ]
    },
    "component": {
      "bom-ref": "cpp@0.0.0-dev",
      "type": "application",
      "name": "cpp",
      "version": "0.0.0-dev"
    }
  },
  "components": [
    {
      "bom-ref": "cpp_library_template@latest:pkg:gem/cpp_library_template@latest",
      "type": "application",
      "name": "cpp_library_template",
      "version": "latest",
      "purl": "pkg:gem/cpp_library_template@latest",
      "components": [
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/hello_world",
          "type": "application",
          "name": "hello_world",
          "purl": "pkg:generic/hello_world"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:conan/hello_world@1.0.0",
          "type": "library",
          "name": "hello_world",
          "version": "1.0.0",
          "purl": "pkg:conan/hello_world@1.0.0"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:conan/mathter@1.1.1",
          "type": "library",
          "name": "mathter",
          "version": "1.1.1",
          "purl": "pkg:conan/mathter@1.1.1"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:conan/xsimd@11.1.0",
          "type": "library",
          "name": "xsimd",
          "version": "11.1.0",
          "purl": "pkg:conan/xsimd@11.1.0"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:conan/ms-gsl@4.0.0",
          "type": "library",
          "name": "ms-gsl",
          "version": "4.0.0",
          "purl": "pkg:conan/ms-gsl@4.0.0"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:conan/gtest@1.14.0",
          "type": "library",
          "name": "gtest",
          "version": "1.14.0",
          "purl": "pkg:conan/gtest@1.14.0"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/hello_world",
          "type": "library",
          "name": "hello_world",
          "purl": "pkg:generic/hello_world"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/gtest",
          "type": "library",
          "name": "gtest",
          "purl": "pkg:generic/gtest",
          "properties": [
            {
              "name": "name_with_version",
              "value": "googletest-1.14.0"
            },
            {
              "name": "analyzed_version",
              "value": "googletest-1.14.0"
            },
            {
              "name": "analyzed_source_url",
              "value": "https://github.com/google/googletest/archive/refs/tags/v1.14.0.tar.gz"
            },
            {
              "name": "analyzed_source_filename",
              "value": "gtest-1.14.0.tar.gz"
            },
            {
              "name": "analyzed_source_hash",
              "value": "8ad598c73ad796e0d8280b082cebd82a630d73e73cd3c70057938a6501bba5d7"
            },
            {
              "name": "PkgProvides",
              "value": "gmock, gmock_main, gtest, gtest_main"
            },
            {
              "name": "available_versions",
              "value": "1.14.0-1, 1.13.0-1, 1.12.1-1, 1.11.0-2, 1.11.0-1, 1.10.0-1, 1.8.1-1, 1.8.0-5, 1.8.0-4, 1.8.0-3, 1.8.0-2, 1.8.0-1, 1.7.0-5, 1.7.0-4, 1.7.0-2"
            }
          ],
          "evidence": {
            "identity": {
              "field": "purl",
              "confidence": 0.5,
              "methods": [
                {
                  "technique": "source-code-analysis",
                  "confidence": 0.5,
                  "value": "Filename /Users/oriavraham/Development/examples/cpp/cpp_library_template/tests/CMakeLists.txt"
                }
              ]
            }
          }
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/PackageTest",
          "type": "library",
          "name": "PackageTest",
          "purl": "pkg:generic/PackageTest"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/Mathter@1.1.1",
          "type": "library",
          "name": "Mathter",
          "version": "1.1.1",
          "purl": "pkg:generic/Mathter@1.1.1",
          "evidence": {
            "identity": {
              "field": "purl",
              "confidence": 0,
              "methods": [
                {
                  "technique": "source-code-analysis",
                  "confidence": 0.5,
                  "value": "Filename /Users/oriavraham/Development/examples/cpp/cpp_library_template/src/CMakeLists.txt"
                }
              ]
            }
          }
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/Microsoft.GSL@4.0.0",
          "type": "library",
          "name": "Microsoft.GSL",
          "version": "4.0.0",
          "purl": "pkg:generic/Microsoft.GSL@4.0.0",
          "evidence": {
            "identity": {
              "field": "purl",
              "confidence": 0,
              "methods": [
                {
                  "technique": "source-code-analysis",
                  "confidence": 0.5,
                  "value": "Filename /Users/oriavraham/Development/examples/cpp/cpp_library_template/src/CMakeLists.txt"
                }
              ]
            }
          }
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/mathter",
          "type": "library",
          "name": "mathter",
          "purl": "pkg:generic/mathter",
          "evidence": {
            "identity": {
              "field": "purl",
              "confidence": 0,
              "methods": [
                {
                  "technique": "source-code-analysis",
                  "confidence": 0.5,
                  "value": "Filename /Users/oriavraham/Development/examples/cpp/cpp_library_template/build/Release/generators/conandeps_legacy.cmake"
                }
              ]
            }
          }
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/Microsoft.GSL",
          "type": "library",
          "name": "Microsoft.GSL",
          "purl": "pkg:generic/Microsoft.GSL",
          "evidence": {
            "identity": {
              "field": "purl",
              "confidence": 0,
              "methods": [
                {
                  "technique": "source-code-analysis",
                  "confidence": 0.5,
                  "value": "Filename /Users/oriavraham/Development/examples/cpp/cpp_library_template/build/Release/generators/conandeps_legacy.cmake"
                }
              ]
            }
          }
        }
      ]
    }
  ],
  "dependencies": [
    {
      "ref": "cpp@0.0.0-dev",
      "dependsOn": [
        "cpp_library_template@latest:pkg:gem/cpp_library_template@latest"
      ]
    }
  ]
}

```

**cdxgen** was able to pick up all direct dependencies, most likely due to `CMakeLists.txt` parsing, since it doesn’t not contain any information for transitive dependencies or licenses, the SBOM is missing dependency tree. you can also note the `purl` are incorrect, and prefixed with `generic` .

Generating `conan.lock` might yield better results:

```tsx
conan lock create conanfile.py
```

Rerun, output should be:

```tsx
{
  "$schema": "http://cyclonedx.org/schema/bom-1.5.schema.json",
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "version": 1,
  "metadata": {
    "tools": {
      "components": [
        {
          "bom-ref": "pkg:npm/@cyclonedx/cdxgen@10.6.1",
          "type": "application",
          "author": "OWASP Foundation",
          "publisher": "OWASP Foundation",
          "group": "@cyclonedx",
          "name": "cdxgen",
          "version": "10.6.1",
          "purl": "pkg:npm/%40cyclonedx/cdxgen@10.6.1"
        }
      ]
    },
    "component": {
      "bom-ref": "cpp@0.0.0-dev",
      "type": "application",
      "name": "cpp",
      "version": "0.0.0-dev"
    }
  },
  "components": [
    {
      "bom-ref": "cpp_library_template@latest:pkg:gem/cpp_library_template@latest",
      "type": "application",
      "name": "cpp_library_template",
      "version": "latest",
      "purl": "pkg:gem/cpp_library_template@latest",
      "components": [
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/hello_world",
          "type": "application",
          "name": "hello_world",
          "purl": "pkg:generic/hello_world"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:conan/hello_world@1.0.0",
          "type": "library",
          "name": "hello_world",
          "version": "1.0.0",
          "purl": "pkg:conan/hello_world@1.0.0"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:conan/mathter@1.1.1",
          "type": "library",
          "name": "mathter",
          "version": "1.1.1",
          "purl": "pkg:conan/mathter@1.1.1"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:conan/xsimd@11.1.0",
          "type": "library",
          "name": "xsimd",
          "version": "11.1.0",
          "purl": "pkg:conan/xsimd@11.1.0"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:conan/ms-gsl@4.0.0",
          "type": "library",
          "name": "ms-gsl",
          "version": "4.0.0",
          "purl": "pkg:conan/ms-gsl@4.0.0"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:conan/gtest@1.14.0",
          "type": "library",
          "name": "gtest",
          "version": "1.14.0",
          "purl": "pkg:conan/gtest@1.14.0"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/hello_world",
          "type": "library",
          "name": "hello_world",
          "purl": "pkg:generic/hello_world"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/gtest",
          "type": "library",
          "name": "gtest",
          "purl": "pkg:generic/gtest",
          "properties": [
            {
              "name": "name_with_version",
              "value": "googletest-1.14.0"
            },
            {
              "name": "analyzed_version",
              "value": "googletest-1.14.0"
            },
            {
              "name": "analyzed_source_url",
              "value": "https://github.com/google/googletest/archive/refs/tags/v1.14.0.tar.gz"
            },
            {
              "name": "analyzed_source_filename",
              "value": "gtest-1.14.0.tar.gz"
            },
            {
              "name": "analyzed_source_hash",
              "value": "8ad598c73ad796e0d8280b082cebd82a630d73e73cd3c70057938a6501bba5d7"
            },
            {
              "name": "PkgProvides",
              "value": "gmock, gmock_main, gtest, gtest_main"
            },
            {
              "name": "available_versions",
              "value": "1.14.0-1, 1.13.0-1, 1.12.1-1, 1.11.0-2, 1.11.0-1, 1.10.0-1, 1.8.1-1, 1.8.0-5, 1.8.0-4, 1.8.0-3, 1.8.0-2, 1.8.0-1, 1.7.0-5, 1.7.0-4, 1.7.0-2"
            }
          ],
          "evidence": {
            "identity": {
              "field": "purl",
              "confidence": 0.5,
              "methods": [
                {
                  "technique": "source-code-analysis",
                  "confidence": 0.5,
                  "value": "Filename /Users/oriavraham/Development/examples/cpp/cpp_library_template/tests/CMakeLists.txt"
                }
              ]
            }
          }
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/PackageTest",
          "type": "library",
          "name": "PackageTest",
          "purl": "pkg:generic/PackageTest"
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/Mathter@1.1.1",
          "type": "library",
          "name": "Mathter",
          "version": "1.1.1",
          "purl": "pkg:generic/Mathter@1.1.1",
          "evidence": {
            "identity": {
              "field": "purl",
              "confidence": 0,
              "methods": [
                {
                  "technique": "source-code-analysis",
                  "confidence": 0.5,
                  "value": "Filename /Users/oriavraham/Development/examples/cpp/cpp_library_template/src/CMakeLists.txt"
                }
              ]
            }
          }
        },
        {
          "bom-ref": "cpp_library_template@latest:pkg:generic/Microsoft.GSL@4.0.0",
          "type": "library",
          "name": "Microsoft.GSL",
          "version": "4.0.0",
          "purl": "pkg:generic/Microsoft.GSL@4.0.0",
          "evidence": {
            "identity": {
              "field": "purl",
              "confidence": 0,
              "methods": [
                {
                  "technique": "source-code-analysis",
                  "confidence": 0.5,
                  "value": "Filename /Users/oriavraham/Development/examples/cpp/cpp_library_template/src/CMakeLists.txt"
                }
              ]
            }
          }
        }
      ]
    }
  ],
  "dependencies": [
    {
      "ref": "cpp@0.0.0-dev",
      "dependsOn": [
        "cpp_library_template@latest:pkg:gem/cpp_library_template@latest"
      ]
    }
  ]
}

```

For unknown reasons, `cdxgen` combined the results from `CMakeLists.txt` and `conan.lock`. Even though the dependency graph exists in the lockfile, it failed to parse it and construct a dependency tree in the SBOM.

**Trivy**

```tsx
manifest-cli sbom --name=cpp --version=0.0.0-dev --generator=trivy -vvv -f test-trivy.json .
```

**Trivy** takes a different approach, it explicitly uses `conan.lock` files to find all dependencies, then it uses the local conan cache folder and parse [`conanfile.py`](http://conanfile.py) for licenses data.

```tsx
{
  "$schema": "http://cyclonedx.org/schema/bom-1.5.schema.json",
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "version": 1,
  "metadata": {
    "component": {
      "bom-ref": "cpp@0.0.0-dev",
      "type": "application",
      "name": "cpp",
      "version": "0.0.0-dev"
    }
  },
  "components": [
    {
      "bom-ref": ".@:772e09eb-bff9-46f2-af6a-930e8ced753e",
      "type": "application",
      "name": ".",
      "properties": [
        {
          "name": "aquasecurity:trivy:SchemaVersion",
          "value": "2"
        }
      ],
      "components": [
        {
          "bom-ref": ".@:cfe81db4-3e82-4573-adf3-068719b5c7c0",
          "type": "application",
          "name": "conan.lock",
          "properties": [
            {
              "name": "aquasecurity:trivy:Class",
              "value": "lang-pkgs"
            },
            {
              "name": "aquasecurity:trivy:Type",
              "value": "conan"
            }
          ]
        },
        {
          "bom-ref": ".@:pkg:conan/gtest@1.14.0",
          "type": "library",
          "name": "gtest",
          "version": "1.14.0",
          "purl": "pkg:conan/gtest@1.14.0",
          "properties": [
            {
              "name": "aquasecurity:trivy:PkgID",
              "value": "gtest/1.14.0"
            },
            {
              "name": "aquasecurity:trivy:PkgType",
              "value": "conan"
            }
          ]
        },
        {
          "bom-ref": ".@:pkg:conan/hello_world@1.0.0",
          "type": "library",
          "name": "hello_world",
          "version": "1.0.0",
          "purl": "pkg:conan/hello_world@1.0.0",
          "properties": [
            {
              "name": "aquasecurity:trivy:PkgID",
              "value": "hello_world/1.0.0"
            },
            {
              "name": "aquasecurity:trivy:PkgType",
              "value": "conan"
            }
          ]
        },
        {
          "bom-ref": ".@:pkg:conan/mathter@1.1.1",
          "type": "library",
          "name": "mathter",
          "version": "1.1.1",
          "purl": "pkg:conan/mathter@1.1.1",
          "properties": [
            {
              "name": "aquasecurity:trivy:PkgID",
              "value": "mathter/1.1.1"
            },
            {
              "name": "aquasecurity:trivy:PkgType",
              "value": "conan"
            }
          ]
        },
        {
          "bom-ref": ".@:pkg:conan/ms-gsl@4.0.0",
          "type": "library",
          "name": "ms-gsl",
          "version": "4.0.0",
          "purl": "pkg:conan/ms-gsl@4.0.0",
          "properties": [
            {
              "name": "aquasecurity:trivy:PkgID",
              "value": "ms-gsl/4.0.0"
            },
            {
              "name": "aquasecurity:trivy:PkgType",
              "value": "conan"
            }
          ]
        },
        {
          "bom-ref": ".@:pkg:conan/xsimd@11.1.0",
          "type": "library",
          "name": "xsimd",
          "version": "11.1.0",
          "purl": "pkg:conan/xsimd@11.1.0",
          "properties": [
            {
              "name": "aquasecurity:trivy:PkgID",
              "value": "xsimd/11.1.0"
            },
            {
              "name": "aquasecurity:trivy:PkgType",
              "value": "conan"
            }
          ]
        }
      ]
    }
  ],
  "dependencies": [
    {
      "ref": ".@:772e09eb-bff9-46f2-af6a-930e8ced753e",
      "dependsOn": [
        ".@:cfe81db4-3e82-4573-adf3-068719b5c7c0"
      ]
    },
    {
      "ref": ".@:cfe81db4-3e82-4573-adf3-068719b5c7c0",
      "dependsOn": [
        ".@:pkg:conan/gtest@1.14.0",
        ".@:pkg:conan/hello_world@1.0.0",
        ".@:pkg:conan/mathter@1.1.1",
        ".@:pkg:conan/ms-gsl@4.0.0"
      ]
    },
    {
      "ref": ".@:pkg:conan/gtest@1.14.0",
      "dependsOn": []
    },
    {
      "ref": ".@:pkg:conan/hello_world@1.0.0",
      "dependsOn": [
        ".@:pkg:conan/mathter@1.1.1",
        ".@:pkg:conan/ms-gsl@4.0.0"
      ]
    },
    {
      "ref": ".@:pkg:conan/mathter@1.1.1",
      "dependsOn": [
        ".@:pkg:conan/xsimd@11.1.0"
      ]
    },
    {
      "ref": ".@:pkg:conan/ms-gsl@4.0.0",
      "dependsOn": []
    },
    {
      "ref": ".@:pkg:conan/xsimd@11.1.0",
      "dependsOn": []
    },
    {
      "ref": "cpp@0.0.0-dev",
      "dependsOn": [
        ".@:772e09eb-bff9-46f2-af6a-930e8ced753e"
      ]
    }
  ]
}

```

The generator produced an accurate and comprehensive dependency tree, inclusive of license data for all identified direct and indirect components, meeting all requirements for a high-quality SBOM.

### conan v2

The maintainers of Conan have made substantial changes when upgrading to v2. In addition to changes in the CLI, they have completely overhauled the lockfile structure.

For instance, the v1 lockfile for the project appeared as follows:

```tsx
{
 "graph_lock": {
  "nodes": {
   "0": {
    "ref": "hello_world/1.0.0",
    "options": "enable_testing=False\nfPIC=True\nshared=False\nmathter:with_xsimd=True",
    "requires": [
     "1",
     "3"
    ],
    "build_requires": [
     "4"
    ],
    "path": "conanfile.py",
    "context": "host"
   },
   "1": {
    "ref": "mathter/1.1.1",
    "options": "with_xsimd=True\nxsimd:xtl_complex=False",
    "package_id": "5ab84d6acfe1f23c4fae0ab88f26e3a396351ac9",
    "prev": "0",
    "requires": [
     "2"
    ],
    "context": "host"
   },
   "2": {
    "ref": "xsimd/11.1.0",
    "options": "xtl_complex=False",
    "package_id": "5ab84d6acfe1f23c4fae0ab88f26e3a396351ac9",
    "prev": "0",
    "context": "host"
   },
   "3": {
    "ref": "ms-gsl/4.0.0",
    "options": "",
    "package_id": "5ab84d6acfe1f23c4fae0ab88f26e3a396351ac9",
    "prev": "0",
    "context": "host"
   },
   "4": {
    "ref": "gtest/1.14.0",
    "options": "build_gmock=True\ndisable_pthreads=False\nfPIC=True\nhide_symbols=False\nno_main=False\nshared=False",
    "package_id": "01fd5ff1a3e2609b618e12489366c49edfc13adc",
    "context": "host"
   }
  },
  "revisions_enabled": false
 },
 "version": "0.4",
 "profile_host": "[settings]\narch=armv8\narch_build=armv8\nbuild_type=Release\ncompiler=gcc\ncompiler.libcxx=libstdc++\ncompiler.version=13\nos=Linux\nos_build=Linux\n[options]\n[build_requires]\n[env]\n"
}
```

when in v2 it changed to a flat structure:

```tsx
{
    "version": "0.5",
    "requires": [
        "xsimd/11.1.0#2eb8a15496fbb27ad27c95a9a84534ee%1715440299.116",
        "ms-gsl/4.0.0#60ed7b1ae7ff8fbfe0bf9a1c1f0443a8%1700512686.639",
        "mathter/1.1.1#9b9d4d684c0eed20c5c0ce5b4f4cc800%1718560368.069",
        "gtest/1.14.0#25e2a474b4d1aecf5ff4f0555dcdf72c%1715096213.863"
    ],
    "build_requires": [],
    "python_requires": [],
    "config_requires": []
}
```

This had a significant effect on the capabilities of current generators, particularly impacting graph dependency parsing. We are also currently tracking other bugs and issues. The graph data is absent, and its generateable by running:

```tsx
conan graph info -f json .
```

However, in version 2, the maintainers introduced a new extensions module for SBOM generation.

**conan sbom extension**

As previously discussed, you will need to have Conan v2+ installed and configured.

Install the extension by running:

```tsx
conan config install https://github.com/conan-io/conan-extensions.git
```

Now, create a lockfile:

```tsx
conan install . --build=missing --lockfile-out=conan.lock
```

Run SBOM generation:

```tsx
conan sbom:cyclonedx --format 1.4_json .
```

```tsx
{
  "components": [
    {
      "author": "Conan",
      "bom-ref": "pkg:conan/gtest@1.14.0?rref=25e2a474b4d1aecf5ff4f0555dcdf72c",
      "description": "Google's C++ test framework",
      "externalReferences": [
        {
          "type": "website",
          "url": "https://github.com/google/googletest"
        }
      ],
      "licenses": [
        {
          "license": {
            "id": "BSD-3-Clause"
          }
        }
      ],
      "name": "gtest",
      "purl": "pkg:conan/gtest@1.14.0?rref=25e2a474b4d1aecf5ff4f0555dcdf72c",
      "type": "library",
      "version": "1.14.0"
    },
    {
      "author": "Conan",
      "bom-ref": "pkg:conan/mathter@1.1.1?rref=9b9d4d684c0eed20c5c0ce5b4f4cc800",
      "description": "Powerful 3D math and small-matrix linear algebra library for games and science.",
      "externalReferences": [
        {
          "type": "website",
          "url": "https://github.com/petiaccja/Mathter"
        }
      ],
      "licenses": [
        {
          "license": {
            "id": "MIT"
          }
        }
      ],
      "name": "mathter",
      "purl": "pkg:conan/mathter@1.1.1?rref=9b9d4d684c0eed20c5c0ce5b4f4cc800",
      "type": "library",
      "version": "1.1.1"
    },
    {
      "author": "Conan",
      "bom-ref": "pkg:conan/ms-gsl@4.0.0?rref=60ed7b1ae7ff8fbfe0bf9a1c1f0443a8",
      "description": "Microsoft's implementation of the Guidelines Support Library",
      "externalReferences": [
        {
          "type": "website",
          "url": "https://github.com/microsoft/GSL"
        }
      ],
      "licenses": [
        {
          "license": {
            "id": "MIT"
          }
        }
      ],
      "name": "ms-gsl",
      "purl": "pkg:conan/ms-gsl@4.0.0?rref=60ed7b1ae7ff8fbfe0bf9a1c1f0443a8",
      "type": "library",
      "version": "4.0.0"
    },
    {
      "author": "Conan",
      "bom-ref": "pkg:conan/xsimd@11.1.0?rref=2eb8a15496fbb27ad27c95a9a84534ee",
      "description": "C++ wrappers for SIMD intrinsics and parallelized, optimized mathematical functions (SSE, AVX, NEON, AVX512)",
      "externalReferences": [
        {
          "type": "website",
          "url": "https://github.com/xtensor-stack/xsimd"
        }
      ],
      "licenses": [
        {
          "license": {
            "id": "BSD-3-Clause"
          }
        }
      ],
      "name": "xsimd",
      "purl": "pkg:conan/xsimd@11.1.0?rref=2eb8a15496fbb27ad27c95a9a84534ee",
      "type": "library",
      "version": "11.1.0"
    }
  ],
  "dependencies": [
    {
      "ref": "pkg:conan/gtest@1.14.0?rref=25e2a474b4d1aecf5ff4f0555dcdf72c"
    },
    {
      "dependsOn": [
        "pkg:conan/gtest@1.14.0?rref=25e2a474b4d1aecf5ff4f0555dcdf72c",
        "pkg:conan/mathter@1.1.1?rref=9b9d4d684c0eed20c5c0ce5b4f4cc800",
        "pkg:conan/ms-gsl@4.0.0?rref=60ed7b1ae7ff8fbfe0bf9a1c1f0443a8"
      ],
      "ref": "pkg:conan/hello_world@1.0.0"
    },
    {
      "dependsOn": [
        "pkg:conan/xsimd@11.1.0?rref=2eb8a15496fbb27ad27c95a9a84534ee"
      ],
      "ref": "pkg:conan/mathter@1.1.1?rref=9b9d4d684c0eed20c5c0ce5b4f4cc800"
    },
    {
      "ref": "pkg:conan/ms-gsl@4.0.0?rref=60ed7b1ae7ff8fbfe0bf9a1c1f0443a8"
    },
    {
      "ref": "pkg:conan/xsimd@11.1.0?rref=2eb8a15496fbb27ad27c95a9a84534ee"
    }
  ],
  "metadata": {
    "component": {
      "author": "Conan",
      "bom-ref": "pkg:conan/hello_world@1.0.0",
      "name": "hello_world",
      "purl": "pkg:conan/hello_world@1.0.0",
      "type": "library",
      "version": "1.0.0"
    },
    "timestamp": "2024-06-17T11:06:39.879233+00:00",
    "tools": [
      {
        "externalReferences": [
          {
            "type": "build-system",
            "url": "https://github.com/CycloneDX/cyclonedx-python-lib/actions"
          },
          {
            "type": "distribution",
            "url": "https://pypi.org/project/cyclonedx-python-lib/"
          },
          {
            "type": "documentation",
            "url": "https://cyclonedx-python-library.readthedocs.io/"
          },
          {
            "type": "issue-tracker",
            "url": "https://github.com/CycloneDX/cyclonedx-python-lib/issues"
          },
          {
            "type": "license",
            "url": "https://github.com/CycloneDX/cyclonedx-python-lib/blob/main/LICENSE"
          },
          {
            "type": "release-notes",
            "url": "https://github.com/CycloneDX/cyclonedx-python-lib/blob/main/CHANGELOG.md"
          },
          {
            "type": "vcs",
            "url": "https://github.com/CycloneDX/cyclonedx-python-lib"
          },
          {
            "type": "website",
            "url": "https://github.com/CycloneDX/cyclonedx-python-lib/#readme"
          }
        ],
        "name": "cyclonedx-python-lib",
        "vendor": "CycloneDX",
        "version": "6.4.4"
      },
      {
        "externalReferences": [
          {
            "type": "website",
            "url": "https://github.com/conan-io/conan-extensions"
          }
        ],
        "name": "conan extension sbom:cyclonedx"
      }
    ]
  },
  "serialNumber": "urn:uuid:c448d7e6-e23c-41c4-8146-d5821afe0ae3",
  "version": 1,
  "$schema": "http://cyclonedx.org/schema/bom-1.4.schema.json",
  "bomFormat": "CycloneDX",
  "specVersion": "1.4"
}
```

As expected, the deep integration with Conan's core functions enabled the extension to generate high-quality SBOM. This includes licenses, a dependency tree, direct and transitive dependencies, along with accurate component metadata.

## Conclusion

In regards to SBOM generation, the C/++ community using Conan is fairly covered by open source generators that can generate quality outputs.

If you're using conan v1, we generally recommend t**rivy** for most scenarios because it can provide license data. However, if license data isn't important to you and you prefer the addition of CPE, s**yft** would be the next best option.

Regrettably, it’s fair to state that as of today, **cdxgen** should not be used in most scenarios due to its inferior results compared to other options. However, it may be useful for projects that lack conan support.

If you want to use conan V2, your only option is to use the conan extension solution. Luckily, it provides a high-quality SBOM, although only CDX SBOM(up to V1.4) is supported.
