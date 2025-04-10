# FHIR Artifact Analyzer

## 1. Visão geral
O **FHIR Artifact Analyzer** é uma ferramenta para identificar, validar e facilitar a consulta acerca de artefatos do padrão FHIR (versão 4.0.1). Estes artefatos podem estar disponibilizadas em uma entrada definida por um conjunto formado por guias de implementação (arquivos .tgz), diretórios e inclusive arquivos, quaisquer um deles disponível localmente ou não.

As funcionalidas oferecidas devesão ser exploradas por meio de duas interfaces, uma via linha de comandos e outra via interface gráfica (web).

_Contexto_. O uso do FHIR é registrado por artefatos em um Guia de Implementação, geralmente um NPM Package (.tgz). Também é possível
que tais artefatos sejam fornecidos em um diretório contendo os arquivos JSON ou até identificados unicamente, arquivo por arquivo. Cada arquivo, 
seja identificado individualmente, contido em um diretório ou arquivo .tgz define um recurso FHIR. Adicionalmente, instâncias de recursos FHIR formam um grafo, 
pois há várias formas de estabelecer [relações](https://www.hl7.org/fhir/R4/references.html) entre elas. Nesse contexto é "fácil se perder", pois são muitos recursos, 
dos mais variados tipos e relações entre eles. Este projeto visa criar uma ferramenta
para facilitar a navegação entre estes artefatos tanto para integradores (consumidores)
quanto para aqueles responsáveis pela criação de perfis FHIR.

O padrão FHIR, versão 4.0.1, está disponível [AQUI](https://hl7.org/fhir/R4/index.html).

## 2. Objetivos
- **Identificar**: localizar todos os artefatos FHIR contidos na entrada fornecida, como guias de implementação, pacotes `.tgz`, arquivos JSON e outras formas.
- **Validar**: Verificar URLs canônicas e conformidade com o padrão FHIR 4.0.1, inclusive duplicidades e outros. 
- **Organizar**: indexar os artefatos para busca eficiente e geração de grafos dirigidos, além de dados estatísticos.
- **Visualizar**: fornecer uma interface via linha de comandos e outra web para explorar os artefatos e suas relações.
- **Exportar**: permitir a exportação dos resultados em formatos JSON e CSV e PNG (grafos).

## 3. Funcionalidades principais

### 3.1 Identificação da entrada
- A entrada pode ser fornecida por meio de:
  - Pacotes `.tgz` (NPM Package), por exemplo, `http://hl7.org/fhir/us/core/package.tgz`.
  - Arquivos .zip contendo artefatos, seja um arquivo disponível localmente ou acessível via URL de acesso público.
  - Diretórios (e possivelmente subdiretórios). Por exemplo, pode-se indicar um diretório e, adicionalmente, que os subdiretórios deste diretório devem ser consultados.
  - Arquivos individuais identificados, por exemplo, *x.json* e *c:\teste\y.json*. Arquivos podem ser locais ou estarem disponíveis remotamente.
  - Quando arquivos forem fornecidos como entrada, deve ser possível fornecer expressões regulares para indicar os arquivos relevantes para serem considerados.
- Estes itens de entrada identificam arquivos cujos conteúdos são instâncias de recursos FHIR serializados em JSON. Em particular, os recursos FHIR relevantes estão restritos a: (a) `StructureDefinition`; (b) `CodeSystem`, (c) `ValueSet`, (d) `CapabilityStatement`; (e) `ImplementationGuide`; (f) `OperationDefinition` e (g) `SearchParameter`. Observe que a entrada pode conter instâncias de outros recursos distintos destes considerados relevantes e, neste caso, tais instâncias podem ser simplesmente ignoradas. 
- Arquivos contendo instâncias relevantes deverão ser carregados e do conteúdo extraído metadados para apoiar as demais funcionalidades. Por exemplo, cada arquivo tem um nome, tem o tipo do recurso nele contido, por exemplo, `StructureDefinition` (dentre aqueles identificados acima), e assim por diante.
- A entrada deve ser monitorada periodicamente por mudanças. Na ocorrência de uma mudança, o conteúdo correspondente deve ser recarregado para refletir a atualização no conteúdo. 
  - Caso a entrada seja remota, periodicamente mudanças deverão ser verificadas, provavelmente por meio de uma requisição (HTTP HEAD) com o header `Last-Modified`.
  - Caso a entrada seja local, arquivo ou diretório, `fs.watch` (NodeJS) e `Watchservice` (Java) ilustram bibliotecas que podem ser empregadas, dentre outras opções usando outras linguagens. 

### 3.2 Validação
- URLs Canônicas:
  - Verificar se as URLs canônicas estão acessíveis. Isto pode ser feito via requisição HEAD (http) seguida da verificação de `Content-Type` e `Content-Length`, por exemplo.
- Referências
  - Verificar se referências literais (Reference.reference) contidas (internas), relativas ou absolutas estão acessíveis (incluir validação sintática).
  - Verificar se referências literais (Reference.reference) urn:oid e urn:uuid são sintaticamente válidas e se estão acessíveis.
  - Verificar se referência lógica (Reference.identifier) está acessível (caso seja URL).
- Conformidade com FHIR:
  - Validar a estrutura dos artefatos usando o HL7 FHIR Validator.

### 3.3 Busca e Filtros
- Busca global por nome ou sequência de caracteres a ser encontrada no nome dos artefatos, url canônica, comentários, descrições e ou outros elementos relevantes.
- Filtros por:
  - Tipo de artefato.
  - Status de validação.
  - Referências (artefatos referenciados ou referenciadores).

### 3.4 Visualização de Grafo e estatísticas

- Algumas estatísticas relevantes:
  - Total de artefatos por tipo (recurso)
  - Total de artefatos
  - Total de falhas (conforme validações)
- Exibir as relações entre artefatos em um grafo dirigido. Instâncias de recursos FHIR estão ligadas a outras por meio de vários tipos: (a) canonical; (b) uri e (c) [Reference](https://www.hl7.org/fhir/R4/references.html#Reference). Noutras palavras, nas instâncias dos recursos "relevantes", já identificados, `StructureDefinition` e outros, os elementos destes tipos devem ser utilizados para formar o grafo a ser exibido. 
- Permitir interatividade:
  - Zoom, pan, seleção para destaque de nós (centralizar) e de conexões.
  - Aplicação de filtros no grafo.

### 3.5 Exportação
- Gerar resultados em:
  - **JSON**: estrutura completa dos metadados.
  - **CSV**: dados tabulares para análise simples.
  - **PNG**: grafo exibido

## 4. Arquitetura do Sistema
### 4.1 Diagrama de Arquitetura Geral
```mermaid
graph TD
    A[Entrada de Artefatos] -->|Guias, .tgz, JSON| B[Módulo de Extração]
    B --> C[Módulo de Validação]
    C --> D[Módulo de Indexação]
    D --> E[Módulo de Exportação]
    D --> F[Módulo de API REST]
    F --> G[Interface Web]
    G -->|Upload/Download| F
```

### 4.2 Componentes Principais
1. **Backend**:
   - Linguagem: Java (Spring Boot).
   - Módulos:
     - Extração.
     - Validação.
     - Indexação.
     - Exportação.
     - API REST.
2. **Frontend**:
   - Linguagem: JavaScript/TypeScript.
   - Frameworks:
     - React (interface gráfica).
     - D3.js (visualização de grafos).
3. **Armazenamento**:
   - Local: Arquivos temporários.
   - Nuvem: Banco de dados leve (opcional).

## 5. Fluxo de Processamento
### 5.1 Diagrama de Fluxo
```mermaid
flowchart TD
    Start[Início] --> Input[Entrada de Artefatos]
    Input --> Extraction[Extração de Artefatos]
    Extraction --> Validation[Validação de Artefatos]
    Validation --> Indexing[Indexação de Artefatos]
    Indexing -->|Busca e Filtros| Graph[Construção do Grafo]
    Indexing -->|Exportação| Output[Saída: JSON/CSV]
    Graph --> Web[Interface Web]
    Output --> Web
    Web --> End[Fim]
```

### 5.2 Descrição do Fluxo
1. **Entrada**:
   - O usuário fornece os artefatos (localmente ou via interface web).
2. **Extração**:
   - Identificação e classificação dos artefatos.
3. **Validação**:
   - Verificação de URLs e conformidade com o padrão FHIR.
4. **Indexação**:
   - Organização dos metadados e mapeamento de relações.
5. **Construção do Grafo**:
   - Representação das relações entre artefatos.
6. **Saída**:
   - Geração de arquivos JSON e CSV.
7. **Interface Web**:
   - Exibição interativa dos resultados.


## 6. Requisitos Não Funcionais
- **Desempenho**:
  - Suportar até 10.000 artefatos (~50 MB de dados).
  - Processamento síncrono em tempo real.
- **Internacionalização**:
  - Pronto para múltiplos idiomas (ex.: `en`, `pt`).
- **Portabilidade**:
  - Compatível com execução local e em nuvem.
- **Usabilidade**:
  - Interface intuitiva com feedback visual.



## 7. Estrutura Inicial do Repositório
```plaintext
project-root/
├── backend/
│   ├── extraction/         # Módulo de Extração
│   ├── validation/         # Módulo de Validação
│   ├── indexing/           # Módulo de Indexação
│   ├── export/             # Módulo de Exportação
│   ├── api/                # Módulo de API REST
│   └── config/          # Configurações e arquivos auxiliares
├── frontend/
│   ├── components/         # Componentes React
│   ├── pages/              # Páginas principais
│   ├── services/           # Comunicação com a API REST
│   ├── styles/             # Estilização
│   └── locales/            # Arquivos de idioma
├── docs/                   # Documentação do projeto
├── tests/                  # Testes unitários e de integração
└── docker/                 # Configurações para containerização
```


## 8. Prazos e Milestones
- **Mês 1**:
  - Configuração inicial.
  - Implementação dos módulos de extração e validação.
- **Mês 2**:
  - Implementação dos módulos de indexação, exportação e API REST.
- **Mês 3**:
  - Desenvolvimento da interface web.
  - Testes e refinamentos.


## 9. Contato
Para dúvidas ou sugestões, crie uma *issue* correspondente no presente projeto.


## Onboarding

- Visão geral do FHIR. O que você precisa fazer para colaborar com o projeto (30min).
- Visão geral do projeto (30min).
- Ambientação com FHIR e o projeto.
