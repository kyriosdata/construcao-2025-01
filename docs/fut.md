FHIRUT - FHIR Unit Test Suite  
Este documento descreve o FUT, uma ferramenta para automatizar testes de conformidade de instâncias de recursos FHIR.

Contexto  
O FHIR é um padrão flexível, o que torna a verificação de conformidade um processo complexo. Felizmente, essa complexidade conta com o apoio de ferramentas, incluindo o validator_cli.jar, a ferramenta oficial recomendada pela HL7 para tal operação.

Abaixo segue um exemplo de como essa ferramenta pode ser empregada:

java -jar validator_cli.jar -version 4.0.1 -output output.json input.json  
produz uma instância do recurso OperationOutcome, depositada no arquivo output.json, contendo o resultado da avaliação da instância contida no arquivo input.json. Para mais informações sobre o uso do validador, consulte a documentação disponível.

Em essência, o objetivo deste projeto é criar uma solução que receba múltiplas entradas (instâncias) de recursos a serem validadas, bem como as expectativas correspondentes. A solução requisita a validação de cada uma delas, recupera os resultados, compara com as expectativas também fornecidas e apresenta os resultados em formato de "fácil consumo" para o usuário final.

Visão geral  
O FUT é uma ferramenta projetada para simplificar e automatizar o processo de validação de instâncias de recursos FHIR. Um caso de uso típico é o suporte ao desenvolvimento de Guias de Implementação. Neste cenário, o desenvolvedor de um guia de implementação define casos de teste em conformidade com os perfis fornecidos (artefatos de conformidade) e verifica se instâncias de recursos atendem a tais artefatos. A ferramenta é inspirada no conceito do JUnit, mas adaptada ao contexto do FHIR.

Ilustração de usos  
Sem argumentos  
fut  
Executa todos os testes contidos no diretório corrente. Os testes são definidos por arquivos com a extensão .yaml.

Indicação específica de teste a ser executado  
fut teste/x.yml y.yml  
Executa o teste registrado no arquivo x.yml, contido no diretório teste, e o teste em y.yml contido no diretório corrente.

Vários testes  
fut patient-*.yml  
Executa todos os testes disponíveis no diretório corrente cujos arquivos têm como prefixo patient- e sufixo .yml.

Componentes  
- Casos de teste: unidades de teste individuais, contendo a definição do contexto (artefatos de conformidade), a instância a ser validada e os resultados esperados.
- Máquina de execução de testes: o componente responsável por requisitar a execução dos testes, tarefa a ser realizada pelo validador FHIR oficial (veja documentação).
- Comparador de resultados: o componente que compara os resultados obtidos, fornecidos pelo validador, com os resultados esperados.
- Gerador de relatórios: o componente que gera relatórios detalhados e visualmente atrativos dos resultados dos testes (comparação do esperado com o obtido).

1. Definição de caso de teste  
Um caso de teste pode ser definido por meio de um arquivo YAML, por exemplo, conforme a estrutura fornecida abaixo:

```yaml
test_id: Patient-001  # Identificador único do caso de teste.
description: Nome do paciente não é fornecido. # Descrição do caso de teste
context:  # Define com o que a instância deve estar em conformidade
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
  status: success  # Nível geral esperado ('success', 'error', 'warning', 'information').
  errors: []  # Lista de erros esperados (lista vazia indica sucesso).
  warnings: []  # Lista de avisos esperados.
  informations: []  # Lista de mensagens informativas esperadas.
  invariants: # Opcional.
    - expression: "OperationOutcome.issues.count() = 0"
      expected: true # Opcional. O padrão é true.
```

Regras para instance_path e instance:  
- instance_path: se fornecida, indica a instância a ser verificada.
- Convenção: se instance_path não é fornecida, a ferramenta irá procurar por:
  - Arquivo com o mesmo nome do arquivo YAML, mas com a extensão .json.
  - Se não encontrado, um arquivo com o nome test_id e a extensão .json será empregado.
  - Se instance_path não é fornecida e a convenção não resulta na localização de um arquivo, então a execução do teste falha.

2. Execução de testes  
A ferramenta identifica os testes a serem executados.  
Para cada teste (arquivo YAML):
- Lê o arquivo
- Extrai as informações nele contidas
- Monta a requisição correspondente para o validador (parâmetros)
- Executa o validador (validator_cli) com os parâmetros
- Captura a resposta fornecida pelo validador
- Compara a resposta fornecida pelo validador com as respostas esperadas
- A representação visual da comparação é produzida.

Convém observar que a execução concorrente dos testes é desejável e segura, pois testes devem ser, idealmente, independentes.  
Timeout deve ser implementado por teste.  
A versão mais recente do validator_cli deve ser empregada. Se não disponível localmente, deve ser obtida.

4. Comparação  
Compara o que foi produzido pelo validador com o que foi especificado em expected_results. Convém lembrar que a resposta do validador é uma instância de OperationOutcome.

Observe a sequência de passos seguinte para compreensão. Primeiro, execute o validador via linha de comando:

