# Scanner_LAOB — Scanner antropométrico 3D

Scanner corporal 3D do laboratório LAOB. Ele **reconstrói o corpo em 3D para medir o volume** e,
junto com a massa medida numa balança de precisão, **calcula a densidade corporal e o percentual de
gordura** (equação do tipo Siri, com correção do volume pulmonar).

> **Estado (nov/2026):** a versão funcional de referência é a **RTX Anthropometric V8.2.5 CLI**.
> O módulo de super-resolução (RTX_SuperResolution) e a instalação de **3 Orbbec Femto Mega** no arco
> ainda são **estudos**: nada foi implementado nem medido, e o trabalho aguarda a chegada dos sensores.

---

## Arquitetura

```
 3× Orbbec Femto Mega (RGB-D ToF, sincronizados por hub Orbbec, alimentação PoE)
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

## Em números

| Item | Valor |
| :--- | :--- |
| Sensores | 3× Femto Mega: profundidade 1024×1024, 120°×120°, até 25 fps, 0,25–2,21 m; RGB 4K 80°×51° |
| Posições (mm) | FM-01 (−1110, 1100, −310) · FM-02 (+1110, 1100, +310) · FM-03 (0, 2130, −280), inclinada −72,7° |
| Arco (CAD) | 2554 × 2246 × 842 mm |
| GPU | RTX 3060 (Ampere, CC 8.6, 12 GB) |
| Malha científica (LOD 0) | sem simplificação; é a única usada para volume e %GC |
| Equação de gordura | `%GC = A/D − B` com 8 pares (A, B) por sexo/etnia; Siri clássica = 495/450 |

## Documentação

| Documento | Conteúdo |
| :--- | :--- |
| [Arquitetura](docs/scanner-3d/arquitetura.md) | Componentes, fluxo de dados, níveis de detalhe (LOD) |
| [Instalação dos sensores](docs/scanner-3d/instalacao-sensores.md) | Posições no arco, sincronismo, PoE |
| [Super-resolução](docs/scanner-3d/super-resolucao.md) | Matemática (Cuba, 2014) e plano CUDA na RTX 3060 |
| [Modelo biomédico](docs/scanner-3d/modelo-biomedico.md) | Volume pulmonar, densidade, %GC e exemplo numérico |
| [Histórico e referências](docs/scanner-3d/historico-e-referencias.md) | Do laser (2005) ao RGB-D (2014) e ao sistema atual; inventário do Drive |
| [posicoes_3_femto_mega.csv](docs/scanner-3d/posicoes_3_femto_mega.csv) | Coordenadas propostas dos sensores |

## Equipe

Laboratório LAOB — Prof. Mario Gazziro, Victor Vieira Colombo.
