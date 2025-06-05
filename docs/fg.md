# Documentação do FHIR Guard (fg)

## Visão geral
A interface de linha de comando (CLI) `fg` é uma ferramenta para gerenciar 
e executar a aplicação FHIR Guard. Ela fornece uma interface consistente 
e fácil de usar para instalar, atualizar, iniciar, parar e monitorar 
diferentes versões desta aplicação.

> **Nota sobre configuração**: O FHIR Guard não possui mecanismos para configurar parâmetros específicos de cada aplicação. As aplicações são executadas com suas configurações padrão.

> **Nota sobre segurança**: 
> - O FHIR Guard não possui operações restritas ou que demandem credenciais
> - Todas as operações são realizadas com as permissões do usuário que executa o comando
> - Não há níveis de segurança configuráveis
> - Não há políticas de segurança personalizáveis
> - O acesso aos arquivos segue as permissões do sistema operacional

> **Nota sobre integração**:
> - O FHIR Guard é uma ferramenta independente
> - Não há suporte para integração com outras ferramentas
> - Não há APIs ou interfaces para integração externa
> - O sistema opera de forma isolada

## Requisitos de operação
- Sistema operacional suportado (versões mais recentes de): Windows, macOS, Linux
  - Não há requisitos específicos de versão do sistema operacional
- Mínimo de RAM: 2GB
- Armazenamento: 1GB
- Rede: acesso HTTP(S) de saída
  - Suporte a proxy configurável para downloads e atualizações
  - Não há requisitos específicos de configuração de firewall
  - Não há suporte para outros protocolos além de HTTP(S)
  - Não há configurações de timeout ou retentativas de conexão
- JRE: O sistema requer JRE versão 17, que será gerenciado e instalado automaticamente pelo FHIR Guard
  - Nota: O FHIR Guard utiliza sua própria instalação do JDK/JRE e não faz uso de versões do Java eventualmente instaladas no sistema
- Permissões: Apenas permissões de leitura e escrita no diretório de instalação são necessárias
- Dependências: Não há necessidade de bibliotecas nativas ou outras dependências além do JDK

> **Nota sobre o sistema de arquivos**:
> - O sistema utiliza UTF-8 para codificação de caracteres em nomes de arquivos e diretórios
> - Caminhos de arquivo são tratados de forma compatível com cada sistema operacional
> - Recomenda-se evitar caracteres especiais em nomes de arquivos e diretórios
> - O comprimento máximo de caminhos segue as limitações do sistema operacional

> **Nota sobre data, hora e localização**:
> - Todos os timestamps são armazenados em UTC
> - A interface e documentação estão disponíveis apenas em português do Brasil
> - Formatos de data e hora seguem o padrão brasileiro (dd/mm/aaaa)

> **Nota sobre logs**:
> - Os logs são armazenados em arquivos de texto no diretório de instalação
> - O formato dos logs inclui timestamp, nível de log e mensagem
> - Os logs são rotacionados automaticamente quando atingem 10MB
> - São mantidos os últimos 5 arquivos de log
> - O nível de log pode ser configurado através da variável de ambiente FG_LOG_LEVEL

> **Nota sobre performance**:
> - O FHIR Guard não possui mecanismos de configuração de limites de recursos
> - Não há configurações específicas de performance para diferentes versões
> - O sistema não fornece mecanismos de monitoramento de recursos

> **Nota sobre backup e recuperação**:
> - O FHIR Guard não possui mecanismos de backup automático
> - Não há mecanismos de recuperação em caso de falha
> - O sistema não suporta exportação ou importação de configurações

> **Nota sobre tratamento de erros**:
> - O sistema não possui tratamento específico para diferentes tipos de erro
> - Os erros são apenas registrados nos logs
> - A execução continua após o registro do erro
> - Não há mecanismos de recuperação ou retry automático

## Opções globais
- `--dir string`: especifica o diretório de trabalho (também pode ser definido através da variável de ambiente `FG_HOME`).
  - Se `FG_HOME` não estiver definida e `--dir` não for um parâmetro fornecido, o diretório padrão será o diretório `.fg` no diretório *home* do usuário (será criado, se necessário).
- `--help, -h`: Exibe informações de ajuda para qualquer comando.

### Variáveis de ambiente

| Variável de Ambiente | Descrição |
|---------------------|-----------|
| `FG_HOME` | Define o diretório de trabalho padrão. Se não definida, o diretório padrão será `.fg` no diretório *home* do usuário. |
| `FG_LOG_LEVEL` | Define o nível de log. Valores aceitos: `debug`, `info`, `warn`, `error`. |

