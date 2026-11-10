# Arquitetura — RTX Anthropometric V8.2.5

Fonte: *Relatório Técnico RTX Anthropometric V8.2.5 — Arquitetura Integrada* (Drive, 03/11/2026).

## Componentes

| Componente | Responsabilidade |
| :--- | :--- |
| 3× Orbbec Femto Mega | Aquisição RGB-D sincronizada do corpo |
| RTX_SuperResolution (CUDA / RTX 3060) | Fusão temporal de vários quadros, redução de ruído e aumento da resolução da profundidade |
| RTX Anthropometric V8.2.5 | Registro e fusão multivista, reconstrução 3D, reparo topológico, **cálculo do volume** |
| RTX Mesh Data Layer (ply-js) | Estrutura de malha, serialização PLY binária, LODs, compressão, transporte e armazenamento |
| AllBodyScan biomédico | Densidade corporal e %GC com correção pulmonar e coeficientes por sexo/etnia |
| Balança de precisão | Massa corporal medida (Mg) |

## Fluxo de dados

1. Captura RGB-D (3 vistas)
2. Super-resolução temporal
3. Fusão e reconstrução 3D
4. Malha científica (LOD 0)
5. Geração dos LODs 1, 2 e 3
6. Armazenamento, transporte e uso (banco de malhas, visualizadores web/desktop, APIs, relatórios antropométricos)

## Níveis de detalhe (LOD)

Somente o **LOD 0** é usado para volume e %GC; os outros são derivados para visualização e rede
e nunca alteram a malha científica.

| LOD | Nome | Uso | Simplificação | Triângulos (exemplo p/ 1 M) |
| :--- | :--- | :--- | :--- | :--- |
| 0 | Científico | Medições, volume, %GC | nenhuma | 1.000.000 |
| 1 | Alta qualidade | Revisão técnica, análise qualitativa | −25 a −40 % | 600 k – 750 k |
| 2 | Intermediário | Visualização interativa, rede | −60 a −80 % | 200 k – 400 k |
| 3 | Leve | Pré-visualização, celular, catálogos, teleatendimento | −85 a −95 % | 50 k – 150 k |

## Pilha de software proposta

- **Python:** aquisição, sincronização, calibração, CLI, integração.
- **CUDA:** pré-processamento, alinhamento temporal, fusão ponderada, regularização, solver iterativo, profundidade → XYZ.
- **Validação:** erros geométricos, estabilidade de volume, perfis de desempenho.
- Regra: a V8.2.5 permanece **inalterada** como referência funcional até a nova cadeia ser validada.

## Dados de exemplo no Drive

- `coca_UP.ply` (26 MB), `coca_LOW.ply` (44 MB) — nuvens/malhas de teste (objeto "coca", vistas superior e inferior).
- `Brenda2.ply` (8,8 MB) — malha corporal de exemplo.
- `ply-js-main.zip` — biblioteca usada pela Mesh Data Layer.
- `RTX_Anthropometric.zip` (35 MB) — código da versão RTX Anthropometric (não versionado aqui).
