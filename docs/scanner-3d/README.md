# Scanner antropométrico 3D (RTX Anthropometric)

Documentação técnica do scanner corporal 3D do LAOB, consolidada a partir da pasta
compartilhada do Google Drive do Prof. Mario Gazziro (conteúdo de 01 a 04/11/2026).

O objetivo do sistema é **medir o volume do corpo humano com uma reconstrução 3D e, junto
com a massa medida numa balança, estimar a densidade corporal e o percentual de gordura**
(método do tipo pletismografia por volume, com a equação de Siri e variantes).

> **Estado (nov/2026):** a versão funcional de referência é a **RTX Anthropometric V8.2.5 CLI**.
> O módulo **RTX_SuperResolution** e a instalação dos **3 Orbbec Femto Mega** no arco são
> **estudos/propostas** — nada foi implementado nem medido ainda; o desenvolvimento aguarda a
> chegada dos sensores.

## Documentos

| Arquivo | Conteúdo |
| :--- | :--- |
| [arquitetura.md](arquitetura.md) | Componentes, fluxo de dados e níveis de detalhe (LOD) da malha |
| [instalacao-sensores.md](instalacao-sensores.md) | Posição dos 3 Femto Mega no arco, sincronismo e alimentação |
| [super-resolucao.md](super-resolucao.md) | Matemática da super-resolução temporal RGB-D e plano de implementação CUDA na RTX 3060 |
| [modelo-biomedico.md](modelo-biomedico.md) | Equações de volume pulmonar, densidade e %GC do AllBodyScan, com exemplo numérico |
| [historico-e-referencias.md](historico-e-referencias.md) | Linha do tempo do projeto (laser 2005 → RGB-D 2014 → 2015 → atual) e inventário do Drive |
| [posicoes_3_femto_mega.csv](posicoes_3_femto_mega.csv) | Coordenadas propostas dos sensores (mm) |

## Resumo em uma figura

```
 3× Orbbec Femto Mega (RGB-D, sincronizados por hub Orbbec, alimentados por PoE)
        │  sequências curtas de quadros por sensor
        ▼
 RTX_SuperResolution (CUDA, RTX 3060)   ← proposta
   fusão temporal · redução de ruído · super-resolução 2×
        ▼
 RTX Anthropometric V8.2.5 (CLI)        ← versão de referência
   registro multivista · reconstrução 3D · reparo da malha · volume V
        ▼
 RTX Mesh Data Layer (ply-js)
   PLY binário · LOD 0..3 · compressão · transporte/armazenamento
        ▼
 AllBodyScan biomédico  ← massa Mg (balança de precisão)
   Vp (pulmão) → D = Mg / (V − Vp) → %GC = A/D − B
```
