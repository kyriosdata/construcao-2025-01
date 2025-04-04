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

## Ilustração de usos

### Nenhum argumento

**fut**
Executa todos os testes contidos no diretório corrente. Os testes são definidos
por arquivos com a extensão .yaml.

### Indicação específica de teste a ser executado

**fut teste/x.yml y.yml**
Executa o teste registrado no arquivo **x.yml**, contido no diretório **teste**,
e o teste em **y.yml** contido no diretório corrente.

### Vários testes

**fut patient-*.yml**
Executa todos os testes disponíveis no diretório corrente 
cujos arquivos têm como prefixo **patient-** e sufixo **.yml**.

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
description: Nome do paciente não é fornecido. # Descrição do caso de teste
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

**Regras para `instance_path` e `instance`:**

1.  `instance_path`: se fornecida, então indica a instância a ser verificada.
2.  Convenção: se `instance_path` não é fornecida, a ferramenta irá procurar por:
    *   Arquivo com o mesmo nome do arquivo YAML, mas com a extensão `.json`.
    *   Se não encontrado, um arquivo com o nome `test_id` e a extensão `.json` será empregado.
3.  Se `instance_path` não é fornecida e a convenção não resulta na localização de um
arquivo, então a execução do teste falha. 

## 2. Execução de testes

1.  A ferramenta identifica os testes a serem executados.
2.  Para cada teste (arquivo YAML):
    *   Lê o arquivo
    *   Extrai as informações nele contidas
    *   Monta a requisição correspondente para o validador (parâmetros)
    *   Executa o validador (`validator_cli`) com os parâmetros
    *   Captura a resposta fornecida pelo validador
    *   Compara a resposta fornecida pelo validador com as respostas esperadas
3.  A representação visual da comparação é produzida.
4. Convém observar que a execução concorrente dos testes é desejável e segura,
pois testes devem ser, idealmente, independentes.
5. Timeout deve ser implementado por teste.
6. A versão mais recente do `validator_cli` deve ser empregada. Se não disponível
localmente, deve ser obtida.

## 4. Comparação

*   Compara o que foi produzido pelo validador com o que foi especificado em `expected_results`. 
Convém lembrar que a resposta do validador é uma instância de [OperationOutcome](https://www.hl7.org/fhir/R4/operationoutcome.html). 

Observe a sequência de passos seguinte para compreensão. Primeiro, execute o validador via linha de comandos:

- `java -jar validator_cli.jar -version 4.0.1 homem.json -ig perfil.json -output saida.json`

Neste caso a instância **homem.json** será validada. O único artefato de conformidade é o
arquivo **perfil.json** e a resposta, a instância de OperationOutcome será depositada no
arquivo **saida.json**. O arquivo contendo a instância a ser testada e o arquivo contendo 
o perfil estão disponíveis no diretório **fut**. A saída é indicada abaixo:

```json
{
  "resourceType" : "OperationOutcome",
  "text" : {
    "status" : "generated",
    "div" : "<div xmlns=\"http://www.w3.org/1999/xhtml\"><p class=\"res-header-id\"><b>OperationOutcome </b></p><table class=\"grid\"><tr><td><b>Severity</b></td><td><b>Location</b></td><td><b>Code</b></td><td><b>Details</b></td><td><b>Diagnostics</b></td><td><b>Source</b></td></tr><tr><td>error</td><td>Patient.gender</td><td>value</td><td>Value is 'male' but is fixed to 'female' in the profile https://perfil.com/mulher|1.0.0#Patient.gender</td><td/><td>InstanceValidator</td></tr><tr><td>warning</td><td>Patient</td><td>invariant</td><td>Constraint failed: dom-6: 'A resource should have narrative for robust management' (defined in http://hl7.org/fhir/StructureDefinition/DomainResource) (Best Practice Recommendation)</td><td/><td>InstanceValidator</td></tr></table></div>"
  },
  "extension" : [{
    "url" : "http://hl7.org/fhir/StructureDefinition/operationoutcome-file",
    "valueString" : "homem.json"
  }],
  "issue" : [{
    "extension" : [{
      "url" : "http://hl7.org/fhir/StructureDefinition/operationoutcome-issue-line",
      "valueInteger" : 9
    },
    {
      "url" : "http://hl7.org/fhir/StructureDefinition/operationoutcome-issue-col",
      "valueInteger" : 22
    },
    {
      "url" : "http://hl7.org/fhir/StructureDefinition/operationoutcome-issue-source",
      "valueString" : "InstanceValidator"
    },
    {
      "url" : "http://hl7.org/fhir/StructureDefinition/operationoutcome-message-id",
      "valueCode" : "_DT_Fixed_Wrong"
    }],
    "severity" : "error",
    "code" : "value",
    "details" : {
      "text" : "Value is 'male' but is fixed to 'female' in the profile https://perfil.com/mulher|1.0.0#Patient.gender"
    },
    "expression" : ["Patient.gender"]
  },
  {
    "extension" : [{
      "url" : "http://hl7.org/fhir/StructureDefinition/operationoutcome-issue-line",
      "valueInteger" : 1
    },
    {
      "url" : "http://hl7.org/fhir/StructureDefinition/operationoutcome-issue-col",
      "valueInteger" : 4
    },
    {
      "url" : "http://hl7.org/fhir/StructureDefinition/operationoutcome-issue-source",
      "valueString" : "InstanceValidator"
    },
    {
      "url" : "http://hl7.org/fhir/StructureDefinition/operationoutcome-message-id",
      "valueCode" : "http://hl7.org/fhir/StructureDefinition/DomainResource#dom-6"
    }],
    "severity" : "warning",
    "code" : "invariant",
    "details" : {
      "text" : "Constraint failed: dom-6: 'A resource should have narrative for robust management' (defined in http://hl7.org/fhir/StructureDefinition/DomainResource) (Best Practice Recommendation)"
    },
    "expression" : ["Patient"]
  }]
}
```

Observe que o arquivo acima possui um erro e um aviso. A ideia é que este resultado seja verificado com o que é esperado e apresentado o resultado da comparação de forma adequada para o usuário.

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

