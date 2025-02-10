# SBOM Generation for C/++

## Overview

SBOM generation has become essential for detecting vulnerabilities, ensuring compliance, and addressing license issues. Although it may seem straightforward, the quality of results varies based on ecosystems, generators, formats, and specific project files.

By utilizing this repository, a fork from [egendron93/cpp_library_template](https://github.com/egendron93/cpp_library_template), which is an open-source C++ library with Conan as the package manager, we aim to demonstrate different outputs and guide you through the SBOM generation process and its expected results. This will assist you in reproducing the process in your own projects with Manifest.

<aside>

⚠️ This repository assumes the use of Conan. To date, other package managers and C/++ projects without package managers are not supported by any OSS generator.

</aside>

## Prerequisites

- Install **Manifest CLI** by following the [installation guide](https://github.com/manifest-cyber/cli).
- **conan** **v1** is installed and configured. (v2 is required for conan extensions):
- python3 or python installed. (pip3 or pip)
- (optional) Node, NPM installed (**cdxgen** only)

## Conan installation

Conan is a Python-based software package manager. Installation should be as easy as running:

```
pip install conan
```

This will install v2 by default, to install a specific version run:

for python run:

```
pip install "conan==1.66.0"
```

for python3 run:

```
pip3 install "conan<2"
```

if using `pipx`, you can run:

```
pipx ensurepath && pipx install "conan==1.66.0"
```

To verify conan is installed correctly, run:

```
conan --version
```

You should see output similar to:

```
Conan version 1.x.y
```

For more information, visit the [official installation guide](https://docs.conan.io/2/installation.html).

## SBOM Quality

A quality SBOM possesses key traits that ensure comprehensiveness and accuracy:

- Detects **all direct dependencies**.
- Identifies **transitive dependencies** to capture the full hierarchy of components.
- Includes **detailed license data**.
- Provides **precise identifiers** such as Package URLs (purl), CPEs, and SHA for integrity and traceability.
- Features a **clear dependency tree** illustrating interconnections among components.
- Includes **accurate metadata** for each component, including the root component.

## Generate SBOM

In this example, we compare the results of **Syft**, **Trivy**, and **Cdxgen**.

> ⚠️ Other generators exist, but they do not provide support for C/++ and Conan projects.

> ⚠️ Important! We are currently using **Conan v1** because not all features have been ported and breaking changes fixed for v2. If you prefer to use v2, feel free to skip this part.

</aside>

### Installing Generators

The **manifest-cli** requires proper installation of the underlying generators. You can install them using:

```bash
manifest-cli install -g cdxgen
manifest-cli install -g syft
manifest-cli install -g trivy
```

> ⚠️ If you install into the default location (e.g. /usr/local/bin), you may need to use `sudo` alternatively, install into a custom directory with the `-d` flag (for example, /path/to/your/bin) and ensure that directory is in your $PATH, or call the binary directly.

### Project Files

Conan projects have different configurations such as `conanfile.txt`, `conaninfo.txt`, `conanfile.py`, and `conanfile.lock`.

This project uses [`conanfile.py`](http://conanfile.py), which is the more robust and recommended way by Conan. See the `legacy` branch for an example with `conanfile.txt` and `conaninfo.txt`.

### Generating SBOM with Manifest

The example project uses a mix of those, lets attempt to generate an SBOM and compare the results.

#### **Syft**

Run:

Syft uses `conan.lock` file to detect direct and transitive dependencies, to generate one run:

```bash
conan lock create conanfile.py --lockfile-out=conan.lock
```

Then, to generate a SBOM run:

```bash
manifest-cli sbom --name=cpp --version=0.0.0-dev --generator=syft -f syft-sbom.json .
```

The output should look like this

```tsx
{
  "$schema": "http://cyclonedx.org/schema/bom-1.6.schema.json",
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "version": 1,
  "metadata": {
    "timestamp": "2025-02-10T14:34:11-06:00",
    "tools": {
      "components": [
        {
          "type": "application",
          "author": "anchore",
          "name": "syft",
          "version": "1.18.1"
        }
      ]
    },
    "component": {
      "bom-ref": "0a68eb57-c88a-5f34-9e9d-27f85e68af4f",
      "type": "application",
      "name": "cpp",
      "version": "0.0.0-dev"
    }
  },
  "components": [
    {
      "bom-ref": "pkg:conan/gtest@1.14.0?package-id=4c8f783013cfb2eb",
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
      "bom-ref": "pkg:conan/hello_world@1.0.0?package-id=1c0e4535a9fb07fc",
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
      "bom-ref": "pkg:conan/mathter@1.1.1?package-id=6a9872a2438c3cba",
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
      "bom-ref": "pkg:conan/ms-gsl@4.0.0?package-id=8561fa79ef7b4a03",
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
      "bom-ref": "pkg:conan/xsimd@11.1.0?package-id=82c175e62a3194ad",
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
  ],
  "dependencies": [
    {
      "ref": "pkg:conan/hello_world@1.0.0?package-id=1c0e4535a9fb07fc",
      "dependsOn": [
        "pkg:conan/mathter@1.1.1?package-id=6a9872a2438c3cba",
        "pkg:conan/ms-gsl@4.0.0?package-id=8561fa79ef7b4a03"
      ]
    },
    {
      "ref": "pkg:conan/mathter@1.1.1?package-id=6a9872a2438c3cba",
      "dependsOn": [
        "pkg:conan/xsimd@11.1.0?package-id=82c175e62a3194ad"
      ]
    }
  ]
}
```

The results appear to include all build and runtime dependencies with an accurate dependency tree; however, they lack license data. This type of information is not contained in the `conan.lock` file. Syft also generates CPEs which are helpful for license and vulnerability matching.

#### **Cdxgen**

Cdxgen supports parsing `CMakeLists.txt` and `conan.lock` (among other files).

```bash
conan lock create conanfile.py --lockfile-out=conan.lock
```

then generate with:

```bash
manifest-cli sbom --name=cpp --version=0.0.0-dev --generator=cdxgen -vvv -f cdxgen-sbom.json .
```

```tsx
{
  "$schema": "http://cyclonedx.org/schema/bom-1.6.schema.json",
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "version": 1,
  "metadata": {
    "timestamp": "2025-02-10T14:35:56-06:00",
    "lifecycles": [
      {
        "phase": "build"
      },
      {
        "phase": "post-build"
      }
    ],
    "tools": {
      "components": [
        {
          "bom-ref": "pkg:npm/@cyclonedx/cdxgen@11.1.7",
          "type": "application",
          "authors": [
            {
              "name": "OWASP Foundation"
            }
          ],
          "publisher": "OWASP Foundation",
          "group": "@cyclonedx",
          "name": "cdxgen",
          "version": "11.1.7",
          "purl": "pkg:npm/%40cyclonedx/cdxgen@11.1.7"
        }
      ]
    },
    "authors": [
      {
        "name": "OWASP Foundation"
      }
    ],
    "component": {
      "bom-ref": "0a68eb57-c88a-5f34-9e9d-27f85e68af4f",
      "type": "application",
      "name": "cpp",
      "version": "0.0.0-dev"
    },
    "properties": [
      {
        "name": "cdx:bom:componentTypes",
        "value": "conan\\ngeneric"
      }
    ]
  },
  "components": [
    {
      "bom-ref": "pkg:conan/hello_world@1.0.0",
      "type": "library",
      "name": "hello_world",
      "version": "1.0.0",
      "purl": "pkg:conan/hello_world@1.0.0"
    },
    {
      "bom-ref": "pkg:conan/mathter@1.1.1",
      "type": "library",
      "name": "mathter",
      "version": "1.1.1",
      "purl": "pkg:conan/mathter@1.1.1"
    },
    {
      "bom-ref": "pkg:conan/xsimd@11.1.0",
      "type": "library",
      "name": "xsimd",
      "version": "11.1.0",
      "purl": "pkg:conan/xsimd@11.1.0"
    },
    {
      "bom-ref": "pkg:conan/ms-gsl@4.0.0",
      "type": "library",
      "name": "ms-gsl",
      "version": "4.0.0",
      "purl": "pkg:conan/ms-gsl@4.0.0"
    },
    {
      "bom-ref": "pkg:conan/gtest@1.14.0",
      "type": "library",
      "name": "gtest",
      "version": "1.14.0",
      "purl": "pkg:conan/gtest@1.14.0"
    },
    {
      "bom-ref": "pkg:generic/hello_world@1.0.0",
      "type": "library",
      "name": "hello_world",
      "version": "1.0.0",
      "purl": "pkg:generic/hello_world@1.0.0"
    },
    {
      "bom-ref": "pkg:generic/gtest",
      "type": "library",
      "name": "gtest",
      "purl": "pkg:generic/gtest",
      "properties": [
        {
          "name": "name_with_version",
          "value": "googletest-1.15.0"
        },
        {
          "name": "analyzed_version",
          "value": "googletest-1.15.0"
        },
        {
          "name": "analyzed_source_url",
          "value": "https://github.com/google/googletest/archive/refs/tags/v1.15.0.tar.gz"
        },
        {
          "name": "analyzed_source_filename",
          "value": "gtest-1.15.0.tar.gz"
        },
        {
          "name": "analyzed_source_hash",
          "value": "7315acb6bf10e99f332c8a43f00d5fbb1ee6ca48c52f6b936991b216c586aaad"
        },
        {
          "name": "PkgProvides",
          "value": "gmock, gmock_main, gtest, gtest_main"
        },
        {
          "name": "available_versions",
          "value": "1.15.0-1, 1.14.0-2, 1.14.0-1, 1.13.0-1, 1.12.1-1, 1.11.0-2, 1.11.0-1, 1.10.0-1, 1.8.1-1, 1.8.0-5, 1.8.0-4, 1.8.0-3, 1.8.0-2, 1.8.0-1, 1.7.0-5, 1.7.0-4, 1.7.0-2"
        }
      ],
      "evidence": {
        "identity": [
          {
            "field": "purl",
            "confidence": 0.5,
            "methods": [
              {
                "technique": "source-code-analysis",
                "confidence": 0.5,
                "value": "Filename /Users/oriavraham/Development/examples/examples-cpp-conan/tests/CMakeLists.txt"
              }
            ]
          }
        ]
      }
    },
    {
      "bom-ref": "pkg:generic/hello_world",
      "type": "library",
      "name": "hello_world",
      "purl": "pkg:generic/hello_world",
      "evidence": {
        "identity": [
          {
            "field": "purl",
            "methods": [
              {
                "technique": "source-code-analysis",
                "confidence": 0.5,
                "value": "Filename /Users/oriavraham/Development/examples/examples-cpp-conan/test_package/CMakeLists.txt"
              }
            ]
          }
        ]
      }
    },
    {
      "bom-ref": "pkg:generic/PackageTest",
      "type": "library",
      "name": "PackageTest",
      "purl": "pkg:generic/PackageTest"
    },
    {
      "bom-ref": "pkg:generic/Mathter@1.1.1",
      "type": "library",
      "name": "Mathter",
      "version": "1.1.1",
      "purl": "pkg:generic/Mathter@1.1.1",
      "evidence": {
        "identity": [
          {
            "field": "purl",
            "methods": [
              {
                "technique": "source-code-analysis",
                "confidence": 0.5,
                "value": "Filename /Users/oriavraham/Development/examples/examples-cpp-conan/src/CMakeLists.txt"
              }
            ]
          }
        ]
      }
    },
    {
      "bom-ref": "pkg:generic/Microsoft.GSL@4.0.0",
      "type": "library",
      "name": "Microsoft.GSL",
      "version": "4.0.0",
      "purl": "pkg:generic/Microsoft.GSL@4.0.0",
      "evidence": {
        "identity": [
          {
            "field": "purl",
            "methods": [
              {
                "technique": "source-code-analysis",
                "confidence": 0.5,
                "value": "Filename /Users/oriavraham/Development/examples/examples-cpp-conan/src/CMakeLists.txt"
              }
            ]
          }
        ]
      }
    }
  ],
  "dependencies": [
    {
      "ref": "pkg:generic/hello_world@1.0.0"
    }
  ]
}
```

#### **Trivy**

**Trivy** takes a different approach, it explicitly uses `conan.lock` files to find all dependencies, then it uses the local conan cache folder and parse [`conanfile.py`](http://conanfile.py) for license data.

Install dependencies and generate a lockfile with:

```bash
conan install . --build=missing
conan lock create conanfile.py --lockfile-out=conan.lock
```

then generate with:

```bash
manifest-cli sbom --name=cpp --version=0.0.0-dev --generator=trivy -vvv -f trivy-sbom.json .
```

```tsx
{
  "$schema": "http://cyclonedx.org/schema/bom-1.6.schema.json",
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "version": 1,
  "metadata": {
    "timestamp": "2025-02-10T14:45:09-06:00",
    "tools": {
      "components": [
        {
          "type": "application",
          "group": "aquasecurity",
          "name": "trivy",
          "version": "0.58.2"
        }
      ]
    },
    "component": {
      "bom-ref": "0a68eb57-c88a-5f34-9e9d-27f85e68af4f",
      "type": "application",
      "name": "cpp",
      "version": "0.0.0-dev"
    }
  },
  "components": [
    {
      "bom-ref": "2051f1f4-0ca3-492f-8b63-1039d24ed686",
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
      "bom-ref": "pkg:conan/gtest@1.14.0",
      "type": "library",
      "name": "gtest",
      "version": "1.14.0",
      "licenses": [
        {
          "license": {
            "name": "BSD-3-Clause"
          }
        }
      ],
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
      "bom-ref": "pkg:conan/hello_world@1.0.0",
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
      "bom-ref": "pkg:conan/mathter@1.1.1",
      "type": "library",
      "name": "mathter",
      "version": "1.1.1",
      "licenses": [
        {
          "license": {
            "name": "MIT"
          }
        }
      ],
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
      "bom-ref": "pkg:conan/ms-gsl@4.0.0",
      "type": "library",
      "name": "ms-gsl",
      "version": "4.0.0",
      "licenses": [
        {
          "license": {
            "name": "MIT"
          }
        }
      ],
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
      "bom-ref": "pkg:conan/xsimd@11.1.0",
      "type": "library",
      "name": "xsimd",
      "version": "11.1.0",
      "licenses": [
        {
          "license": {
            "name": "BSD-3-Clause"
          }
        }
      ],
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
  ],
  "dependencies": [
    {
      "ref": "2051f1f4-0ca3-492f-8b63-1039d24ed686",
      "dependsOn": [
        "pkg:conan/gtest@1.14.0",
        "pkg:conan/hello_world@1.0.0"
      ]
    },
    {
      "ref": "d86f70b0-a6a9-4a25-9b4c-488d2854d499",
      "dependsOn": [
        "2051f1f4-0ca3-492f-8b63-1039d24ed686"
      ]
    },
    {
      "ref": "pkg:conan/gtest@1.14.0"
    },
    {
      "ref": "pkg:conan/hello_world@1.0.0",
      "dependsOn": [
        "pkg:conan/mathter@1.1.1",
        "pkg:conan/ms-gsl@4.0.0"
      ]
    },
    {
      "ref": "pkg:conan/mathter@1.1.1",
      "dependsOn": [
        "pkg:conan/xsimd@11.1.0"
      ]
    },
    {
      "ref": "pkg:conan/ms-gsl@4.0.0"
    },
    {
      "ref": "pkg:conan/xsimd@11.1.0"
    }
  ]
}
```

Trivy produced an accurate and comprehensive dependency tree, inclusive of license data for all identified direct and indirect components, meeting all requirements for a high-quality SBOM.

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

```bash
conan config install https://github.com/conan-io/conan-extensions.git
```

Now, create a lockfile:

```bash
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

If you're using conan v1, we generally recommend t**Trivy** for most scenarios because it can provide license data. However, if license data isn't important to you and you prefer the addition of CPE, s**yft** would be the next best option.

Regrettably, it’s fair to state that as of today, **cdxgen** should not be used in most scenarios due to its inferior results compared to other options. However, it may be useful for projects that lack conan support.

If you want to use conan V2, your only option is to use the conan extension solution. Luckily, it provides a high-quality SBOM, although only CDX SBOM(up to V1.4) is supported.

# Appendix: Migrating a C++ Project to Conan

## Install Conan

First, you need to install Conan. If you don’t already have it, you can install it using pip:

```
pip install conan # for v1 "conan==1.66.0"

```

## Create a conanfile.py

In the root directory of your C++ project, create a file named conanfile.py. This file will manage your dependencies.

Here is an example of what `conanfile.py` might look like:

```python
from conan import ConanFile
from conan.tools.cmake import CMakeToolchain, CMake, cmake_layout

class MyProjectConan(ConanFile):
    name = "myproject"
    version = "1.0"
    license = "MIT"
    author = "John Doe <john.doe@example.com>"
    url = "https://github.com/myproject/myproject"
    description = "A short description of my project"
    topics = ("conan", "cmake", "example")
    settings = "os", "compiler", "build_type", "arch"
    options = {"shared": [True, False]}
    default_options = {"shared": False}
    requires = [
        "boost/1.85.0",
        "fmt/8.0.1",
    ]
    build_requires = [
        "gtest/1.11.0"
    ]
    generators = "CMakeDeps", "CMakeToolchain"

    def layout(self):
        cmake_layout(self)

    def generate(self):
        tc = CMakeToolchain(self)
        tc.generate()

    def build(self):
        cmake = CMake(self)
        cmake.configure()
        cmake.build()

    def imports(self):
        self.copy("*.dll", dst="bin", src="bin")
        self.copy("*.dylib*", dst="bin", src="lib")

    def build_requirements(self):
        self.tool_requires("cmake/3.21.1")  # Specify a specific version of CMake if needed

    def package(self):
        self.copy("*.h", dst="include", src="src")
        self.copy("*.lib", dst="lib", keep_path=False)
        self.copy("*.dll", dst="bin", keep_path=False)
        self.copy("*.so", dst="lib", keep_path=False)
        self.copy("*.dylib", dst="lib", keep_path=False)
        self.copy("*.a", dst="lib", keep_path=False)

    def package_info(self):
        self.cpp_info.libs = ["myproject"]
```

Replace `boost/1.85.0`, `fmt/8.0.1`, and `gtest/1.11.0` with the actual dependencies your project needs.
The `build_requirements` method ensures that any necessary build tools, like a specific version of `CMake`, are available.

## Integrate Conan with CMake

If your project uses `CMake`, you’ll need to update your `CMakeLists.txt` to integrate with Conan. Here is an example of a typical `CMakeLists.txt` file before and after the integration:

```
cmake_minimum_required(VERSION 3.10)
project(MyProject)

set(CMAKE_CXX_STANDARD 11)

add_executable(MyProject main.cpp)
```

Updated `CMakeLists.txt` with Conan integration:

```
cmake_minimum_required(VERSION 3.10)
project(MyProject)

set(CMAKE_CXX_STANDARD 11)

find_package(Boost REQUIRED)
find_package(fmt REQUIRED)
find_package(GTest REQUIRED)

add_executable(MyProject main.cpp)

target_link_libraries(MyProject Boost::Boost fmt::fmt GTest::GTest)
```

## Install Dependencies with Conan

conan v1:

```
conan install . --build=missing
conan lock create conanfile.py --lockfile-out=conan.lock
```

The first command will download and build any missing dependencies, generating the necessary files for CMake to find them.
The second command will create a lockfile named `conan.lock`, which will ensure the exact versions of dependencies are used in future builds.

conan v2:

```
conan install . --build=missing --lockfile-out=conan.lock
```

Does the same but with single command.

## Build

Now, you can build your project using CMake. Run the following command:

```
conan build .
```

## Update source code to use packages

Ensure that your source code includes the headers and links against the libraries provided by conan. For example:

```
#include <boost/asio.hpp>
#include <fmt/core.h>
#include <gtest/gtest.h>

int main() {
    boost::asio::io_context io_context;
    fmt::print("Hello, world!\n");
    return 0;
}
```

Make sure to adjust include paths and library links as needed, based on the dependencies specified in your `conanfile.py`.

## Conclusion

By following these steps, you have successfully migrated your vanilla C++ project to use Conan v1/v2 with `conanfile.py` and generate a `conan.lock` needed for quality SBOMs and for dependency management.

This setup ensures consistent builds, and makes it easier to handle updates and new dependencies in the future. The addition of a lockfile ensures reproducible builds by locking the exact versions of dependencies used.
