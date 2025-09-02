# 🛠️ Como Construir o Tooltip Inteligente Financeiro

> **Passo a passo completo para implementar**

## 📋 Pré-requisitos

### **Dados Necessários:**
- ✅ Tabela de recebimentos com valores e datas
- ✅ Tabela de pagamentos com valores e datas
- ✅ Tabela de calendário configurada
- ✅ Relacionamentos corretos entre tabelas

### **Conhecimentos:**
- DAX básico a intermediário
- Conceitos de Time Intelligence
- Configuração de tooltips no Power BI

---

## 🏗️ ETAPA 1: Medidas Base

### 1.1 Medidas de Recebimentos
```dax
Conta a Receber "RECEBIDO" = 
CALCULATE(
    SUM(fRecebimentos[Valor]),
    fRecebimentos[Status] = "RECEBIDO"
)

Conta a Receber "RECEBIDO" LM = 
CALCULATE(
    [Conta a Receber "RECEBIDO"],
    DATEADD(dCalendario[Data], -1, MONTH)
)
```

### 1.2 Medidas de Pagamentos
```dax
Conta a Pagar "PAGO" = 
CALCULATE(
    SUM(fPagamentos[Valor]),
    fPagamentos[Status] = "PAGO"
)

Conta a Pagar "PAGO" LM = 
CALCULATE(
    [Conta a Pagar "PAGO"],
    DATEADD(dCalendario[Data], -1, MONTH)
)
```

### 1.3 Medida de Lucro
```dax
Lucro = [Conta a Receber "RECEBIDO"] - [Conta a Pagar "PAGO"]
```

---

## 🎯 ETAPA 2: Medidas de Apoio

### 2.1 Formatação do Mês
```dax
Mes TXT = "📅 Referente ao mês: " & SELECTEDVALUE(dCalendario[Mes Abreviado])
```

### 2.2 Indicadores Visuais
```dax
Target TXT = "🎯 Meta"

Actual TXT = "Atual"
```

### 2.3 Ícone Dinâmico
```dax
Meta Icon = 
VAR Lucro = [Lucro]
VAR Up = "👍"
VAR Down = "👎"
RETURN
SWITCH(
    TRUE(),
    Lucro >= 0, Up,
    Lucro < 0, Down
)
```

### 2.4 Cor Condicional
```dax
Cor Formatada = 
VAR Recebido = [Conta a Receber "RECEBIDO"]
VAR Pagamento = [Conta a Pagar "PAGO"]
VAR Resultado = 
    SWITCH(
        TRUE(),
        Recebido >= Pagamento, "#27896C",  // Verde
        Recebido < Pagamento, "#D64550"    // Vermelho
    )
RETURN Resultado
```

---

## 🧠 ETAPA 3: Medida Principal - Resumo Financeiro

