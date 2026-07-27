---
name: "Delphi Code Review"
description: "Delphi code review checklist — quality, security, performance, SOLID, memory"
---

# Delphi Code Review — Skill

## Quick Checklist

### 🚨 BLOQUEANTE (deve ser corrigido)
- [ ] SQL parametrizado (nunca concatenado) — risco de SQL Injection
- [ ] `try..finally` para cada recurso alocado — risco de memory leak
- [ ] Um recurso por bloco `try..finally` — múltiplos recursos inseguros
- [ ] Credenciais hardcoded (senha, IP, usuário de BD no fonte)
- [ ] Bloco `except` vazio — engole exceções
- [ ] `const` em parâmetros de interface — quebra ARC, causa leak

### ⚠️ ATENÇÃO (risco relevante)
- [ ] `with` statement — causa ambiguidade, dificulta debug
- [ ] `Break` / `Continue` — usar condição do loop
- [ ] `Exit` fora de guard clause
- [ ] `Real` como tipo de ponto flutuante
- [ ] RecordCount para verificar existência — usar `IsEmpty`
- [ ] Locate em tabelas grandes — usar `FindKey`/`FindNearest`
- [ ] Notação húngara (`sNome`, `iCount`) — usar prefixos F/A/L
- [ ] Variáveis globais de unit — usar `class var`

### 💡 RECOMENDAÇÃO (qualidade)
- [ ] Métodos >30 linhas — extrair em métodos menores
- [ ] Classes >500 linhas — considerar extração (SRP)
- [ ] Magic numbers — usar constantes nomeadas
- [ ] Poliade (4+ parâmetros) — usar DTO/record
- [ ] Lógica de negócio nos Forms — delegar para Services
- [ ] Componentes com nome padrão (Button1, Edit1) — renomear com prefixo
- [ ] SQL espalhado nos Forms — centralizar em units de SQL
- [ ] Código sem documentação de regras de negócio

### Corretude
- [ ] Code does what it's supposed to do
- [ ] Edge cases handled (nil, empty list, zero value)
- [ ] Error handling implemented with specific exceptions
- [ ] No obvious bugs

### Security
- [ ] Parameterized SQL queries (without string concatenation)
- [ ] Validated and sanitized input
- [ ] No hardcoded credentials or passwords
- [ ] No SQL injection via `Format` or concatenation in queries

### Performance
- [ ] No N+1 queries (avoid loop with query inside)
- [ ] No unnecessary loops
- [ ] Large objects released as early as possible
- [ ] `TObjectList` with `OwnsObjects` configured correctly

### Code Quality
- [ ] Self-descriptive names following Pascal Guide
- [ ] DRY — no duplicate code
- [ ] SOLID — principles respected
- [ ] Methods ≤ 20 lines
- [ ] Guard clauses instead of deep nesting

### Memory Management
- [ ] `try/finally` with `Free` for temporary objects
- [ ] Interfaces for automatic reference counting
- [ ] `Assigned()` before accessing references that may be nil
- [ ] Destructor `Destroy` with `override` freeing owned fields
- [ ] No memory leaks in exception paths

### Pascal Nomenclature
- [ ] PascalCase for all identifiers
- [ ] Prefix `T` in classes, `I` in interfaces, `E` in exceptions
- [ ] Prefix `F` in private fields, `A` in parameters, `L` in local variables
- [ ] Units: `Projeto.Camada.Dominio.Funcionalidade.pas`
- [ ] Components: 3-letter prefix (`btn`, `edt`, `lbl`, etc.)

### Tests
- [ ] Unit tests for new code
- [ ] Edge cases tested
- [ ] Readable and maintainable tests

### Documentation
- [ ] XMLDoc for public methods and properties
- [ ] Comments in Portuguese when necessary
- [ ] Do not comment self-explanatory code

## Anti-Patterns to Flag

```pascal
//❌ Magic numbers
if ACustomer.Age > 18 then

//✅ Named constants
const MINIMUM_AGE = 18;
if ACustomer.Age > MINIMUM_AGE then

//❌ with statement
with AQuery do begin
  SQL.Text := '...';
  Open;
end;

//✅ Explicit reference
AQuery.SQL.Text := '...';
AQuery.Open;

//❌ Generic Catch
except
  on E: Exception do ShowMessage(E.Message);

//✅ Specific exceptions
except
  on E: EFDDBEngineException do
    raise EDatabaseException.Create('Falha: ' + E.Message);

//❌ Logic in OnClick
procedure TfrmMain.btnSaveClick(Sender: TObject);
begin
  //50 lines of business logic here
end;

//✅ Delegate for Service
procedure TfrmMain.btnSaveClick(Sender: TObject);
begin
  FService.SaveCustomer(GetFormData);
end;

// ❌ Memory leak
function GetItems: TStringList;
begin
  Result := TStringList.Create;
  LoadItems(Result); //if LoadItems throws exception, leak!
end;

//✅ Safe
function GetItems: TStringList;
begin
  Result := TStringList.Create;
  try
    LoadItems(Result);
  except
    Result.Free;
    raise;
  end;
end;
```

## Review Comments Guide

```
🔴 BLOQUEANTE: Memory leak — objeto não liberado em caso de exception
🔴 BLOQUEANTE: SQL injection — query usando concatenação de string

🟡 SUGESTÃO: Extrair método — este bloco tem 35 linhas
🟡 SUGESTÃO: Usar interface em vez de classe concreta (DIP)

🟢 NIT: Renomear variável 'S' para nome descritivo
🟢 NIT: Preferir guard clause a nesting

❓ PERGUNTA: O que acontece se ACustomer for nil aqui?
❓ PERGUNTA: Este objeto é liberado por quem?
```

## Specific SOLID Checklist

| Principle | Check |
|-----------|-----------|
| **SRP** | Does class have ONE responsibility? Service does not access data? |
| **OCP** | Do new features add classes, not modify existing ones? |
| **LSP** | Does either implementation of the interface work in place of the other? |
| **ISP** | Doesn't interface have methods that implementers don't use? |
| **DIP** | Constructor takes interfaces, not concrete classes? |
