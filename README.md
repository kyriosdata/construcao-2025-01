# Construção de Software (2025/1)

Apresenta projetos a serem utilizados pelas atividades práticas da disciplina Construção de Software (2025/01). Todos eles pertinentes
ao padrão [FHIR](https://hl7.org/fhir/R4). FHIR é um padrão para troca de dados em saúde. Alguns dos elementos relevantes:

- Recursos são "estruturas de dados". Por exemplo, para o registro de um paciente usa-se o recurso Patient. Uma instância serializada em JSON deste recurso pode ser obtida [aqui](https://www.hl7.org/fhir/R4/patient-example.json.html).
- Estas estruturas são representadas em JSON (além de XML e Turtle).
- Conjunto de instâncias de recursos são agrupados em arquivos .tgz (NPM Package).
- Informação em saúde é uma "rede" (grafo) de instâncias de recursos. Uma instância referencia outras. 

Opções:

- [FHIR Artifact Analyser](faa.md)
- [FHIR Guard](fg.md)
- [FHIR Unit Test](fut.md)
