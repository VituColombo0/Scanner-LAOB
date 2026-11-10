# Instalação dos sensores no arco

Fontes: `simulacao_3_femto_mega_arco.pdf/.png`, `posicoes_3_femto_mega.csv`,
`00.100-Mont. carenagem scanner.STEP` e o diagrama de ligações (`IMG-20261102-WA0085.jpg`).

> Pré-projeto feito sobre a geometria do STEP. **Não há validação** de colisão, cobertura do corpo,
> oclusões, interferência de IR entre sensores nem precisão volumétrica.

## Estrutura existente (CAD)

Caixa envolvente da carenagem: **2554 mm (largura) × 2246 mm (altura) × 842 mm (profundidade)**.

## Posições propostas

Origem no centro do arco, no chão; Y para cima; o alvo é o eixo central do corpo.

| Sensor | Função | X (mm) | Y (mm) | Z (mm) | Alvo (X,Y,Z) | Distância ao alvo | Elevação |
| :--- | :--- | ---: | ---: | ---: | :--- | ---: | ---: |
| FM-01 | Lateral esquerda / entrada | −1110 | 1100 | −310 | (0, 1100, 0) | 1152,5 mm | 0° |
| FM-02 | Lateral direita / saída | +1110 | 1100 | +310 | (0, 1100, 0) | 1152,5 mm | 0° |
| FM-03 | Superior / entrada | 0 | 2130 | −280 | (0, 1230, 0) | 942,5 mm | −72,7° |

As laterais ficam em lados opostos de Z (−310 / +310), ou seja, levemente cruzadas, e não frente a frente
(provavelmente para reduzir a incidência direta do IR de uma no sensor da outra; a confirmar). A distância ≈ 1,15 m fica dentro
da faixa útil do modo de profundidade (0,25–2,21 m).

## Femto Mega — dados usados no projeto

- Profundidade (ToF) WFOV: **120° × 120°, 1024 × 1024, até 25 fps, alcance 0,25–2,21 m**.
- RGB: **80° × 51°**, 4K até 25 fps, **rolling shutter** (campo mais estreito que a profundidade).

## Ligações

- **Sincronismo:** hub de sincronismo Orbbec. Um sensor é *Primary* (Sync Out) e os outros são *Secondary* (Sync In).
- **Energia e dados:** switch/hub **PoE**, com uplink (NET) para o computador de controle.
- Computador de controle com GPU **RTX 3060** (confirmar se é a versão de 12 GB).
