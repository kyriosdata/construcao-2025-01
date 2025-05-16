# Construção de Software (2025/1)

Apresenta projetos a serem utilizados nas atividades práticas da disciplina [Construção de Software](docs/plano-construcao-2025-01.pdf) (2025/01)
e permite acompanhar o [desempenho](https://docs.google.com/spreadsheets/d/1I3StF95ieTRd_je2aaprwDr5gfE4BAmdifdjKdcDRBA/edit?usp=sharing).

Os projetos são pertinentes
ao padrão [FHIR](https://hl7.org/fhir/R4). FHIR é um padrão para troca de dados em saúde. Alguns dos elementos relevantes:

- Recursos são "estruturas de dados". Por exemplo, para o registro de um paciente usa-se o recurso Patient. Uma instância serializada em JSON deste recurso pode ser obtida [aqui](https://www.hl7.org/fhir/R4/patient-example.json.html).
- Estas estruturas são representadas em JSON (além de XML e Turtle).
- Conjunto de instâncias de recursos são agrupados em arquivos .tgz (NPM Package).
- Informação em saúde é uma "rede" (grafo) de instâncias de recursos. Uma instância referencia outras. 

Opções:

- FHIR Artifact Analyser ([requisitos](docs/faa.md)) ([design preliminar](docs/faa-design.md))
- FHIR Guard ([requisitos](docs/fg.md)) ([design preliminar](docs/fg-design.md))
- FHIR Unit Test ([requisitos](docs/fut.md)) ([design preliminar](docs/fut-design.md))

# Sprint 6 (07/05 ...)

- Conforme o plano, deveríamos estar hoje, 14/05, na Sprint 6, contudo, o conteúdo dos
repositórios está aquém desta expectativa. Precisamos fazer um realinhamento das
Sprints planejadas da disciplina. E, sobretudo, identificar com clareza as lições
aprendidas, por exemplo, a resposta para a seguinte pergunta pode ajudar a localizar
as lições aprendidas: qual a origem desde "não alinhamento"?

# Sprint 5 (25/04, 30/04 e 09/05)
- (40% da aplicação implementada). Isso não significa
que o código apresentado é definitivo, pois pode exigir
refatoração, por exemplo, visando eliminar débito técnico.
Neste ponto já devem ter sido abordados de forma prática, o
que varia conforme a aplicação, vários temas como projeto
detalhado, especificação de projeto, análise sintática (parsing),
expressões regulares, parametrização (generics), closure,
logging, configuração de software em tempo de execução.
Internacionalização. Técnicas de construção baseadas em
estado e tabelas. Naturalmente, conforme a demanda do trabalho em questão. 

# Sprint 4 (16/04, 18/04 e 23/04)

- Conclusão da arquitetura da aplicação. Os principais 
componentes já devem estar identificados, assim como a 
interação entre eles. Possivelmente, quase toda ela exercitada,
estendendo o objetivo da Sprint 3.

# Sprint 3 (04/04, 09/04 e 11/04)
- Código que exercita a arquitetura proposta para a aplicação.
- Nenhuma funcionalidade é esperada, mas a identificação de componentes e a conexão deve estar disponível e executável por meio de código.

# Sprint 2 (21/03)
- Proposta de plano de construção. Possivelmente conterá:
  - Uma análise dos requisitos (modelos, diagramas conceituais, ...)
  - Arquitetura (grandes componentes da aplicação a ser desenvolvida)
  - Tecnologias: Java, C# ou outra utilizada, bibliotecas, ferramentas como IDE, testes, ...
  - Repositório do projeto (github)
  - Critérios de aceitação (testes, medidas aplicáveis, ...)
  - Estratégia para comunicação entre membros da equipe (discord, ...)

# Sprint 1 (19/03)
- Esclarecimento de dúvidas.
- Identificação do projeto a ser realizado pelo grupo.
- Atualização da planilha (indicação do repositório empregado pelo grupo)

# 26/03
- Todos os elementos necessários para a implementação da aplicação são dominados? Se não,
trabalhar cada um deles neste período. Exemplo: operação para início de processo externo (gestão de processo), download de arquivos, interface gráfica, interface via linha de comandos, IDEs, ferramentas em geral e outros.
- Resultado esperado (1). Refinar o plano, em particular, com a identificação precisa das tecnologias a serem utilizadas.
- Resultado esperado (2). A organização da aplicação, componentes, está definida.
- Resultado esperado (3). Domínio das ferramentas necessárias (bibliotecas e outros).

