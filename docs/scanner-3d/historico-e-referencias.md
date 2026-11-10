# Histórico e referências

## Linha do tempo

| Ano | Trabalho | Tecnologia |
| :--- | :--- | :--- |
| 2005 | **Mario A. Gazziro** — *Projeto e Construção de um Scanner Antropométrico Baseado no Método de Triangulação a Laser* (mestrado, UFSCar) | Triangulação a laser |
| 2005 | França, Gazziro, Ide, Saito — *A 3D scanning system based on laser triangulation and variable field of view* (IEEE) | Laser + câmera giratórios, FOV variável |
| 2014 | **Manolo Canales Cuba** — *Reconstrução Tridimensional do Corpo Humano a partir de Dados RGB-D* (mestrado, UFABC; orient. João Paulo Gois) | RGB-D (Kinect), super-resolução, MATLAB |
| 2015 | **Michel Alves Lacerda** — *Aprimoramento do método de estimação de gordura corporal por meio de um Scanner 3D antropométrico* (mestrado, USP Bioengenharia; coorient. Gazziro) | Estimação de %GC |
| 2017? | AllBodyScan versão 17.03.02 (C++; ano deduzido do nome) | Software de cálculo de gordura |
| 2026 | RTX Anthropometric V8.2.5 + estudos RTX_SuperResolution / 3× Femto Mega | RGB-D ToF + GPU |

## Triangulação a laser (artigo de 2005), em resumo

Um laser projeta uma linha, a câmera vê o deslocamento `x'` e, com o ângulo θ (encoder), a focal `f` e a linha de base `b`:

```
z = b·f / (f·cot θ − x')    x = b·x' / (f·cot θ − x')    y = b·y' / (f·cot θ − x')
b = z·tan θ   (calibração com alvo a distância conhecida)
```

A triangulação é precisa, mas tem alcance e variação de profundidade limitados. O ToF tem mais alcance e menos
precisão. O protótipo gira **câmera e laser** (3 eixos, motores com encoder, RS-232, redução 879:1) e
**ajusta o FOV** por camadas para cobrir mais profundidade com boa resolução. A calibração usa o método de
Heikkilä & Silvén (4 etapas), a segmentação usa mediana + Prewitt/Canny + limiar, e a reconstrução faz
alinhamento + fusão + malha triangular.

## Inventário da pasta do Drive

| Arquivo | O que é |
| :--- | :--- |
| `Relatório Técnico RTX Anthropometric V8.2.5.png` | Arquitetura integrada e LODs |
| `RTX_SuperResolution_Tutorial_Matematico.pdf` | Matemática da super-resolução |
| `RTX_SuperResolution_Arquitetura_GPU_RTX3060.pdf` | Plano CUDA/memória |
| `Relatorio_Modelos_Biomedicos_AllBodyScan.pdf` | Auditoria das equações de %GC |
| `simulacao_3_femto_mega_arco.pdf/.png`, `posicoes_3_femto_mega.csv` | Simulação da instalação dos sensores |
| `00.100-Mont. carenagem scanner.STEP` | CAD da carenagem/arco (13 MB) |
| `IMG-20261102-WA0085.jpg` | Diagrama de sincronismo + PoE |
| `gordura.ods` | Planilha de voluntário (exemplo de cálculo) |
| `RTX_Anthropometric.zip`, `17.03.02.zip`, `ply-js-main.zip`, `programa_manolo_aluno_gois_scanner (1).zip` | Códigos-fonte (não copiados para cá) |
| `coca_UP.ply`, `coca_LOW.ply`, `Brenda2.ply` | Malhas/nuvens de exemplo |
| `DissMAG.pdf`, `Disertacao_manolo_final.pdf`, `TDE_MichelAlvesLacerda_ME_Corrigida.pdf`, artigo IEEE | Referências acadêmicas |