java -jar validator_cli.jar -version 4.0.1 homem.json -ig perfil.json -output saida.json  
Neste caso, a instância homem.json será validada. O único artefato de conformidade é o arquivo perfil.json e a resposta, a instância de OperationOutcome, será depositada no arquivo saida.json. O arquivo contendo a instância a ser testada e o arquivo contendo o perfil estão disponíveis no diretório fut. A saída é indicada abaixo:

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

5. Geração de relatórios  
Formato principal: HTML (navegável e visualmente atraente).  
Formato adicional: JSON (para possível integração com outras ferramentas).

Estrutura do relatório HTML (sugestão inicial):

Resumo:
- Estatísticas gerais (testes executados, aprovados/reprovados/avisos, gráficos).
- Tempo de execução por teste, por todos os testes.

Detalhes por Teste:
- test_id, description, contexto.
- Instância (formatada).
- Resultados Esperados.
- Resultados Obtidos (tabela com diagnósticos, localização e severidade do OperationOutcome).
- Diferenças destacadas.
- Saída bruta do validador (opcional).

6. Backend  
Responsável pela requisição de execuções de validação e processamento dos resultados. É esperado que possa se integrar a outros validadores.

Principais responsabilidades:

Carga dos casos de teste:
- Carrega as definições de testes de arquivos YAML.
- Implementa a localização de arquivos de instâncias (instance_path, instance ou convenção de nomes).
- Valida o arquivo YAML (formato compatível com o esperado).
- Lida com erros potenciais de forma adequada (ex: arquivos ausentes, YAML inválido).

Gerenciamento da suíte:
- Carrega definições de suítes (seja por estrutura de diretórios ou arquivos de manifesto).
- Fornece uma interface para acessar casos de teste dentro de uma suíte.
- Permite filtrar e selecionar casos de teste com base em vários critérios (ex: suíte, ID, descrição).

Preparação de contexto:
- Com base no contexto definido no caso de teste:
  - Coleta os Implementation Guides (IGs) necessários.
  - Coleta os perfis relevantes (StructureDefinitions).
  - Carrega quaisquer recursos adicionais especificados (ValueSets, CodeSystems, etc.).
  - Prepara essas informações em um formato adequado para o validador FHIR escolhido.

Execução do validador:
- Interface com o validador FHIR escolhido (ex: validator_cli). Isso deve ser feito por meio de uma camada de abstração (ex: uma classe Validator) para facilitar a troca de validadores no futuro.
- Constrói os argumentos de linha de comando apropriados (ou chamadas de API) para o validador, com base no contexto do caso de teste e da instância.
- Executa o validador, lidando com possíveis erros e timeouts.
- Captura a saída padrão (stdout) e o erro padrão (stderr) do validador.

Processamento de resultados:
- Analisa a saída do validador (que pode estar em JSON, XML ou outro formato, dependendo do validador).
- Extrai informações relevantes:
  - Status geral da validação (success, error, warning).
  - Mensagens individuais (erros, avisos, informações).
  - Localização de cada mensagem (XPath ou FHIRPath).
  - Severidade de cada mensagem.

Comparação de resultados:
- Compara os resultados extraídos com os expected_results definidos no caso de teste.
- Identifica e categoriza discrepâncias:
  - Erros/avisos/informações inesperados.
  - Erros/avisos/informações esperados que não foram encontrados.
  - Incompatibilidades de severidade ou localização.
  - Avaliação de expressões FHIRPath e comparação com o resultado esperado.

Armazenamento de dados:
- Se for necessário persistência (ex: para armazenar resultados de testes ao longo do tempo, ou para uma interface gráfica), o backend lida com o armazenamento e recuperação desses dados. Isso pode ser em:
  - Arquivos simples (JSON, YAML).

Preparação de dados para relatório:
- Prepara todos os dados necessários para o gerador de relatórios, incluindo um relatório estruturado.

Considerações arquiteturais:
- Modularidade: o backend deve ser projetado como um conjunto de módulos independentes (classes ou funções) com responsabilidades claras. Isso facilita a manutenção, testes e extensões.
- Abstração: use camadas de abstração (interfaces, classes abstratas) para desacoplar componentes. Por exemplo, a classe Validator deve abstrair os detalhes específicos de interação com o validator_cli ou qualquer outro validador.
- Tratamento de erros: implemente tratamento robusto de erros em todo o backend para lidar graciosamente com situações inesperadas (ex: entrada inválida, erros do validador, problemas no sistema de arquivos). Forneça mensagens de erro informativas ao usuário.
- Configuração: permita a configuração de vários aspectos do backend, como:
  - Caminho para o executável do FHIR validator.
  - Configurações de timeout.
  - Opções de paralelização.
  - Opções de armazenamento de dados.
- Logging: use uma biblioteca de logging para registrar eventos importantes, informações de depuração e erros. Isso é crucial para troubleshooting.
- Testabilidade: o backend deve ser amplamente testado.
