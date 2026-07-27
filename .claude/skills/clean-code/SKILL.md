---
name: "Delphi Clean Code"
description: "Pragmatic clean code standards for Delphi — concise, direct, no over-engineering"
---

# Delphi Clean Code — Skill

> **CRITICAL SKILL** — Be **concise, direct and solution-focused**.

## Fundamental Principles

| Principle | Rule |
|-----------|-------|
| **SRP** | A function/class does ONE thing |
| **DRY** | Don't repeat code — extract and reuse |
| **KISS** | Simplest solution that works |
| **YAGNI** | Don't build what wasn't asked for |
| **Boy Scout** | Leave the code better than you found it |

## Naming Rules (Delphi Style Guide)

| Element | Prefix | Convention | Example |
|----------|--------|-----------|---------|
| **Fields** | `F` | PascalCase | `FCustomerName`, `FValorTotal` |
| **Parameters** | `A` | PascalCase | `ACustomerName`, `AValor` |
| **Local vars** | `L` | PascalCase, reveal intent | `LCustomerCount` not `N` |
| **Constants** | `C_` | UPPER_SNAKE | `C_MAX_RETRY_COUNT`, `C_SQL_PEDIDOS` |
| **Classes/Types** | `T` | PascalCase | `TCliente`, `TPedidoService` |
| **Interfaces** | `I` | PascalCase | `IClienteService`, `IRepository` |
| **Exceptions** | `E` | PascalCase | `EClienteNaoEncontrado` |
| **Methods** | (verb) | Verb + noun | `CalcularICMS`, `GetCustomerById`, `ValidarCPF` |
| **Booleans** | — | Question form | `IsActive`, `HasPermission`, `CanEdit` |

**Proibido:** notação húngara (`sNome`, `iCount`), underline em identificadores (exceto `C_`), `p` como prefixo de parâmetro.

> **Rule:** If you need a comment to explain a name, rename it.

## Method Rules

| Rule | Description |
|-------|-----------|
| **Short** | Maximum 20 lines, ideal 5-10 |
| **One Thing** | Do one thing and do it well |
| **One Level** | One level of abstraction per method |
| **Few Args** | Maximum 3 arguments, prefer 0-2 |
| **No Side Effects** | Don't mute inputs unexpectedly |

## Code Structure

| Standard | Application |
|--------|-----------|
| **Guard Clauses** | Early returns for edge cases |
| **Flat > Nested** | Avoid deep nesting (max 2 levels) |
| **Composition** | Small compound methods |
| **Colocation** | Related code together |

### Guard Clauses in Delphi

```pascal
//❌ BAD — excessive nesting
procedure ProcessOrder(AOrder: TOrder);
begin
  if Assigned(AOrder) then
  begin
    if AOrder.Items.Count > 0 then
    begin
      if AOrder.IsValid then
      begin
        //real logic here
      end;
    end;
  end;
end;

//✅ BOM — guard clauses
procedure ProcessOrder(AOrder: TOrder);
begin
  if not Assigned(AOrder) then
    raise EArgumentNilException.Create('AOrder cannot be nil');
  if AOrder.Items.Count = 0 then
    raise EBusinessRuleException.Create('Order must have items');
  if not AOrder.IsValid then
    raise EValidationException.Create('Order validation failed');

  //real logic here — no nesting
end;
```

## Anti-Patterns (DO NOT DO)

| ❌ Pattern | ✅ Fix |
|-----------|------------|
| Comment each line | Delete obvious comments |
| Method > 20 lines | Share by responsibility |
| Magic numbers | Named constants |
| `with` statement | Explicit local variables |
| Global variables | Constructor injection |
| Generic Catch | Specific exceptions |
| Logic in `OnClick` | Delegate to Service |
| God class / God unity | One class = one responsibility |
| Ignore `Free` | `try/finally` always |

## Memory Management

```pascal
//✅ Temporary objects — always try/finally
LList := TStringList.Create;
try
  LList.Add('item');
  //use LList
finally
  LList.Free;
end;

//✅ Interfaces — reference counting automático
var LService: IMyService;
LService := TMyService.Create; //automatically released

//✅ Owner pattern for visual components
LButton := TButton.Create(Self); //Self frees automatically
```

## AI Code Style

| Situation | Action |
|----------|------|
| User requests feature | Write directly |
| User reports bug | Correct, don't explain |
| Requirement unclear | Ask, don't assume |

## 🔴 Before Editing (THINK FIRST!)

| Question | Why |
|----------|---------|
| **Which units use this?** | They can break |
| **What does this unit matter?** | Interfaces can change |
| **What tests cover this?** | Tests may fail |
| **Is it a shared component?** | Multiple points affected |

> 🔴 **Rule:** Edit the file + all dependents in the SAME task.

## 🔴 Self-Check (MANDATORY)

| Check | Pergunta |
|-------|----------|
| ✅ **Goal achieved?** | Did I do exactly what was asked? |
| ✅ **Edited files?** | Have I modified everything necessary? |
| ✅ **Does the code work?** | Have I tested/verified? |
| ✅ **No errors?** | Compiles without warnings? |
| ✅ **Nothing forgotten?** | Edge cases treated? |
| ✅ **Memory safe?** | Objects released correctly? |

> 🔴 **Rule:** If ANY check fails, correct it before finishing.
