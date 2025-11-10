# CodSpeed Composite Action for Faster-Ethereum Python Suite

[![CI](https://github.com/BobTheBuidler/codspeed/actions/workflows/test.yml/badge.svg)](https://github.com/BobTheBuidler/codspeed/actions/workflows/test.yml)

This composite GitHub Action builds and/or installs a Python wheel, then runs CodSpeed benchmarks using pytest with a constant test pattern. It is designed specifically for the faster-ethereum Python suite (faster-eth-abi, faster-eth-utils, faster-hexbytes, faster-web3.py) and is **not a general-purpose CodSpeed action**.

## Faster-Ethereum Python Suite Repositories

- [faster-eth-abi](https://github.com/BobTheBuidler/faster-eth-abi)
- [faster-eth-utils](https://github.com/BobTheBuidler/faster-eth-utils)
- [faster-hexbytes](https://github.com/BobTheBuidler/faster-hexbytes)
- [faster-web3.py](https://github.com/BobTheBuidler/faster-web3.py)

## Features

- Supports both building the wheel internally or using a pre-built wheel artifact.
- Handles sharded and non-sharded CodSpeed runs.
- Validates input combinations and provides clear error messages.
- Designed for maintainability and consistency across the faster-ethereum suite.

## Inputs

| Name                   | Required | Default         | Description                                                                                   |
|------------------------|----------|-----------------|-----------------------------------------------------------------------------------------------|
| python-version         | Yes      |                 | Python version to use for the CodSpeed run.                                                   |
| hash-key               | No       |                 | Hash key for mypycify caching (files/globs). Required if not using wheel-artifact-name.       |
| pip-cache-dependency-path | No    |                 | Dependency files for pip cache.                                                               |
| wheel-artifact-name    | No       |                 | Name of the pre-built wheel artifact to download (if provided, skips build step).             |
| install-wheel-command  | No       | pip install ... | Custom shell command to install the built wheel.                                              |
| install-codspeed-deps  | Yes      |                 | Shell command to install CodSpeed dependencies (e.g. 'pip install -r requirements-codspeed.txt'). |
| ccache                | No        | false           | Enable ccache for mypycify.                                                                   |
| shards                | No        | 1               | Number of CodSpeed shards (for parallelization).                                              |
| mode                  | No        | instrumentation | CodSpeed run mode (e.g. 'instrumentation', 'walltime').                                      |

## Usage

### Build the Wheel Internally

```yaml
- uses: BobTheBuidler/codspeed@master
  with:
    python-version: "3.11"
    hash-key: "setup.py"
    pip-cache-dependency-path: "setup.py"
    install-wheel-command: "pip install --no-binary :all: $(find testpkg/dist -name '*.whl')"
    install-codspeed-deps: "pip install pytest-codspeed"
    shards: "1"
```

### Use a Prebuilt Wheel Artifact

```yaml
- name: Build wheel
  run: |
    cd testpkg
    python3 -m pip install --upgrade pip setuptools wheel
    python3 setup.py bdist_wheel
    cd ..
- name: Upload Wheel Artifact
  uses: actions/upload-artifact@v4
  with:
    name: testpkg-wheel
    path: testpkg/dist/*.whl

- uses: BobTheBuidler/codspeed@master
  with:
    python-version: "3.11"
    wheel-artifact-name: "testpkg-wheel"
    install-wheel-command: "pip install --no-binary :all: $(find . -name '*.whl')"
    install-codspeed-deps: "pip install pytest-codspeed"
    shards: "1"
```

### Sharded CodSpeed Run

```yaml
- uses: BobTheBuidler/codspeed@master
  with:
    python-version: "3.11"
    hash-key: "setup.py"
    pip-cache-dependency-path: "setup.py"
    install-wheel-command: "pip install --no-binary :all: $(find testpkg/dist -name '*.whl')"
    install-codspeed-deps: "pip install pytest-codspeed"
    shards: "3"
```

### Input Validation

The action will fail with a clear error if:
- Both `wheel-artifact-name` and `hash-key` are provided.
- Neither `wheel-artifact-name` nor `hash-key` is provided.
- `install-codspeed-deps` is missing.

## Troubleshooting

- **"You must provide either 'wheel-artifact-name' or 'hash-key'"**: Provide one of these inputs to specify how the wheel should be obtained.
- **"You must provide only one of 'wheel-artifact-name' or 'hash-key', not both."**: Only one method can be used at a time.
- **"'install-codspeed-deps' is required."**: You must specify how to install CodSpeed dependencies.

## License

MIT
