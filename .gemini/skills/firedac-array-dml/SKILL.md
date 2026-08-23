---
name: firedac-array-dml
description: Expert guide and reference for high-performance batch data processing, bulk inserts, upserts, and ETL data synchronization in Delphi using FireDAC Array DML (Params.ArraySize, AsIntegers[i], AsStrings[i], AsFloats[i], Execute(LBatchSize, 0)). Use when writing data sync routines, migrating datasets, optimizing slow while not Eof do ExecSQL loops, or implementing low-latency database batch operations for MySQL, SQLite, PostgreSQL, Firebird, and SQL Server.
---
`
# FireDAC Array DML & High-Performance Batch Processing
`
Guia definitivo para processamento em lote de alta performance e sincronizacao de dados volumosos no Delphi utilizando FireDAC Array DML e ETL nativo.
`
## 1. Por que usar FireDAC Array DML?
`
Em operacoes convencionais de banco de dados, loops linha a linha geram um gargalo severo de roundtrips de rede e disk-syncs.
`
Com o FireDAC Array DML, os parametros sao enviados em blocos de memoria contigua diretamente para a API nativa do banco:
- Reducao de 99% no trafego de rede (WAN/LAN): 5.000 registros sao transferidos em apenas 10 pacotes binarios (lotes de 500).
- Eliminacao do overhead de parse de SQL: O banco de dados compila o comando SQL uma unica vez e consome a matriz de dados em pipeline.
`
## 2. Units Obrigatorias no uses
`
Para que o compilador Delphi reconheca as propriedades de matriz (AsIntegers[i], AsStrings[i], AsFloats[i], etc.), inclua no uses:
`
``pascal
uses
  FireDAC.Comp.Client,
  FireDAC.Stan.Param,
  FireDAC.DatS,
  FireDAC.DApt;
`
`
## 3. Padrao Canonico de Implementacao
`
``pascal
// {ATLAS-CUSTOM-START} [YYYY-MM-DD] [NomeDaRotina-FireDAC-ArrayDML]
const
  C_BATCH_SIZE = 500; // Tamanho ideal de lote para balancear memoria e throughput
`
procedure SincronizarTabela(AConexao: TFDConnection; AQryOrigem: TDataSet);
var
  LQryExec: TFDQuery;
  LBatchCount: Integer;
begin
  if AQryOrigem.IsEmpty then
    Exit;
`
  LQryExec := TFDQuery.Create(nil);
  try
    LQryExec.Connection := AConexao;
    
    // Comando parametrizado de destino
    LQryExec.SQL.Text :=
      'INSERT INTO cliente (codigo, nome, cnpj, limite) ' +
      'VALUES (:codigo, :nome, :cnpj, :limite) ' +
      'ON DUPLICATE KEY UPDATE ' +
      'nome = VALUES(nome), cnpj = VALUES(cnpj), limite = VALUES(limite)';
`
    if not AConexao.InTransaction then
      AConexao.StartTransaction;
    try
      AQryOrigem.First;
      while not AQryOrigem.Eof do
      begin
        LBatchCount := 0;
        LQryExec.Params.ArraySize := C_BATCH_SIZE;
`
        while (not AQryOrigem.Eof) and (LBatchCount < C_BATCH_SIZE) do
        begin
          // Atribuicao direta por indice sem Variant / RTTI
          LQryExec.ParamByName('codigo').AsIntegers[LBatchCount] := AQryOrigem.FieldByName('CODIGO').AsInteger;
          LQryExec.ParamByName('nome').AsStrings[LBatchCount]    := Copy(AQryOrigem.FieldByName('NOME').AsString, 1, 60);
          LQryExec.ParamByName('cnpj').AsStrings[LBatchCount]    := Copy(AQryOrigem.FieldByName('CNPJ').AsString, 1, 20);
          LQryExec.ParamByName('limite').AsFloats[LBatchCount]   := AQryOrigem.FieldByName('LIMITE').AsFloat;
`
          Inc(LBatchCount);
          AQryOrigem.Next;
        end;
`
        // Executa o lote inteiro em 1 chamada binaria
        if LBatchCount > 0 then
          LQryExec.Execute(LBatchCount, 0);
      end;
`
      if AConexao.InTransaction then
        AConexao.Commit;
    except
      on E: Exception do
      begin
        if AConexao.InTransaction then
          AConexao.Rollback;
        raise;
      end;
    end;
  finally
    LQryExec.Free;
  end;
end;
// {ATLAS-CUSTOM-END} [YYYY-MM-DD] [NomeDaRotina-FireDAC-ArrayDML]
`
`
## 4. Sintaxes de Upsert Idempotente por SGDB
`
O Array DML opera em conjunto com instrucoes atomicas de Upsert (Inserir ou Atualizar):
`
### A. MySQL / MariaDB
``sql
INSERT INTO produto (codigo, descricao, pr_venda)
VALUES (:codigo, :descricao, :pr_venda)
ON DUPLICATE KEY UPDATE
  descricao = VALUES(descricao),
  pr_venda  = VALUES(pr_venda)
`
`
### B. SQLite (FMX / Mobile / Desktop Local)
``sql
INSERT OR REPLACE INTO produto (codigo, descricao, pr_venda)
VALUES (:codigo, :descricao, :pr_venda)
`
`
### C. PostgreSQL (9.5+)
``sql
INSERT INTO produto (codigo, descricao, pr_venda)
VALUES (:codigo, :descricao, :pr_venda)
ON CONFLICT (codigo) DO UPDATE
SET descricao = EXCLUDED.descricao,
    pr_venda  = EXCLUDED.pr_venda
`
`
### D. Firebird (2.1+)
``sql
UPDATE OR INSERT INTO produto (codigo, descricao, pr_venda)
VALUES (:codigo, :descricao, :pr_venda)
MATCHING (codigo)
`
`
## 5. Comparativo: Array DML vs. TFDBatchMove
`
| Criterio | FireDAC Array DML | TFDBatchMove |
|---|---|---|
| Controle de Transformacao | Total (codigo Pascal tipado e sanitizacao direta) | Requer eventos OnValueMapping |
| Idempotencia / Upsert | Nativo via SQL customizado do SGDB | Depende de Mode = dmAppendUpdate |
| Performance de Execucao | Maxima absoluta (Zero alocacoes extras) | Equivalente (Usa Array DML internamente) |
| Feedback de Progresso | Incremento granular por lote na UI/Thread | Evento OnProgress |
| Ideal para | Sincronizacoes com regras de negocio, ERP - Mobile | Migracao direta 1:1 de tabelas e arquivos |
`
## 6. Checklist de Boas Praticas
`
- [ ] Tamanho do Lote (ArraySize): Usar entre 100 e 1000 (padrao recomendado: 500).
- [ ] Limitacao de Strings: Aplicar Copy(Str, 1, MaxLen) para evitar estouro de buffer no driver.
- [ ] Tratamento de Nulos: Tratar valores nulos antes da indexacao (ex: if Field.IsNull then ...).
- [ ] Bloco Transacional: Envolver cada lote ou a sincronizacao completa em try..except com Rollback.
- [ ] Threading / Assincronismo: Se executado em background thread (TThread.CreateAnonymousThread), atualizar a UI exclusivamente via TThread.Synchronize ou TThread.Queue.
- [ ] Delimitadores de Customizacao: Envolver blocos customizados com // {ATLAS-CUSTOM-START} e // {ATLAS-CUSTOM-END}.
`