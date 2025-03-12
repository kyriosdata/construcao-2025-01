# FHIRUT - FHIR Unit Test Suite

This document describes FHIRUT, a tool for automating validation tests of FHIR resources. The pronunciation could be like "FHIR ROOT".

## Context
FHIR is a flexible standard, which makes compliance checking a complex process. HL7 recommends using the reference implementation of the FHIR validator (`validator_cli`).

For example,

```
java -jar validator_cli.jar -version 4.0.1 -output output.json input.json
```

produces an instance of the [OperationOutcome](https://hl7.org/fhir/R4/operationoutcome.html) resource, deposited in the **output.json** file, containing the result of evaluating the instance contained in the **input.json** file. For more information on using the validator, consult the available [documentation](https://confluence.hl7.org/display/FHIR/Using+the+FHIR+Validator).

In essence, the purpose of this project is to create a solution that receives multiple inputs (instances) of resources to be validated, as well as the corresponding expectations, and presents the results to the user according to the established expectations.

## Overview

FHIRUT is a tool designed to simplify and automate the process of validating FHIR resources. A key use case for FHIRUT is supporting Implementation Guide development. It allows users to define test cases, organize them into suites, run validation automatically, and view the results in a clear and concise manner. The tool is inspired by the concept of JUnit, but adapted to the FHIR context.

## Key Components

1.  Test Cases: individual test units, containing the definition of the context, the instance to be validated, and the expected results.
2.  Test Suites: logical groupings of related test cases.
3.  Test Runner: the component responsible for executing the tests, using a FHIR validator (such as `validator_cli`).
4.  Result Comparator: the component that compares the obtained results with the expected results.
5.  Report Generator: the component that generates detailed and visually appealing reports of the test results.
6.  Graphical User Interface (Optional): an interface to facilitate user interaction.

## 1. Test Case Definition

Test cases are defined in YAML files, with the following structure:

```yaml
test_id: Patient-001  # Unique identifier of the test case (string).
description: Checks the basic structure of a Patient resource. # Description (string).
context:  # Defines the validation context.
  igs:  # List of Implementation Guides (IGs) to be used.
    - br-core-r4  # IDs of the IGs (list of strings).
  profiles:  # List of profiles (StructureDefinitions) to be applied.
    - br-patient  # IDs of the profiles or canonical URLs (list of strings).
  resources:  # (Optional) Additional FHIR resources (ValueSet, CodeSystem, etc.).
    - valuesets/my-valueset.json  # Path to the file or the inline resource.
instance_path: instances/patient_example.json  # Path to the instance file (recommended).
# If instance_path is not provided, the tool searches, by convention:
# 1. <yaml file name>.json
# 2. <test_id>.json

expected_results:  # Defines the expected validation results.
  status: success  # Expected overall level ('success', 'error', 'warning', 'information').
  errors: []  # List of expected errors (empty list indicates success).
  warnings: []  # List of expected warnings.
  informations: []  # List of expected information messages.
  invariants: # Optional.
    - expression: "OperationOutcome.issues.count() = 0"
      expected: true # Optional. Default is true.
```

**Rules for `instance_path` and `instance`:**

1.  `instance_path`: If provided, the tool will use the file at the specified path.
2.  Convention: If `instance_path` is not provided, the tool will search for:
    *   A file with the same name as the YAML file, but with a `.json` extension.
    *   If not found, a file with the name of the `test_id` and a `.json` extension.
3.  Error: If no instance is found, an error will be produced.

## 2. Test Suite Organization

Test suites can be organized in two ways:

*   **Directory Structure:**
    *   Each directory within a root `tests/` directory represents a suite.
    *   The YAML files within each directory are the test cases.

    ```
    tests/
      ├── suite_br-core/
      │   ├── Patient-001.yaml
      │   ├── Patient-002.yaml
      │   └── Observation-001.yaml
      ├── suite_medications/
      │   ├── Medication-001.yaml
      │   └── MedicationRequest-001.yaml
      └── ...
    ```

*   **Manifest File (YAML):**
    *   A YAML file that lists the suites and their respective paths.

    ```yaml
    suites:
      - name: BR Core
        path: tests/suite_br-core
      - name: Medications
        path: tests/suite_medications

    ```

## 3. Test Execution

1.  The tool reads the suite configuration (directory or manifest).
2.  For each test case (YAML file):
    *   Reads the file.
    *   Extracts the context (IGs, profiles, additional resources).
    *   Locates and loads the instance (using `instance_path`, `instance`, or the convention).
    *   Executes the FHIR validator (`validator_cli` or other) with the appropriate parameters.
    *   Captures the validator's output.
    *   Processes the output to extract the results.
3.  Parallelization is possible, but file access must be handled carefully.
4.  Timeout must be implemented.
5. The most recent version of `validator_cli` is downloaded and used.

## 4. Result Comparison

*   Compares the obtained results with the `expected_results`. The answer of the `validator_cli` is an instance of [OperationOutcome](https://www.hl7.org/fhir/R4/operationoutcome.html).
*   Records the discrepancies.

## 5. Report Generation

*   Main Format: HTML (browsable and visually appealing).
*   Additional format: JSON (for possible integration with other tools).

HTML Report Structure (initial suggestion):

*   Summary:
    *   Overall statistics (tests executed, passed/failed/warnings, graphs).
    *   Execution time.
    *   List of suites with summarized results.

*   Details per Suite:
    *   Suite name.
    *   Suite statistics.
    *   Table with test results (link to details).

*   Details per Test:
    *   `test_id`, `description`, context.
    *   Instance (formatted).
    *   Expected Results.
    *   Obtained Results (table with OperationOutcome's diagnostics, location, severity).
    *   Highlighted differences.
    *   Raw validator output (optional).

## 6. User Interface

*   MVP: Command-line interface (CLI) + HTML Reports.
*   Later Phase: Graphical User Interface (GUI)
    *   GUI Features:
        *   Dashboard.
        *   Suite Manager.
        *   Test Case Manager (create, edit, view, delete).
        *   Test Execution (selection, progress, real-time results).
        *   Result Visualization (navigation, filters, comparison).
        *   Settings.

### Suggested Technologies (for those that uses Python)

**Note**: while specific technologies are suggested, they are not mandatory. The only requirement is that all tools used be open source.

*   Language: Python.
*   Graphical User Interface (Optional): Streamlit.
*   Validator: `validator_cli` (or other).
*   Test Case Format: YAML.
*   Reports: HTML, Jinja2, Plotly (optional).

## 7. Backend

The backend is the core of FHIRUT, responsible for all the heavy lifting involved in test execution, validation, and result processing. It is designed to be modular and extensible, allowing for future enhancements and integration with different FHIR validators.

Key Responsibilities:

1.  Test Case Loading and Parsing:
    *   Reads test case definitions from YAML files.
    *   Handles the logic for locating instance files (using `instance_path`, `instance`, or the naming convention).
    *   Parses the YAML structure and validates it against a predefined schema (to ensure correct format).
    *   Handles potential errors gracefully (e.g., missing files, invalid YAML).

2.  Suite Management:
    *   Loads suite definitions (either from directory structures or manifest files).
    *   Provides an interface for accessing test cases within a suite.
    *   Allows for filtering and selecting test cases based on various criteria (e.g., suite, ID, description).

3.  Context Preparation:
    *   Based on the `context` defined in the test case:
        *   Collects the necessary Implementation Guides (IGs).
        *   Collects the relevant profiles (StructureDefinitions).
        *   Loads any additional resources specified (ValueSets, CodeSystems, etc.).
        *   Prepares this information in a format suitable for the chosen FHIR validator.

4.  Validator Execution:
    *   Interfaces with the chosen FHIR validator (e.g., `validator_cli`). This should be done through an abstraction layer (e.g., a `Validator` class) to make it easy to switch validators in the future.
    *   Constructs the appropriate command-line arguments (or API calls) for the validator, based on the test case context and instance.
    *   Executes the validator, handling potential errors and timeouts.
    *   Captures the validator's standard output (stdout) and standard error (stderr).

5.  Result Processing:
    *   Parses the validator's output (which might be in JSON, XML, or another format, depending on the validator).
    *   Extracts relevant information:
        *   Overall validation status (success, error, warning).
        *   Individual messages (errors, warnings, information).
        *   Location of each message (XPath or FHIRPath).
        *   Severity of each message.

6.  Result Comparison:
    *   Compares the extracted results with the `expected_results` defined in the test case.
    *   Identifies and categorizes discrepancies:
        *   Unexpected errors/warnings/information.
        *   Expected errors/warnings/information that were not found.
        *   Mismatches in severity or location.
    *   FHIRPath expression evaluation and comparison with expected result.

7.  Data Storage:
    *   If persistence is required (e.g., for storing test results over time, or for a GUI), the backend handles storing and retrieving this data. This could be in:
        *   Simple files (JSON, YAML).

8. Report Data Preparation:
   * Prepares all data needed by the report generator, including a structured report.

Architectural Considerations:

*   Modularity: the backend should be designed as a set of independent modules (classes or functions) with clear responsibilities. This makes it easier to maintain, test, and extend.
*   Abstraction: use abstraction layers (interfaces, abstract classes) to decouple components. For example, the `Validator` class should abstract away the specific details of interacting with `validator_cli` or any other validator.
*   Error handling: implement robust error handling throughout the backend to gracefully handle unexpected situations (e.g., invalid input, validator errors, file system issues). Provide informative error messages to the user.
*   Configuration: allow for configuration of various aspects of the backend, such as:
    *   The path to the FHIR validator executable.
    *   Timeout settings.
    *   Parallelization options.
    *   Data storage options.
*   Logging: use a logging library to record important events, debug information, and errors. This is crucial for troubleshooting.
* Testability: The backend must itself be comprehensively tested.

