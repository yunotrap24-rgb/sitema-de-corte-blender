# Sistema de Corte Blender

Ferramenta para Blender focada em **desenhar cortes diretamente sobre a superfície de um modelo 3D**, tratando o objeto como uma superfície contínua, sem obrigar o usuário a seguir as edges ou a topologia existente.

## Ideia principal

A ferramenta deve se comportar visualmente como uma mistura de **Knife Tool + lápis/desenho livre**, porém com uma diferença fundamental:

> O traço não deve ser corrigido, encaixado ou reconstruído seguindo as edges da malha original.

O usuário desenha sobre o modelo como se ele fosse uma escultura lisa. O sistema captura a posição do traço sobre a superfície, estabiliza o desenho, fecha o contorno e cria uma nova geometria independente que poderá ser usada para gerar uma face e posteriormente um cutter para Boolean.

## Comportamento esperado

1. O usuário ativa o modo de desenho.
2. Clica e arrasta o mouse sobre o modelo como um lápis.
3. Cada posição do mouse é projetada sobre a superfície do objeto.
4. O traço permanece independente das edges existentes.
5. Pequenas tremidas da mão podem ser suavizadas por um estabilizador.
6. Ao aproximar o final do traço do ponto inicial, o sistema pode fechar automaticamente o loop.
7. O loop final é reconstruído como uma curva limpa e editável.
8. O loop pode ser convertido em geometria.
9. A área fechada pode gerar uma face/superfície editável.
10. Essa superfície pode ganhar profundidade e virar um objeto cortador.
11. O cortador pode ser usado em um modificador Boolean sobre a peça original.

## Regra mais importante: NÃO seguir as edges

O sistema não deve funcionar assim:

```text
mouse -> edge mais próxima -> caminho pelas edges da malha
```

Ele deve funcionar assim:

```text
mouse -> raycast na superfície -> ponto 3D -> curva independente suavizada
```

Internamente o Blender continuará detectando a superfície através dos polígonos, pois toda malha é formada por faces. Porém esses polígonos servem apenas para descobrir **onde está a superfície**.

Eles não definem o formato do traço final.

Isso permite trabalhar com:

- esculturas;
- modelos triangulados;
- scans 3D;
- remesh;
- malhas com topologia ruim;
- modelos com muitos polígonos;
- modelos em que as edges não acompanham o corte desejado.

## Fluxo conceitual

```text
DESENHO DO MOUSE
      ↓
RAYCAST NA SUPERFÍCIE
      ↓
PONTOS 3D BRUTOS
      ↓
ESTABILIZAÇÃO / SMOOTH
      ↓
CURVA INDEPENDENTE
      ↓
FECHAMENTO DO LOOP
      ↓
CURVA FECHADA LIMPA
      ↓
GEOMETRIA / FACE
      ↓
EXTRUSÃO DO CUTTER
      ↓
BOOLEAN
```

## Sensação de uso desejada

A prioridade é fazer o usuário sentir que está desenhando sobre uma **escultura lisa**, e não editando uma malha poligonal.

O usuário não deveria precisar pensar em:

- vertices;
- edges;
- edge loops existentes;
- quantidade de triângulos;
- direção da topologia.

Ele simplesmente desenha o formato desejado sobre a peça.

## Estabilização do traço

A ferramenta deverá possuir estabilização ajustável para remover tremidas sem destruir o formato que o usuário desenhou.

Exemplo de controle:

```text
Stabilization: 0%   -> acompanha a mão praticamente sem correção
Stabilization: 50%  -> suavização moderada
Stabilization: 100% -> curva fortemente estabilizada
```

A estabilização deve trabalhar sobre os pontos capturados pelo desenho, e **não procurando edges próximas**.

## Fechamento do loop

Quando o usuário estiver terminando um contorno fechado, a ferramenta deverá detectar a proximidade entre o último ponto e o primeiro.

Dentro de uma distância configurável:

```text
final do traço
     ↓
próximo do início?
     ↓ sim
snap controlado
     ↓
loop fechado
```

O fechamento deve evitar uma emenda visivelmente quebrada ou com quina causada por tremor da mão.

## Geometria final

O traço durante o desenho pode existir apenas como uma prévia/Curve.

Não é necessário criar uma nova edge na malha original a cada movimento do mouse.

A abordagem desejada é:

```text
Stroke
  ↓
Smooth Curve
  ↓
Closed Curve
  ↓
Mesh / Surface
  ↓
Cutter
```

Isso mantém o sistema independente da topologia da peça original.

## Objetivo do projeto

Criar para Blender uma ferramenta capaz de:

> **Desenhar um contorno livre diretamente sobre qualquer superfície 3D, independente da topologia original, suavizar e estabilizar esse contorno, fechá-lo como um loop limpo e transformá-lo em uma superfície editável capaz de gerar cortes Boolean precisos.**

## Documentação

- [`docs/ARQUITETURA.md`](docs/ARQUITETURA.md) — arquitetura técnica proposta.
- [`ROADMAP.md`](ROADMAP.md) — etapas sugeridas para desenvolver e testar o sistema.

## Status

Projeto em fase inicial de definição e prototipagem.
