# Modelo biomédico — AllBodyScan

Fonte: `Relatorio_Modelos_Biomedicos_AllBodyScan.pdf` (01/11/2026), que audita
`17.03.02/AllBodyScan/usuario.cpp`, função `Usuario::calcularGordura()`, linhas 186–231 (código em `17.03.02.zip`).
É uma **auditoria da implementação**, não uma validação clínica.

## Equações

Volume pulmonar (correção), com `Mg = PESO × 1000` (massa em g) e `H = ALTURA`:

| Condição | Vp |
| :--- | :--- |
| SEXO = 0 | `[0,0472·(H + 2) + 0,000009·Mg − 5,92] × 1000` |
| SEXO ≠ 0 | `[0,036·(H + 2) + 0,0000031·Mg − 3,182] × 1000` |

Densidade e gordura:

```
D   = Mg / (V − Vp)        (g/cm³; V = volume do scanner em cm³)
%GC = A / D − B            (forma de Siri; A=495, B=450 é a Siri clássica)
```

## Coeficientes (A, B) por SEXO e ETNIA (códigos como estão no código-fonte)

| SEXO | ETNIA 0 | ETNIA 1 | ETNIA 2 | ETNIA 3 |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 495 / 450 | 497 / 452 | 437 / 393 | 437 / 393 |
| ≠ 0 | 501 / 457 | 476 / 428 | 485 / 439 | 485 / 439 |

## Exemplo real (planilha `gordura.ods`)

Voluntário: M, 57 anos, afro-americano, 168 cm, 78 600 g, volume 75 729 cm³, Vp 2 717 cm³.

- `D = 78 600 / (75 729 − 2 717) = 1,07654 g/cm³` ✔ (bate com a planilha)
- `%GC = 437 / 1,07654 − 393 = 12,93 %` ✔ (a adipometria deu 12,5 %)

Pela planilha, **SEXO = 0 é masculino** e "afro-americano" usa o par 437/393 (ETNIA 2 ou 3).

**Ponto em aberto:** com H em cm e Mg em g, a fórmula de Vp do código dá ≈ 2 811 cm³, e a planilha usa
2 717 cm³. É preciso confirmar as unidades/origem do Vp usado (e o "+2" na altura, que o código não justifica).

## Limitações

- Distinguir volume residual pulmonar, volume gasoso torácico e volume realmente deslocado.
- Etnia é uma aproximação populacional; os pares A/B diferentes de 495/450 precisam ter a fonte bibliográfica rastreada.
- A idade não entra no %GC (só nas faixas de classificação).
- Massa livre de gordura (100 − %GC) não é o mesmo que massa muscular.
- Validar contra um método de referência (DXA ou modelo multicompartimental).