```dax
Resumo Financeiro = 

VAR _Recebidos = [Conta a Receber "RECEBIDO"]
VAR _Pagamentos = [Conta a Pagar "PAGO"]
VAR _RecebidosAnterior = [Conta a Receber "RECEBIDO" LM]
VAR _PagamentosAnterior = [Conta a Pagar "PAGO" LM]

-- Saldo atual e anterior
VAR _SaldoAtual = _Recebidos - _Pagamentos
VAR _SaldoAnterior = _RecebidosAnterior - _PagamentosAnterior

-- Variações percentuais
VAR _VariacaoRecebidos = DIVIDE(_Recebidos - _RecebidosAnterior, _RecebidosAnterior)
VAR _VariacaoPagamentos = DIVIDE(_Pagamentos - _PagamentosAnterior, _PagamentosAnterior)
VAR _VariacaoSaldo = DIVIDE(_SaldoAtual - _SaldoAnterior, ABS(_SaldoAnterior))

-- Formatação das variações
VAR _VariacaoRecebidosTexto = FORMAT(_VariacaoRecebidos, "0.0%")
VAR _VariacaoPagamentosTexto = FORMAT(_VariacaoPagamentos, "0.0%")
VAR _VariacaoSaldoTexto = FORMAT(_VariacaoSaldo, "0.0%")

-- Ícones para variações
VAR _IconeRecebidos = 
    SWITCH(
        TRUE(),
        ABS(_VariacaoRecebidos) = 0, "🔁",
        _VariacaoRecebidos > 0, "📈",
        _VariacaoRecebidos < 0, "📉",
        " "
    )

VAR _IconePagamentos = 
    SWITCH(
        TRUE(),
        ABS(_VariacaoPagamentos) = 0, "🔁",
        _VariacaoPagamentos > 0, "📈",
        _VariacaoPagamentos < 0, "📉",
        " "
    )

-- Meses para comparação
VAR _MesAtual = SELECTEDVALUE(dCalendario[Mes Abreviado])
VAR _MesAnterior = CALCULATE(SELECTEDVALUE(dCalendario[Mes Abreviado]), DATEADD(dCalendario[Id Data], -1, MONTH))

-- Análise da situação financeira
VAR _SituacaoFinanceira =
    SWITCH(
        TRUE(),
        -- EXCELENTE: Saldo positivo E melhorou significativamente
        _SaldoAtual > 0 && _VariacaoSaldo > 0.15, "Performance Excepcional",
        
        -- MUITO BOM: Saldo positivo E melhorou
        _SaldoAtual > 0 && _VariacaoSaldo > 0, "Crescimento Sólido",
        
        -- BOM: Saldo positivo mas estável
        _SaldoAtual > 0 && _VariacaoSaldo >= -0.1, "Desempenho Estável",
        
        -- ATENÇÃO: Saldo positivo mas deteriorando
        _SaldoAtual > 0 && _VariacaoSaldo < -0.1, "Revisão Necessária",
        
        -- CRÍTICO: Saldo negativo
        _SaldoAtual <= 0, "Intervenção Urgente",
        
        "Status Indefinido"
    )

-- Mensagens contextuais
VAR _TextoSituacao = 
    SWITCH(
        _SituacaoFinanceira,
        "Performance Excepcional", "🏆 PERFORMANCE EXCEPCIONAL: Fluxo de caixa robusto com crescimento expressivo. Operação demonstra excelência operacional.",
        "Crescimento Sólido", "📊 CRESCIMENTO SÓLIDO: Indicadores financeiros em trajetória ascendente. Gestão eficaz de receitas e despesas.",
        "Desempenho Estável", "⚖️ DESEMPENHO ESTÁVEL: Equilíbrio financeiro mantido. Operação dentro dos parâmetros estabelecidos.",
        "Revisão Necessária", "🔍 REVISÃO NECESSÁRIA: Margem operacional em declínio. Recomenda-se análise estratégica dos custos.",
        "Intervenção Urgente", "🚨 INTERVENÇÃO URGENTE: Fluxo de caixa negativo detectado. Ação executiva imediata requerida.",
        "📈 Análise em Processamento"
    )

-- Recomendações estratégicas baseadas na situação
VAR _RecomendacaoEstrategica =
    SWITCH(
        _SituacaoFinanceira,
        "Performance Excepcional", "💼 ESTRATÉGIA: Oportunidade para expansão ou constituição de reservas estratégicas.",
        "Crescimento Sólido", "📈 ESTRATÉGIA: Momento adequado para investimentos planejados e otimização de portfólio.",
        "Desempenho Estável", "⚡ ESTRATÉGIA: Manter disciplina operacional e explorar oportunidades de eficiência.",
        "Revisão Necessária", "🎯 ESTRATÉGIA: Implementar revisão de custos e reavaliar estrutura de despesas.",
        "Intervenção Urgente", "⚠️ ESTRATÉGIA: Ativar plano de contingência e acelerar ciclo de recebimentos.",
        ""
    )

RETURN
_TextoSituacao & UNICHAR(10) & UNICHAR(10) &

"💰 RECEBIDO: " & FORMAT(_Recebidos, "R$ #,##0.00") & " " & _IconeRecebidos & " " & _VariacaoRecebidosTexto & 
" (" & _MesAtual & " → " & _MesAnterior & ")" & UNICHAR(10) &

"💸 PAGAMENTO: " & FORMAT(_Pagamentos, "R$ #,##0.00") & " " & _IconePagamentos & " " & _VariacaoPagamentosTexto & 
" (" & _MesAtual & " → " & _MesAnterior & ")" & UNICHAR(10) & UNICHAR(10) &

"📈 LUCRO LÍQUIDO: " & FORMAT(_SaldoAtual, "R$ #,##0.00") & UNICHAR(10) &
"📊 EVOLUÇÃO DO SALDO (" & _MesAnterior & " → " & _MesAtual & "): " & _VariacaoSaldoTexto & UNICHAR(10) &
"💡 SALDO ANTERIOR: " & FORMAT(_SaldoAnterior, "R$ #,##0.00") & UNICHAR(10) & UNICHAR(10) &

_RecomendacaoEstrategica
```

