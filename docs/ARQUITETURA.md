# Arquitetura proposta

Este documento descreve uma arquitetura inicial para o **Sistema de Corte Blender**.

O objetivo é preservar a sensação de desenho livre sobre uma superfície contínua e evitar que o traço seja condicionado pela topologia do objeto.

---

## 1. Princípio central

A malha original é usada somente como **superfície de referência espacial**.

Ela não deve servir como caminho para o traço.

Portanto:

- não procurar a edge mais próxima;
- não fazer shortest path entre vertices;
- não obrigar o stroke a passar por vertices existentes;
- não reconstruir o desenho seguindo edge loops;
- não depender de boa topologia.

O sistema deve obter posições 3D na superfície e depois criar uma representação própria do desenho.

---

## 2. Componentes principais

### 2.1 Surface Draw

Responsável por ler o movimento do mouse e encontrar a posição correspondente sobre a superfície do modelo.

Fluxo sugerido:

```text
Mouse 2D
  ↓
raio da câmera
  ↓
raycast
  ↓
hit position 3D
  ↓
ponto do stroke
```

Dados úteis por amostra:

- posição 3D;
- normal da superfície;
- posição 2D do mouse;
- distância em relação ao ponto anterior;
- timestamp opcional;
- objeto atingido.

O sistema deve evitar adicionar pontos demais quando o mouse praticamente não se moveu.

---

### 2.2 Stroke Sampler

Responsável por controlar a densidade de pontos.

O desenho do mouse costuma gerar pontos com espaçamento irregular. Antes da suavização é recomendável reamostrar o stroke usando uma distância aproximadamente constante.

Objetivos:

- evitar concentração excessiva de pontos;
- facilitar smoothing;
- melhorar estabilidade;
- reduzir custo de processamento;
- produzir curvas mais previsíveis.

---

### 2.3 Stroke Stabilizer

Responsável por remover pequenas tremidas da mão.

A correção deve atuar sobre o **stroke independente**, nunca sobre edges da malha.

Possíveis abordagens para protótipo:

- moving average;
- Chaikin smoothing;
- spline/interpolação;
- filtros com preservação de forma;
- simplificação + reconstrução suave.

É importante evitar smoothing excessivo que altere a intenção do usuário.

Pode existir um parâmetro:

```text
stabilization = 0.0 ... 1.0
```

Também pode ser útil separar:

- `input_smoothing` — suavização durante o desenho;
- `final_smoothing` — limpeza aplicada depois que o stroke termina.

---

### 2.4 Surface Constraint

Depois de suavizar uma curva, alguns pontos podem se afastar levemente da superfície.

Por isso é necessário reprojetá-los.

A reprojeção não deve significar snap em edge.

Ela deve significar:

```text
ponto suavizado
   ↓
projeção/raycast para superfície
   ↓
ponto corrigido sobre a superfície
```

Dependendo do ângulo e da geometria, pode ser necessário usar mais de uma estratégia de projeção:

- direção da câmera;
- normal armazenada do hit original;
- closest point on mesh;
- combinação dessas técnicas.

Esse módulo será importante principalmente em superfícies muito curvas.

---

### 2.5 Loop Closure

Responsável por fechar o stroke quando o final chega próximo do início.

Parâmetros possíveis:

- distância mínima em pixels;
- distância máxima em espaço 3D;
- número mínimo de pontos;
- comprimento mínimo do stroke;
- ângulo de chegada.

Apenas aproximar o último ponto do primeiro pode criar uma quina artificial. A solução deve suavizar a região da emenda para preservar continuidade.

Estados possíveis:

```text
OPEN
CLOSE_PREVIEW
CLOSED
```

`CLOSE_PREVIEW` pode mostrar visualmente ao usuário que, se soltar o botão, o loop será fechado.

---

### 2.6 Curve Builder

Ao terminar o desenho, os pontos finais devem gerar uma Curve própria.

Essa Curve será independente do objeto original.

Ela pode armazenar:

- spline fechada;
- pontos de controle;
- informações de projeção;
- referência ao objeto fonte;
- parâmetros de smoothing.

A vantagem de manter Curve nessa etapa é permitir edição posterior sem destruir a peça original.

---

### 2.7 Surface Patch

Objetivo: transformar o contorno fechado em uma região utilizável.

Essa é uma das partes mais delicadas do projeto, porque o loop pode atravessar uma superfície curva.

Não se deve simplesmente preencher o contorno em um plano se a intenção for preservar a curvatura da peça.

