Automated AWS Lambda Layer Build and Release Workflow
=====================================================

This repository provides a GitHub Actions workflow to automate the creation, packaging, and release of AWS Lambda layers. The workflow is triggered when changes are made to the `requirements.txt` file in a pull request. It builds a Lambda layer for multiple Python versions, packages it into a `.zip` file, and publishes it as a GitHub release.

* * * * *

Workflow Overview
-----------------

The workflow performs the following steps:

  1.  Trigger : Activates when changes are made to the `requirements.txt` file in a pull request.
  2.  Matrix Strategy : Builds the Lambda layer for multiple Python versions (`pypy3.10`, `3.9`, `3.10`, `3.11`, `3.12`, `3.13`).
  3.  Layer Name Generation : Dynamically generates a layer name based on the first two libraries listed in `requirements.txt`.
  4.  Packaging : Creates a `.zip` file containing the dependencies specified in `requirements.txt`.
  5.  Release : Publishes the `.zip` file as a GitHub release with version tags.

* * * * *

Workflow Details
----------------

### Trigger

The workflow is triggered when changes are made to the `requirements.txt` file in a pull request:

```yaml
on:
  pull_request:
    paths:
      - 'requirements.txt'
```

  -   Condition : Only runs if the `requirements.txt` file is modified.
  -   Purpose : Ensures that the workflow is only triggered when dependencies are updated.

* * * * *

### Jobs

#### Job: Build

The `build` job runs on the latest Ubuntu environment and uses a matrix strategy to test multiple Python versions.

```yaml

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["pypy3.10", "3.9", "3.10", "3.11", "3.12", "3.13"]
```

-   Matrix Strategy : Tests the Lambda layer across multiple Python versions to ensure compatibility.

* * * * *

#### Steps

1.  Checkout Code Clones the repository to the GitHub Actions runner:

```yaml
- name: Checkout code
  uses: actions/checkout@v2
```

2.  Set Up Python Configures the specified Python version from the matrix:

```yaml
- name: Set Up Python
  uses: actions/setup-python@v5
  with:
    python-version: ${{ matrix.python-version }}
```

3.  Run Layer Name Script Dynamically generates a layer name based on the first two libraries in `requirements.txt`:

```yaml
- name: Run layer name script
  run: |
    def get_layer_name():
        with open('requirements.txt', 'r') as file:
            lines = file.readlines()
        libraries = [line.split('==')[0] for line in lines if '==' in line]
        if len(libraries) >= 2:
            layer_name = f'{libraries[0]}_{libraries[1]}'
        elif libraries:
            layer_name = libraries[0]
        else:
            layer_name = 'default_layer'
        print(layer_name)
    layer_name = get_layer_name()
    echo "LAYER_NAME=$(python -c 'print(get_layer_name())')" >> $GITHUB_ENV
```

  -   Logic :
      -   Reads `requirements.txt` and extracts library names (e.g., `boto3`, `requests`).
      -   Combines the first two library names to form the layer name (e.g., `boto3_requests`).
      -   If fewer than two libraries are specified, uses the first library name or defaults to `default_layer`.
4.  Package Layer Creates a `.zip` file containing the dependencies:

```yaml
- name: Package layer
  run: zip -r9 ${LAYER_NAME}_layer_${{ matrix.python-version }}.zip python
```
  -   Output : Generates a `.zip` file named `<layer_name>_layer_<python-version>.zip` (e.g., `boto3_requests_layer_3.9.zip`).
5.  Publish Release Publishes the `.zip` file as a GitHub release:

```yaml
- name: Publish Release
  if: ${{ github.event_name == 'pull_request' }}
  uses: softprops/action-gh-release@v1
  with:
    files: ${LAYER_NAME}_layer_${{ matrix.python-version }}.zip
    tag_name: v1.0-${LAYER_NAME}_${{ matrix.python-version }}
```

  -   Files : Includes the `.zip` file in the release.
  -   Tag Name : Uses a versioned tag format (e.g., `v1.0-boto3_requests_3.9`).

* * * * *

How to Use This Workflow
------------------------

### 1\. Update `requirements.txt`

Add or modify the dependencies in the `requirements.txt` file. For example:

```
boto3==1.26.0
requests==2.28.1
numpy==1.23.5
```

### 2\. Create a Pull Request

Push your changes to a branch and create a pull request. The workflow will automatically trigger.

### 3\. Review the Build Results

  -   Check the GitHub Actions logs to ensure the workflow completes successfully.
  -   Verify that the `.zip` files are generated and published as GitHub releases.

### 4\. Deploy the Lambda Layer

Download the `.zip` file from the GitHub release and upload it to AWS Lambda as a new layer:

  1.  Go to the AWS Management Console.
  2.  Navigate to the Lambda service.
  3.  Create a new layer or update an existing one using the `.zip` file.

* * * * *

Customization
-------------

### Modify Python Versions

To add or remove Python versions, update the `matrix` section in the workflow:

```yaml
matrix:
  python-version: ["3.8", "3.9", "3.10"]  # Example: Add or remove versions
```

### Change Layer Naming Logic

If you want to customize how the layer name is generated, modify the `Run layer name script` step in the workflow.

* * * * *

Notes
-----

1.  Dependencies :

    -   Ensure that all dependencies in `requirements.txt` are compatible with the specified Python versions.
2.  GitHub Releases :

    -   Each release includes a `.zip` file for each Python version.
    -   Tags follow the format `v1.0-<layer_name>_<python-version>`.
3.  Error Handling :

    -   If the workflow fails, check the logs for debugging information. Common issues include missing dependencies or syntax errors in `requirements.txt`.

* * * * *

Conclusion
----------

This workflow automates the process of building and releasing AWS Lambda layers, ensuring consistency and compatibility across multiple Python versions. By leveraging GitHub Actions, you can streamline dependency management and simplify the deployment of Lambda layers. If you have any questions or need further assistance, feel free to ask!

* * * * *
