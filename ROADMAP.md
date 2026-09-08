# Roadmap inicial

Este roadmap organiza o desenvolvimento em etapas pequenas para evitar construir Boolean, interface e edição antes de validar a parte mais importante: **desenhar um loop estável sobre a superfície sem seguir as edges existentes**.

## Fase 1 — Prova de conceito do desenho

Objetivo: conseguir desenhar sobre um objeto no viewport.

- [ ] Criar estrutura básica de add-on do Blender.
- [ ] Criar um Modal Operator.
- [ ] Ler movimento do mouse.
- [ ] Fazer raycast contra o objeto ativo.
- [ ] Guardar os hit points em espaço 3D.
- [ ] Mostrar uma linha de preview no viewport.
- [ ] Cancelar desenho com Esc/RMB.

### Critério de sucesso

O usuário consegue rabiscar sobre uma esfera/escultura e o traço acompanha a superfície sem procurar edges próximas.

---

## Fase 2 — Amostragem e estabilização

Objetivo: transformar o movimento irregular do mouse em uma curva limpa.

- [ ] Definir distância mínima entre amostras.
- [ ] Reamostrar pontos com espaçamento mais uniforme.
- [ ] Implementar smoothing inicial.
- [ ] Criar parâmetro `Stabilization`.
- [ ] Comparar stroke bruto e stroke estabilizado.
- [ ] Evitar perda excessiva de detalhes desejados.
- [ ] Reprojetar pontos suavizados sobre a superfície.

### Critério de sucesso

Um círculo desenhado à mão deve ficar visivelmente mais limpo sem ser deformado pelas edges da malha.

---

## Fase 3 — Fechamento do loop

Objetivo: permitir criar contornos fechados de forma natural.

- [ ] Marcar visualmente o ponto inicial.
- [ ] Detectar aproximação do cursor ao início.
- [ ] Mostrar estado de `Close Preview`.
- [ ] Fazer snap controlado do final para o início.
- [ ] Suavizar a região da emenda.
- [ ] Garantir continuidade do loop.
- [ ] Impedir fechamento acidental de strokes muito pequenos.

### Critério de sucesso

O usuário consegue desenhar um contorno e fechá-lo sem deixar uma emenda tremida ou uma pequena abertura.

---

## Fase 4 — Curve editável

Objetivo: transformar o resultado em um elemento que possa ser ajustado depois do desenho.

- [ ] Criar Curve a partir do loop processado.
- [ ] Manter a Curve independente da malha original.
- [ ] Permitir selecionar o loop após criação.
- [ ] Definir sistema para editar pontos de controle.
- [ ] Preservar vínculo/referência com o objeto fonte quando necessário.

### Critério de sucesso

Depois de desenhar, o usuário pode ajustar o contorno sem alterar a topologia da peça original.

---

## Fase 5 — Validação de independência da topologia

Objetivo: comprovar que o comportamento não depende do edge flow.

Testar o mesmo tipo de desenho em:

- [ ] esfera low-poly;
- [ ] esfera subdividida;
- [ ] malha triangulada;
- [ ] remesh de escultura;
- [ ] scan 3D;
- [ ] superfície com topologia irregular.

### Critério de sucesso

O stroke final deve seguir a intenção do desenho e não mudar de caminho apenas porque a direção das edges mudou.

---

## Fase 6 — Detecção de problemas no loop

- [ ] Detectar autointerseções.
- [ ] Detectar segmentos muito curtos.
- [ ] Detectar mudanças bruscas no stroke.
- [ ] Tratar perda temporária do raycast.
- [ ] Evitar saltos para a parte traseira do modelo.
- [ ] Definir comportamento em cavidades.

---

## Fase 7 — Surface Patch / região interna

Objetivo: transformar o loop em uma região utilizável para corte.

- [ ] Pesquisar e testar estratégias para preencher loops sobre superfícies curvas.
- [ ] Criar protótipo de patch.
- [ ] Comparar patch planar vs. patch adaptado à superfície.
- [ ] Garantir estabilidade em áreas curvas.
- [ ] Permitir editar/confirmar a região antes do corte.

### Observação

Esta fase deve ser desenvolvida somente depois que o sistema de desenho estiver sólido.

---

## Fase 8 — Cutter e Boolean

- [ ] Converter região/loop em cutter.
- [ ] Adicionar profundidade de corte.
- [ ] Criar opção de atravessar toda a peça.
- [ ] Definir direção de extrusão.
- [ ] Adicionar margem/offset opcional.
- [ ] Aplicar Boolean Difference.
- [ ] Opção para manter ou apagar o cutter.
- [ ] Testar Boolean em malhas densas.

---

## Fase 9 — Interface

Possíveis controles:

- [ ] Draw Tool.
- [ ] Stabilization.
- [ ] Sample Spacing.
- [ ] Close Distance.
- [ ] Final Smooth.
- [ ] Cutter Depth.
- [ ] Through All.
- [ ] Offset.
- [ ] Apply Boolean.
- [ ] Keep Cutter.

A interface deve permanecer simples. A prioridade é que o fluxo pareça uma ferramenta de desenho, não um conjunto complicado de operações de modelagem.

---

## Fase 10 — Otimização

- [ ] Testar em modelos com muitos polígonos.
- [ ] Usar malha avaliada quando apropriado.
- [ ] Evitar criar geometria real durante cada Mouse Move.
- [ ] Otimizar raycasts.
- [ ] Reduzir quantidade de pontos sem perder precisão.
- [ ] Testar Undo/Redo.
- [ ] Testar estabilidade em sessões longas.

---

# Primeiro marco importante

Antes de avançar para Boolean, o projeto deve conseguir demonstrar este fluxo com qualidade:

```text
Ativar ferramenta
      ↓
Desenhar livremente sobre a peça
      ↓
Traço acompanha a superfície
      ↓
Traço não segue edges
      ↓
Sistema remove tremidas
      ↓
Usuário aproxima do início
      ↓
Loop fecha suavemente
      ↓
Curve editável é criada
```

Quando esse fluxo estiver funcionando bem, o núcleo do projeto estará validado.