Possíveis estratégias de implementação devem ser testadas em protótipos:

1. criar uma superfície temporária a partir do contorno;
2. projetar/ajustar essa superfície sobre o modelo;
3. gerar um cutter volumétrico diretamente a partir do loop;
4. usar a curva como limite para construir volume ao longo da direção desejada.

O melhor método dependerá do comportamento final esperado para cortes em regiões muito curvas.

---

### 2.8 Cutter Generator

Depois do loop estar pronto, o sistema cria geometria própria para Boolean.

Exemplo:

```text
Closed Surface/Loop
       ↓
offset opcional
       ↓
extrusão / profundidade
       ↓
Cutter Mesh
       ↓
Boolean Difference
```

Configurações futuras possíveis:

- profundidade de corte;
- corte atravessando a peça inteira;
- direção da extrusão;
- margem/offset;
- manter cutter após Boolean;
- Boolean Difference/Intersect.

---

## 3. Representação durante o desenho

Durante o mouse drag, a ferramenta deve preferencialmente trabalhar com uma representação leve.

Sugestão:

```text
lista de pontos Python
        +
preview desenhado no viewport
```

Somente depois do stroke terminar é necessário criar Curve/Mesh real no Blender.

Benefícios:

- melhor desempenho;
- menos alterações na cena;
- Undo mais limpo;
- nenhuma dependência da topologia durante o movimento;
- fácil descarte caso o usuário cancele.

---

## 4. Operador modal

A ferramenta provavelmente deverá ser implementada como um **Modal Operator** do Blender.

Estados sugeridos:

```text
IDLE
DRAWING
CLOSE_PREVIEW
PROCESSING
EDIT_LOOP
READY_TO_CUT
```

Eventos iniciais possíveis:

- LMB press: começa stroke;
- Mouse Move: adiciona amostras;
- LMB release: finaliza;
- Esc/RMB: cancela;
- Enter: confirma loop;
- controle futuro para editar pontos.

---

## 5. Preview no viewport

O usuário precisa enxergar claramente o que está desenhando.

O preview pode exibir:

- stroke bruto durante o desenho;
- stroke suavizado;
- marcador do ponto inicial;
- indicador de fechamento;
- loop final;
- preview da área/cutter.

Idealmente o preview deve aparecer ligeiramente acima da superfície para evitar z-fighting visual, sem alterar a posição lógica do stroke.

---

## 6. Independência da topologia

Este é um requisito funcional e deve ser tratado como critério de teste.

O mesmo formato desenhado deve se comportar de maneira semelhante em:

- uma esfera low-poly;
- a mesma esfera subdividida;
- uma escultura remeshed;
- um scan triangulado;
- uma malha com edge flow completamente diferente.

A densidade e a direção das edges podem afetar apenas a precisão geométrica da superfície disponível, mas **não devem decidir por onde passa o stroke**.

---

## 7. Problemas técnicos esperados

### Superfície muito low-poly

Mesmo sem snap em edges, uma superfície realmente facetada continua sendo facetada geometricamente.

Se necessário, no futuro pode existir uma opção de usar uma superfície avaliada com modificadores como referência para o raycast.

### Mudança brusca de silhueta

O cursor pode passar para outra parte da peça quando o raycast da câmera deixa de atingir a região esperada.

Será necessário testar estratégias para continuidade do stroke.

### Cavidades e oclusão

Uma projeção simples da câmera pode atingir uma superfície que não corresponde à intenção do usuário.

### Loops autointersectados

Desenhos que cruzam eles mesmos precisam ser detectados antes de gerar uma face/cutter.

### Superfícies extremamente curvas

Construir a região interna de um loop fechado pode exigir algoritmos específicos para preservar a relação com a superfície.

---

## 8. Critérios para o primeiro protótipo

O primeiro protótipo não precisa fazer Boolean.

Ele deve provar apenas estes pontos:

1. desenhar com o mouse sobre um objeto;
2. capturar pontos por raycast;
3. exibir stroke independente das edges;
4. suavizar o stroke;
5. reprojetar a curva suavizada sobre a superfície;
6. detectar proximidade do início;
7. fechar o loop;
8. criar uma Curve editável.

Se isso funcionar bem, a base da ferramenta estará validada.

---

## 9. Regra para decisões futuras

Sempre que houver duas soluções possíveis, priorizar a que melhor respeitar este princípio:

> O usuário está desenhando sobre uma escultura, não navegando pela topologia de uma malha.
