# RTX_SuperResolution — super-resolução temporal de profundidade

Fontes: `RTX_SuperResolution_Tutorial_Matematico.pdf` e `RTX_SuperResolution_Arquitetura_GPU_RTX3060.pdf`
(02/11/2026). Base: dissertação de **Manolo Canales Cuba (UFABC, 2014)** e seus scripts MATLAB
(`scriptsuper.m`, `superre.m`, `main3.m`, `getMatrixGrad.m`).

> **Estudo conceitual: o módulo não foi implementado.** Nenhuma taxa de quadros, latência ou ganho foi medido.

## 1. Modelo de observação

Hipótese simples (pessoa e sensor parados): `D_i(x,y) = D*(x,y) + η_i(x,y)`. Nesse caso juntar quadros
**reduz ruído**, mas não cria detalhe novo.

Modelo geral proposto: `D_i = M_i · S_i · B_i · W_i(D*) + η_i`, onde
W é a reprojeção/deslocamento, B o desfoque, S a subamostragem e M a máscara de validade.
Super-resolução **real** exige amostras complementares (deslocamentos subpixel) e registro confiável.

## 2. Pré-processamento (como no MATLAB)

- Normalização: `d_i = D_i / 4095` (`MAX_VALUE_DEPTH = 2^12 − 1`); variável invertida `u_i = 1 − d_i`.
- Corte por limites de distância; ampliação inicial `FACTOR = 2` com interpolação *nearest*
  (aumenta a grade, mas não acrescenta informação física).
- Estimativa inicial: a média válida é `ū = Σ m_i u_i / max(1, Σ m_i)`, mas o solver **começa no quadro central**
  (`iavgimg = ceil(C/2); Xk = img_depth{iavgimg}`).

## 3. Energia e atualização

`E(u) = E_dados(u) + λ·E_reg(u)`, com `E_dados = ½ Σ_i Σ_xy w_i (u − u_i)²`.

- Pesos `w_i` são **fotométricos** (diferença em cinza do RGB para uma referência média), sem calibração estatística do ruído.
- Regularização `type = 4` (**isotrópica não linear**), diferenças finitas numa janela **5×5** (`getMatrixGrad.m`):
  `H_(a,b)(u) = [u(x+a,y+b) − u(x,y)] / √(a²+b²)`
  `R(u) = Σ H_(a,b)(u) / [1 + (H_(a,b)(G_σ * u) / λ_iso)²]` (suaviza o ruído e preserva as bordas).
- Passo explícito (gradiente descendente):
  `u^(k+1) = u^(k) − Δt · [ W·e_dados − 2λ·R(u^(k)) ]`, com **dt = 0,008, λ = 6** no código original
  (valores ligados à normalização original, não recomendados para o Femto Mega).
- Atenção: `reproject_depthmap_color` está **comentada** em `superre.m`, ou seja, o código original não faz registro temporal subpixel.

## 4. Volta para 3D

Câmera pinhole calibrada: `X = (x − c_x)·Z / f_x`, `Y = (y − c_y)·Z / f_y`, `Z = D(x,y)`
(intrínsecos medidos de cada sensor; na dissertação, `depthToCloud.m`).

## 5. Implementação proposta na RTX 3060 (Ampere, CC 8.6, 12 GB GDDR6)

- **CUDA cores** fazem o trabalho (cálculo por pixel, stencil). Tensor Cores e RT Cores são opcionais e não aceleram esse método.
- Fluxo: buffers circulares na RAM → cópia assíncrona **pinned + CUDA streams** → máscaras/pesos → alinhamento → solver → XYZ+RGB na GPU.
- **Onde ficam os dados:** pilhas de quadros e estados `X(k)` na VRAM global; tiles com halo em *shared memory*;
  acumuladores em registradores; parâmetros pequenos em *constant memory*.
- **Tile:** bloco 16×16 threads; stencil de raio 2 → **tile 20×20** (halo de 2 px) = 1600 B em float32.
- Layout `D[t,y,x]` para acesso coalescido; redução temporal por thread:
  `s = Σ_t w_t D_t`, `q = Σ_t w_t`, `D_fundida = s / max(q, ε)`.
- **Ping-pong** `X_A → X_B` a cada iteração (sem sobrescrever vizinhos, sem voltar à CPU).
- Orçamento de memória (hipótese: 3 sensores × 10 quadros, 640×576, SR 2×): profundidade 42,2 MiB + RGB 42,2 MiB +
  máscaras 10,5 MiB + 8 buffers float32 2× por sensor 135 MiB = **≈ 230 MiB** (sem contexto CUDA, nuvens, bibliotecas).
- O kernel deve ser **limitado por banda de memória**: `desempenho ≤ min(P_pico, B_mem × intensidade aritmética)`.
  Medir com CUDA Events, Nsight Systems/Compute.
- Otimizações: manter os dados na GPU (evitar ida e volta NumPy↔GPU↔Open3D), fundir kernels, CUDA Graphs, cópia assíncrona de tiles,
  float32 primeiro (precisão mista só depois de validar o volume).

## 6. Validação planejada

- Comparar **1, 3, 5, 10 e 15 quadros** por sensor.
- Medir a dispersão em regiões planas (`σ_D`), a preservação de detalhes, a distância entre superfícies e a repetibilidade do volume.
- **Repetibilidade ≠ exatidão:** usar objetos com dimensões/volume conhecidos.
- Riscos: respiração, oscilação postural, mãos e bordas de oclusão (borrão e superfícies duplicadas). Regiões nunca vistas não podem ser recuperadas.
