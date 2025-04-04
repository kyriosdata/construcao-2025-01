# FHIRUT - FHIR Unit Test Suite

Este documento descreve FUT, uma ferramenta para automatizar testes de conformidade 
de instâncias de recursos FHIR. 

## Contexto

FHIR é um padrão flexível, o que torna a verificação de conformidade um processo complexo. 
Felizmente, a complexidade da verificação conta com o apoio de ferramentas, 
inclusive o *validator_cli.jar*, a ferramenta oficial recomendada pela HL7 para tal operação.

Abaixo segue um exemplo de como esta ferramenta pode ser empregada:
```
java -jar validator_cli.jar -version 4.0.1 -output output.json input.json
```

produz uma instância do recurso [OperationOutcome](https://hl7.org/fhir/R4/operationoutcome.html), depositada no arquivo *output.json*, contendo o resultado da avaliação da instância contida no arquivo *input.json*. Para mais informações sobre o uso do validador, consulte a [documentação](https://confluence.hl7.org/display/FHIR/Using+the+FHIR+Validator) disponível.

Em essência, o objetivo deste projeto é criar uma solução que receba múltiplas entradas (instâncias) de recursos a serem validadas, bem como as expectativas correspondentes, a solução requisita a validação de cada uma delas, recupera os resultados, compara com as expectativas também fornecidas, e apresenta
os resultados em formato de "fácil consumo" para o usuário final.

## Visão geral

FUT é uma ferramenta projetada para simplificar e automatizar o processo de validação de 
instâncias de recursos FHIR. Um caso de uso típico é o suporte ao desenvolvimento de Guias de Implementação. Neste cenário, o desenvolvedor de um guia de implementação 
define casos de teste em conformidade com os perfis fornecidos (artefatos de conformidade) e verifica se instâncias de recursos atendem tais artefatos. A ferramenta é inspirada no conceito do JUnit, mas adaptada ao contexto do FHIR.

## Componentes

1. Casos de teste: unidades de teste individuais, contendo a definição do contexto (artefatos de conformidade), a instância a ser validada e os resultados esperados.

2. Máquina de execução de testes: o componente responsável por requisitar a execução dos testes, tarefa a ser realizada pelo validador FHIR oficial (veja [documentação](https://confluence.hl7.org/display/FHIR/Using+the+FHIR+Validator)).

3. Comparador de resultados: o componente que compara os resultados obtidos, fornecidos pelo validador, com os resultados esperados.

4. Gerador de relatórios: o componente que gera relatórios detalhados e visualmente atrativos dos resultados dos testes (comparação do esperado com o obtido).

## 1. Definição de caso de teste

Um caso de teste pode ser definido por meio de um arquivo YAML, por exemplo,
conforme a estrutura fornecida abaixo:

```yaml
test_id: Patient-001  # Identificador único do caso de teste. 
description: Nome obrigatório não é fornecido. # Descrição do caso de teste
context:  # Define com o que a instância deve estar em conformidade com
  igs:  # Lista de guias de implementação a serem utilizados
    - br.go.ses.core#0.0.1
  profiles:  # Perfis a serem utilizados na validação
    - https://fhir.saude.go.gov.br/r4/core/StructureDefinition/individuo 
  resources:  # Instâncias adicionais de recursos de conformidade
    - valuesets/my-valueset.json  # Arquivo específico
    - instancias/ # todos os artefatos de conformidade contidos neste diretório
instance_path: instances/patient_example.json  # Instância a ser validada
# Se a instância não é indicada explicitamente, considerar:
# 1. <nome do arquivo yaml>.json
# 2. <test_id>.json

expected_results:  # Resultados esperados
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

