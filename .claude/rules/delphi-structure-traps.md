# Armadilhas Estruturais em Refatoração Delphi — Regras Claude

Estas regras existem porque cada item abaixo **já quebrou o build** deste
repositório. Nenhuma delas é questão de estilo: todas produzem erro de
compilação ou dano visível ao operador.

## Posição do `uses` da implementation

O `uses` da `implementation` vem **imediatamente após** a palavra
`implementation`, antes de qualquer método. Inserir um método entre
`implementation` e o `uses` é **erro de sintaxe** (`E2029`), não preferência de
organização — e gera dezenas de erros em cascata que escondem a causa real.

```pascal
// ✅ correto
implementation

uses
  Vcl.Dialogs;

procedure TMinhaClasse.Fazer;
begin
end;
```

```pascal
// ❌ E2029 + cascata
implementation

procedure TMinhaClasse.Fazer;
begin
end;

uses
  Vcl.Dialogs;
```

## Ordem dentro da declaração de classe

Campos vêm antes de métodos e propriedades, **dentro da mesma seção de
visibilidade**. Ao inserir um campo novo numa seção que já tem métodos, o campo
tem de **subir** — não basta anexar no fim da seção. Campo declarado depois de
método na mesma seção é `E2169`.

```pascal
// ✅ correto
private
  FNome: string;
  FIdade: Integer;
  procedure Validar;
```

```pascal
// ❌ E2169
private
  FNome: string;
  procedure Validar;
  FIdade: Integer;
```

## Unit nova entra no `uses` de todos os `.dpr` que a consomem

Não apenas do projeto em que você compilou. Neste repositório há `.dpr` fora do
`groupproj` (`Retaguarda.dpr`), que nunca é coberto por uma compilação de
conveniência. Unit ausente do `.dpr` é `F2613 Unit not found`.

Verificação: `atlas-erp/scripts/verify-changes.ps1` (checagem DPR).

## Rename de componente varre os call sites

Renomear um componente exige varrer **todos** os usos, em `.pas` **e** `.dfm` —
não só a declaração e o handler. Call site órfão é `E2003`.

## Edição byte-safe de fonte Delphi

Fontes `.pas`/`.dfm` deste repositório misturam UTF-8, Windows-1252 e, em alguns
arquivos, BOM legítimo. Para editar por script:

- Use `ReadAllBytes` / `WriteAllBytes`.
- Use `Encoding.GetEncoding(28591)` (Latin1) **apenas como mapa byte↔char 1:1**,
  nunca como declaração de que o arquivo é Latin1.
- **Nunca** `ReadAllText` / `WriteAllText`: eles detectam BOM e ignoram o
  encoding informado, convertendo `U+FFFD` em `?` e apagando o BOM.

Conferência obrigatória após editar: o delta de bytes tem de bater com o
esperado — `0` num movimento puro de linhas, `+7×N` numa troca de identificador
com 7 caracteres a mais.

## Critério de pronto em refatoração estrutural: compilar

Diff limpo, revisão de texto e auditoria de linhas **não substituem o
compilador**. Os erros desta página têm em comum que a linha continua presente
no arquivo, apenas no lugar errado — nenhuma auditoria textual detecta isso de
forma confiável.

Portão obrigatório antes de afirmar "pronto":

```powershell
.\scripts\verify-changes.ps1          # no atlas-erp
.\scripts\verify-changes.ps1 -All     # suite inteira
```