---

## 🎨 ETAPA 4: Configuração Visual

### 4.1 Criar o Gauge Chart
1. **Adicionar visual:** Gauge Chart
2. **Configurar valor:** Medida principal (ex: `[Lucro]`)
3. **Definir mínimo/máximo:** Baseado nos seus dados
4. **Aplicar formatação condicional:** Use `[Cor Formatada]`

### 4.2 Configurar Tooltip
1. **Selecionar o Gauge**
2. **Ir em Visualizações > Tooltip**
3. **Adicionar campos ao tooltip:**
   - `[Mes TXT]`
   - `[Target TXT]`
   - `[Resumo Financeiro]`
   - `[Actual TXT]`
   - `[Meta Icon]`

### 4.3 Configurar Formatação
```
🎯 Título: "Comparativo: Recebido vs Pagamento"
📊 Subtítulo: Valores e análise contextual
🎨 Cores: Verde (#27896C) / Vermelho (#D64550)
📝 Fonte: Segoe UI ou similar
```

---

## ⚙️ ETAPA 5: Customizações Avançadas

### 5.1 Ajustar Thresholds
Modifique os valores na medida principal:
```dax
-- Personalizar limites de classificação
_SaldoAtual > 0 && _VariacaoSaldo > 0.20,  // Mudou de 0.15 para 0.20
_SaldoAtual > 0 && _VariacaoSaldo >= -0.15, // Mudou de -0.1 para -0.15
```

### 5.2 Personalizar Mensagens
Ajuste as mensagens para seu contexto empresarial:
```dax
"Performance Excepcional", "🚀 CRESCIMENTO EXPLOSIVO: [sua mensagem customizada]",
```

### 5.3 Adicionar Novos Cenários
```dax
-- Adicionar novos níveis de classificação
_SaldoAtual > 0 && _VariacaoSaldo > 0.30, "Crescimento Excepcional",
_SaldoAtual > 0 && _VariacaoSaldo > 0.15, "Performance Excepcional",
```

---

## ✅ ETAPA 6: Testes e Validação

### 6.1 Cenários de Teste
- **📈 Crescimento:** Receitas crescendo, pagamentos controlados
- **📉 Declínio:** Receitas caindo, pagamentos aumentando
- **🔁 Estabilidade:** Variações mínimas
- **🚨 Crise:** Saldo negativo
- **🎯 Recuperação:** Saindo do negativo

### 6.2 Validações
- [ ] Cálculos matemáticos corretos
- [ ] Formatação de valores adequada
- [ ] Ícones aparecem corretamente
- [ ] Mensagens fazem sentido para o contexto
- [ ] Tooltip renderiza sem erros
- [ ] Performance adequada (< 3 segundos)

---

## 🚀 ETAPA 7: Próximos Passos

### 7.1 Evoluções Possíveis
- Adicionar comparação com budget/meta
- Incluir projeções futuras
- Integrar análise de sazonalidade
- Criar versões para diferentes departamentos

### 7.2 Implementação em Escala
- Replicar para outras métricas
- Criar biblioteca de tooltips inteligentes
- Padronizar para diferentes usuários
- Automatizar atualizações

---

## 🔧 Soluções de Problemas

### Problemas Comuns:
1. **Tooltip não aparece:** Verificar se está habilitado nas configurações
2. **Valores em branco:** Checar relacionamentos e filtros
3. **Performance lenta:** Otimizar medidas DAX
4. **Formatação quebrada:** Verificar aspas duplas no código
5. **Datas incorretas:** Validar configuração da tabela calendário

### Dicas de Performance:
- Use `VAR` para reutilizar cálculos
- Evite medidas complexas dentro de `SWITCH`
- Teste com datasets menores primeiro
- Use `DIVIDE` para evitar erros de divisão por zero

---

**🎯 Resultado Final:** Um tooltip que não apenas mostra dados, mas conta uma história, sugere ações e facilita decisões executivas.