> **Nota sobre variáveis de ambiente**: O FHIR Guard suporta apenas as variáveis de ambiente listadas acima. Não há mecanismos adicionais para configurar o ambiente de execução das aplicações.

O diretório home do usuário pode ser identificado usando os comandos abaixo:

| Sistema Operacional | Comando para Exibir o Diretório Home do Usuário |
|------------------|------------------------------------------|
| **Linux**        | `echo $HOME`                             |
| **macOS**        | `echo $HOME`                             |
| **Windows**      | `echo %USERPROFILE%`                     |

## Comandos

### Comandos básicos

#### `fg available`
- Lista todas as versões disponíveis, independentemente do que está instalado localmente. Depende de acesso à internet para recuperação de arquivo contendo a descrição das versões disponíveis.
- Mostra as versões e, para cada uma delas, a data de lançamento.

Exemplo de saída:

```plaintext
Versão      Data        JDK
------      ----------  ---
2.0.0       2024-04-05  24
1.2.0       2024-03-10  17
1.1.0       2024-02-22  11
1.0.0       2024-01-15  8
```

Para esta saída acima, um documento JSON parcial, ilustrando
apenas parte das duas primeiras versões disponíveis seria:

```plaintext
{
  "versoes" : [
    {
      "versao" : "2.0.0",
      "data" : "2024-04-05"
    },
    {
      "versao" : "1.2.0",
      "data" : "2024-03-10"
    }
  ]
}
```

### Gerenciamento de instalação

A aplicação FHIR Guard está disponível em muitas versões. 
Cada versão é uma coleção de aplicações Java e arquivos, cada um com um ciclo de vida independente 
e sua própria versão. Cada aplicação é composta por arquivos jar e outros. Em particular, 
pode ser executada pelo JDK indicado em cada versão.

> **Nota sobre dados**: Cada versão do FHIR Guard mantém seus próprios dados quando necessário. Não há necessidade de migração de dados entre versões, pois cada versão opera de forma independente.

#### `fg install [versão]`
- Instala uma versão específica da aplicação FHIR Guard.
- Suporta apenas versionamento semântico: `x.y.z` (ex: `1.0.0`, `2.1.3`).
- Saída de sucesso (em verde): `Versão [versão] instalada com sucesso`.
- Saída de erro (em vermelho): `Falha ao instalar versão [versão]: [motivo do erro]`.

#### `fg update`
- Verifica se existe uma versão mais recente do que a mais recente em `$FG_HOME/`.
- Se uma versão mais recente existir, baixa, instala e define como a versão padrão atual.
- Exibe:
  - Saída de sucesso (em verde): `Atualizado para versão [nova versão]. Esta é agora a versão padrão.`
  - Nenhuma atualização disponível: `Nenhuma versão mais recente disponível. Você tem disponível a versão mais recente: [versão atual].`
  - Saída de erro (em vermelho): `Falha ao atualizar: [motivo do erro]`

> **Nota sobre atualizações**: 
> - Conectividade com a internet é necessária para o processo de atualização verificar e baixar novas versões
> - O sistema não verifica compatibilidade antes de atualizar
> - Não há mecanismo de rollback em caso de falha na atualização
> - Recomenda-se manter uma cópia da versão anterior caso seja necessário retornar a ela
> - Atualizações devem ser iniciadas manualmente através do comando `fg update`
> - Não há verificações de integridade das atualizações

#### `fg uninstall [versão]`
- Remove uma versão específica da aplicação.
- Requer confirmação: `"Confirmar desinstalação da versão [versão]? (s/N)"`
- Não pode desinstalar uma versão em execução
- Saída de sucesso (em verde): `"Versão [versão] desinstalada com sucesso"`
- Saída de erro (em vermelho): `"Falha ao desinstalar versão [versão]: [motivo do erro]"`

> **Nota sobre desinstalação**:
> - A desinstalação é sempre manual e requer confirmação do usuário
> - Não há suporte para desinstalação silenciosa ou automática
> - O processo simplesmente remove os arquivos da versão especificada
> - Não há verificações adicionais após a remoção dos arquivos

#### `fg list`
- Mostra todas as versões instaladas da aplicação.
- A versão instalada mais recente é automaticamente definida como padrão e marcada com um asterisco (*).

Exemplo de saída:
  ```
```

## Primeiros passos
Referência rápida para operações comuns:

1. Instalar a versão mais recente (caso já não esteja disponível): `fg update`
2. Iniciar a aplicação: `fg start [nome]`
3. Verificar status: `fg status`
4. Visualizar logs: `fg logs [nome]`
5. Parar aplicação: `fg stop [nome]`