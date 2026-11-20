# Central de Gestão de Serviços Públicos

Projeto acadêmico em TypeScript: módulo de gestão de solicitações de zeladoria urbana, com integração à API de Localidades do IBGE.

## Contexto de negócio

Uma central municipal recebe solicitações de moradores sobre problemas urbanos, como buracos em vias, iluminação pública, limpeza e drenagem. O módulo organiza o ciclo de atendimento:

1. Abertura da solicitação pelo cidadão/atendente.
2. Triagem: conferência das informações, identificação de duplicidade e classificação de prioridade.
3. Aprovação ou rejeição por um gestor (rejeição exige justificativa).
4. Encaminhamento para equipe responsável.
5. Execução e registro de evidências.
6. Validação da conclusão por fiscal/solicitante.
7. Encerramento ou reabertura justificada.

O projeto é uma **simulação acadêmica**. Não é um sistema oficial da Prefeitura de Porto Alegre e não envia solicitações à prefeitura.

## API pública utilizada

API de Localidades do IBGE, sem autenticação:

- Base URL: `https://servicodados.ibge.gov.br/api/v1/localidades`
- `GET /estados` — lista as Unidades da Federação.
- `GET /estados/43/municipios` — lista municípios do Rio Grande do Sul (43 é o identificador da UF no IBGE).
- `GET /municipios/4314902` — consulta Porto Alegre pelo código IBGE.

Documentação oficial: https://servicodados.ibge.gov.br/api/docs/localidades

### Como a API participa da regra de negócio

Ao abrir uma solicitação, o sistema precisa associá-la a um município reconhecido. O módulo consulta o IBGE e utiliza o código oficial e o nome da localidade. A API não fornece chamados, aprovações, equipes, prazos ou execução: esses dados e regras pertencem ao nosso domínio.

## Escopo implementado inicialmente com apoio de IA

- Modelos tipados de estado e município.
- Serviço de integração com a API do IBGE.
- Tratamento da resposta externa como `unknown` e validação em tempo de execução.
- Configuração TypeScript estrita e source maps.

## Requisitos

- Node.js 18 ou superior (usa `fetch` nativo).
- npm.

## Executar

```bash
npm install
npm run check
npm run build
npm start
```

O programa consulta Porto Alegre no IBGE e cria uma solicitação demonstrativa em memória. Nenhum dado é enviado à prefeitura.

## Estrutura

```text
src/
├── domain/
│   ├── models/
│   │   ├── catalogo-solicitacao.ts
│   │   ├── municipio.ts
│   │   └── solicitacao.ts
│   ├── types/
│   │   └── filtro-solicitacao.ts
|   |   └── fluxo-solicitacao.ts
|   |   └── resultado-condicional.ts
|   |   └── resultado-operacao.ts
│   └── use-cases/
│       └── consultar-solicitacao.ts
│       └── criar-solicitacao.ts
│       └── gerencia-solicitacao.ts
├── infrastructure/
│   └── repositories/
│       └── solicitacao-repository.ts
├── integrations/
│   └── ibge/
│       └── ibge-localidades.service.ts
└── index.ts
```

## 1. Decisões técnicas

Durante o desenvolvimento da Central de Gestão de Serviços Públicos,
foram tomadas decisões técnicas considerando a organização, a manutenção
e a segurança da tipagem do projeto.

Entre as principais decisões estão:
-   Utilização de TypeScript com configuração estrita (`strict: true`),
    evitando tipagens implícitas e aumentando a segurança durante o
    desenvolvimento.
-   Organização do código em camadas de domínio, infraestrutura e
    integrações externas.
-   Utilização de uniões de tipos literais para representar categorias e
    status das solicitações.
-   Utilização de interfaces para definir os contratos dos modelos de
    domínio.
-   Utilização de um repositório em memória para armazenar as
    solicitações durante a execução, mantendo o escopo do projeto
    acadêmico.
-   Integração com a API de Localidades do IBGE para obter os municípios
    utilizados no cadastro.

Também foi adotada a utilização de tipos literais em vez de enums para
representar categorias e status, por serem adequados ao domínio e
permitirem uma modelagem direta das possibilidades aceitas pelo sistema.

## 2. Refatorações realizadas

Durante a implementação, foram identificadas oportunidades de melhoria
na organização e na tipagem do código.

Entre as refatorações realizadas, destacam-se:

-   Substituição da verificação de propriedades com `in` por
    `Object.hasOwn()`, garantindo que somente propriedades próprias do
    catálogo sejam consideradas válidas.
-   Restrição do parâmetro genérico do Conditional Type com
    `extends TipoOperacao`, limitando as operações aos valores
    previstos.
-   Remoção de variável declarada e não utilizada na consulta de
    solicitações.
-   Exclusão do histórico de movimentações dos campos disponíveis para
    filtragem, pois a comparação por referência de arrays não seria
    adequada para essa finalidade.
-   Aproveitamento do método `listarPorStatus()` do repositório,
    evitando duplicação da lógica de filtragem no ponto de entrada da
    aplicação.
-   Melhoria do tratamento de validações no cadastro, utilizando
    `ResultadoOperacao<Solicitacao>` para representar situações de
    sucesso e falha.

Também foi implementado o histórico de movimentações, utilizando tuplas
para registrar o status anterior, o novo status e a data da alteração.

## 3. Problemas de tipagem identificados

Durante a evolução do projeto, foram encontrados problemas relacionados
à relação entre os tipos do domínio e os campos utilizados nos filtros.

A inclusão de `historicoStatus` na interface `Solicitacao`, por exemplo,
exigiu uma revisão do tipo `FiltroSolicitacao` e do catálogo de campos
permitidos.

Outro problema ocorreu quando o catálogo de filtros continuou exigindo
uma propriedade que havia sido removida do tipo filtrável. A
configuração foi ajustada para manter a correspondência entre o Mapped
Type e o catálogo validado com `satisfies`.

Essas situações demonstraram a importância de manter os modelos, os
tipos auxiliares e suas utilizações consistentes.

## 4. Utilização de Inteligência Artificial

A Inteligência Artificial foi utilizada como ferramenta de apoio durante
o desenvolvimento, principalmente para:

-   Sugerir estruturas de tipos e interfaces.
-   Identificar inconsistências de tipagem.
-   Sugerir refatorações.
-   Revisar código.
-   Auxiliar na interpretação das mensagens de erro do compilador.

As sugestões não foram aceitas automaticamente. Foram analisadas
considerando a necessidade real do sistema.

Um exemplo foi a proposta inicial de utilizar uma interseção de tipos
para acrescentar informações de auditoria. A proposta foi questionada
por não representar uma necessidade concreta do projeto e ficou para
avaliação posterior.

Outro exemplo foi a sugestão de excluir o histórico dos campos
filtráveis, pois não faria sentido comparar arrays de movimentações por
referência.

As alterações foram aplicadas gradualmente e verificadas por meio do
compilador TypeScript.

## 5. Validação e evidências

A validação do projeto foi realizada utilizando os seguintes comandos:

-   `npm run check`: verifica os tipos sem gerar arquivos.
-   `npm run build`: compila o projeto TypeScript.

Na versão validada durante a revisão, ambos os comandos foram executados
sem apresentar erros.

Também foram realizados testes manuais pelo terminal, incluindo
cadastro, aprovação, encaminhamento, início da execução, conclusão e
consulta do histórico de movimentações.

## 6. Considerações finais

O desenvolvimento permitiu aplicar recursos estudados durante a disciplina de
TypeScript.

As refatorações foram realizadas buscando melhorar a clareza, a
consistência e a manutenção do código, evitando a inclusão de recursos
apenas para demonstrar conceitos da linguagem.